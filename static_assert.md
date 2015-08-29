# Boost.StaticAssert

* lib: `boost/libs/static_assert`
* repo: `boostorg/static_assert`
* commit: `bb3cc11f0913defd9a18b338a69fcd7b4c8ddb5c`, 2014-10-02

------
### BOOST_STATIC_ASSERT

#### Header

* `<boost/static_assert.hpp>`

#### Macro

* `BOOST_STATIC_ASSERT(...)`
* `BOOST_STATIC_ASSERT_MSG(..., msg)`

#### Rationale

If the compiler supports `static_assert`, invoke `static_assert(expr, #expr)`
or `static_assert(expr, msg)`, othereise results in a typedef declaration that
will cause template instantiation failure if tested argument is false.

------
### Dependency

#### Boost::Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`.

------
### Standard Facilities

* Language: `static_assert`.
* Proposals:
  * N3928 - `static_assert` with no message.
