# Boost.Utility/ResultOf

* lib: `boost/libs/utility`
* repo: `boostorg/utility`
* commit: `dda83c5`, 2025-08-26

------
### Result Of

```c++
struct tr1_result_of<F>;
struct result_of<F,...Args> : tr1_result_of<F(Args...)> {};
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Core

* `<boost/core/enable_if.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

------
### Standard Facilities
