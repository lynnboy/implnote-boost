# Boost.Convert

* lib: `boost/libs/convert`
* repo: `boostorg/convert`
* commit: `b59c1945`, 2015-8-8

------
### `convert` facade API

#### Header `<boost/convert.hpp>`

#### Main API

* `convert<TO>(TI const& v_in, Converter const& cvt) -> optional<TO>` - main API
* `convert<TO>(TI const& v_in) -> optional<TO>` - Use a default-constructed default converter.
  Default converter should be declared as `cnv::by_default` by user.
* Converter can be wrapped by `boost::ref`.

#### API Wrapper

* `convert<TO>(TI const& v_in, Converter const& cvt, throw_on_failure) -> TO` - throw on failure.
* `convert<TO>(TI const& v_in, Converter const& cvt, F const& fallback) -> TO` - `F` is convertible to `TO`.
* `convert<TO>(TI const& v_in, Converter const& cvt, F fallback) -> TO` - `F()` is convertible to `TO`.

#### API Like `bind`

* `cnv::apply<TO, TI>(Converter const& cvt) -> cnv::reference<Converter, TO, TI>`
  Similar to `[cvt, fallback](TI const& v_in){ return convert<TO>(v_in cvt).value_or(fallback.value()); }`
* `cnv::apply<TO>(Converter const& cvt) -> cnv::reference<Converter, TO, void>`
  Similar to `[cvt, fallback](auto const& v_in){ return convert<TO>(v_in, cvt).value_or(fallback.value()); }`

Thus the returned `reference` is a functor can be used by algorithms.

------
### Trait `is_cnv` To Detect Converter Type

Header `<boost/convert/detail/is_converter.hpp>`

```c++
template <typename T, typename TI, typename TO> concept bool is_cnv =
  ( is_class_v<T> || is_function_v<T> ) &&
    requires (T const& cnv, TI const & i, optional<TO>& o) { { cnv(i, o) }; }
```

------
### Base class for converter

* Header `<boost/convert/base.hpp>`
* Header `<boost/convert/parameters.hpp>`

Base class `cnv::cnvbase<DerivedConverter>`, provide basic support for string-oriented conversion.

#### String Conversion Support

* Auto support string-like types:
  * `char` and `wchar_t` pointer and array
  * Any class with accessible member named `begin` and `end`
* Conversible types are `int`, `uint`, `long`, `ulong`, `short`, `ushort`, `float`, `double`, and `long double`.
* Require derived class to implement `to_str` and `str_to`.

#### Declared Keyword-recognized Parameters And Formatting Support

* `adjust`, `base`, `fill`, `locale`, `notation`, `precision`, `skipws`, `uppercase`, `width`
* Provides `operator()(Keyword value) -> DerivedConverter&` API (`locale` and `notation` are unsupported)
  * Option values are stored as `cnvbase` data members.
* `skipws` is handled before call `str_to` of derived class.
* A temporary buffer is used to accept output of `to_str`, then perform format adjustment
  * `uppercase`, `width`, `adjust`, `fill` are handled
* `base`, `precision` are left to derived converter class to handle.

#### Direct Conversion Support

Provide conversion when input value can be `>>` to `optional<TO>`.

------
### Converters

#### `cnv::lexical_cast`

Header `<boost/convert/lexical_cast.hpp>`

Just wrap a call to `lexical_cast`, catch any exception and left the `optional` empty.
Not derived from `cnvbase`.

#### `cnv::spirit`

Header `<boost/convert/spirit.hpp>`

Use Boost.Spirit builtin parsers and generators for required type.
Does not handle the `base` and `precision` options

#### `cnv::basic_stream<CharT>`

* Header `<boost/convert/stream.hpp>`
* Header `<boost/make_default.hpp>`

* Wrap a `basic_stringstream<CharT>` to perform conversion for strings, not copyable, but movable.
* Not derived from `cnvbase`, but supports all keyword parameters, which are set on underlying
  `stringstream` instance.
* Also accept standard manipulators and `std::locale` by `operator()`.
* The converted-to type use `make_default` to create temporary object.
* Typedefs for `char` and `wchar_t` are provided as `cnv::cstream` and `cnv::wstream`.

#### `cnv::printf`

Header `<boost/convert/printf.hpp>`

* Use `snprintf` and `sscanf` to implement `to_str` and `str_to`
* Supports `precision`, and supports `base` for 10, 16, and 8 for integer types, don't support `base` for floats.
* The converted-to type use `make_default` to create temporary object.

#### `cnv::strtol`

Header `<boost/convert/printf.hpp>`

* Use `snprintf` and `sscanf` to implement `to_str` and `str_to`
* Supports arbitrary `base` for integer I/O, recognize `0x` and `0` format.
* Supports `precison` for `float` and `double` output.
* `float` and `double` input is implemented by `strtold`, and only support `char` strings.

------
### Helper Traits

#### `is_fun`

Header `<boost/convert/detail/is_fun.hpp>`

```c++
template <typename T, typename TO> concept bool is_fun =
  ! is_convertible_v<T, TO> && requires (T const& f) { { f() } -> TO; };
```

####  `BOOST_DECLARE_HAS_MEMBER(trait_name, member_name)`

Header `<boost/convert/detail/has_member.hpp>`

```c++
template <typename T> concept trait_name = requires (T t) { t.member_name; };
```

####  `BOOST_DECLARE_HAS_MEMBER(trait_name, member_name)`

Header `<boost/convert/detail/is_callable.hpp>`

```c++
template <typename T, typename Sig> concept trait_name = false;
template <typename T, typename R, typename ...A> concept trait_name<T, R(A...)> =
  requires (T t, A...a) { { t.member_name(a...) } -> R; };
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Optional

* `<boost/optional.hpp>`

#### Boost.MPL

* `<boost/mpl/bool.hpp>`
* `<boost/mpl/vector.hpp>`, `<boost/mpl/find.hpp>` - used by `cnv::printf`.

#### Boost.TypeTraits

* `<boost/type_traits/remove_const.hpp>`
* `<boost/type_traits/make_unsigned.hpp>`, `<boost/type_traits/is_same.hpp>`
* `<boost/type_traits.hpp>`

#### Boost.Range

* `<boost/range/iterator.hpp>`

#### Boost.Core

* `<boost/ref.hpp>`
* `<boost/utility/enable_if.hpp>`
* `<boost/core/noncopyable.hpp>` - required by `cnv::basic_stream`.

#### Boost.FunctionTypes

* `<boost/function_types/is_function_pointer.hpp>`
* `<boost/function_types/function_arity.hpp>`
* `<boost/function_types/result_type.hpp>`

#### Boost.Parameter

* `<boost/parameter/keyword.hpp>`

#### Boost.LexicalCast

* `<boost/lexical_cast.hpp>` - required by `cnv::lexical_cast`.

#### Boost.Spirit

* `<boost/spirit/include/qi.hpp>`, `<boost/spirit/include/karma.hpp>` - required by `cnv::spirit`.

#### Boost.Math

* `<boost/math/special_functions/round.hpp>` - required by `cnv::strtol`.

------
### Standard Facilities

