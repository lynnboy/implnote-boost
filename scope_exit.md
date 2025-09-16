# Boost.ScopeExit

* lib: `boost/libs/scope_exit`
* repo: `boostorg/scope_exit`
* commit: `1c467753`, 2016-10-07

------
### Boost Scope Exit

Header `<boost/scope_exit.hpp>`

#### Macros

* `BOOST_SCOPE_EXIT(capture_list)`, `BOOST_SCOPE_EXIT_ID(id, capture_list)`
* `BOOST_SCOPE_EXIT_TPL(capture_list)`, `BOOST_SCOPE_EXIT_ID_TPL(id, capture_list)`
  Used within template code, to workaround some GCC version.
* `BOOST_SCOPE_EXIT_ALL(capture_list)`, `BOOST_SCOPE_EXIT_ALL_ID(id, capture_list)`
  Only available for C++11 mode
* `BOOST_SCOPE_EXIT_END`, `BOOST_SCOPE_EXIT_END_ID(id)`
  On C++11 mode, can be simply `;`

The `capture_list` can be `void` or comma-separated `capture` list or sequence of `(capture)`.
Each `capture` can be `var`, `&var`, or `this_` (for capturing `this`).
For `BOOST_SCOPE_EXIT_ALL`, a leading `&` or `=` capture should be the first one.
The `_ID` version is needed when multiple macros used at the same code line.

#### Switch

* `BOOST_SCOPE_EXIT_CONFIG_USE_LAMBDAS` to force use C++ 11 lambda implementation.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Function

* `<boost/function.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/*.hpp>`

#### Boost.TypeOf

* `<boost/typeof/typeof.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/enable_if.hpp>`, `<boost/type_traits/integral_constant.hpp>`

------
### Standard Facilities

Language: lambda expression (C++11)
Proposals:
  N4189, P0052 - Generic Scope Guard and RAII Wrapper for the Standard Library
