# Boost.Core/EnableIf

* lib: `boost/libs/core`
* repo: `boostorg/core`
* commit: `1dff5587`, 2017-02-21

------
### `enable_if` family

Header `<boost/core/enable_if.hpp>`

* `enable_if_c<bool, T=void>`
* `enable_if<Cond, T=void>` - test `Cond::value`
* `lazy_enable_if<Cond, T>` - when `Cond::value`, use (instantiate) `T::type`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

------
### Standard Facilities

Standard Library: `enable_if<bool, T=void>` (C++11), `enable_if_t` (C++14)
