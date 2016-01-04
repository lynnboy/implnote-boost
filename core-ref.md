# Boost.Core/Ref

* lib: `boost/libs/core`
* repo: `boostorg/core`
* commit: `6345b931`, 2016-01-01

------
### Reference Wrapping

Header `<boost/core/ref.hpp>`

* Class `reference_wrapper<T>`
  * `T` as member `type`
  * Ctor for `T&`, no ctor for `T&&`
  * accessors `T& get() const`, `T* get_pointer() const`, and `operator T&() const`
* `ref(T&) -> reference_wrapper<T> const`
* `cref(T const&) -> reference_wrapper<const T> const`
* Trait `is_reference_wrapper<T>`, `unwrap_reference<T>`
* `unwrap_ref(T&) -> unwrap_reference<T>`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Core

* `<boost/utility/addressof.hpp>`

------
### Standard Facilities

Standard Library: `ref`, `cref` and `reference_wrapper` in `<functional>` (C++11)
