# Boost.Functional/Factory

* lib: `boost/libs/functional/factory`
* repo: `boostorg/functional`
* commit: `3c468dd`, 2025-06-26

------
### Factory Wrappers

Header `<boost/functional/factory.hpp>` and `<boost/functional/value_factory.hpp>`

```c++
enum factory_alloc_propagation {
  factory_alloc_for_pointee_and_deleter,  factory_passes_alloc_to_smart_pointer
};

template<class R, class A> class detail::fc_alocate { // RAII wrapper for allocate storage
  A a_; allocator_traits<A>::pointer p_; // data members
  pointer release() { pointer p = p_; p_ = pointer(); return p; } // return and set to nullptr
public:
  fc_allocate(const A& a) : a_(a), p_(a_.allocate(1)) {} // allocate on ctor
  ~fc_allocate() { if (p_) a_.deallocate(p_, 1); } // deallocate on dtor
  A& state() { return a_; } // get allocator
  A::value_type* get() { return to_address(p_); } // get storage
  R release(fc_tag<factory_alloc_for_pointee_and_deleter>) {
    return R(release(p_), fc_delete<A>(a_), a_);
  }
  R release(fc_tag<factory_passes_alloc_to_smart_pointer>) {
    return R(release(p_), fc_delete<A>(a_));
  }
};

template<typename Pointer, typename Allocator=void,
  factory_alloc_propagation AP=factory_alloc_for_pointee_and_deleter>
class factory : allocator { // when not 'void', maybe EBO
public:
  using result_type = remove_cv<Pointer>::type;
  using type = pointer_traits<result_type>::element_type;
  using allocator = std::allocator_traits<Allocator>::rebind_alloc<type>; // when not 'void'
  using AC=std::allocator_traits<allocator>;
  
  factory(Allocator const& a = Allocator());
  template<typename ...Args> result_type operator()(Args &... args) const {
    if constexpr (is_same_v<Allocator, void>) { // for 'Allocator' is 'void'
      return result_type(new type(std::forward<Args>(args)...));
    } else {
      detai::fc_allocate<result_type, allocator> s(base::get());
      AC::construct(s.state(), s.get(), std::forward<Args>(args)...); // construct
      return s.release(detail::fc_tag<Policy>());
    }
  }
};

template<typename T>
class value_factory {
public:
  using result_type = T;
  template<typename ...Args> result_type operator()(Args&&... args) const
  { return result_type(std::forward<Args>(args)...); }
};
```

* Factories wrapps object construction expressions, which are not `bind`able.
* `factory` supports `T*`, as well as smart pointer types.
* `factory` used with `Allocator`, can be used with smart pointers like `shared_ptr`, which accepting
  a deleter and an allocator.
* If the smart pointer doesn't accept allocator, use `factory_passes_alloc_to_smart_pointer`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Core

* `<boost/core/empty_value.hpp>`
* `<boost/core/pointer_traits.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/remove_cv.hpp>`

------
### Standard Facilities

Language: lambda expression (C++11)
