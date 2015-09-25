# Boost.PropertyMap/Parallel

* lib: `boost/libs/property_map`
* repo: `boostorg/property_map`
* commit: `ba0bed2e`, 2014-09-01

------
### Parallel Property Map Concepts

#### Process Group

```c++
struct attach_distributed_object {}; // ctor selecting tag
enum trigger_receive_context {
  trc_none, trc_in_synchronization, trc_early_receive, trc_out_of_band, trc_irecv_out_of_band
};

struct process_group_tag {};  // tag type root, virtual inheritance
  // linear
  // messaging: immediate, bsp, batch
  // locking
  // spawning

concept bool ProcessGroup<PG,T> = Serializable<T> &&
  require (PG& pg, const PG& cpg, T& val, T const& cval, T pval[], std::size_t n,
      PG::process_id_type id, int tag) {
    typename PG::process_id_type;
    PG();
    PG(cpg, attach_distributed_object()); { cpg.base() } -> PG;
    wait(pg);
    { probe(cpg) } -> optional<std::pair<int,int>>; // source id and tag
    synchronize(pg);
    { process_id(cpg) } -> PG::process_id_type;
    { num_processes(cpg) } -> int;
    send(pg, id, tag, cval);
    { receive(cpg, id, tag, val) } -> PG::process_id_type;
    { receive(cpg, tag, pval, n) } -> std::pair<PG::process_id_type, std::size_t>;
    { receive(cpg, id, tag, pval, n) } -> std::pair<PG::process_id_type, std::size_t>;
};

concept bool OobProcessGroup<PG,T,U,F1,F2,Result> = ProcessGroup<PG,T> &&
  require (PG& pg, PG const& cpg, F1 const& f1, F2 const& f2, int tag, int id, T& val, T const& cval, U& u) {
    pg.template trigger<T>(tag, f1);
    pg.template trigger_with_reply<T>(tag, f2);
    { cpg.trigger_context() } -> trc;
    send_oob(cpg, id, tag, cval);
    send_oob_with_reply(cpg, id, tag, cval, u);
    receive_oob(cpg, id, tag, val);
    pg.poll();
};
```

* Basic process group supports BSP model: superstep(computation,synchronization) loop
* Out-of-band messages will invoke registered trigger handler function

------
### Distributed Property Map

Header `<boost/property_map/parallel/parallel_property_maps.hpp>`

```c++
concept bool Reduce<Red, Key, T> = requires(Red const& r, Key const& k, T const local, T const& remote) {
  { T::non_default_resolver } -> bool;      // flag whether 'r(k)' is unusable for default value
  { r(k) } -> T;                            // Make a default value for ghost cell
  { r(k, local, remote) } -> T;             // Make a value from local and remote
};

struct basic_reduce<T>;     // return default 'T()' for ghost, always use remote for combine

enum consistency_model {    // combinable bit flags
  cm_forward = 0b1,
  cm_backward = 0b10,
  cm_bidirectional = 0b11,
  cm_flush = 0b100,
  cm_reset = 0b1000,
  cm_clear = 0b10000,
};

class distributed_property_map<PG, GM, SM>
  requires ProcessGroup<PG> && ReadablePropertyMap<GM> && PropertyMap<SM>
{
  enum property_map_messages {        // totally 5 messages used to implement dist-pm
    pm_put, pm_get, pm_multiget, pm_multiget_reply, pm_multiput
  };

  using ghost_cells_type = multi_index_container<pair<key_type, value_type>,  // how to store ghost cells
                            sequenced<>, hashed_unique<pair_first>>;

  struct data_t {               // data holder type
    PG        process_group;    // the process group;
    GM        global;           // maps key to <owner, local_key> pair
    SM        storage;          // local map
    shared_ptr<ghost_cells_type>    ghost_cells;        // storage for ghost cells
    size_t    max_ghost_cells = 1000000;  // 0 for infinite
    function<value_type(key_type)>  get_default_value;  // get default value for ghost
    bool      has_default_resolver;         // flag so that the resolver can't provide default value
    consistency_model     model = cm_forward;           // the consistency model flags
    function<void()>      reset;            // resets all ghost cells to default
    void      clear() { ghost_cells->clear(); }         // clear out all ghost cells
    void      flush() {                     // flush all values destined for remote processes
      vector<vector<pair<SM::key_type, value_type>>> values(num_processes(process_group));
      for (auto cell : ghost_cells) {
        auto g = get(global, cell.first);   // get owner/local_key pair for each cell
        values[g.first].emplace_back(g.second, cell.second); put each owner's cell
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
  using key_type = GM::key_type;      // a key go through GM and becomes <owner, local_key> pair
  using value_type = SM::value_type;  // value_type and reference are the same as local storage map's
  using reference = SM::reference;
  using category = SM::category == lvalue_property_map_tag ? read_write_property_map_tag : SM::category;

  distributed_property_map<Reduce=basic_reduce>(PG const& pg, GM const& gm, SM const& sm, // ctor
          Reduce const& r=Reduce()) {
    data.reset(new data_t {pg, gm, sm});
    set_reduce(r);
  }
  ~distributed_property_map();
  
  reference operator[](key_type const& key) const {   // return either local or ghost
    auto p = get(data->global, key);
    if (p.first == process_id(data->process_group)) return data->storage[p.second];
    else return cell(key);
  }
  
  PG process_group() const { return data->process_group.base(); }
  SM [const]& base() [const] { return data->storage; }
  GM [const]& global() [const] { return data->global; }

  void set_reduce<Reduce>(Reduce const& r) {
    data->process_group.replace_handler([] { assert(false); }); // all messages are handled by triggers

    auto handle_put =            [=](int, int, pair<key_type, value_type> const& req, trc) {
      auto local_key = get(data->global, req.first).second;
      auto local = get(data->storage, local_key);
      auto reduced = r(req.first, local, req.second);
      put(data->storage, local_key, reduced);
    };
    auto handle_get =            [=](int, int, key_type const& key, trc) -> value_type {
      return get(data->storage, get(data->global, key).second);       // immediately reply via OOB return
    };
    auto handle_multiget =       [=](int source, int, vector<key_type> const& keys, trc) {
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
    auto handle_multiput =       [=](int, int, vector<pair<SM::key_type, value_type>> const values&, trc) {
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
* 

------
### Predefined Property Maps

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Core

* `<boost/detail/iterator.hpp>` - deprecated
* `<boost/type.hpp>` - by `dynamic_property_map`

#### Boost.TypeTraits

* `<boost/type_traits.hpp>`, `<boost/type_traits/is_same.hpp>`, `<boost/type_traits/is_convertible.hpp>`

#### Boost.MPL

* `<boost/mpl/and.hpp>`, `<boost/mpl/not.hpp>`, `<boost/mpl/or.hpp>`, `<boost/mpl/if.hpp>`
* `<boost/mpl/has_xxx.hpp>`, `<boost/mpl/assert.hpp>`, `<boost/mpl/bool.hpp>`

#### Boost.Utility

* `<boost/utility/result_of.hpp>` - function PM

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>`, `<boost/concept_archetype.hpp>`

#### Boost.SmartPtr

* `<boost/smart_ptr/shared_array.hpp>` - by `shared_array_property_map`
* `<boost/shared_ptr.hpp>` - by `vector_property_map`
* `<boost/smart_ptr.hpp>` - by `dynamic_property_map`

#### Boost.Iterator

* `<boost/iterator/iterator_adaptor.hpp>` - by iterator generator

#### Boost.LexicalCast

* `<boost/lexical_cast.hpp>` - by `dynamic_property_map`

#### Boost.Any

* `<boost/any.hpp>` - by `dynamic_property_map`

#### Boost.Function

* `<boost/function/function3.hpp>` - by `dynamic_property_map`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>` - by `dynamic_property_map`

------
### Standard Facilities
