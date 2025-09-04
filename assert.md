# Boost.Assert

* lib: `boost/libs/assert`
* repo: `boostorg/assert`
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
### `BOOST_CURRENT_FUNCTION`

#### Header

* `<boost/current_funtion.hpp>`

#### Macro

* `BOOST_CURRENT_FUNCTION`

#### Switch Macro

* `BOOST_DISABLE_CURRENT_FUNCTION` -- Use `(unknown)`.

------
### Class `source_location`

#### Header

* `<boost/assert/source_location.hpp>`

#### Class `source_location`

```c++
struct source_location {
  constexpr source_location() : source_location("", 0, "", 0) {}
  constexpr source_location(char const* file, uint_least32_t line,
            char const* function, uint_least32_t column = 0) noexcept;
  constexpr source_location(std::source_location const&) noexcept;
  constexpr char const* file_name() noexcept;
  constexpr char const* function_name() noexcept;
  constexpr uint_least32_t line() noexcept;
  constexpr uint_least32_t column() noexcept;
  std::string to_string() const;
  friend bool operator==(source_location const& s1, source_location const& s2) noexcept;
  friend bool operator!=(source_location const& s1, source_location const& s2) noexcept;
};
template<class E, class T>
  std::basic_ostream<E, T>& operator<<(std::basic_ostream<E, T>& os, source_location const& loc);
```

* Macro `BOOST_CURRENT_LOCATION` used as initializer.
  It is a `source_location` rvalue initialized with `std::source_location::current()` if available, otherwise with `__FILE__` etc.
* Disabled by `BOOST_DISABLE_CURRENT_LOCATION`.
* Supports `<<` on ostreams, `to_string()`.
* Supports `==` and `!=`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/cstdint.hpp>`

------
### Standard Facilities

* Preprocessor: `__func__`, `__FILE__`, `__LINE__`.
* Standard Library: `<cassert>`, `source_location` (C++20)
