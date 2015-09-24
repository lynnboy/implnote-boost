# Boost.PropertyMap

* lib: `boost/libs/property_map`
* repo: `boostorg/property_map`
* commit: `ba0bed2e`, 2014-09-01

------
### Property Map Concepts

Header `<boost/property_map/property_map.hpp>`

```c++
struct readable_property_map_tag {};
struct writable_property_map_tag {};
struct read_write_property_map_tag : readable_property_map_tag, writable_property_map_tag {};
struct lvalue_property_map_tag : read_write_property_map_tag {};

bool is_property_map<PA> = requires {
  typename PA::key_type; typename PA::value_type; typename PA::reference; typename PA::category;
};

struct property_traits<PMap> {
  // if is_property_map<PMap> then define key_type, value_type, reference, category types
}

template <class PMap>
concept bool ReadablePropertyMap = CopyConstructible<PMap> && requires {
    typename boost::property_traits<PMap>::value_type;
    typename boost::property_traits<PMap>::reference;
    typename boost::property_traits<PMap>::key_type;
    requires (PMap pmap, boost::property_traits<PMap>::key_type key) {
      {get(pmap, key)} -> boost::property_traits<PMap>::reference;
    }
  };

template <class PMap>
concept bool WritablePropertyMap = CopyConstructible<PMap> && requires {
    typename boost::property_traits<PMap>::value_type;
    typename boost::property_traits<PMap>::key_type;
    requires { Convertible<typename boost::property_traits<PMap>::category, writable_property_map_tag> }
    requires (PMap pmap, key_type key, value_type val) {
      put(pmap, key, val);
    }
  };

template <class PMap>
concept bool ReadWritePropertyMap = ReadablePropertyMap<PMap> && WritablePropertyMap<PMap> &&
  Convertible<typename boost::property_traits<PMap>::category, read_write_property_map_tag>;

template <class PMap>
concept bool LvaluePropertyMap = ReadablePropertyMap<PMap> && requires {
    requires { Convertible<typename boost::property_traits<PMap>::category, lvalue_property_map_tag> }
    requires (PMap pmap, key_type key) {
      { pmap[k] } -> value_type const&;
    }
  };

template <class PMap>
concept bool Mutable_LvaluePropertyMap = ReadWritePropertyMap<PMap> && requires {
    requires { Convertible<typename boost::property_traits<PMap>::category, lvalue_property_map_tag> }
    requires (PMap pmap, key_type key) {
      { pmap[k] } -> value_type &;
    }
  };
```

* Concept classes and archetype classes are provide for *Boost.ConceptCheck*.

------
### Property Map Wrapper

```c++
struct property_map_function<PropMap> requires ReadablePropertyMap<PropMap> {
  PropMap pm; // stored map

  using param_type = property_traits<PropMap>::key_type;
  using result_type = property_traits<IndexMap>::value_type;

  property_map_function(PropMap const&); //ctor, init 'pm'
  result_type operator()(param_type const& k) const; // call 'get(pm, k)'
}:
auto make_property_map_function(PropMap const&) -> property_map_function<PropMap>;
```

* Lvalue property map
* Use `IndexMap` as adaptor to map key to index
* `safe_iterator_property_map` asserts index range when accessing.

------
### Predefined Property Maps

------
#### Builtin Pointer

* Lvalue property map
* Provide `property_traits<T*>`, `key_type` is `ptrdiff_t`
* Property accessing by indexing.

------
#### Dummy Property Map

```c++
class dummy_property_map;
```

* Read and write property map, do nothing, value type is `int`, always return `0`.

------
#### Identity Property Map

```c++
class typed_identity_property_map<T> {
  ValueType& operator[](KeyType const&) const { return *value; } // get reference
};
using identity_property_map = typed_identity_property_map<std::size_t>;
```

* Readonly property map, return a copy of the key.

------
#### Static Property Map

```c++
class static_property_map<ValueType, KeyType=void> {
  ValueType value;    // value copy
public:
  static_property_map(ValueType v) : value(v) {}  // store a copy
  ValueType operator[](T) const { return value; } // always return by value
};
auto make_static_property_map(ValueType const&) -> static_property_map<ValueType>;
```

* Readonly property map, always return a value copy.

------
#### Reference Property Map

```c++
class ref_property_map<KeyType, ValueType> {
  ValueType* value;   // value pointer
public:
  static_property_map(ValueType v) : value(&v) {} // store a reference
  ValueType& operator[](KeyType const&) const { return *value; } // get reference
};
```

* Lvalue property map, access a reference to the origin value.

------
#### Iterator Property Map

```c++
class iterator_property_map<RAIter, IndexMap,
  T=iterator_traits<RAIter>::value_type, R=iterator_traits<RAIter>::reference>
  requires ReadPropertyMap<IndexMap> {
  RAIter iter, IndexMap index;
public:
  using key_type = property_traits<IndexMap>::key_type; // and value_type, reference, category
  iterator_property_map(RAIter, IndexMap const& = IndexMap()); //ctor
  R operator[](key_type) const; // use index to get index from key
}:
class safe_iterator_property_map<RAIter, IndexMap, T, R> { // similar to iterator_property_map
  property_traits<IndexMap>::value_type n_;
public:
  safe_iterator_property_map(RAIter, std::size_t n = 0, IndexMap const& = IndexMap()); // ctor
  R operator[](key_type) const; // asserts for index out of range
}:
auto make_iterator_property_map(RAIter, IndexMap) -> iterator_property_map<RAIter, IndexMap>;
auto make_safe_iterator_property_map(RAIter, std::size_t, IndexMap)
    -> safe_iterator_property_map<RAIter, IndexMap>;
```

* Lvalue property map
* Use `IndexMap` as adaptor to map key to index
* `safe_iterator_property_map` asserts index range when accessing.

------
#### Associative Property Map

```c++
class [const_]associative_property_map<C>
  requires UniquePairAssociativeContainer<C> {
  C* c; // reference to container
public:
  UniquePairAssociativeContainer(C&); // ctor with reference
  [const] C::value_type& operator[](C::key_type const &) const; // fwd to c
}:
auto make_assoc_property_map([const] C&) -> [const_]associative_property_map<C>;
```

* Lvalue property map
* The associative container is only referenced by property map.

------
#### Vector Based Property Map

Header `<boost/property_map/vector_property_map.hpp>`

```c++
class vector_property_map<T, IndexMap=identity_property_map> requires ReadPropertyMap<IndexMap> {
  shared_ptr<std::vector<T>> store; // shared storage
  IndexMap index; // key mapper
public:
  using key_type = property_traits<IndexMap>::key_type; // and value_type, reference, category

  vector_property_map([unsigned initial_size,]IndexMap const& = IndexMap()); // ctor, new storage

  storage_begin() [const]; storage_end() [const]; // access the underlying storage
  IndexMap [const]& get_index_map() [const];      // access the index key mapper

  reference operator[](key_type const&) const;    // use 'index' to get index from key
}:
auto make_iterator_property_map<T>(IndexMap) -> vector_property_map<T, IndexMap>;
```

* Lvalue property map
* The property map owns the storage, and shared between copies of the map.

------
#### Shared Array Based Property Map

Header `<boost/property_map/shared_array_property_map.hpp>`

```c++
class shared_array_property_map<T, IndexMap> requires ReadPropertyMap<IndexMap> {
  boost::shared_array<T> data; // shared storage
  IndexMap index; // key mapper
public:
  using key_type = property_traits<IndexMap>::key_type; // and value_type, reference, category

  shared_array_property_map(size_t n, IndexMap const& = IndexMap()); // ctor, new array with size 'n'
  T& operator[](key_type) const;    // use 'index' to get index from key
}:
auto make_iterator_property_map<T>(IndexMap) -> vector_property_map<T, IndexMap>;
```

* Lvalue property map

------
#### Function Property Map

Header `<boost/property_map/function_property_map.hpp>`

```c++
class function_property_map<Func, Key, Ret=result_of<const Func(const Key&)> {
  Func f;   // store function
public:
  using value_type = remove_cv_t<remove_reference_t<Ret>>;

  function_property_map(Func f=Func()) : f(f){}
  Ret operator[](const Key& k) const { return f(k); }
}:
auto make_function_property_map<Key[,Ret]>(Func const&) -> function_property_map<Func,Key,Ret>;
```

* Wraps a function as a property map.
* If `Ret` is non-const reference, it is lvalue property map, otherwise a readable property map.

------
#### Transform Value Property Map

Header `<boost/property_map/transform_value_property_map.hpp>`

```c++
class transform_value_property_map<Func, PM,
      Ret=result_of<const Func(property_traits<PM>::reference)> {
  Func f; PM pm;
public:
  using value_type = remove_cv_t<remove_reference_t<Ret>>;

  transform_value_property_map(Func f, PM pm) : f(f), pm(pm){}
  Ret operator[](const Key& k) const { return f(get(pm, k)); }
}:
auto make_transform_value_property_map<[Ret]>(Func const&, PM const&)
    -> transform_value_property_map<Func,Key,Ret>;
```

* Wraps a function to convert value from a property map.
* If `Ret` is non-const reference, it is lvalue property map, otherwise a readable property map.

------
#### Compose Property Map

Header `<boost/property_map/compose_property_map.hpp>`

```c++
class compose_property_map<FPMap, GPMap> {
  FPMap f; GPMap g;
public:
  // 'key_type' from 'GPMap', other types from 'FPMap'

  compose_property_map(FPMap const& f, GPMap const& g) : f(f), g(g){}
  reference operator[](const key_type& k) const { return f[get(g, k)]; } // use 'f[]' for reference
}:
auto make_compose_property_map(FPMap const&, GPMap const&) -> compose_property_map<FPMap,GPMap>;
```

* Compose two maps like `f(g(k))`

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
