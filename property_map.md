# Boost.PropertyMap

* lib: `boost/libs/property_map`
* repo: `boostorg/property_map`
* commit: `b035da1`, 2025-07-03

------
### Property Map Concepts

Header `<boost/property_map/property_map.hpp>`

```c++
// traits for member types: key_type, value_type, reference, category
struct has_key_type<T>; struct has_value_type<T>; struct has_reference<T>; struct has_category<T>;

bool is_property_map<PA> = requires {
  typename PA::key_type; typename PA::value_type; typename PA::reference; typename PA::category;
};

struct property_traits<PMap> {
  using key_type = is_property_map<PMap> ? PMap::key_type; // also for `value_type`, `reference`, `category` types
};

// get/put support for lvalue property maps
struct put_get_helper<Ref,PMap> {}; // CRTP base class for lvalue property map classes
Ref get(put_get_helper<Ref,PMap> const& pa, Key const& k) { return static_cast<PMap const&>(pa)[k]; }
void put(put_get_helper<Ref,PMap> const& pa, Key k, Value const& v) { static_cast<PMap const&>(pa)[k] = v; }

// concept declarations
enum detail::ePropertyMapID { R, W, RW, LV, B, RAITER, LAST };
struct readable_property_map_tag { enum { id = R }; };
struct writable_property_map_tag { enum { id = W }; };
struct read_write_property_map_tag : readable_property_map_tag, writable_property_map_tag { enum { id = RW }; };
struct lvalue_property_map_tag : read_write_property_map_tag { enum { id = LV }; };

template <class PMap, class Key>
concept ReadablePropertyMap = is_property_map<property_traits<PMap>> && // traits defined member types for PMap
  Convertible<TR::category, readable_property_map_tag> &&
  requires(PMap pmap, Key k) { {get(pmap, key)} -> convertible_to<TR::value_type>; }; // can `get`

template <class PMap, class Key>
concept WritablePropertyMap = is_property_map<property_traits<PMap>> && // traits defined member types for PMap
  Convertible<TR::category, writable_property_map_tag> &&
  requires(PMap pmap, Key k, TR::value_type val) { {put(pmap, key, val)}; }; // can `put`

template <class PMap, class Key>
concept ReadWritePropertyMap = ReadablePropertyMap<PMap, Key> && WritablePropertyMap<PMap, Key> &&
  Convertible<TR::category, read_write_property_map_tag>;

template <class PMap, class Key>
concept LvaluePropertyMap = ReadablePropertyMap<PMap, Key> &&
  Convertible<TR::category, lvalue_property_map_tag> &&
  (same_as<TR::value_type&, TR::reference> || same_as<TR::value_type const&, TR::reference>)
  requires(PMap pmap, Key k) { {pmap[k]} -> convertible_to<TR::reference>; }; // can `put`

template <class PMap, class Key>
concept Mutable_LvaluePropertyMap = ReadWritePropertyMap<PMap, Key> &&
  Convertible<TR::category, lvalue_property_map_tag> && same_as<TR::value_type&, TR::reference>
  requires(PMap pmap, Key k) { {pmap[k]} -> convertible_to<TR::reference>; }; // can `put`
```

* Concept classes and archetype classes are provide for *Boost.ConceptCheck*.

------
### Property Map Wrapper

------
#### Wraps Property Map As Function Object

```c++
class property_map_function<PropMap> requires ReadablePropertyMap<PropMap> {
  PropMap pm; // stored map
public:
  using param_type = property_traits<PropMap>::key_type;
  using result_type = property_traits<IndexMap>::value_type;
  explicit property_map_function(PropMap const& pm) : pm(pm) {}
  result_type operator()(param_type const& k) const { return get(pm, k); }
}:
auto make_property_map_function(PropMap const&) -> property_map_function<PropMap>;
```

* Lvalue property map
* Use `PropMap` as adaptor to map key to index
* `safe_iterator_property_map` asserts index range when accessing.

------
#### Plug Property Map Into Iterator

Header `<boost/property_map/property_map_iterator.hpp>`

```c++
class detail::lvalue_pmap_iter<Iterator, LvaluePMap> : public iterator_adaptor<...> {
  LvaluePMap m_map;   // stored map
public:
  // ctor(), ctor(Iterator const&, LvaluePMap), initialize base iterator and map member
  reference dereference() const {return m_map[*base_reference()];}
}:

class detail::readable_pmap_iter<Iterator, ReadablePMap> : public iterator_adaptor<...> {
  ReadablePMap m_map;   // stored map
public:
  // ctor(), ctor(Iterator const&, LvaluePMap), initialize base iterator and map member
  reference dereference() const {return get(m_map, *base_reference());}
}:

struct property_map_iterator_generator<PMap, Iter>:
  (requires LValuePropertyMap<PMap>) ? lvalue_pmap_iter<Iter,PMap> : readable_pmap_iter<Iter,PMap>;

auto make_property_map_iterator(PMap, Iterator) -> property_map_iterator_generator<PMap, Iter>::type;
```

* Maps upon the result of iterator access.

------
### Predefined Property Maps

------
#### Builtin Pointer

```c++
struct property_traits<[const] T*> { // treate pointers as lvalue property map
  using value_type = T; using reference = [const]T&;
  using key_type = ptrdiff_t; using category = lvalue_property_map_tag;
};

const T& get<T>(const T* pa, ptrdiff_t k) { return pa[k]; }
void put<T,V>(T* pa, ptrdiff_t k, const V& val) { pa[k] = val; }
```

* Lvalue property map
* `k` -> `p[k]` -> `r`
* Provide `property_traits<T*>`, `key_type` is `ptrdiff_t`
* Property accessing by indexing.

------
#### Dummy Property Map

```c++
struct detail::dummy_pmap_reference {
  dummy_pmap_reference& operator= <T> (const T&) { return *this; }
  operator int() {return 0;}
};
struct dummy_property_map : public put_get_helper<dummy_pmap_reference, dummy_property_map> {
  using value_type = int; using key_type = void; using reference = dummy_pmap_reference;
  using category = read_write_property_map_tag;
  ctor(); ctor(int); ctor(dummy_property_map const&);
  reference operator[] <V> (V) const {return {};}
};
```

* Read and write property map, do nothing, value type is `int`, always return `0`.

------
#### Identity Property Map

```c++
struct typed_identity_property_map<T> : put_get_helper<T, typed_identity_property_map> {
  using key_type = T; using value_type = T; using reference = T;
  using category = readable_property_map_tag;
  value_type operator[](KeyType const& v) const { return v; } // get the key itself
};
using identity_property_map = typed_identity_property_map<std::size_t>;
```

* Readonly property map, return a copy of the key.
* `k` -> `r`

------
#### Static Property Map

```c++
class static_property_map<ValueType, KeyType=void> : public put_get_helper<ValueType,static_property_map> {
  ValueType value;    // value copy
public:
  using key_type = KeyType; using value_type = ValueType; using reference = ValueType;
  using category = readable_property_map_tag;
  static_property_map(ValueType v) : value(v) {}  // store a copy
  ValueType operator[] <T>(T) const { return value; } // always return by value
};
auto make_static_property_map<K,V>(V const&) -> static_property_map<V,K>;
```

* Readonly property map, always return a copy of the specified value.

------
#### Reference Property Map

```c++
class ref_property_map<KeyType, ValueType> : public put_get_helper<ValueType&,ref_property_map> {
  ValueType* value;   // value pointer
public:
  using key_type = KeyType; using value_type = ValueType; using reference = ValueType&;
  using category = lvalue_property_map_tag;
  ref_property_map(ValueType& v) : value(&v) {} // store a reference
  ValueType& operator[](KeyType const&) const { return *value; } // always return reference to specified value
};
```

* Lvalue property map, access a reference to the origin value.

------
#### Iterator Property Map

```c++
template<RandomAccessIterator RAIter, ReadablePropertyMap IndexMap,
class T=iterator_traits<RAIter>::value_type, class R=iterator_traits<RAIter>::reference>>
class iterator_property_map : public put_get_helper<R,iterator_property_map> { // for get/put
  RAIter iter, IndexMap index;
public:
  using key_type = property_traits<IndexMap>::key_type;
  using value_type = T; using reference = R; using category = lvalue_property_map_tag;
  iterator_property_map(RAIter, IndexMap const& = IndexMap()); //ctor
  R operator[](key_type) const { return *(iter + get(index, v)); }
}:
template<RandomAccessIterator RAIter, ReadablePropertyMap IndexMap,
class T=iterator_traits<RAIter>::value_type, class R=iterator_traits<RAIter>::reference>>
class safe_iterator_property_map : public put_get_helper<R,iterator_property_map> { // similar to iterator_property_map
  property_traits<IndexMap>::value_type n; // and also iter, index
public: // types
  safe_iterator_property_map(RAIter, std::size_t n = 0, IndexMap const& = IndexMap()); // ctor
  R operator[](key_type) const { assert(get(index, v) < n); return *(iter + get(index, v)); } // asserts for index out of range
}:
auto make_iterator_property_map(RAIter, IndexMap) -> iterator_property_map<RAIter, IndexMap>;
auto make_iterator_property_map(RAIter, IndexMap, V) -> iterator_property_map<RAIter, IndexMap, V, V&>;
auto make_safe_iterator_property_map(RAIter, std::size_t, IndexMap) -> safe_iterator_property_map<RAIter, IndexMap>;
auto make_safe_iterator_property_map(RAIter, std::size_t, IndexMap, V) -> safe_iterator_property_map<RAIter, IndexMap, V, V&>;
```

* Lvalue property map
* `k` -> `IndexMap` -> `iter[x]` -> `r`
* Use `IndexMap` as adaptor to map key to index, can use `identity_property_map` for no-mapping.
* `safe_iterator_property_map` asserts index range when accessing.

------
#### Associative Property Map

```c++
template<UniquePairAssociativeContainer C>
class associative_property_map<C> : public put_get_helper<value_type&, associative_property_map> {
  C* m_c; // reference to container
public:
  using key_type = C::key_type; using value_type = C::value_type::second_type; using reference = value_type&;
  using category = lvalue_property_map_tag;
  ctor(); ctor(C&); // ctor with reference
  reference operator[](key_type const & k) const { return (*m_c)[k]; }
}:
template<UniquePairAssociativeContainer C>
class const_associative_property_map<C> : public put_get_helper<value_type const&, associative_property_map> {
  C const* m_c; // reference to container
public:
  using key_type = C::key_type; using value_type = C::value_type::second_type; using reference = value_type const&;
  using category = lvalue_property_map_tag;
  ctor(); ctor(C const&); // ctor with reference
  reference operator[](key_type const & k) const { return m_c->find(k)->second; }
}:
auto make_assoc_property_map([const] C&) -> [const_]associative_property_map<C>;
```

* Lvalue property map
* `k` -> `container` -> `r`
* The associative container is only referenced by property map.

------
#### Vector Based Property Map

Header `<boost/property_map/vector_property_map.hpp>`

```c++
class vector_property_map<T, IndexMap=identity_property_map>
  : public put_get_helper<<std::vector>::reference, vector_property_map> {
  shared_ptr<std::vector<T>> store; // shared storage
  IndexMap index; // key mapper
public:
  using key_type = property_traits<IndexMap>::key_type;
  using value_type = T; using reference = std::vector<T>::reference;
  using category = lvalue_property_map_tag;

  vector_property_map([unsigned initial_size,]IndexMap const& = IndexMap()); // ctor, new storage

  storage_begin() [const]; storage_end() [const]; // access the underlying storage
  IndexMap [const]& get_index_map() [const];      // access the index key mapper

  reference operator[](key_type const&) const;    // use 'index' to get index from key. auto resize vector for large index
}:
auto make_iterator_property_map<T,IndexMap>(IndexMap) -> vector_property_map<T, IndexMap>;
```

* Lvalue property map
* `k` -> `IndexMap` -> `vector` -> `r`
* The property map owns the storage, and shared between copies of the map.
* dynamic sized.

------
#### Shared Array Based Property Map

Header `<boost/property_map/shared_array_property_map.hpp>`

```c++
class shared_array_property_map<T, IndexMap> : put_get_helper<T&, shared_array_property_map> {
  boost::shared_array<T> data; // shared storage
  IndexMap index; // key mapper
public:
  using key_type = property_traits<IndexMap>::key_type;
  using value_type = T; using reference = T&;
  using category = lvalue_property_map_tag;

  shared_array_property_map(size_t n, IndexMap const& = IndexMap()); // ctor, new array with size 'n'
  T& operator[](key_type) const;    // use 'index' to get index from key
}:
auto make_shared_array_property_map<T, IndexMap>(size_t n, const T&, const IndexMap&) -> shared_array_property_map<T, IndexMap>;
```

* Lvalue property map
* `k` -> `IndexMap` -> `shared_array[n]` -> `r`
* Fixed size (decided on ctor)

------
#### Function Property Map

Header `<boost/property_map/function_property_map.hpp>`

```c++
class function_property_map<Func, Key, Ret=result_of_t<const Func(const Key&)>
  : public put_get_helper<Ret, function_property_map> {
  Func f;   // store function
public:
  using value_type = remove_cv_t<remove_reference_t<Ret>>;
  using key_type = Key; using reference = Ret;
  using category = (is_reference_v<Ret> && !is_const_v<Ret>) ? lvalue_property_map_tag : readable_property_map_tag;

  function_property_map(Func f=Func()) : f(f){}
  Ret operator[](const Key& k) const { return f(k); }
}:
auto make_function_property_map<Func,Key[,Ret]>(Func const&) -> function_property_map<Func,Key[,Ret]>;
```

* Wraps a function as a property map.
* `k` -> `Func` -> `r`
* If `Ret` is non-const reference, it is lvalue property map, otherwise a readable property map.

------
#### Transform Value Property Map

Header `<boost/property_map/transform_value_property_map.hpp>`

```c++
class transform_value_property_map<Func, PM, Ret=result_of_t<const Func(property_traits<PM>::reference)>
  : public put_get_helper<Ret, transform_value_property_map> {
  Func f; PM pm;
public:
  using value_type = remove_cv_t<remove_reference_t<Ret>>;
  using key_type = property_traits<PM>::key_type; using reference = Ret;
  using category = (is_reference_v<Ret> && !is_const_v<Ret>) ? lvalue_property_map_tag : readable_property_map_tag;

  transform_value_property_map(Func f, PM pm) : f(f), pm(pm){}
  Ret operator[](const Key& k) const { return f(get(pm, k)); }
}:
auto make_transform_value_property_map<[Ret,]PM,Func>(Func const&, PM const&)
    -> transform_value_property_map<Func,Key[,Ret]>;
```

* Wraps a function to convert value from a property map.
* `k` -> `PM` -> `Func` -> `r`
* If `Ret` is non-const reference, it is lvalue property map, otherwise a readable property map.

------
#### Compose Property Map

Header `<boost/property_map/compose_property_map.hpp>`

```c++
class compose_property_map<FPMap, GPMap> {
  FPMap f; GPMap g;
public:
  using cateogory = property_traits<FPMap>::category
  // 'key_type' from 'GPMap', other types from 'FPMap'

  compose_property_map(FPMap const& f, GPMap const& g) : f(f), g(g){}
  reference operator[](const key_type& k) const { return f[get(g, k)]; } // use 'f[]' for reference
}:
auto get(compose_property_map const& m, key_type const& k) { return get(m.f, get(m.g, k)); }
void put(compose_property_map const& m, key_type const& k value_type const& v) { return put(m.f, get(m.g, k), v); }
auto make_compose_property_map<FPMap,GPMap>(FPMap const&, GPMap const&) -> compose_property_map<FPMap,GPMap>;
```

* `k` -> `GPMap` -> `FPMap` -> `r`
* Compose two maps like `f(g(k))`

------
### Dynamic Property Maps

Header `<boost/property_map/dynaimc_property_map.hpp>`

```c++
// polymorphic base, interface
class dynamic_property_map {
public:
  virtual ~dynamic_property_map() {}
  virtual any get(any const&) = 0;
  virtual string get_string(any const&) = 0;
  virtual void put(any const&, any const&) = 0;
  virtual type_info const& key() const = 0;
  virtual type_info const& value() const = 0;
};

// exceptions
struct dynamic_property_exception : std::exception {};
struct property_not_found : dynamic_property_exception {};
struct dynamic_get_failure : dynamic_property_exception {};
struct dynamic_const_put_error : dynamic_property_exception {};

class detail::dynamic_property_map_adaptor<PMap> : public dynamic_property_map {
  PMap pmap;
public:
  using key_type = property_traits<PMap>::key_type; // and value_type, category
  dynamic_property_map_adaptor(PMap const&);  // ctor
  // get(), get_string(), get value or streamed to string
  // put(), throw if not writable, otherwise accept value_type or string
  // key(), value(), return 'typeid()' for types
  [const] PMap& base() [const]; // access
};

struct dynamic_properties {
  using property_maps_type = std::multimap<string, shared_ptr<dynamic_property_map>>;
  using generate_fn_type = function<shared_ptr<dynamic_property_map>(string const&, any const&, any const&)>;
  property_maps_type property_maps;
  generate_fn_type generate_fn;
  
  // ctor(generate_fn), default ctor, dtor
  // begin(), end(), lower_bound(), insert() forwards to 'property_maps'
  dynamic_properties[&] property<PMap>(string const& name, PMap pmap) [const]; // const version copies
  shared_ptr<dynamic_property_map> generate<Key, Value>(string const& name, Key const&, Value const&);
};

bool put<Key,Value>(string const& name, dynamic_properties&, Key const&, Value const&);
Value get<Value,Key>(string const& name, dynamic_properties const&, Key const&[,type<Value>]);
string get<Key>(string const& name, dynamic_properties const&, Key const&); // call 'get_string()'

shared_ptr<dynamic_property_map> ignore_other_properties(string const&, any const&, any const&);
```

* `dynamic_properties` stores multiple property maps for each name
* `generate` creates map via function object but not store, `property` wraps map as adaptor and store
* `get` throws if not found, `put` create new map by `generate` if not found

------
### Dependency

#### Boost.Any

* `<boost/any.hpp>` - by `dynamic_property_map`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>`, `<boost/concept_archetype.hpp>`, `<boost/concept/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Core

x* `<boost/detail/iterator.hpp>` - deprecated
* `<boost/type.hpp>` - by `dynamic_property_map`

#### Boost.Function

* `<boost/function/function3.hpp>` - by `dynamic_property_map`

#### Boost.Iterator

* `<boost/iterator/iterator_adaptor.hpp>` - by iterator generator

#### Boost.LexicalCast

* `<boost/lexical_cast.hpp>` - by `dynamic_property_map`

#### Boost.MPL

* `<boost/mpl/and.hpp>`, `<boost/mpl/not.hpp>`, `<boost/mpl/or.hpp>`, `<boost/mpl/if.hpp>`
* `<boost/mpl/has_xxx.hpp>`, `<boost/mpl/assert.hpp>`, `<boost/mpl/bool.hpp>`

#### Boost.SmartPtr

* `<boost/smart_ptr/shared_array.hpp>` - by `shared_array_property_map`
* `<boost/smart_ptr/shared_ptr.hpp>` - by `vector_property_map`
* `<boost/smart_ptr.hpp>` - by `dynamic_property_map`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>` - by `dynamic_property_map`

#### Boost.TypeIndex

* `<boost/type_index.hpp>` - by `dynamic_property_map`

#### Boost.TypeTraits

* `<boost/type_traits.hpp>`, `<boost/type_traits/is_same.hpp>`, `<boost/type_traits/is_convertible.hpp>`

#### Boost.Utility

* `<boost/utility/result_of.hpp>` - by `function_property_map` & `transform_value_property_map`

------
### Standard Facilities
