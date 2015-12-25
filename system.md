# Boost.System

* lib: `boost/libs/system`
* repo: `boostorg/system`
* commit: `e374f766`, 2015-11-21

------
### System Error Code

#### Header `<boost/system/error_code.hpp>`

* `enum errc::errc_t` - mimic `enum class errc`
* Trait `is_error_code_enum<T>` and `is_error_condition_enum<T>`
* `class error_code` and `class error_condition`
* Function `make_error_code(errc_t)` and `make_error_condition(errc_t)`
* `class error_category`
* Function `system_category()` and `generic_category()`

#### Header `<boost/system/system_error.hpp>`

* Exception class `class system_error: public std::runtime_error`.

#### Platform-specific

Header `<boost/system/linux_error.hpp>`, `<boost/system/windows_error.hpp>` and `<boost/system/cygwin_error.hpp>`

Provide platform-specific enum for additional error_code, and provide
corresponding `make_error_code()` overload for that enum type.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/config/auto_link.hpp>` - for library linkage
* `<boost/cstdint.hpp>`
* `<boost/config/abi_prefix.hpp>`, `<boost/config/abi_suffix.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Core

* `<boost/noncopyable.hpp>`
* `<boost/utility/enable_if.hpp>`

#### Boost.Predef

* `<boost/predef/platform.hpp>`

------
### Standard Facilities

Standard Library: `<system_error>` (C++11)
