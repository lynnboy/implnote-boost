# Boost.NoWide

* lib: `boost/libs/nowide`
* repo: `boostorg/nowide`
* commit: `2a8f682`, 2025-09-19

UTF-8 converting functions, and Windows UTF-8 compatibility layer.

------
### UTF-8 Converting

* Header `<boost/nowide/utf/utf.hpp>`

```c++
namespace boost::nowide::utf;
using code_point = uint32_t;
static const codepoint illegal = 0xFFFF'FFFFu, incomplete = 0xFFFF'FFFEu;
bool is_valid_codepoint(code_point v); // <= 0x10FFFF and not surrogates
struct utf_traits<Ch,size=sizeof(Ch)>; // primary
struct utf_traits<Ch,1> { // for UTF-8, (char, char8_t)
    using char_type = Ch;
    static int trail_length(char_type ci); // 0~3, -1 for invalid
    static const int max_width = 4;
    static int width(code_point v); // 1~4 Ch for v
    static bool is_trail(char_type ci); static bool is_lead(char_type ci);
    static code_point decode <It> (It& p, It e); // return illegal/incomplete for error cases
    static code_point decode_valid <It> (It& p); // assure p points valid utf-8 seq
    static It encode <It> (code_point v, It out); // write UTF-8 to out
};
struct utf_traits<Ch,2> { // for UTF-16 (char16_t, wchar_t on Win)
    using char_type = Ch;
    static bool is_single_codepoint(uint16_t x);
    static bool is_first_surrogate(uint16_t x); static bool is_second_surrogate(uint16_t x);
    static code_point combine_surrogate(uint16_t w1, uint16_t w2);

    static int trail_length(char_type c); // 0 or 1, -1 for invalid
    static const int max_width = 2;
    static int width(code_point v); // 1~2 Ch for v
    static bool is_trail(char_type ci); static bool is_lead(char_type ci);
    static code_point decode <It> (It& p, It e); // return illegal/incomplete for error cases
    static code_point decode_valid <It> (It& p); // assure p points valid utf-16 char or sur-pair
    static It encode <It> (code_point v, It out); // write UTF-16 char or sur-pair to out
};
struct utf_traits<Ch,4> { // for UTF-32 (char32_t, wchar_t on POSIX)
    using char_type = Ch;
    static int trail_length(char_type c); // 0, -1 for invalid
    static const int max_width = 1;
    static int width(code_point v); // always 1
    static bool is_trail(char_type ci); static bool is_lead(char_type ci); // always false, true
    static code_point decode <It> (It& p, It e); // read 1 Ch, return illegal/incomplete for error cases
    static code_point decode_valid <It> (It& p); // read 1 Ch
    static It encode <It> (code_point v, It out); // write 1 Ch
};
```

-----
* Header `<boost/nowide/utf/convert.hpp>`

```c++
namespace boost::nowide::utf;
size_t strlen <Ch> (const Ch* s);
CharOut* convert_buffer<CharOut,CharIn> (CharOut* buf, size_t n, const CharIn* f, const CharIn* l);
std::basic_string<CharOut> convert_string<CharOut,CharIn> (const CharIn* f, const CharIn* l);
std::basic_string<CharOut> convert_string<CharOut,CharIn> (const std::basic_string<CharIn>& s);
```

-----
* Header `<boost/nowide/convert.hpp>`

```c++
using detail::is_char_type<Ch> = Ch == {char|wchar_t|char8_t|char16_t|char32_t};
using detail::is_c_string<Str> = {Str match const T*} && is_char_type<T>;
using detail::const_data_result<Str> = decltype(declval<const Str>().data());
using detail::is_string_container<Str> =
    is_integral_v<decltype(Str::npos)> && is_integral_v<declype(declval<Str>().size())> // `npos` and `size()`
    && is_c_string<const_data_result<Str>>; // `data` gets c-string
using detail::requires_narrow_string_container<Str> = is_string_container<Str> && sizeof(const_data_result<Str>) == 1;
using detail::requires_wide_string_container<Str> = is_string_container<Str>  && sizeof(const_data_result<Str>) > 1;
using detail::requires_narrow_char<Ch> = std::is_char_v<Ch> && sizeof(Ch) == 1;
using detail::requires_wide_char<Ch> = std::is_char_v<Ch> && sizeof(Ch) > 1;

char* narrow(char* buf, size_t n, const wchar_t* f, const wchar_t* l); // convert_buffer
char* narrow(char* buf, size_t n, const wchar_t* s); // [s,s+strlen(s))
wchar_t* widen(wchar_t* buf, size_t n, const char* f, const char* l); // convert_buffer
wchar_t* widen(wchar_t* buf, size_t n, const char* s); // [s,s+strlen(s))
std::string narrow <Ch>(const Ch* s, size_t n) requires std::is_char_v<Ch> && sizeof(Ch) > 1; // convert_string
std::string narrow <Ch>(const Ch* s) requires requires_wide_char<Ch>; // [s,s+strlen(s))
std::string narrow <SV>(const SV& s) requires requires_wide_string_container<SV>; // convert_string
std::wstring wide <Ch>(const Ch* s, size_t n) requires std::is_char_v<Ch> && sizeof(Ch) > 1; // convert_string
std::wstring wide <Ch>(const Ch* s) requires requires_narrow_char<Ch>; // [s,s+strlen(s))
std::wstring wide <SV>(const SV& s) requires requires_narrow_string_container<SV>; // convert_string
```

-----
### Standard Library Replacements

* Header `<boost/nowide/cstdio.hpp>`
* Header `<boost/nowide/cstdlib.hpp>`
* Header `<boost/nowide/stat.hpp>`

Filenames and mode strings are treated as UTF-8.

-----
* Header `<boost/nowide/utf8_codecvt.hpp>`

Provides C++ locale component.

-----
* Header `<boost/nowide/filebuf.hpp>`
* Header `<boost/nowide/fstream.hpp>`

Accepts UTF-8 filenames.

-----
* Header `<boost/nowide/iostream.hpp>`

Define class `winconsole_istream` and `winconsole_ostream`, and define `cin`, `cout`, `cerr`, and `clog`.
Internally, use classs `console_input_buffer_base` and `console_output_buffer_base`, to handle buffering and UTF-8/UTF-16 conversion,
and implemented basedon Windows Console API calls.

-----
### Utilities

* Header `<boost/nowide/stackstring.hpp>`

```c++
class basic_stackstring<ChOut=wchar_t, CharIn=char, bufferSize=256> {
    output_char buffer_[buffer_size], *data_ = nullptr;
protected: bool uses_stack_memory() const { return data_ == buffer_; }
    size_t length() const { if (!data_) return 0; return strlen(data); } // not O(1)
public: const size_t buffer_size = bufferSize;
    using output_char = CharOut; using input_char = CharIn;
    ctor(); ctor(self const&);
    explicit ctor(const input_char* str); // init by convert
    ctor(const input_char* b, const input_char* e);
    ~dtor() { clear(); }
    self& operator=(self const& other) {
        if (this!=&other) { clear(); const size_t len = other.length();
            if (other.uses_stack_memory()) data_ = buffer; // stack
            else if (other.data_) data_ = new output_char[len + 1]; // heap
            else { data_ = nullptr; return *this; } // empty
            std::memcpy(data_, other.data_, sizeof(output_char)*(len+1)); // copy data
        }
        return *this;
    }
    output_char* convert(const input_char* str); // [str,str+strlen(str)), if (!str), just clear() and return get()
    output_char* convert(const input_char* b, const input_char* e) {
        clear(); if (b) {/*try convert_buffer() fill buffer, allocate heap buffer if not enough*/} return get();
    }
    <const> output_char* get() <const> { return data_; }
    void clear() { if (!uses_stack_memory()) delete[] data_; data_ = nullptr;}

    friend void swap(self& lhs, self& rhs); // if both on heap, just swap data_, otherwise swap buffer content too.
};
using wstackstring = basic_stackstring<wchar_t, char, 256>;
using stackstring = basic_stackstring<char, wchar_t, 256>;
using wshort_stackstring = basic_stackstring<wchar_t, char, 16>;
using short_stackstring = basic_stackstring<char, wchar_t, 16>;
```

An RAII class to hold UTF string converting result.

-----
* Header `<boost/nowide/filesystem.hpp>`

```c++
std::locale nowide_filesystem();
```

Provides a locale installation API to imbue UTF-8 converting into Boost.FileSystem `path`.

-----
* Header `<boost/nowide/quoted.hpp>`

```c++
struct detail::is_path<T> { static constexpr bool value = ...; }; // detects existance of `T::make_preferred` and `T::filename`
struct detail::quoted<Path> {
    Path value; // wrapped value.
    friend std::basic_ostream<Ch>& operator<< <Ch> (std::basic_ostream<Ch>& out, const self& path);
    friend std::basic_istream<Ch>& operator>> <Ch> (std::basic_istream<Ch>& in, const self& path);
};
detail::quoted<Path&> quoted <Path> (Path& path) requires is_path<Path>::value { return {path}; }
```

Input/output with `std::quoted` upon path's `native` value, converted as needed.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/auto_link.hpp>`
* `<boost/version.hpp>`

#### Boost.FileSystem

* `<boost/filesystem/path.hpp>`

------
### Standard Facilities
