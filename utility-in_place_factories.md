# Boost.Utility/InPlaceFactories

* lib: `boost/libs/utility`
* repo: `boostorg/utility`
* commit: `dda83c5`, 2025-08-26

------
### `in_place_factory`

* Header `<boost/utility/in_place_factory.hpp>`

```c++
struct in_place_factory_base{};
struct in_place_factoryN<A1...An> : in_place_factory_base {
    A1 const& m_a1; ... An const& m_an;
    explicit ctor(A1 const& a1, ..., An const& an) ;
    void* apply<T>(void* address) const { return new(address) T{m_a1, ..., m_an}; }
    void* apply<T>(void* address, size_t n) const // array version
    { for (char* next=address=apply<T>(address); !!--n;) apply<T>(next=next+sizeof(T)); return address; }
};
inline in_place_factoryN<A1,...,An> in_place<A1,...,An>(A1 const& a1,..., An const& an) { return {a1,...,an}; }
```

------
### `typed_in_place_factory`

* Header `<boost/utility/typed_in_place_factory.hpp>`

```c++
struct typed_in_place_factory_base{};
struct typed_in_place_factoryN<T,A1...An> : typed_in_place_factory_base {
    using value_type = T;
    A1 const& m_a1; ... An const& m_an;
    explicit ctor(A1 const& a1, ..., An const& an) ;
    void* apply(void* address) const { return new(address) T{m_a1, ..., m_an}; }
    void* apply(void* address, size_t n) const // array version
    { for (char* next=address=apply(address); !!--n;) apply(next=next+sizeof(T)); return address; }
};
inline typed_in_place_factoryN<T,A1,...,An> in_place<T,A1,...,An>(A1 const& a1,..., An const& an) { return {a1,...,an}; }
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

------
### Standard Facilities
