# Boost.PropertyMap/Parallel

* lib: `boost/libs/property_map_parallel`
* repo: `boostorg/property_map_parallel`
* commit: `d04a8f0`, 2025-05-03

------
### Parallel Property Map Concepts

#### Process Group & Triggers

* Header `<boost/property_map/parallel/process_group.hpp>`

```c++
struct attach_distributed_object {}; // ctor selecting tag
enum trigger_receive_context {
  trc_none, trc_in_synchronization, trc_early_receive, trc_out_of_band, trc_irecv_out_of_band
};

struct process_group_tag {};  // tag type root, virtual inheritance
  //    / linear    / immediate
  // pg - messaging - bsp
  //    \ locking   \ batch
  //    \ spawning

concept ProcessGroup<PG,T> = Serializable<T> &&
  require (PG& pg, const PG& cpg, T& val, T const& cval, T pval[], std::size_t n, PG::process_id_type id, int tag) {
    typename PG::process_id_type;
    { PG() }; { PG(cpg, attach_distributed_object()); }; // ctors
    { cpg.base() } -> convertible_to<PG>; // helper operations
    { pg.poll() }; // helper operations

    { probe(cpg) } -> optional<std::pair<int,int>>; // source id and tag
    { wait(pg) }; { synchronize(pg) };
    { process_id(cpg) } -> PG::process_id_type; { num_processes(cpg) } -> int; // process query
    { send(cpg, id, tag, cval) }; // message transmission
    { receive(cpg, id, tag, val) } -> PG::process_id_type;
    { receive(cpg, tag, pval, n) } -> pair<PG::process_id_type, size_t>;
    { receive(cpg, id, tag, pval, n) } -> pair<PG::process_id_type, size_t>;
};

concept bool OobProcessGroup<PG,T,U,F1,F2,Result> = ProcessGroup<PG,T> &&
  require (PG& pg, PG const& cpg, F1 const& f1, F2 const& f2, int tag, int id, T& v, T const& cv, U& u) {
    { pg.template trigger<T>(tag, f1) };
    { pg.template trigger_with_reply<T>(tag, f2) };
    { cpg.trigger_context() } -> trigger_receive_context;

    { send_oob(cpg, id, tag, cv) };
    { send_oob_with_reply(cpg, id, tag, cv, u) };
    { receive_oob(cpg, id, tag, v) };
};
```

* Basic process group supports BSP model: superstep(computation,synchronization) loop
* Out-of-band messages will invoke registered trigger handler function

------
#### Distributed Property Map

Header `<boost/property_map/parallel/parallel_property_maps.hpp>`

```c++
concept bool Reduce<Red, Key, T> = requires(Red const& r, Key const& k, T const& local, T const& remote) {
  { T::non_default_resolver } -> bool;      // flag whether 'r(k)' is unusable for default value
  { r(k) } -> T;                            // Make a default value for ghost cell
  { r(k, local, remote) } -> T;             // Make a value from local and remote
};

struct basic_reduce<T> {     // return default 'T()' for ghost, always use remote for combine
  static const bool non_default_resolver = false;
  T operator() <Key> (Key const&) const { return T(); }
  T operator() <Key> (Key const&, T const&, T const& remote) const { return remote; }
};

enum consistency_model {    // combinable bit flags
  cm_forward = 0b1,
  cm_backward = 0b10,
  cm_bidirectional = 0b11,
  cm_flush = 0b100,
  cm_reset = 0b1000,
  cm_clear = 0b10000,
};

template<PropertyGroup PG, ReadablePropertyMap GMap, PropertyMap SMap> // Global, Storage
class distributed_property_map {
  enum property_map_messages {        // totally 5 messages used to implement dist-pm
    pm_put, pm_get, pm_multiget, pm_multiget_reply, pm_multiput
  };
  using local_category = TR<SMap>::category; using local_key_type = TR<SMap>::key_type;
  using owner_local_pair = TR<GMap>::value_type;
  using process_id_type = PG::process_id_type;

public:
  using key_type = TR<GMap>::key_type;
  using value_type = TR<SMap>::value_type; using reference = TR<SMap>::reference;
  using category = is_base_of<lvalue_property_map_tag, local_category> ? read_write_property_map_tag : local_category;
  using process_group_type = PG;

  using ghost_cells_type = multi_index_container<pair<key_type, value_type>,  // how to store ghost cells
                            indexed_by<sequenced<>, hashed_unique<[](auto& p){p.first}>>>;
  using iterator = ghost_cells_type::iterator;
  using key_iterator = ghost_cells_type::nth_index<1>::type::iterator;

  struct data_t {               // data holder type
    PG        process_group;    // the process group;
    GMap      global;           // maps key to <owner, local_key> pair
    SMap      storage;          // local map
    shared_ptr<ghost_cells_type>    ghost_cells;        // storage for ghost cells
    size_t    max_ghost_cells = 1000000;    // 0 for infinite
    function<value_type(key_type)>  get_default_value;  // get default value for ghost
    bool      has_default_resolver;         // flag so that the resolver can provide default value
    consistency_model     model = cm_forward;           // the consistency model flags
    function<void()>      reset;            // resets all ghost cells to default
    void      clear() { ghost_cells->clear(); }         // clear out all ghost cells
    void      flush() {                     // flush all values destined for remote processes
      vector<vector<pair<local_key_type, value_type>>> values(num_processes(process_group));
      for (auto cell : ghost_cells) {
        auto g = get(global, cell.first);   // get owner/local_key pair for each cell
        values[g.first].emplace_back(g.second, cell.second); // put each owner's cell
      }
      for (int p = 0; p < values.size(); ++p)
        if (!values[p].empty()) send(process_group, p, pm_multiput, values[p]);
    }
    void      refresh_ghost_cells() {       // send requests to refresh ghost cells
      vector<vector<key_type>> keys(num_processes(process_group));
      for (auto cell : ghost_cells) keys[get(global, cell.first).first].push_back(cell.first);
      auto id = process_id(process_group);  // self id
      for (int p = (id+1)%keys.size(); p != id; p = (p+1)%keys.size())  // cycle from next until self
        if (!keys[p].empty()) send(process_group, p, pm_multiget, keys[p]);
    }
  };
  shared_ptr<data_t>  data;     // all data

public:
  // def-ctor
  distributed_property_map<Reduce=basic_reduce<value_type>>(PG const& pg, GMap const& gm, SMap const& sm, Reduce const& r=Reduce())
    : data(new data_t(pg, gm, sm, r, Reduce::non_default_resolver)) {
    data->ghost_cells.reset(new ghost_cells_type());
    set_reduce(r);
  }
  ~distributed_property_map();
  
  reference operator[](key_type const& key) const {   // return either local or ghost
    auto p = get(data->global, key);
    if (p.first == process_id(data->process_group)) return data->storage[p.second];
    else return cell(key);
  }
  
  PG process_group() const { return data->process_group.base(); }
  SMap [const]& base() [const] { return data->storage; }
  GMap [const]& global() [const] { return data->global; }

  void set_reduce<Reduce>(Reduce const& r) {
    data->process_group.replace_handler([] { assert(false); }); // all messages are handled by triggers

    auto handle_put =            [=](int, int, pair<key_type, value_type> const& req, trc) {
      auto local_key = get(data->global, req.first).second;
      auto local = get(data->storage, local_key);
      auto reduced = r(req.first, local, req.second);
      put(data->storage, local_key, reduced);
    };
    auto handle_get = [=](int, int, key_type const& key, trc) -> value_type {
      return get(data->storage, get(data->global, key).second);       // immediately reply via OOB return
    };
    auto handle_multiget = [=](int source, int, vector<key_type> const& keys, trc) {
      vector<pair<key_type, value_type>> results; results.reserve(keys.size());
      for (auto k : msg) {
        auto local_key = get(data->global, k).second;
        results.emplace_back(k, get(data->storage, local_key));
      }
      send(data->process_group, source, pm_multiget_reply, results);  // reply via message
    };
    auto handle_multiget_reply = [=](int, int, vector<pair<key_type, value_type>> const& msg, trc) {
      auto const& key_index = data->ghost_cells->get<1>();  // the hashed key index
      for (auto p : msg) {
        auto pos = data->ghost_cells->project<0>(key_index.find(p.first));  // get positional iterator
        if (pos != data->ghost_cells->end()) pos->second = p.second;    // update ghost cell's value
      }
    };
    auto handle_multiput = [=](int, int, vector<pair<SM::key_type, value_type>> const values&, trc) {
      for (auto p : values) {
        auto local = get(data->storage, p.first);   // first is local key
        auto reduced = r(p.first, local, p.second); // second is remote pushed value
        put(data->storage, p.first, reduced);
      }
    };

    data->process_group.trigger           (pm_put,            handle_put);
    data->process_group.trigger_with_reply(pm_get,            handle_get);
    data->process_group.trigger           (pm_multiget,       handle_multiget);
    data->process_group.trigger           (pm_multiget_reply, handle_multiget_reply);
    data->process_group.trigger           (pm_multiput,       handle_multiput);

    data->get_default_value = r;
    data->has_default_resolver = Reduce::non_default_resolver;
    data->reset = []() {          // clear all ghost cell to default value
      for (auto p : ghost_cells) p.second = data->get_default_value(p.first);
    };
    set_consistency_model(data->model);
  }

  int get_consistency_model() const;
  void set_consistency_model(consistency_model model) {
    data->model = model;
    if (model & cm_backward) data->max_ghost_cells = 0; // don't drop ghost cells for backward model
    if (model != cm_forward) {
      data->process_group.replace_on_synchronize_handle(
        [weak_ptr<data_t> data=data]() {    // callback before each synchronize
          if (data->model & cm_flush)                 data->flush();
          if (data->model & cm_backward && !(data->model & (cm_flush | cm_clear | cm_reset)))
                                                      data->refresh_ghost_cells();
          if (data->model & cm_clear)                 data->clear();
          if (data->model & cm_reset && data->reset)  data->reset();
        });
    }
  }
  
  void set_max_ghost_cells(size_t max_ghost_cells) {  // throw if 'cm_backward'
    data->max_ghost_cells = max(max_ghost_cells, 2);  // at least 2
    prune_ghost_cells();
  }

  void clear() { data->clear(); }
  void reset() { data->reset && data->reset(); }
  void flush() { data->flush(); }

private:
  void do_synchronize() {
    if (data->model & cm_backward) { synchronize(data->process_group); return; }
    data->refresh_ghost_cells();
    synchronize(data->process_group); synchronize(data->process_group); // multiget and reply
  }
  void request_put(int p, key_type const& k, value_type const& v) const {
    send(data->process_group, p, pm_put, make_pair(k, v));
  }
  value_type& cell(key_type const& key, bool request_if_missing = true) const {
    auto const& key_index = data->ghost_cells->get<1>();  // the hashed key index
    auto pos = data->ghost_cells->project<0>(key_index.find(key));  // get positional iterator
    if (pos == data->ghost_cells->end()) {
      value_type value;
      if (data->has_default_resolver) value = data->get_default_value(key);
      else if (request_if_missing)
        send_oob_with_reply(data->process_group, get(data->global, key).first, pm_get, key, value);
      else value = value_type{};    // default
      pos = data->ghost_cells->emplace_front(key, value).first;       // at front

      if (data->max_ghost_cells > 0) prune_ghost_cells();             // prune from back
    } else if (data->max_ghost_cells > 0)
      data->ghost_cells->relocate(data->ghost_cells->begin(), pos);   // move to front, recent at front
    return pos->second;
  }
  void prune_ghost_cells() const {
    if (data->max_ghost_cells == 0) return;
    while (data->ghost_cells->size() > data->max_ghost_cells) {
      if (data->model & cm_flush) {
        auto const& victim = data->ghost_cells->back();
        send(data->process_group, get(data->global, victim.first).first, pm_put, victim);
      }
      data->ghost_cells->pop_back();
    }
  }
};
void request(distributed_property_map const& pm, key_type const& key) {
  if (get(pm.global(), key).first != process_id(pm.process_group()))
      pm.cell(key, false);    // just allocate ghost cell, will send 'pm_multiget' on next sync
}
value_type get(distributed_property_map const& pm, key_type const& key) {   // similar to operator[]
  auto p = get(pm.global(), key);
  if (p.first == process_id(pm.process_group())) return get(pm.base(), p.second);
  else return pm.cell(key);
}
void put(distributed_property_map const& pm, key_type const& key, value_type const& value) {
  auto p = get(pm.global(), key);
  if (p.first == process_id(pm.process_group())) put(pm.base(), p.second, value);
  else {
    if (pm.get_consistency_model() & cm_forward) pm.request_put(p.first, key, value);
    pm.cell(key, false) = value;
  }
}
void local_put(distributed_property_map const& pm, key_type const& key, value_type const& value) {
  auto p = get(pm.global(), key);
  if (p.first == process_id(pm.process_group())) put(pm.base(), p.second, value);
  else pm.cell(key, false) = value;
}
void cache(distributed_property_map const& pm, key_type const& key, value_type const& value) {
  auto p = get(pm.global(), key);
  if (p.first != process_id(pm.process_group()))
    pm.cell(key, false) = value;
}
void synchronize(distributed_property_map const& pm) { pm.do_synchronize(); }

auto make_distributed_property_map(PG const&, GM global, SM storage,[Reduce])
    -> distributed_property_map<PG, GM, SM, Reduce>;
```

* Assert only `put` on writable and non-const lvalue property maps.
* Use 5 messages, all handled by registered trigger handlers.
  * `pm_put` caused by `put`, synchronized
  * `pm_get` caused by `get`, out-of-band, immediately wait for response
  * `pm_multiget` and `pm_multiget_reply`, caused by refreshing ghost cells, synchronized
  * `pm_multiput`, caused by flushing ghost cells to owners, synchronized
* Reducer have two roles:
  * Combiner for local value and remote value, decide how to update owned value against received `put` value
  * Default value provider, used by `reset`-ing all ghost cells to default value, or provide initial value
    for new ghost cells
  * `non_default_resolver` flag means use the reducer provided value as initial value for ghost cells,
    otherwise `cell(key, true)` will get value via oob `pm_get`, and `cell(key, false)` will use default-constructed
    value as initial value for ghost cell.
* Setting `max_ghost_cells` will cause ghost cells be discarded, `cm_backward` don't allow discarding
* Consistency models: (default is `cm_forward`)
  * when `cm_forward` is set, `put` will send `pm_put`, otherwise only cache the value in ghost cell
  * `cm_backward` not set:
    * ghost cell may discarded, but remaining cells are refreshed just before `synchronize`
    * on synchronize, handle `cm_flush`, `cm_clear` and `cm_reset` in order.
  * `cm_backward` is set:
    * ghost cell never discarded, but on `synchronize`, they are refreshed only none of `cm_flush`, `cm_clear`
      and `cm_reset` are set, because it is meaningless.
    * otherwise `cm_flush`, `cm_clear` and `cm_reset` are handled normally.
  * `cm_flush` ensure no value loss, discarded ghost cell will send `pm_put` before remove
  * `cm_bidirectional` (`cm_forward | cm_backward`) will keep all ghost cells in sync
  * `cm_forward | cm_flush | cm_clear` suitable to handle accumulated values
* `operator[]` is like `get`, but use `[]` on local storage map.
* `local_put` will store in ghost cell and don't send `pm_put`, `cache` don't store in local storage map
* `request` will allocate ghost cell and expect it being refreshed on next `synchronize`

------
#### Caching Property Map

```c++
class caching_property_map<PM> requires DistributedPropertyMap<PM> {
  PM property_map;
public:
  // value_type, key_type, reference, category
  // ctor(pm), base() [const] init/get `property_map`
  // set_reduce(r), reset() forward to 'property_map'
};
// get(), local_put(), cache(), forward to `pm.base()`'s
void put<PM,K,V>(caching_property_map<PM> const& pm, K cosnt& k, V const& v) { local_put(pm.base(), k, v); }
auto make_caching_property_map<PM>(PM const&) -> caching_property_map<PM>;
```

* Wraps a `distributed_property_map` and disable `pm_put` (by calling `local_put` in `put`)

------
#### Local Property Map

```c++
template<ProcessGroup PG, ReadablePropertyMap GMap, PropertyMap SMap>
class local_property_map {
  PG process_group;  mutable GMap global_; mutable SMap storage;
public:
  // `process_group_type`, `value_type`, `reference`, `category` from SMap, `key_type` from GMap
  // ctor(), ctor(pg, gm, sm)
  reference operator[] (key_type const& k) { return storage[get(global_, k).second]; }
  // getters: global(), base(), process_group()[const]
};
reference get<PG,GMap,SMap>(local_property_map const& pm, key_type const& k)
{ return get(pm.base(), get(pm.global(),k).second); }
void put<PG,GMap,SMap>(local_property_map const& pm, key_type const&k, value_type const& v)
{ put(pm.base(), get(pm.global(), k).second, v); }
```

* Provides same API as `distributed_property_map`, but don't use `process_group` at all, just wraps a local map

------
#### Global Index Map

```c++
class global_index_map<IndexMap, GMap> {
  GMap global; IndexMap index_map;
  share_ptr<vector<value_type>> starting_index; // store index ranges for each process
public:
  using key_type = IndexMap::key_type; using value_type = IndexMap::value_type; using reference = value_type;
  using category = readable_property_map_tag;     // only readable, data prepared on ctor
  global_index_map<PG>(PG pg, value_type num_local_indices, IndexMap index_map, GMap gm)
    : index_map(index_map), global(gm) {
    starting_index.reset(new vector(num_processes(pg) + 1));    // process 0 is master node
    send(pg, 0, 0, num_local_indices);  // report self capacity to process 0
    synchronize(pg);
    if (process_id(pg) == 0) {
      (*starting_index)[0] = 0;
      for (int dest = 1; dest < num_processes(pg); ++dest) {
        value_type n; receive(pg, dest, 0, n);      // receive reported size from each process
        (*starting_index)[dest + 1] = (*starting_index)[dest] + n;      // register index range
      }
      for (int dest = 1; dest < num_processes(pg); ++dest)
        send(pg, dest, 1, &starting_index->front(), num_processes(pg)); // synchronize to all process
      synchronize(pg);                  // send result to each process
    } else {
      synchronize(pg);                  // wait for result
      receive(pg, 0, 1, &starting_index->front(), num_processes(pg));   // get data
    }
  }
};
value_type get<IndexMap,GMap>(global_index_map const& gim, key_type const& key) {
  auto owner = get(gim.global, key).first;
  auto offset = get(gim.index_map, key);
  return (*gim.starting_index)[owner] + offset;
}
```

* Maintains a global index range map, maps `key_type` to a global index.

------
### Distributed Specialization For Predefined Property Maps

#### Iterator Property Maps

```c++
class [safe_]iterator_property_map<RAIter, local_property_map<PG,GM,SM>, T, Ref>
  : public distributed_property_map<PG, GM, [safe_]iterator_property_map<RAIter, SM, T, Ref>>;
class [safe_]iterator_property_map<RAIter, distributed_property_map<PG,GM,SM>, T, Ref>
  : public distributed_property_map<PG, GM, [safe_]iterator_property_map<RAIter, SM, T, Ref>>;
auto make_iterator_property_map(RAIter, local_property_map<PG,GM,SM>)
    -> distributed_property_map<PG, GM, iterator_property_map<RA,SM>>;
```

* Specializations of Boost.PropertyMap's `iterator_property_map` and `safe_iterator_property_map` templates.
* Inject a `distributed_property_map` to serve as index map.

#### Vector Property Maps

```c++
class vector_property_map<T, local_property_map<PG,GM,SM>>
  : public distributed_property_map<PG,GM, vector_property_map<T,SM>>;
class vector_property_map<T, distributed_property_map<PG,GM,SM>>
  : public distributed_property_map<PG,GM, vector_property_map<T,SM>>;
```

* Specializations of Boost.PropertyMap's `vector_property_map` template.
* Inject a `distributed_property_map` to serve as index map.

------
### Dependency

#### Boost.PropertyMap

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.ConceptCheck

* `<boost/concept_archetype.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/version.hpp>`
* `<boost/cstdint.hpp>`

#### Boost.Function

* `<boost/function/function1.hpp>`

#### Boost.MPI

* `<boost/mpi/datatype.hpp>`

#### Boost.MPL

* `<boost/mpl/if.hpp>`, `<boost/mpl/bool.hpp>`, `<boost/mpl/assert.hpp>`
* `<boost/mpl/or.hpp>`, `<boost/mpl/and.hpp>`, `<boost/mpl/has_xxx.hpp>`

#### Boost.MultiIndex

* `<boost/multi_index_container.hpp>`
* `<boost/multi_index/hashed_index.hpp>`, `<boost/multi_index/member.hpp>`, `<boost/multi_index/sequenced_index.hpp>`

#### Boost.Optional

* `<boost/optional.hpp>`

#### Boost.PropertyMap

* `<boost/property_map/property_map.hpp>`

#### Boost.Serialization

* `<boost/serialization/is_bitwise_serializable.hpp>`
* `<boost/serialization/utility.hpp>`

#### Boost.SmartPtr

* `<boost/shared_ptr.hpp>`, `<boost/weak_ptr.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits.hpp>`
* `<boost/type_traits/is_same.hpp>`, `<boost/type_traits/is_base_and_derived.hpp>`

------
### Standard Facilities
