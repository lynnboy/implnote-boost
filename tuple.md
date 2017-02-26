# Boost.Tuple

* lib: `boost/libs/tuple`
* repo: `boostorg/tuple`
* commit: `895af797`, 2016-10-07

------
### Tuple library

#### Header

* `<boost/tuple/tuple.hpp>`
* `<boost/tuple/tuple_comparison.hpp>`
* `<boost/tuple/tuple_io.hpp>`

#### API for `tuple`

* `struct null_type` denote empty tuple.
* `class tuple<T0=null_type, ... T9=null_type>`.
* `get<int N, class HT, class TT>(cons<HT, TT>& c)` and const version.
* `struct length<Tuple> { int value; }`
* `make_tuple<T0, ...>(const T0&, ...)`
* `tie<T0, ...>(T0&, ...)`, supports `ignore`.

##### Comparison
* Operators `==`, `!=`, `<`, `<=`, `>`, `>=`

##### I/O Support
* Custom manipulators `set_open(CharT)`, `set_close(CharT)`, and `set_delimiter(CharT)`.
* Operators `<<` and `>>`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/cv_traits.hpp>`
* `<boost/type_traits/function_traits.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Core

* `<boost/utility/swap.hpp>`.
* `<boost/ref.hpp>`.

------
### Standard Facilities

Standard Library: `<tuple>` (C++11)
