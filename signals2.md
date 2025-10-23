# Boost.Signals2

* lib: `boost/libs/signals2`
* repo: `boostorg/signals2`
* commit: `5dcb2af`, 2025-03-21

------
### `BOOST_ASSERT` / `BOOST_VERIFY`

#### Header

* `<boost/assert.hpp>`

#### Macro

* `BOOST_ASSERT(expr)`
* `BOOST_ASSERT_MSG(expr, msg)`
* `BOOST_VERIFY(expr)`
* `BOOST_VERIFY_MSG(expr, msg)`
* `BOOST_ASSERT_IS_VOID` - Whether assertion is not in effect.

#### Switch Macro

* `BOOST_DISABLE_ASSERTS` -- Force NO-OP.
* `BOOST_ENABLE_ASSERT_HANDLER` -- Use _custom handlers_ instead of standard `<cassert>`.
* `BOOST_ENABLE_ASSERT_DEBUG_HANDLER` -- Use _custom handlers_, if not defined `NDEBUG`.
* Default behavior is to use standard `<cassert>`.
* `BOOST_ASSERT_HANDLER_IS_NORETURN` -- Specify _custom _handlers_ are `[[noreturn]]`

#### User Provided

* `boost::assertion_failed`
* `boost::assertion_failed_msg`

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Bind

* `<boost/config.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/cstdint.hpp>`

#### Boost.Core

* `<boost/config.hpp>`

#### Boost.Function

* `<boost/function.hpp>`

#### Boost.Iterator

* `<boost/config.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.Optional

* `<boost/optional.hpp>`

#### Boost.Parameter

* `<boost/parameter.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.SmartPtr

* `<boost/smart_ptr/**.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Tuple

* `<boost/tuple.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits.hpp>`

#### Boost.Variant

* `<boost/variant.hpp>`

------
### Standard Facilities

* Preprocessor: `__func__`, `__FILE__`, `__LINE__`.
* Standard Library: `<cassert>`, `source_location` (C++20)
