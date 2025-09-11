# Boost.LexicalCast

* lib: `boost/libs/lexical_cast`
* repo: `boostorg/lexical_cast`
* commit: `a16040a`, 2025-08-22

------
### Lexical Type Cast API

Header `<boost/lexical_cast.hpp>`

```c++
class bad_lexical_cast : public std::bad_cast {
  const type_info *source, *target;
  public: const type_info& source_type() const noexcept; const type_info& target_type() const noexcept;
};

struct buffer_view<Ch> {const Ch *begin, *end; }; // `signed/unsigned char` are casted to `char`
std::basic_ostream<Elem,Tr>& operator<< <Ch,Elem,Tr>(std::basic_ostream<Elem,Tr>& os, buffer_view<Ch> r);

bool detail::dynamic_num_converter_impl<Target,Source>::try_convert(Source a, Target& res);

struct detail::optimized_src_stream<Ch,Tr,BufSize>; // optimized implementation of stream_in() overloads
struct detail::ios_src_stream<Ch,Tr>; // wrap a basic_ostream and an unlocked buffer, provides stream_in() overloads on `<<`
struct detail::to_target_stream<Ch,Tr>; // wrap a pair of char pointers (range of buffer), provides optimized stream_out() overloads, `>>` for only-streamable types

bool detail::lexical_converter_impl<Target,Source>::try_convert(Source a, Target& res) {
  using source_char_type = /*deduce known lib types (stage 1) or ostream-able types (stage 2)*/;
  using target_char_type = /*deduce known lib types (stage 1) or istream-able types (stage 2)*/;
  using char_type = wider_char<target_char_type,source_char_type>; using traits = std::char_traits<char_type>;
  using from_src_stream = supports_invoke(optimized_src_stream::stream_in(exact<Source>)) ?
    optimized_src_stream : ios_src_stream;
  from_src_stream src_stream{};
  if (!src_stream.stream_in(exact<Source>(arg))) return false; // write to src_stream's buffer
  to_target_stream out(src_stream.cbegin(), src_stream.cend());
  if (!out.stream_out(result)) return false; // write to result
  return true;
}

bool try_lexical_convert<Target,Source>(const Source& arg, Target& res) {
  if constexpr (is_arithmetic_v<Source> && !is_character_v<Source> && /*and Target too*/)
    return dynamic_num_converter_impl<Target,Source>::try_convert (arg, res);
  
}
bool try_lexical_convert<Target,Ch>(const Ch* s, std::size_t n, Target& res)
{ return try_lexical_convert(buffer_view{s, n}, res); } // `signed/unsigned char` are casted to `char`

Target lexical_cast<Target,Source>(const Source & a) {
  Target res{};
  if (!try_lexical_convert(a, res)) throw_bad_cast<Source,Target>();
  return res;
}

Target lexical_cast<Target>(const Ch* s, std::size_t n) // Ch is every supported character type
{ return lexical_cast<Target>(buffer_view{s, n}); } // `signed/unsigned char` are casted to `char`
```

* `lexical_cast` require `Target` to be default-constructible and copy-constructible.
* `bad_lexical_cast` is thrown by `lexical_cast` when conversion failed.

* Direct assignment is performed when `Target` and `Source` are
  * the same character type, or converting between `char`, `signed char` or `unsigned char`
  * the same `std::basic_string<CharT>` type, or from array/pointer of `CharT` to `std::basic_string<CharT>`
* When both types are arithmetic types (except character types), perform checked converting (`dynamic_num_converter`)
  * When `Target` is unsigned, and `Source` is signed or floating-point and `< 0`, then cast negated arg and use complement value.
  * If the value will loss precision after convert (narrowing), then conversion fail, otherwise assign to `Target`
* Otherwise, perform `std::stringstream` based converting (`lexical_converter`).
  * Recognize `Source` as character type, [const] pointer of, char-pointer-pair of, `basic_string{_view}` of, and `array` of, for stage 1 type
  * Prefer using `stringstream` over `wstringstream`, for stage 2 type
  * Defined optimized istream and ostream like classes providing `<<` and `>>` for all primitive types
  * If unsupported by optimized stream-like types, use stringstream types.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`
* `<boost/limits.hpp>`

#### Boost.Container

* `<boost/container/container_fwd.hpp>`

#### Boost.Core

* `<boost/core/cmath.hpp>`
* `<boost/core/snprintf.hpp>`
* `<boost/core/enable_if.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

------
### Standard Facilities

* Proposals:
  * N1973 - Lexical Conversion Library Proposal for TR2 (not accepted)
  * P0117R0 - Generic `to_string`/`to_wstring` functions
