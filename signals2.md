# Boost.Signals2

* lib: `boost/libs/signals2`
* repo: `boostorg/signals2`
* commit: `5dcb2af`, 2025-03-21

------
### API

```c++
void detail::adl_predestruct(...) {}
class postconstructor_invoker<T> {
    shared_ptr<T> _sp; mutable bool _postconstructed{false};
    ctor(const shared_ptr<T>& sp) : _sp(sp) {}
public:
    operator const shared_ptr<T>& () const { return postconstruct(); }
    const shared_ptr<T>& postconstruct<...Args>(Args&&...args) const
    { if (!_postconstructed) { adl_postconstruct(_sp, sp.get(), std::forward<Args>(args)...); _postconstructed=true;} return _sp; }
};
struct detail::sp_aligned_storage<n,a> { union type{ char data_[n]; boost::type_with_alignment<a>::type align_; }; };
class detail::deconstruct_deleter<T> {
    bool initialized_{false}; sp_aligned_storage<sizeof(T), alignment_of_v<T>>::type storage_;
    void destroy() { if (initialized_) { using adl_predestruct; adl_predestruct(p); p->~T(); initialized_=false; }} // remove const
public: ctor() {} ctor(self&const) {}
    ~dtor {destroy();} void operator()(T*) { destroy();}
    void* address() { return storage_.data_; }
    void set_initialized() { initialized_ = true; }
};
class deconstruct_access {
    static postconstructor_invoker<T> deconstruct<T,...Args>(Args&&...args) {
        shared_ptr<T> pt{nullptr, deconstruct_deleter<T>{}};
        auto* pd = get_deleter(pt); void* pv = pd->address();
        new(pv) T(std::forward<Args>(args)...); pd->set_initialized();
        shared_ptr<T> retval{pt, (T*)pv}; sp_enable_shared_from_this(&retval, retval.get(), retval.get());
        return retval;
    }
};
postconstructor_invoker<T> deconstruct<T,...Args>(Args&&...args) { return deconstruct_access::deconstruct<T>(std::forward<Args>(args)...); }

class postconstructible { // disabled ADL
protected: ctor(){} virtual ~dtor(){}
    virtual void postconstruct() = 0;
    friend void adl_postconstruct<T>(const shared_ptr<T>&, self* p) { p->postconstruct(); }
};
class predestructible { // disable ADL
protected: ctor(){} virtual ~dtor(){}
    virtual void predestruct() = 0;
    friend void adl_predestruct(self* p) { p->predestruct(); }
    friend void adl_postconstruct<T>(const shared_ptr<T>&, ...) {}
};

void detail::do_postconstruct(const postconstructible* ptr) { ptr->postconstruct(); } // remove const
void detail::do_postconstruct(...) {}
void detail::do_predestruct(const predestruct* ptr) { try {ptr->predestruct();} catch(...){assert(false);} }
void detail::do_predestruct(...) {}
struct predestructing_deleter { void operator()(const T* ptr) const { do_predestruct(ptr); checked_delete(ptr); } };
shared_ptr<T> deconstruct_ptr<T>(T* ptr) {
    if (!ptr) return shared_ptr<T>{ptr};
    shared_ptr<T> shared{ptr, predestructing_deleter<T>{}}; do_postconstruct(ptr); return shared;
}
shared_ptr<T,D> deconstruct_ptr<T>(T* ptr, D deleter) {
    shared_ptr<T> shared(ptr, deleter); if (!ptr) return shared;
    do_postconstruct(ptr); return shared;
}

struct dummy_mutex { void lock(){} bool try_lock() { retrun true; } void unlock() {} };
struct mutex { // no-op, pthread, and win32 versions
    ctor(); ~dtor();
    void lock(); bool try_lock(); vid unlock();
};

struct expired_slot: bad_weak_ptr { /* what*/ };
struct no_slots_error: std::exception { /*what*/ };

struct last_value<T> { using result_type = T;
    T operator() <InIt>(InIt first, InIt last) const {
        if (first == last) throw_exception(no_slots_error{});
        optional<T> value;
        for (;first != last;++first){ try{ value = move_if_not_lvalue_reference<T>(*first); }catch(const expired_slot&){} }
        if (value) return value.get(); else throw_exception(no_slots_error{});
    }
};
struct last_value<void> { using result_type = void;
    void operator() <InIt>(InIt first, InIt last) const
    { for (;first != last;++first){ try{ *first; }catch(const expired_slot&){} } }
};

struct optional_last_value<T> { using result_type = optional<T>;
    optional<T> operator()<InIt>(InIt first, InIt last) const {
        optional<T> value;
        while (first!=last){ try { value=move_if_not_lvalue_reference<T>(*first); } catch(const expired_slot&){} ++first; }
        return value;
    }
};
struct optional_last_value<void> { using result_type = void;
    result_type operator()<InIt>(InIt first, InIt last) const
    { while (first!=last){ try { *first; } catch(const expired_slot&){} ++first; } }
};

class detail::scope_guard_impl_base { // delete copy-assign
protected: mutable bool dismissed_{false};
  ctor(const self& other) : dismissed_(other.dismissed_) { other.dismiss(); }
  ~dtor(){}
  static void safe_execute<J>(J& j) { try{ if (!j.dismissed_) j.execute(); } catch(...) {} }
public: ctor(){}
  void dismiss() const { dismissed_=true; }
};
using detail::scoped_guard = const scope_guard_impl_base&;

class detail::obj_scope_guard_impl2<Obj,MemFun,P1,P2> : public scope_guard_impl_base {
protected: Obj& obj_; MemFun mem_fum_; const P1 p1_; const P2 p2_;
public: ctor(Obj& obj, MemFun mem_fum, P1 p1, P2 p2) : obj_(obj), mem_fum_(mem_fum), p1_(p1), p2_(p2) {}
  ~dtor() { safe_execute(*this); }
  void execute() { (obj_.*mem_fun_)(p1_, p2_); }
};
obj_scope_guard_impl2<Obj,MemFun,P1,P2> detail::make_obj_guard(Obj& obj, MemFun mem_fun, P1 p1, P2 p2)
{ return obj_scope_guard_impl2<Obj,MemFun,P1,P2>(obj,mem_fun,p1,p2); }

struct detail::does_nothing { void operator()<T>(const T&) const {} };
using null_output_iterator = boost::function_output_iterator<does_nothing>;

class detail::unique_lock<Mutex> : public noncopyable { Mutex& _mutex;
public: ctor(Mutex &m) : _mutex{m} { _mutex.lock(); } ~dtor() { _mutex.unlock(); }
};

void null_deleter(const void*) {}
class detail::garbage_collecting_lock<Mutex> : public noncopyable {
  auto_buffer<shared_ptr<void>, store_n_objects<10>> garbage; unique_lock<Mutex> lock;
public: ctor(Mutex& m) : lock{m} {}
  void add_trash(const shared_ptr<void>& piece_of_trash) { garbage.push_back(piece_of_trash); }
};

class detail::connection_body_base {
  mutable bool _connected{true}; mutable unsigned m_slot_refcount{1};
protected: weak_ptr<void> _weak_blocker;
  virtual shared_ptr<void> release_slot() const = 0;
public: ctor() {} virtual ~dtor() {}
  void disconnect() { garbage_collecting_lock<self> local_lock{*this}; nolock_disconnect(local_lock); }
  void nolock_disconnect<Mutex>(garbage_collecting_lock<Mutex>& lock_arg) const
  { if (_connected) { _connected=false; dec_slot_refcount(lock_arg); }}
  virtual bool connected() const = 0;
  shared_ptr<void> get_blocker() { unique_lock<self> local_lock{*this}; shared_ptr<void> blocker = _weak_blocker.lock();
    if (blocker==shared_ptr<void>()) { blocker.reset(this, &null_deleter); _weak_blocker = blocker; } return blocker; }
  bool blocked() const { return !_weak_blocker.expred(); }
  bool nolock_nograb_blocked() const { return nolock_nograb_connected() == false || blocked(); }
  bool nolock_nograb_connected() const { return _connected; }
  virtual void lock() = 0; virtual void unlock() = 0;
  void inc_slot_refcount<Mutex>(const garbage_collecting_lock<Mutex>&) { ++m_slot_refcount; }
  void dec_slot_refcount<Mutex>(garbage_collecting_lock<Mutex>& lock_arg) const { if (--m_slot_refcount==0) { lock_arg.add_trash(release_slot()); } }
};
class detail::connection_body<GroupKey,SlotType,Mutex> : public connection_body_base {
  mutable shared_ptr<SlotType> m_slot; const shared_ptr<mutex_type> _mutex; GroupKey _group_key;
protected: virtual shared_ptr<void> release_slot() const { shared_ptr<void> released_slot = m_slot; m_slot.reset(); return released_slot; }
public: using mutex_type = Mutex;
  ctor(const SlotType& slot_in, const shared_ptr<mutex_type>& signal_mutex) : m_slot{new SlotType(slot_in)}, _mutex{signal_mutex} {}
  virtual ~dtor() {}
  virtual bool connected() const { garbage_collecting_lock<mutex_type> local_lock{*_mutex};
    nolock_grab_tracked_objects(local_lock, null_output_iterator{}); return nolock_nograb_connected(); }
  const GroupKey& group_key() const { return _group_key; }
  void set_group_key(const GroupKey& key) { _group_key = key; }
  void disconnect_expired_slot<M>(garbage_collecting_lock<M>& lock_arg)
  { if (!m_slot) return; if (slot().expired()) nolock_disconnect(lock_arg); }
  void nolock_grab_tracked_objects<M,OutIt>(garbage_collecting_lock<M>& lock_arg, OutIt inserter) const {
    if (!m_slot) return;
    for (auto obj : slot().tracked_objects()) {
      void_shared_ptr_variant locked_object{apply_visitor(lock_weak_ptr_visitor{}, obj)};
      if (apply_visitor(expired_weak_ptr_visitor{}, obj)) { nolock_disconnect(lock_arg); return; }
      *inserter++ = locked_object;
    }
  }
  virtual void lock() { _mutex->lock(); } virtual void unlock() { _mutex->unlock(); }
  <const> SlotType& slot() <const> { return *m_slot; }
};

class connection {
protected: weak_ptr<connection_body_base> _weak_connection_body;
public: ctor() noexcept {}
  ctor(const self& other) noexcept: _weak_connection_body{other._weak_connection_body} {}
  ctor(const weak_ptr<connection_body_base>& connectionBody) noexcept : _weak_connection_body{connectionBody} {}
  ctor(self&& other) noexcept: _weak_connection_body(std::move(other._weak_connection_body)) { other._weak_connection_body.reset(); }
  self& operator=(self&& other) noexcept { if (&other==this) return *this;
    _weak_connection_body = std::move(other._weak_connection_body); other._weak_connection_body.reset(); return *this; }
  self& operator=(const self& other) noexcept { if (&other==this) return *this;
    _weak_connection_body = other._weak_connection_body; return *this; }
  ~dtor() {}
  void disconnect() const {
    shared_ptr<connection_body_base> cbody{_weak_connection_body.lock()}; if (cbody==0) return;
    cbody->disconnect();
  }
  bool connected() const {
    shared_ptr<connection_body_base> cbody{_weak_connection_body.lock()}; if (cbody==0) return;
    return cbody->connected();
  }
  bool blocked() const {
    shared_ptr<connection_body_base> cbody{_weak_connection_body.lock()}; if (cbody==0) return;
    return cbody->blocked();
  }
  bool operator==(const self& other) const { // also !=, <
    shared_ptr<connection_body_base> cbody{_weak_connection_body.lock()}, othercbody{other._weak_connection_body.lock()};
    return cbody == othercbody;
  }
  void swap(self& other) noexcept { using std::swap; swap(_weak_connection_body, other._weak_connection_body); }
  friend void swap(self& conn1, self& conn2) noexcept { conn1.swap(conn2); }
};

class scoped_connection : public connection { // disable copy
public: ctor() noexcept {} ctor(const self& other) noexcept : base(other){} ~dtor() { disconnect(); }
  self& operator=(const connection& rhs) noexcept { disconnect(); base::operator=(rhs); return *this; }
  ctor({self|base}&& other) noexcept : base(std::move(other)) {}
  self& operator=({self|base}&& other) noexcept { if (&other == this) return *this;
    disconnect(); base::operator=(std::move(other)); return *this; }
  connection release() { connection conn{_weak_connection_body}; _weak_connection_body.reset(); return conn; }
  friend void swap(self& conn1, self& conn2) noexcept { conn1.swap(conn2); }
};

struct weak_ptr_traits<WeakPtr> {};
struct weak_ptr_traits<weak_ptr<T>> { using shared_type = shared_ptr<T>; }; // boost:: and std::
struct shared_ptr_traits<SharedPtr> {};
struct shared_ptr_traits<shared_ptr<T>> { using weak_type = weak_ptr<T>; }; // boost:: and std::

struct detail::foreign_shared_ptr_impl_base { virtual ~dtor() {}; virtual self* clone() const = 0; };
class detail::foreign_shared_ptr_impl<FSP> : public foreign_shared_ptr_impl_base { FSP _p;
public: ctor(const FSP& p) : _p{p} {} virtual self* clone() const { return new self{*this}; } };
class detail::foreign_void_shared_ptr { foreign_shared_ptr_impl_base* _p;
public: ctor(){}  ctor(const self& other) :_p{other._p->clone()} {}
  explicit ctor<FSP>(const FSP& fsp) :_p{new foreign_shared_ptr_impl<FSP>{fsp}} {}
  ~dtor() { delete _p; }
  self& operator=(const self& other) { if (&other==this) return *this; self{other}.swap(*this); return *this; }
  void swap(self& other) { core::invoke_swap(_p, other._p); }
};

struct detail::foreign_weak_ptr_impl_base { virtual ~dtor(){};
  virtual foreign_void_shared_ptr lock() const = 0;
  virtual bool expired() const = 0;
  virtual self* clone() const = 0;
};
class detail::foreign_weak_ptr_impl<FWP> : public foreign_weak_ptr_impl_base { FWP _p;
public: ctor(const FWP& p): _p{p} {}
  virtual foreign_void_shared_ptr lock() const { return {_p.lock()}; }
  virtual bool expired() const { return _p.expired(); }
  virtual self* clone() const { return new self{*this}; }
};
class detail::foreign_void_weak_ptr { scoped_ptr<foreign_weak_ptr_impl_base> _p;
public: ctor(){}  ctor(const self& other):_p{other._p->clone()}{}
  explicit ctor<FWP>(const FWP& fwp): _p{new foreign_weak_ptr_impl<FWP>{fwp}} {}
  self& operator=(const self& other) { if (&other==this) return *this; self{other}.swap(*this); return *this; }
  void swap(self& other) { core::invoke_swap(_p, other._p); }
  foreign_void_shared_ptr lock() const { return _p->lock(); }
  bool expired() const { return _p->expired(); }
};

struct signal_base : public noncopyable { virtual ~dtor(){}
protected: virtual shared_ptr<void> lock_pimpl() const = 0;
};
struct detail::is_signal<T> : mpl::bool_<is_base_of_v<signal_base, T>> {};
struct detail::signal_tag{}; struct detail::reference_tag{}; struct detail::value_tag{};
struct detail::get_slot_tag<S> { using signal_or_value = mpl::if_<is_signal<S>,signal_tag,value_tag>::value;
  using type = mpl::if_<is_reference_wrapper<S>,reference_tag,signal_or_value>::type;
};
F::weak_signal_type detail::get_invocable_slot(const F& signal, signal_tag) { return F::weak_signal_type{signal}; }
const F& detail::get_invocable_slot(const F& f, {reference_tag|value_tag}) { return f; }
get_slot_tag<F>::type detail::tag_type(const F&) { return {}; }

using detail::void_weak_ptr_variant = variant<weak_ptr<trackable_pointee>, weak_ptr<void>, foreign_void_weak_ptr>;
using detail::void_shared_ptr_variant = variant<shared_ptr<void>, foreign_void_shared_ptr>;
struct detail::lock_weak_ptr_visitor { using result_type = void_shared_ptr_variant;
  result_type operator()<WeakPtr>(const WeakPtr& wp) const { return wp.lock(); }
  result_type operator()(const weak_ptr<trackable_pointee>&) const { return shared_ptr<void>{}; }
};
struct detail::expired_weak_ptr_visitor { using result_type = bool;
  result_type operator()<WeakPtr>(const WeakPtr& wp) const { return wp.expired(); }
};
class slot_base {
protected: tracked_container_type _tracked_objects;
  void track_signal(const signal_base& signal) { _tracked_objects.push_back(signal.lock_pimpl()); }
public: using tracked_container_type = std::vector<void_weak_ptr_variant>;
  using locked_container_type = std::vector<void_shared_ptr_variant>;
  const tracked_container_type& tracked_objects() const { return _tracked_objects; }
  locked_container_type lock() const {
    locked_container_type locked_objects{};
    for (auto obj : tracked_objects()) {
      locked_objects.push_back(apply_visitor(lock_weak_ptr_visitor{}, std::forward(obj)));
      if (apply_visitor(expired_weak_ptr_visitor{}, std::forward(obj))) { throw_exception(expired_slot{}); }
    }
    return locked_objects;
  }
  bool expired() const {
    for (auto obj : tracked_objects()) if (apply_visitor(expired_weak_ptr_visitor{}, std::forward(obj))) return true;
    return false;
  }
};

ResultSlot detail::replace_slot_function<ResultSlot, SlotIn, SlotFunction>(const SlotIn& slot_in, const SlotFunction& fun)
{ ResultSlot slot{fun}; slot.track(slot_in); return slot; }

enum detail::slot_meta_group { front_ungrouped_slots, grouped_slots, back_ungrouped_slots };
struct detail::group_key<Group> { using type = std::pair<slot_meta_group,optional<Group>>; };
class detail::group_key_less<Group,GroupCompare> { GroupCompare _group_compare;
public: ctor(){}  ctor(const GroupCompare& group_compare): _group_compare{group_compare}{}
  bool operator() (const group_key<Group>::type& key1, const group_key<Group>::type& key2) const
  { if (key1.first!=key2.first) return key1.first < key2.first;
    if (key1.first!=grouped_slots) return false;
    return _group_compare(key1.second.get(), key2.second.get()); }
};

class grouped_list<Group,GroupCompare,ValueType> {
  using list_type = std::list<ValueType>;
  using map_type = std::map<group_key<Group>::type, list_type::iterator, group_key_compare_type>;
  using <const>_map_iterator = map_type::<const>_iterator;
  list_type _list; map_type _group_map; group_key_compare_type _group_key_compare;

  bool weakly_equivalent(const group_key_type& arg1, const group_key_type& arg2) {
    return !(_group_key_compare(arg1, arg2) || _group_key_compare(arg1, arg2));
  }
  void m_insert(const map_iterator& map_it, const group_key_type& key, const ValueType& value) {
    iterator list_it = get_list_iterator(map_it), new_it = _list.insert(list_it, value);
    if (map_it != _group_map.end() && weakly_equivalent(key, map_it->first)) _group_map.erase(map_it);
    auto lower_bound_it = _group_map.lower_bound(key);
    if (lower_bound_it == _group_map.end() || !weakly_equivalent(lower_bound_it->first, key)) _group_map.insert({key, new_it});
  }
  <const>_iterator get_list_iterator(const const_map_iterator& map_it) <const> {
    <const>_iterator list_it;
    if (map_it == _group_map.end()) list_it = _list.end(); else list_it = map_it->second;
    return list_it;
  }
public: using group_key_compare_type = group_key_less<Group,GroupCompare>;
  using <const>_iterator = list_type::<const>_iterator;
  using group_key_type = group_key<Group>::type;
  ctor(const group_key_compare_type& group_key_compare) : _group_key_compare(group_key_compare) {}
  ctor(const self& other) : _list(other._list), _group_map(other._group_map), _group_key_compare(other._group_key_compare) {
    auto this_list_it = _list.begin(); auto this_map_it = _group_map.begin();
    for (auto other_map_it = other._group_map.begin(); other_map_it!=other._group_map.end(); ++other_map_it, ++this_map_it) {
      this_map_it->second = this_list_it;
      auto other_list_it = other.get_list_iterator(other_map_it); auto other_next_map_it = other_map_it; ++other_next_map_it;
      auto other_next_list_it = other.get_list_iterator(other_next_map_it);
      while (other_list_it != other_next_list_it) {++other_list_it; ++this_list_it;}
    }
  }
  iterator begin() { return _list.begin(); } iterator end() { return _list.end(); }
  iterator lower_bound(const group_key_type& key) { return get_list_iterator(_group_map.lower_bound(key)); }
  iterator upper_bound(const group_key_type& key) { return get_list_iterator(_group_map.upper_bound(key)); }
  void push_front(const group_key_type& key, const ValueType& value) {
    if (key.first == front_ungrouped_slots) m_insert(_group_map.begin(), key, value);
    else m_insert(_group_map.lower_bound(key), key, value);
  }
  void push_back(const group_key_type& key, const ValueType& value) {
    if (key.first == front_ungrouped_slots) m_insert(_group_map.end(), key, value);
    else m_insert(_group_map.upper_bound(key), key, value);
  }
  void erase(const group_key_type& key) {
    auto map_it = _group_map.lower_bound(key);
    auto begin_list_it = get_list_iterator(map_it), end_list_it = upper_bound(key);
    if (begin_list_it != end_list_it) { _list.erase(begin_list_it, end_list_it); _group_map.erase(map_it); }
  }
  iterator erase(const group_key_type& key, const iterator& it) {
    auto map_it = _group_map.lower_bound(key);
    if (map_it->second==it) {
      auto next = it; ++next;
      if (next!=upper_bound(key)) { _group_map[key] = next; } else { _group_map.erase(map_it); }
    }
    return _list.erase(it);
  }
  void clear() { _list.clear(); _group_map.clear(); }
};

struct detail::slot_call_iterator_cache<ReturnType,Function> {
  optional<ReturnType> result;
  auto_buffer<void_shared_ptr_variant, store_n_objects<10>> tracked_ptrs;
  Function f; unsigned connected_slot_count{0}, disconnected_slot_count{0}; connection_body_base* m_active_slot{nullptr};
  ctor(const Function& f_arg) :f{f_arg}{}
  ~dtor() { if (m_active_slot) { garbage_collecting_lock<connection_body_base> lock{*m_active_slot}; m_active_slot->dec_slot_refcount(lock); }}
  void set_active_slot<M>(garbage_collecting_lock<M>& lock, connection_body_base* active_slot) {
    if (m_active_slot) m_active_slot->dec_slot_refcount(lock);
    m_active_slot = active_slot;
    if (m_active_slot) m_active_slot->inc_slot_refcount(lock);
  }
};
class detail::slot_call_iterator_t<Function,Iterator,ConnectionBody>
  : public iterator_facade<slot_call_iterator_t<Function,Iterator,ConnectionBody>, Function::result_type, single_pass_traversal_type> {
  using result_type = Function::result_type; using cache_type = slot_call_iterator_cache<result_type,Function>;
  using lock_type = garbage_collecting_lock<connection_body_base>;
  mutable Iterator iter; Iterator end; cache_type* cache; mutable Iterator callable_iter;

  void set_callable_iter(lock_type& lock, Iterator newValue) const {
    callable_iter = newValue;
    cache->set_active_slot(lock, callable_iter == end ? 0 : (*callable_iter).get());
  }
  void lock_next_callable() const {
    if (iter == callable_iter) return;
    for (; iter!=end; ++iter) {
      cache->traced_ptrs.clear();
      lock_type lock{**iter};
      (*iter)->nolock_grab_tracked_objects(lock, std::back_inserter(cache->traced_ptrs));
      if ((*iter)->nolock_nograb_connected()) ++cache->connected_slot_count; else ++cache->disconnected_slot_count;
      if (!(*iter)->nolock_nograb_blocked()) { set_callable_iter(lock, iter); break; }
    }
    if (iter==end) if (callable_iter!=end) { lock_type lock{**callable_iter}; set_callable_iter(lock, end); }
  }
public: ctor(Iterator iter_in, Iterator end_in, cache_type& c)
    : iter{iter_in}, end{end_in}, cache{&c}, callable_iter{end_in} { lock_next_callable(); }
  reference dereference() const {
    if (!cache->result) try{ cache->result = cache->f(*iter);} cache(...){(*iter)->disconnect(); throw;}
    return cache->result.get();
  }
  void increment() { ++iter; lock_next_callable(); cache->result.reset(); }
  bool equal(const self& other) const { return iter == other.iter; }
};

class detail::variadic_arg_type<0,T,Args...> { using type = T; };
class detail::variadic_arg_type<n,T,Args...> { using type = variadic_arg_type<n-1,Args...>::type; };
struct detail::std_functional_base<...Args>{};
struct detail::std_functional_base<T1> { using argument_type = T1; };
struct detail::std_functional_base<T1,T2> { using first_argument_type = T1; using second_argument_type = T2; };

struct detail::unsigned_meta_array<...values> {};
struct detail::unsigned_meta_array_appender<unsigned_meta_array<Args...>,n> { using type = unsigned_meta_array<Args...,n>; };
struct detail::make_unsigned_meta_array<0> { using type = unsigned_meta_array<>; };
struct detail::make_unsigned_meta_array<1> { using type = unsigned_meta_array<0>; };
struct detail::make_unsigned_meta_array<n> { using type = unsigned_meta_array_appender<make_unsigned_meta_array<n-1>::type, n-1>::type; };

class detail::call_with_tuple_args<R> {
  R m_invoke<Func,...indices,...Args>(Func& func, unsigned_meta_array<indices...>, const tuple<Args...>& args)
  { if constexpr(is_void<Func::result_type>) { func(get<indices>(args)...); return R{}; }
    else return func(get<indices>(args)...); }
public: using result_type = R;
  R operator()<Func,...Args,n>(Func& func, const tuple<Args...>& args, mpl::size_t<n>) const
  { return m_invoke<Func>(func, make_unsigned_meta_array<n>::type{}, args); }
};

class detail::variadic_slot_invoker<R,...Args> { tuple<Args&...> _args;
public: using result_type = R;
  ctor(Args&...args) : _args(args...) {}
  result_type operator()<ConnectionBodyType> (const ConnectionBodyType& connectionBody) const
  { return call_with_tuple_args<result_type>()(connectionBody->slot().slot_function(), _args, mpl::size_t<sizeof...(Args)>{}); }
};

struct detail::variadic_extended_signature<R (Args...)> { using function_type = function<R(const connection&, Args...)>; };

struct detail::bound_extended_slot_function_invoker<R> { using result_type = R;
  result_type operator()<ExtSlotFunc,...Args>(ExtSlotFunc& func, const connection& conn, Args&&...args) { return func(conn, std::forward<Args>(args)...); }
};
class detail::bound_extended_slot_function<ExtSlotFunc> { ExtSlotFunc _fun; shared_ptr<connection> _connection;
public: using result_type = result_type_wrapper<ExtSlotFunc::result_type>::type;
  ctor(const ExtSlotFunc& fun): _fun{fun}, _connection{new connection} {}
  void set_connection(const connection& conn) { *_connection = conn; }
  result_type operator()<...Args>(Args&&...args) <const>
  { return bound_extended_slot_function_invoker<ExtSlotFunc::result_type>{}(_fun, *_connection, std::forward<Args>(args)...); }
  bool contains<T>(const T& other) const { return _fun.contains(other); }
};

class detail::signal_impl<R(Args...),Combiner,Group,GroupCompare,SlotFunc,ExtSlotFunc,Mutex> {
  mutable shared_ptr<invocation_state> _shared_state;
  mutable connection_list_type::iterator _garbage_collector_it;
  const shared_ptr<mutex_type> _mutex;
public: using slot_function_type = SlotFunc; using slot_type = slot<R(Args...), slot_function_type>;
  using extended_slot_function_type = ExtSlotFunc; using extended_slot_type = slot<R (const connection &, Args...), extended_slot_function_type>;
  using nonvoid_slot_result_type = nonvoid<slot_function_type::result_type>::type;
private: using slot_invoker = variadic_slot_invoker<nonvoid_slot_result_type, Args...>;
  using slot_call_iterator_cache_type = slot_call_iterator_cache<nonvoid_slot_result_type, slot_invoker>;
  using group_key_type = group_key<Group>::type;
  using connection_body_type = shared_ptr<connection_body<group_key_type, slot_type, Mutex>>;
  using connection_list_type = grouped_list<Group, GroupCompare, connection_body_type>;
  using bound_extended_slot_function_type = bound_extended_slot_function<extended_slot_function_type>;
  using mutex_type = Mutex;
  class invocation_state { shared_ptr<connection_list_type> _connection_bodies; shared_ptr<combiner_type> _combiner;
  public: ctor(const connection_list_type& connections_in, const combiner_type& combiner_in)
          : _connection_bodies{new connection_list_type{connections_in}}, _combiner{new combiner_type{combiner_in}} {}
    ctor(const self& other, const connection_list_type& connections_in)
      : _connection_bodies{new connection_list_type{connections_in}}, _combiner{other._combiner} {}
    ctor(const self& other, const combiner_type& combiner_in)
      : _connection_bodies{other._connection_bodies}, _combiner{new combiner_type{combiner_in}} {}
    <const> connection_list_type& connection_bodies() <const> { return *_connection_bodies; }
    <const> combiner_type& combiner() <const> { return *_combiner; }
  };
  class invocation_janitor: noncopyable { const slot_call_iterator_cache_type& _cache; const signal_type& _sig; const connection_list_type* _connection_bodies;
  public: ctor(const slot_call_iterator_cache_type& cache, const signal_type& sig, const connection_list_type* connection_bodies)
          : _cache{cache}, _sig{sig}, _connection_bodies{connection_bodies} {}
    ~dtor() { if (_cache.disconnected_slot_count > _cache.connected_slot_count) _sig.force_cleanup_connections(_connection_bodies); }
  };
  void nolock_cleanup_connections_from(garbage_collecting_lock<mutex_type>& lock, bool grab_tracked, const connection_list_type::iterator& begin, unsigned count=0) const {
    auto it = begin; unsigned i = 0;
    for (;it != _shared_state->connection_bodies().end() && (count==0||i<count); ++i) {
      if (grab_tracked) (*it)->disconnect_expired_slot(lock);
      if (!(*it)->nolock_nograb_connected()) it = _shared_state->connection_bodies().erase((*it)->group_key(), it); else ++it;
    }
    _garbage_collector_it = it;
  }
  void nolock_cleanup_connections(garbage_collecting_lock<mutex_type>& lock, bool grab_tracked, unsigned count) const {
    nolock_cleanup_connections_from(lock, grab_tracked,
      _garbage_collector_it==_shared_state->connection_bodies().end() ? _shared_state->connection_bodies().begin() : _garbage_collector_it, count);
  }
  void nolock_force_unique_connection_list(garbage_collecting_lock<mutex_type>& lock) {
    if (!_shared_state.unique()) { _shared_state = make_shared<invocation_state>(*_shared_state, _shared_state->connection_bodies());
      nolock_cleanup_connections_from(lock, true, _shared_state->connection_bodies().begin());
    } else nolock_cleanup_connections(lock, true, 2);
  }
  void force_cleanup_connections(const connection_list_type* connection_bodies) const {
    garbage_collecting_lock<mutex_type> list_lock{*_mutex};
    if (&_shared_state->connection_bodies()!=connection_bodies) return;
    if (!_shared_state.unique()) _shared_state = make_shared<invocation_state>(*_shared_state, _shared_state->connection_bodies());
    nolock_cleanup_connections_from(list_lock, false, _shared_state->connection_bodies().begin());
  }
  shared_ptr<invocation_state> get_readable_state() const { unique_lock<mutex_type> list_lock{*_mutex}; return _shared_state; }
  connection_body_type create_new_connection(garbage_collecting_lock<mutex_type>& lock, const slot_type& slot) {
    nolock_force_unique_connection_list{lock};
    return make_shared<connection_body<group_key_type, slot_type, Mutex>>(slot, _mutex);
  }
  void do_disconnect(const group_type& group, mpl::bool_<true>) { disconnect(group); }
  void do_disconnect<T>(const T& slot, mpl::bool_<false>) {
    shared_ptr<invocation_state> local_state = get_readable_state();
    for (auto conn : local_state->connection_bodies()) {
      garbage_collecting_lock<connection_body_base> lock{*conn};
      if (!conn->nolock_nograb_connected()) continue;
      if (conn->slot().slot_function().contains(slot)) conn->nolock_disconnect(lock);
      else { auto fp = conn->slot().slot_function().target<bound_extended_slot_function_type>();
        if (fp && fp->contains(slot)) conn->nolock_disconnect(lock);
        else { auto fp = conn->slot().slot_function().target<weak_signal_type>();
          if (fp && fp->contains(slot)) conn->nolock_disconnect(lock);
        }
      }
    }
  }
  connection nolock_connect(garbage_collecting_lock<mutex_type>& lock, const slot_type& slot, connect_position position) {
    auto newConn = create_new_connection(lock, slot);
    group_key_type group_key{};
    if (position == at_back) { group_key.first = back_ungrouped_slots; _shared_state->connection_bodies().push_back(group_key, newConn); }
    else { group_key.first = front_ungrouped_slots; _shared_state->connection_bodies().push_front(group_key, newConn); }
    newConn->set_group_key(group_key); return {newConn};
  }
  connection nolock_connect(garbage_collecting_lock<mutex_type>& lock,
      <const group_type& group>, const slot_type& slot, connect_position position) {
    auto newConn = create_new_connection(lock, slot);
    group_key_type group_key{grouped_slots, group};
    if (position == at_back) _shared_state->connection_bodies().push_back(group_key, newConn);
    else _shared_state->connection_bodies().push_front(group_key, newConn);
    newConn->set_group_key(group_key); return {newConn};
  }

public: using combiner_type = Combiner; using result_type = result_type_wrapper<combiner_type::result_type>::type;
  using group_type = Group; using group_compare_type = GroupCompare;
  using slot_call_iterator = detail::slot_call_iterator_t<slot_invoker, connection_list_type::iterator, connection_body<group_key_type, slot_type, Mutex>>;
  using weak_signal_type = weak_signal<R (Args...),Combiner,Group,GroupCompare,SlotFunc,ExtSlotFunc,Mutex>;
  ctor(const combiner_type& combiner_arg, const group_compare_type& group_compare)
    : _shared_state{make_shared<invocation_state>(connection_list_type{group_compare}, combiner_arg)},
      _garbage_collector_it{_shared_state->connection_bodies().end()}, _mutex{new mutex_type{}} {}
  connection connect(<const group_type& group>, const slot_type &slot, connection_position position=at_back)
  { garbage_collecting_lock<mutex_type> lock{*_mutex}; return nolock_connect(lock, <group>, slot, position); }
  connection connect_extended(<const group_type& group>, const extended_slot_type &ext_slot, connection_position position=at_back)
  { garbage_collecting_lock<mutex_type> lock{*_mutex};
    bound_extended_slot_function_type bound_slot{ext_slot.slot_function()};
    auto slot = replace_slot_function<slot_type>(ext_slot, bound_slot);
    connection conn = nolock_connect(lock, <group>, slot, position);
    bound_slot.set_connection(conn); return conn; }
  void disconnect_all_slots() { shared_ptr<invocation_state> local_state = get_readable_state();
    for (auto&& conn : local_state->connection_bodies()) conn.disconnect();
  }
  void disconnect(const group_type& group) { shared_ptr<invocation_state> local_state = get_readable_state();
    group_key_type group_key{grouped_slots, group};
    auto end_it = local_state->connection_bodies().upper_bound(group_key);
    for (auto it = local_state->connection_bodies().lower_bound(group_key); it!=end_it; ++it) { (*it)->disconnect(); }
  }
  void disconnect<T>(const T& slot) { using is_group = is_convertible_v<T,group_type>; do_disconnect(unwrap_ref(slot), is_group{}); }
  result_type operator()(Args ... args)<const> {
    shared_ptr<invocation_state> local_state;
    { garbage_collecting_lock<mutex_type> list_lock{*_mutex}; if (_shared_state.unique()) nolock_cleanup_connections(list_lock, false, 1); local_state = _shared_state;}
    slot_invoker invoker{args...}; slot_call_iterator_cache_type cache{invoker}; invocation_janitor janitor{cache, *this, &local_state->connection_bodies()};
    return combiner_invoker<combiner_type::result_type>{}(local_state->combiner(),
      slot_call_iterator{local_state->connection_bodies().begin(), .end(), cache}, slot_call_iterator{local_state->connection_bodies().end(), .end(), cache});
  }
  size_t num_slots() const { shared_ptr<invocation_state> local_state = get_readable_state(); size_t count=0;
    for (auto obj: local_state->connection_bodies()) if (obj->connected()) ++count;
    return count;
  }
  bool empty() const { shared_ptr<invocation_state> local_state = get_readable_state();
    for (auto obj: local_state->connection_bodies()) if (obj->connected()) return false;
    return true;
  }
  combiner_type combiner() const { unique_lock<mutex_type> lock{*_mutex}; return _shared_state->combiner(); }
  void set_combiner(const combiner_type& combiner_arg) { unique_lock<mutex_type> lock{*_mutex};
    if (_shared_state.unique()) _shared_state->combiner() = combiner_arg;
    else _shared_state = make_shared<invocation_state>(*_shared_state, combiner_arg);
  }
};

class signal<R(Args...),Combiner,Group,GroupCompare,SlotFunc,ExtSlotFunc,Mutex> : public signal_base, public std_functional_base<Args...> {
  using impl_class = signal_impl<R(Args...),Combiner,Group,GroupCompare,SlotFunc,ExtSlotFunc,Mutex>;
  shared_ptr<impl_class> _pimpl;
protected: virtual shared_ptr<void> lock_pimpl() const { return _pimpl; }
public: using weak_signal_type = impl_class::weak_signal_type;
  using slot_function_type = SlotFunc; using slot_type = impl_class::slot_type;
  using extended_slot_function_type = impl_class::extended_slot_function_type; using extended_slot_type = impl_class::extended_slot_type;
  using slot_result_type = slot_function_type::result_type;
  using combiner_type = Combiner; using result_type = impl_class::result_type;
  using group_type = Group; using group_compare_type = GroupCompare;
  using slot_call_iterator = impl_class::slot_call_iterator;
  using signature_type = mpl::identity<R(Args...)>::type;

  struct arg<n>{ using type = variadic_arg_type<n,Args...>::type; };
  constexpr static int arity = sizeof...(Args);

  ctor(const combiner_type& combiner_arg={}, const group_compare_type& group_compare={})
    : _pimpl{new impl_class{combiner_arg, group_compare}} {}
  virtual ~dtor(){}
  ctor(self&& other) noexcept { using std::swap; swap(_pimpl, other._pimpl); }
  self& operator=(self&& rhs) noexcept
  { if (this==&rhs) return *this; _pimpl.reset(); using std::swap; swap(_pimpl, rhs._pimpl); return *this; }
  connection connect(<const group_type& group>, const slot_type& slot, connection_position position=at_back)
  { return (*_pimpl).connect(<group>, slot, position); }
  connection connect_extended(const group_type& group, const extended_slot_type &slot, connect_position position = at_back)
  { return (*_pimpl).connect_extended(<group>, slot, position); }
  void disconnect_all_slots() { if (_pimpl.get()==0) return; (*_pimpl).disconnect_all_slots(); }
  void disconnect(const group_type& group) { if (_pimpl.get()==0) return; (*_pimpl).disconnect(group); }
  void disconnect<T>(const T& slot) { if (_pimpl.get()==0) return; (*_pimpl).disconnect(slot); }
  result_type operator() <const> (Args...args) { return (*_pimpl)(args...); }
  size_t num_slots() const { if (_pimpl.get()==0) return 0; return (*_pimpl).num_slots(); }
  bool empty() const { if (_pimpl.get()==0) return true; return (*_pimpl).empty(); }
  combiner_type combiner() const { return (*_pimpl).combiner(); }
  void set_combiner(const combiner_type& combiner_arg) { return (*_pimpl).set_combiner(combiner_arg); }
  void swap(self& other) noexcept { using std::swap; swap(_pimpl, other._pimpl); }
  bool operator==(const self& other) { return _pimpl.get() == other._pimpl.get(); }
  bool null() const { return _pimpl.get() == 0; }

  friend void swap(self& sig1, self& sig2) { sig1.swap(sig2); }
};

class detail::weak_signal<R(Args...), Combiner,Group,GroupCompare,SlotFunc,ExtSlotFunc,Mutex> {
  using signal_type = signal<R(Args...), Combiner,Group,GroupCompare,SlotFunc,ExtSlotFunc,Mutex>;
  using signal_impl_type = signal_impl<R(Args...), Combiner,Group,GroupCompare,SlotFunc,ExtSlotFunc,Mutex>;
  weak_ptr<signal_impl_type> _weak_pimpl;
public: using result_type = signal_type::result_type;
  ctor(const signal_type& signal) : _weak_pimpl(signal._pimpl) {}
  result_type operator() (Args...args) <const>
  { shared_ptr<signal_impl_type> shared_pimpl{_weak_pimpl.lock()}; return (*shared_pimpl)(args...); }
  bool contains(const signal_type& signal) const { return _weak_pimpl.lock().get() == signal._pimpl.get(); }
  bool contains<T>(const T&) const { return false; }
};

class detail::extended_signature<arity,Signature> : public variadic_extended_signature<Signature> {};

struct detail::void_type{};
struct detail::nonvoid<R>{ using type = R; };
struct detail::nonvoid<void>{ using type = void_type; };
struct detail::result_type_wrapper<R> { using type = R; };
struct detail::combiner_invoker { using result_type = R;
  result_type operator() <Combiner,InIt>(Combiner& combiner, InIt first, InIt last) const { return combiner(first, last); }
};
struct detail::combiner_invoker<void> { using result_type = result_type_wrapper<void>::type;
  result_type operator() <Combiner,InIt>(Combiner& combiner, InIt first, InIt last) const { combiner(first, last); return result_type{}; }
};

struct keywords::signature_type<T> : parameter::template_keyword<struct tag::signature_type,T> {};
struct keywords::combiner_type<T> : parameter::template_keyword<struct tag::combiner_type,T> {};
struct keywords::group_type<T> : parameter::template_keyword<struct tag::group_type,T> {};
struct keywords::group_compare_type<T> : parameter::template_keyword<struct tag::group_compare_type,T> {};
struct keywords::slot_function_type<T> : parameter::template_keyword<struct tag::slot_function_type,T> {};
struct keywords::extended_slot_function_type<T> : parameter::template_keyword<struct tag::extended_slot_function_type,T> {};
struct keywords::mutex_type<T> : parameter::template_keyword<struct tag::mutex_type,T> {};

class signal_type<A0,A1=parameter::void_,...,A6=parameter::void_> {
  using parameter_spec = parameters<required<tag::signature_type, is_function<mpl::_>>,
    optional<tag::combiner_type>, optional<tag::group_type>, optional<tag::group_compare_type>,
    optional<tag::slot_function_type>, optional<tag::extended_slot_function_type>, optional<tag::mutex_type>>;
public: using args = parameter_spec::bind<A0,...,A6>::type;
  using signature_type = value_type<args, tag::signature_type>::type;
  using combiner_type = value_type<args, tag::combiner_type, optional_last_value<function_traits<signature_type>::result_type>>::type;
  using group_type = value_type<args, tag::group_type, int>::type;
  using group_compare_type = value_type<args, tag::group_compare_type, std::less<group_type>>::type;
  using slot_function_type = value_type<args, tag::slot_function_type, function<signature_type>>::type;
  using extended_slot_function_type = value_type<args, tag::extended_slot_function_type, extended_signature<function_traits<signature_type>::arity, signature_type>::function_type>::type;
  using mutex_type = value_type<args, tag::mutex_type, mutex>::type;
  using type = signal<signature_type, combiner_type, group_type, group_compare_type, slot_function_type, extended_slot_function_type, mutex_type>;
};

class shared_connection_block {
  weak_ptr<connection_body_base> _weak_connection_body; shared_ptr<void> _blocker;
public: ctor(const connection& conn={}, bool initially_blocked=true)
    : _weak_connection_body{conn._weak_connection_body} { if (initially_blocked) block(); }
  void block() { if (blocking()) return;
    shared_ptr<connection_body_base> connection_body{_weak_connection_body.lock()};
    if (connection_body==0) { _blocker.reset(nullptr); return; }
    _blocker = connection_body->get_blocker();
  }
  void unblock() { _blocker.reset(); }
  bool blocking() const { shared_ptr<void> empty; return _block < empty || empty < _blocker; }
  connection connection() const { return {_weak_connection_body}; }
};

struct detail::trackable_pointee{};
class trackable{ shared_ptr<trackable_pointee> _tracked_ptr{nullptr};
  weak_ptr<trackable_pointee> get_weak_ptr() const { return _tracked_ptr; }
protected: ctor(){} ctor(const self&){} ~dtor(){}
  self& operator=(const self&) { return *this; }
};
class detail::tracked_objects_visitor { mutable slot_base* slot_;
  void m_visit_reference_wrapper<T>(const reference_wrapper<T>& t, const mpl::bool_<true>&) const
  { m_visit_pointer(t.get_pointer(), mpl::bool_<true>{}); }
  void m_visit_reference_wrapper<T>(const T& t, const mpl::bool_<false>&) const
  { m_visit_pointer(t, mpl::bool_<is_pointer<T>::value>{}); }
  void m_visit_pointer<T>(const T& t, const mpl::bool<true>&) const
  { m_visit_not_function_pointer(t, mpl::bool_<!is_function_t<remove_pointer_t<T>>>{}); }
  void m_visit_pointer<T>(const T& t, const mpl::bool<false>&) const
  { m_visit_pointer(addressof(t), mpl::bool_<true>{}); }
  void m_visit_not_function_pointer<T>(const T* t, const mpl::bool<true>&) const
  { m_visit_signal(t, mpl::bool_<is_signal_v<T>>{}); }
  void m_visit_not_function_pointer<T>(const T&, const mpl::bool<false>&) const {}
  void m_visit_signal<T>(const T* signal, const mpl::bool_<true>&) const { if (signal) slot_->track_signal(*signal); }
  void m_visit_signal<T>(const T& t, const mpl::bool_<false>&) const { add_if_trackable(t); }
  void add_if_trackable(const trackable* trackable) const
  { if (trackable) slot_->_tracked_objects.push_back(trackable->get_weak_ptr()); }
  void add_if_trackable(const void*) const {}
public: ctor(slot_base* slot) : slot_{slot}{}
  void operator() <T> (const T& t) const { m_visit_reference_wrapper(t, mpl::bool_<is_reference_wrapper<T>::value>()); }
};

class slot<R(Args...),SlotFunc> : public slot_base, public std_functional_base {
  SlotFunc _slot_function;
  void init_slot_function<F>(const F& f) {
    _slot_function = get_invocable_slot(f, tag_type(f));
    boost::visit_each(tracked_objects_visitor{this});
  }
public: using slot_function_type = SlotFunc; using result_type = R;
  using signature_type = mpl::identity<R (Args...)>::type;
  struct arg<n> { using type = variadic_arg_type<n,Args...>::type; };
  constexpr static int arity = sizeof...(Args);
  ctor<F>(const F& f) { init_slot_function(f); }
  ctor<Signature,OtherSlotFunc>(const slot<Signature,OtherSlotFunc>& other_slot) : slot_base{other_slot}, _slot_function{other_slot._slot_function} {}
  ctor<A1,A2,...BindArgs>(const A1& arg1, const A2& arg2, const BindArgs&...args) { init_slot_function(bind(arg1, arg2, args...)); }
  R operator()(Args...args) <const> { locked_container_type locked_objects = lock(); return _slot_function{args...}; }
  self& track(const weak_ptr<void>& tracked) { _tracked_objects.push_back(tracked); return *this; }
  self& track(const signal_base& signal) { track_signal(signal); return *this; }
  self& track(const slot_base& slot) { for (auto obj : slot.tracked_objects()) _tracked_objects.push_back(obj); return *this; }
  self& track_foreign<ForeignWeakPtr>(const ForeignWeakPtr& tracked, weak_ptr_traits<ForeignWeakPtr>::shared_type*=0)
  { _tracked_objects.push_back(foreign_void_weak_ptr{tracked}); return *this; }
  self& track_foreign<ForeignSharedPtr>(const ForeignSharedPtr& tracked, shared_ptr_traits<ForeignSharedPtr>::weak_type*=0)
  { _tracked_objects.push_back(foreign_void_weak_ptr{shared_ptr_traits<ForeignSharedPtr>::weak_type{tracked}}); return *this; }
  <const> slot_function_type& slot_function() <const> { return _slot_function; }
};
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Bind

* `<boost/bind/bind.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`

#### Boost.Core

* `<boost/core/addressof.hpp>`
* `<boost/core/allocator_access.hpp>`
* `<boost/core/checked_delete.hpp>`
* `<boost/core/enable_if.hpp>`
* `<boost/core/invoke_swap.hpp>`
* `<boost/core/no_exceptions_support.hpp>`
* `<boost/core/noncopyable.hpp>`
* `<boost/core/ref.hpp>`
* `<boost/core/visit_each.hpp>`

#### Boost.Function

* `<boost/function.hpp>`

#### Boost.Iterator

* `<boost/iterator/reverse_iterator.hpp>`
* `<boost/iterator/iterator_traits.hpp>`

#### Boost.Move

* `<boost/move/utility_core.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.Optional

* `<boost/optional.hpp>`

#### Boost.Parameter

* `<boost/parameter/**.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.SmartPtr

* `<boost/scoped_ptr.hpp>`
* `<boost/shared_ptr.hpp>`, `<boost/shared_ptr/bad_weak_ptr.hpp>`
* `<boost/weak_ptr.hpp>`
* `<boost/smart_ptr/make_shared.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Tuple

* `<boost/tuple/tuple.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

#### Boost.Variant

* `<boost/variant/apply_visitor.hpp>`
* `<boost/variant/variant.hpp>`

------
### Standard Facilities
