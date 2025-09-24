# Boost.TypeOf

* lib: `boost/libs/typeof`
* repo: `boostorg/typeof`
* commit: `26e22b8`, 2025-06-09

------
#### Header

* `<boost/typeof/typeof.hpp>`

#### Macros

* `BOOST_TYPEOF(expr)`, `BOOST_TYPEOF_TPL(expr)`: `remove_cv_ref_t<decltype(expr)>`
* `BOOST_AUTO(var, expr)`, `BOOST_AUTO_TPL(var, expr)`: `auto var = expr`
* `BOOST_TYPEOF_NESTED_TYPEDEF(name,expr)`, `BOOST_TYPEOF_NESTED_TYPEDEF_TPL(name,expr)`: `struct name { using type = BOOST_TYPEOF(expr)};`
* `BOOST_REGISTER_TYPE(x)`, `BOOST_REGISTER_TEMPLATE(x,params)`: nothing
* `BOOST_TYPEOF_UNIQUE_ID()`, `BOOST_TYPEOF_INCREMENT_REGISTRATION_GROUP()`: no-op

* for MSVC version <= 19.00, implemented using a type-information encoding strategy.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

------
### Standard Facilities

* Language: `decltype` (C++11), `auto` (C++11).
