# Boost.TriBool

* lib: `boost/libs/logic`
* repo: `boostorg/logic`
* commit: `45b45781`, 2016-10-07

------
### Tri-Bool Logic

#### Header

* `<boost/logic/tribool_fwd.hpp>`
* `<boost/logic/tribool.hpp>`
* `<boost/logic/tribool_io.hpp>`

#### Class `tribool`

##### Members

* `constexpr tribool() noexcept` - `false_value`
* `constexpr tribool(bool) noexcept` - `false_value`/`true_value`
* `constexpr tribool(indeterminate_keyword_t) noexcept` - `indeterminate_value`
* `constexpr explicit operator bool() const noexcept`
* `enum value_t { false_value, true_value, indeterminate_value } value`

##### Non-members

* `constexpr indeterminate(tribool, indeterminate_t) noexcept -> bool`
* operators `!`, `&&`, `||`, `==`, `!=`, 

##### I/O

* `default_false_name() -> std::basic_string<Char>` - `false`
* `default_true_name() -> std::basic_string<Char>` - `true`
* `get_default_indeterminate_name() -> std::basic_string<Char>` - `indeterminate`
* `class indeterminate_name<CharT> : public std::locale::facet`
  - provide string for `indeterminate` state
  - by default use `get_default_indeterminate_name()`
* `<<` and `>>`, use `indeterminate_name` facet.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`

#### Boost.Core

* `<boost/noncopyable.hpp>`

------
### Standard Facilities
