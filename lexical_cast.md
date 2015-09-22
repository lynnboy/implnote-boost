# Boost.LexicalCast

* lib: `boost/libs/lexical_cast`
* repo: `boostorg/lexical_cast`
* commit: `55b1b832`, 2015-04-27

------
### Lexical Type Cast API

Header `<boost/lexical_cast.hpp>`

```c++
class bad_lexical_cast : public std::bad_cast;

template <typename Target, typename Source>
Target lexical_cast(const Source &);

template <typename Target>
Target lexical_cast(const [signed][unsigned] char*, std::size_t count);

template <typename Target, typename Source>
bool try_lexical_convert(const Source&, Target&);

template <typename Target, typename CharT>
bool try_lexical_convert(const CharT*, std::size_t, Target&);
```

* `lexical_cast` require `Target` to be default-constructible and copy-constructible.
* `bad_lexical_cast` is thrown by `lexical_cast` when conversion failed.

* Direct assignment is performed when `Target` and `Source` are
  * the same character type, or converting between `char`, `signed char` or `unsigned char`
  * the same `std::basic_string<CharT>` type, or from array/pointer of `CharT` to `std::basic_string<CharT>`
* When both types are arithmetic types (except character types), perform checked converting
  * When `Target` is unsigned, and `Source` is signed or floating-point and `< 0`, then negate the source
  * If the value will loss precision after convert, then conversion fail, otherwise assign to `Target`
* Otherwise, perform `std::stringstream` based converting.
  * Recognize `Source` as character type, [const] pointer of, `iterator_range` of, `basic_string` of, and `array` of, for stage 1 type
  * Prefer using `stringstream` over `wstringstream`, for stage 2 type
  * Defined optimized istream and ostream like classes providing `<<` and `>>` for all primitive types
  * If unsupported by optimized stream-like types, use stringstream types.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Core

* `<boost/noncopyable.hpp>`

#### Boost.Utility

* `<boost/utility/value_init.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.MPL

* `<boost/mpl/if.hpp>`, `<boost/mpl/bool.hpp>`
* `<boost/mpl/identity.hpp>`, `<boost/mpl/eval_if.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_arithmetic.hpp>`, `<boost/type_traits/is_integral.hpp>`, `<boost/type_traits/is_float.hpp>`
* `<boost/type_traits/is_same.hpp>`, `<boost/type_traits/is_base_of.hpp>`
* `<boost/type_traits/is_signed.hpp>`, `<boost/type_traits/make_unsigned.hpp>`
* `<boost/type_traits/is_pointer.hpp>`, `<boost/type_traits/is_abstract.hpp>`
* `<boost/type_traits/has_left_shift.hpp>`, `<boost/type_traits/has_right_shift.hpp>`

#### Boost.Math/Special_Functions

* `<boost/math/special_functions/sign.hpp>`, `<boost/math/special_functions/fpclassify.hpp>`

#### Boost.Numeric/Conversion

* `<boost/numeric/conversion/cast.hpp>`

#### Boost.Integer

* `<boost/integer.hpp>`
* `<boost/integer_traits.hpp>`

#### Boost.Range

* `<boost/range/iterator_range_core.hpp>`

#### Boost.Array

* `<boost/array.hpp>`

#### Boost.Container

* `<boost/container/container_fwd.hpp>`

------
### Standard Facilities

* Proposals:
  * N1973 - Lexical Conversion Library Proposal for TR2 (not accepted)
