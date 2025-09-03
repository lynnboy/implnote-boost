# Boost.Core/Swap

* lib: `boost/libs/core`
* repo: `boostorg/core`
* commit: `3f36d50`, 2024-05-29

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

* `<boost/config.hpp>`

------
### Standard Facilities

Standard Library: `swap` for array (C++11)
