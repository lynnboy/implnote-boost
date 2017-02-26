# Boost.Flyweight

* lib: `boost/libs/flyweight`
* repo: `boostorg/flyweight`
* commit: `b85af321`, 2016-01-19

------
### Flyweight Framework

Header `<boost/flyweight.hpp>`

#### Value Policy

```c++
concept bool ValuePolicy<T> = requires {
  typename T::key_type; typename T::value_type; typename T::rep_type;
  requires (const T::rep_type& r) {
    T::construct_value(r);  T::copy_value(r);  T::move_value(r);
    { r } -> T::key_type const &;   { r } -> T::value_type const &;
  }
  requires DefaultConstructible<T::rep_type> && Destructible<T::rep_type>
    && CopyConstructible<T::rep_type> && MoveConstructible<T::rep_type>;
  requires (const T::key_type& k) { {k} -> rep_type; }  // implicitly construcible (move)
  requires (const T::value_type& v) { {v} -> rep_type; } // implicitly construcible (move)
};
```

* A `rep_type` is expected to operates the _key_ part when constructing/copying, the operation
  for _value_ are performed when `construct_value`, `copy_value`, `move_value` are called.
* `rep_type` should be forward-constructs as a `key_type`, and implicitly treated as a `key_type`.

#### Factory

```c++
concept bool Factory<T, Entry> = requires (T const & t) {
  typename T::handle_type;
  { f.insert(r) } -> T::handle_type;
  requires (T::handle_type h) {
    f.erase(h);
    { f.entry(h) } -> Entry const&;
  }
  requires DefaultConstructible<T> && Destructible<T>;
};
```

A factory is a repository to store and fetch entrys.

#### Singleton Holder

```
concept bool Holder<T, C> = requires {
  { T::get() } -> C&; // gets a singleton instance
};
```

#### Locking Policy

```
concept bool LockingPolicy<T> = requires {
  typename T::mutex_type; typename T::lock_type;
  requires DefaultConstructible<T::mutex_type> && Destructible<T::mutex_type>;
  requires Constructible<T::lock_type, T::mutex_type> && Destructible<T::lock_type>;
};
```

#### Tracking Policy

```
concept bool TrackingHelper<T, Handle, Entry, Checker> = requires {const Handle & h, const Checker& c) {
  { T::entry(h) } -> Entry const &;
  T::erase(h, c);
};
concept bool TrackingPolicy<T, Value, Key, Handle, TrackingHelper> = requires {
  using entry_type = typename T::entry_type::apply<Value, Key>::type;
  using handle_type = typename T::handle_type::apply<Handle, TrackingHelper>::type;
  requires Constructible<entry_type, Value>;
  requires (entry_type const& e) { {e} -> Value const&； {e} -> Key const&; }
  requires CopyConstructible<entry_type> && CopyAssignable<entry_type> && Destructible<entry_type>;
  requires Constructible<handle_type, Handle>;
  requires (handle_type const& h) { {h} -> Handle const&； }
  requires CopyConstructible<handle_type> && CopyAssignable<handle_type>
    && Destructible<handle_type> && Swappable<handle_type>;
};
```

* Tracking policy wraps the value and handle types to add bookkeeping info for tracking lifetime.
* A checker predicate is provided when invoking `erase` through the helper to allow actual erasing.

#### Flyweight Core Implementation

```c++
template<ValuePolicy, Tag, TrackingPolicy, FactorySpecifier, LockingPolicy, HolderSpecifier>
struct flyweight_core_tracking_helper{ // implements 'TrackingHelper'
  using handle_type = flyweight_core::handle_type; // entry_type;
  static const entry_type& entry(const handle_type& h) { return flyweight_core::entry(h); }
  static void erase<Checker>(const handle_type& h, Checker chk) {
    flyweight_core::init(); lock_type lock(flyweight_core::mutex());
    if (chk(h)) flyweight_core::factory().erase(h);
  }
}

template<ValuePolicy, Tag, TrackingPolicy, FactorySpecifier, LockingPolicy, HolderSpecifier>
class flyweight_core {
  static bool static_init = init();                                 // statically init
  struct holder_arg { factory_type factory; mutex_type mutex; };    // one factory, one mutex
  using holder_type = HolderSpecifier<holder_arg>::type;            // singleton holder
public:
  using key_type = ValuePolicy::key_type; // value_type, rep_type
  using entry_type = TrackingPolicy::entry_type<rep_type, key_type>::type;  // actual stored type
  using factory_type = FactorySpecifier<entry_type, key_type>::type;        // factory for `entry_type`
  using base_handle_type = factory_type::handle_type;
  using handle_type = TrackingPolicy::handle_type<base_handle_type,
    flyweight_core_tracking_helper<...>>::type;                             // actual handle type
  using mutex_type = LockingPolicy::mutex_type; using lock_type = LockingPolicy::lock_type;
  
  static bool init() {
    if (!static_init) { holder_arg& a = holder_type::get(); static_init=true; }
    return static_init;
  }
  static factory_type& factor(); static mutex_type& mutex();

  static handle_type insert<...Args>(Args...args) {
    init(); lock_type lock(mutex());
    base_handle_type h = factory_type::insert(entry_type(rep_type(std::forward(args)...)));
    try { ValuePolicy::construct_value(entry(h)); } catch(...) { factory().erase(h); throw; }
    return handle_type(h);
  }
  static handle_type insert([const] value_type&[&] x);  // similar as above
  static const entry_type& entry(const base_handle_type& h) { return factory().entry(h); }
  static const value_type& value(const handle_type& h) { return (rep_type const&)(entry(h)); }
  static const key_type& key(const handle_type& h) { return (rep_type const&)(entry(h)); }
};
```

* Erasing go only through helper by the `TrackingPolicy`.
* `TrackingPolicy` wraps `base_handle_type` and `rep_type` as `handle_type` and `entry_type`,
  which are convertible to the wrapped types.
* `Tag` is just a signature aid to distinguish different instances for same value types.

#### Flyweight API

```c++
template <typename T, ...Opts>
class flyweight {
  using value_policy = is_value<T> ? T : default_value_policy;      
  using tag_type = find<Opts..., is_tag, mpl::na>::type;
  using tracking_policy = find<Opts..., is_tracking, refcounted>::type;   // default is refcounted
  using factory_specifier = find<Opts..., is_factory, hashed_factory<>>::type; // default is hash_factory
  using locking_policy = find<Opts..., is_locking, simple_locking>::type;   // default is simple_locking
  using holder_specifier = find<Opts..., is_holder, static_holder>::type;   // default is static_holder
  using core = flyweight_core<value_policy, tag_type, tracking_specifier,
                              factory_specifier, locking_policy, holder_specifier>;

  core::handle_type h;        // flyweight data
public:
  using key_type = value_policy::key_type; // value_type
  class initializer { initializer() { flyweight::init(); } }; // just ensure initialize
  static bool init() { return core::init(); }

  flyweight() : h(core::insert()) {}
  explicit flyweight<...Args>(Args...args) : h(core::insert(std::forward(args)...)) {}
  flyweight<V>(initializer_list<V>) : h(core::insert(list)) {}
  // copy-ctor, move-ctor, copy-assign, move-assign, init-list-assign
  // operator= for copy/move value_type

  const key_type& get_key() const { return core::key(h); }
  const value_type& get() const { return core::value(h); }
  operator const value_type&() const { return get(); }      // flyweight implicit convertible as `value_type`
  
  void swap(flyweight&);
};
// ==, !=, <, >, <=, >=, swap, <<, >>
std::size_t hash_value(const flyweight&);  // call 'boost::hash<const value_type*>'
struct ::std::hash<flyweight<T, ...Opts>>; // call 'std::hash<const value_type*>'
```

* A `flyweight<T, ...>` can be used in many ways just like a `T`
* A `flyweight` wraps a small handle, used to fetch the interned actual value from the factory.
* The options (policies) are recognized in the *Boost.MPL* lambda expression manner.
* The options are manipulated in the *Boost.Parameters* free-ordered keyword parameter manner.

------
### Predefined Policies

#### Value Policies

```c++
struct value_marker;
concept bool is_value<T> = is_base_and_derived<value_marker, T>;
struct value<T> : parameter::template_keyword<value<>,T>;
struct default_value_policy<Value> : value_marker {   // just treat Value both as key and value
  using key_type = Value; using value_type = Value;
  struct rep_type {
    value_type x;
  };
  // empty construct_value(), copy_value(), move_value() functions, no operations needed.
};

struct optimized_key_value<Key,Value,KeyFromValue> : value_marker {
  using key_type = Key; using value_type = Value;
  struct rep_type {
    aligned_storage<key_type, value_type> spc;  // store key before value is constructed
    const value_type*   value_ptr;    // 'value_ptr != &spc' means construct/copy/move delayed
    operator const key_type&() const { return KeyFromValue()(*value_ptr); }
  };
  // construct_value(), copy_value(), move_value() functions, handling 'value_ptr != &spc' case
};

struct regular_key_value<Key,Value> : value_marker {
  using key_type = Key; using value_type = Value;
  struct rep_type { // not constructable from value_type, because key cannot be fetched wherein
    aligned_storage<key_type, value_type> spc;  // store key before value is constructed
    const value_type*   value_ptr;    // 'value_ptr != &spc' means construct/copy/move delayed
  };
  // copy_value(), move_value() functions are meaningless
};

using key_value<Key, Value, KeyFromValue> = is_same<KeyFromValue, no_key_from_value> ?
  regular_key_value<Key,Value> : optimized_key_value<Key,Value,KeyFromValue>;
```

* Any type is by default treated as both the `Key` for identifing and the `Value`, by `default_value_policy`.
* Explicit `key_value<Key,Value>` will store both `Key` and `Value` in the `rep_type`, and disables the ability
  to recognize a `Value` in the repository and converts to a `Key` or `flyweight` for it.
* Explicit `key_value<Key,Value,KeyFromValue>` will store just `Value`, and provide two-way mapping support.

#### Predefined Factories

```c++
struct factory_marker;
concept bool is_factory<T> = is_base_and_derived<factory_marker, T>;
struct factory<T> : parameter::template_keyword<factory<>,T>;

struct hashed_factory<Hash=mpl::na, Pred=mpl::na, Allocator=mpl::na>; // specifier
class hashed_factory_class<Entry, Key, Hash, Pred, Allocator> : factory_marker {
  using index_list = mpl::vector1<multi_index::hashed_unique< // params for hash container
    /*KeyFromValue=*/identity<Entry>,
    /*Hash=*/is_na<Hash> ? boost::hash<Key> : Hash,     // boost::hash by default
    /*Pred=*/is_na<Pred> ? std::equal_to<Key> : Pred>;  // std::equal_to by default
  using container_type = multi_index::multi_index_container<Entry, index_list,
    is_na<Allocator> ? std::allocator<Entry> : Allocator>::type;
  container_type cont; // hash table repository
public:
  typedef const Entry* handle_type;
  // insert, erase, entry
};

struct assoc_container_factory<ContainerSpecifier>; // specifier
class assoc_container_factory_class<Container> : factory_marker {
  Container cont;
public:
  using handle_type = Container::iterator;
  using entry_type = Container::value_type;
  // insert, erase, entry
};

struct set_factory<Compare=mpl::na, Allocator=mpl::na>; // specifier
class set_factory_class<Entry, Key, Compare, Allocator>
  : public assoc_container_factory<std::set<Entry,      // use std::set
      mpl::is_na<Compare> ? std::less<Key> : Compare,   // std::less by default
      mpl_is_na<Allocator> ? std::allocator<Entry> : Allocator> > {}; // std::allocator by default
```

* Default factory is `hashed_factory`

#### Tagging

```c++
struct tag_marker;
concept bool is_tag<T> = is_base_and_derived<tag_marker, T>;
struct tag<T> : parameter::template_keyword<tag<>,T>;
```

* Use `tag<anytype>` as signature tagging facility.

#### Singleton Holders For Factory

```c++
struct holder_marker;
concept bool is_holder<T> = is_base_and_derived<holder_marker, T>;
struct holder<T> : parameter::template_keyword<holder<>,T>;

struct static_holder; // specifier
struct static_holder_class<C> : holder_marker {
  static C& get() { static C c; return c; }     // static local data
};

struct intermodule_holder; // specifier
struct intermodule_holder_class<C> : holder_marker,
  interprocess::ipcdetail::intermodule_singleton<C,true> { };
```

* The type `C` contains the signature of `flyweight_core`, which contains the type tag
* `intermodule_holder` implemented upon *Boost.InterProcess* inter-module singleton.

#### Locking Policy

```c++
struct locking_marker;
concept bool is_locking<T> = is_base_and_derived<locking_marker, T>;
struct locking<T> : parameter::template_keyword<locking<>,T>;

struct no_locking : locking_marker {
  struct mutex_type{}; using lock_type = mutex_type;  // thus 'lock_type l(mutex_type())' has no effect
};

using recursive_lightweight_mutex = lightweight_mutex;  // from Boost.SmartPtr, when no <pthread.h>
struct recursive_lightweight_mutex { /*...*/ };         // on pthread_mutex, PTHREAD_MUTEX_RECURSIVE
struct simple_locking : locking_marker {
  using mutex_type = recursive_lightweight_mutex;
  using lock_type = mutex_type::scoped_lock;
};
```

* On posix, `simple_locking` wraps pthread's recursive mutex, otherwise use *Boost.SmartPtr*'s implementation.

#### Tracking Policy

```c++
struct tracking_marker;
concept bool is_tracking<T> = is_base_and_derived<tracking_marker, T>;
struct tracking<T> : parameter::template_keyword<tracking<>,T>;

struct no_tracking : tracking_marker { // don't change the Value and Handle types
  struct entry_type<Value,Key> -> Value;
  struct handle_type<Handle,TrackingHelper> -> Handle;
};

class refcounted_data<Value,Key> {
  Value x; atomic_count ref; long del_ref;  // value, ref-count, owner-count
public:
  // ctor, copy, assign
  operator const Value&() const; operator const Key&() const;
};
class refcounted_handle<Handle,TrackingHelper> {
  Handle h;
  static bool check_erase(const refcounted_handle& x) {
    return 0 == --TrackingHelper::entry(x).del_ref;   // allow erase if it is the last deleter
  }
public:
  explicit refcounted_handle(const Handle& h_);   // add a ref, if it is the first, add a deleter
  refcounted_handle(const refcounted_handle& x);  // add a ref
  ~refcounted_handle() {
    if (0 == --TrackingHelper::entry(*this).ref)  // last ref
      TrackingHelper::erase(*this, check_erase);  // erase if allowed (last deleter)
  }
  // assign, swap
};
struct refcounted : tracking_marker {
  using entry_type<Value,Key> -> refcounted_data<Value,Key>;
  using handle_type<Handle,TrackingHelper> -> refcounted_handle<Handle,TrackingHelper>;
};
```

* `no_tracking` will cause nothing being erased from a factory.

------
### Serialization Support For Flyweight

Header `<boost/flyweight/serialize.hpp>`

* Implemented `serialize`, `load`, and `save`.
* Utilize serialization load helper and save helper facility to omit redundant value construction.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Core

* `<boost/utility/swap.hpp>`
* `<boost/utility/enable_if.hpp>`
* `<boost/detail/templated_streams.hpp>`
* `<boost/detail/no_exceptions_support.hpp>` - serialization
* `<boost/noncopyable.hpp>` - serialization

#### Boost.Detail

* `<boost/detail/allocator_utilities.hpp>` - required by `set_factory`

#### Boost.TypeTraits

* `<boost/type_traits/is_same.hpp>`, `<boost/type_traits/is_base_and_derived.hpp>`
* `<boost/type_traits/is_convertible.hpp>`
* `<boost/type_traits/aligned_storage.hpp>`, `<boost/type_traits/alignment_of.hpp>` - serializtion, `key_value`

#### Boost.MPL

* `<boost/mpl/apply.hpp>`, `<boost/mpl/aux_/lambda_support.hpp>`, `<boost/mpl/aux_/na.hpp>`
* `<boost/mpl/not.hpp>`, `<boost/mpl/if.hpp>`, `<boost/mpl/or.hpp>`
* `<boost/mpl/assert.hpp>`

#### Boost.Functional/Hash

* `<boost/functional/hash_fwd.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/*.hpp>`

#### Boost.SmartPtr

* `<boost/detail/lightweight_mutex.hpp>` - on no POSIX platform
* `<boost/detail/atomic_count.hpp>`

#### Boost.MultiIndex

* `<boost/multi_index_container.hpp>`
* `<boost/multi_index/hashed_index.hpp>`, `<boost/multi_index/identity.hpp>`
* `<boost/multi_index/random_access_index.hpp>` - serialization

#### Boost.Parameter

* `<boost/parameter/parameters.hpp>`
* `<boost/parameter/parameter.hpp>`, `<boost/parameter/binding.hpp>`

#### Boost.Serialization

* `<boost/serialization/serialization.hpp>` - serialization
* `<boost/serialization/extended_type_info.hpp>` - serialization
* `<boost/serialization/nvp.hpp>`, `<boost/serialization/split_free.hpp>` - serialization

#### Boost.ThrowException

* `<boost/throw_exception.hpp>` - serialization

#### Boost.InterProcess

* `<boost/interprocess/detail/intermodule_singleton.hpp>` - for inter module factory holder

------
### Standard Facilities
