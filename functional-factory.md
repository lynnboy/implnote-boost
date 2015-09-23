# Boost.Functional/Factory

* lib: `boost/libs/functional/factory`
* repo: `boostorg/functional`
* commit: `a4781864`, 2015-01-10

------
### Factory Wrappers

Header `<boost/functional/factory.hpp>`
Header `<boost/functional/value_factory.hpp>`

```c++
enum factory_alloc_propagation {
  factory_alloc_for_pointee_and_deleter,  factory_passes_alloc_to_smart_pointer
};

template<typename Pointer, typename Allocator=void,
  factory_alloc_propagation AP=factory_alloc_for_pointee_and_deleter>
class factory : allocator_type { // when not 'void'
public:
  using result_type = remove_cv<Pointer>::type;
  using value_type = boost::pointee<result_type>::type;
  using allocator_type = Allocator::rebind<value_type>::other; // when not 'void'
  
  factory(Allocator const& a = Allocator());
  template<typename ...T> result_type operator()(T &... a) const {
    return result_type(new value_type(a...)); // for 'Allocator' is 'void'
    value_type* ptr = new (allocate(1)) value_type(a...);
    return result_type(ptr,
      /*deleter*/[](value_type* ptr){ ptr && ptr->~value_type(); this->deallocate(ptr, 1); },
      /*allocator*/ *this); // when AP == factory_alloc_for_pointee_and_deleter
  }
};

template<typename T>
class value_factory {
public:
  using result_type = T;
  value_factory();
  template<typename ...T> result_type operator()(T &... a) const { return T(a...); }
};
```

* Factories wrapps object construction expressions, which are not `bind`able.
* `factory` supports `T*`, as well as smart pointer types.
* `factory` used with `Allocator`, can be used with smart pointers like `smart_ptr`, which accepting
  a deleter and an allocator.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Core

* `<boost/get_pointer.hpp>`

#### Boost.Iterator

* `<boost/pointee.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/remove_cv.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/iteration/iterate.hpp>`
* `<boost/preprocessor/repetition/enum_params.hpp>`, `<boost/preprocessor/repetition/enum_binary_params.hpp>`

------
### Standard Facilities

Language: lambda expression (C++11)
