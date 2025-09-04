# Boost.Core/Swap

* lib: `boost/libs/core`
* repo: `boostorg/core`
* commit: `c6b098d`, 2025-08-26

------
### Swap

Header `<boost/core/invoke_swap.hpp>`

Header `<boost/core/swap.hpp>` is deprecated.

```c++
template<class T> void invoke_swap(T& left, T& right) noexcept(...);
```

Add support for array types, call `std::swap` for each element.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/header_deprecated.hpp>`

------
### Standard Facilities

Standard Library: `swap` for array (C++11)
