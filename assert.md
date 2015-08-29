# Boost::Assert

* lib: `boost/libs/assert`
* repo: `boostorg/assert`
* tag: `boost-1.59.0`

------
### BOOST_ASSERT / BOOST_VERIFY

#### Header

* `<boost/assert.hpp>`

#### Macro

* `BOOST_ASSERT(expr)`
* `BOOST_ASSERT_MSG(expr, msg)`
* `BOOST_VERIFY(expr)`
* `BOOST_VERIFY_MSG(expr, msg)`

#### Switch Macro

* `BOOST_DISABLE_ASSERTS` -- Force NO-OP.
* `BOOST_ENABLE_ASSERT_HANDLER` -- Use _custom handlers_ instead of standard `<cassert>`.
* `BOOST_ENABLE_ASSERT_DEBUG_HANDLER` -- Use _custom handlers_, if not defined `NDEBUG`.
* Default behavior is to use standard `<cassert>`.

#### User Provided

* `boost::assertion_failed`
* `boost::assertion_failed_msg`

------
### BOOST_CURRENT_FUNCTION

#### Header

* `<boost/current_funtion.hpp>`

#### Macro

* `BOOST_CURRENT_FUNCTION`

------
### Dependency

#### Boost::Config

* `<boost/config.hpp>`, when _custom handlers_ are enabled.

------
### Standard Facilities

* Preprocessor: `__func__`, `__FILE__`, `__LINE__`.
* Standard Library: `<cassert>`
* Proposals:
  * N4129 - Source Code Information Capture.
