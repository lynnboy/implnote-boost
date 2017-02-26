# Boost.Functional

* lib: `boost/libs/functional`
* repo: `boostorg/functional`
* commit: `1706cb59`, 2017-02-03

------
### Function Object Wrappers

Header `<boost/functional.hpp>`

* `unary_traits`, `binary_traits`
* `unary_negate`, `binary_negate`, `not1`, `not2`
* `binder1st`, `bindder2nd`, `bind1st`, `bind2nd` - deprecated by `bind`
* `pointer_to_unary_function`, `pointer_to_binary_function`, `ptr_fun` - deprecated by `ref` and `bind`
* `[const_]mem_fun[1]_t`, `[const_]mem_fun[1]_ref_t`, `mem_fun`, `mem_fun_ref` - deprecated by `mem_fn` and `bind`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Utility

* `<boost/call_traits.hpp>`

------
### Standard Facilities

Standard Library: `<functional>` (C++03)
