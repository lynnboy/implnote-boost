# Boost.IO

* lib: `boost/libs/io`
* repo: `boostorg/io`
* commit: `2d9a241`, 2025-06-27

------
### IO State Savers

* Header `<boost/io_fwd.hpp>`
* Header `<boost/io/ios_state.hpp>`

```c++
class ios_flags_saver;
class ios_precision_saver;
class ios_width_saver;
class ios_base_all_saver;

template < typename Ch, class Tr = ::std::char_traits<Ch> >
    class basic_ios_iostate_saver;
template < typename Ch, class Tr = ::std::char_traits<Ch> >
    class basic_ios_exception_saver;
template < typename Ch, class Tr = ::std::char_traits<Ch> >
    class basic_ios_tie_saver;
template < typename Ch, class Tr = ::std::char_traits<Ch> >
    class basic_ios_rdbuf_saver;
template < typename Ch, class Tr = ::std::char_traits<Ch> >
    class basic_ios_fill_saver;
template < typename Ch, class Tr = ::std::char_traits<Ch> >
    class basic_ios_locale_saver;
template < typename Ch, class Tr = ::std::char_traits<Ch> >
    class basic_ios_all_saver;

typedef basic_ios_iostate_saver<char>        ios_iostate_saver;
typedef basic_ios_iostate_saver<wchar_t>    wios_iostate_saver;
typedef basic_ios_exception_saver<char>      ios_exception_saver;
typedef basic_ios_exception_saver<wchar_t>  wios_exception_saver;
typedef basic_ios_tie_saver<char>            ios_tie_saver;
typedef basic_ios_tie_saver<wchar_t>        wios_tie_saver;
typedef basic_ios_rdbuf_saver<char>          ios_rdbuf_saver;
typedef basic_ios_rdbuf_saver<wchar_t>      wios_rdbuf_saver;
typedef basic_ios_fill_saver<char>           ios_fill_saver;
typedef basic_ios_fill_saver<wchar_t>       wios_fill_saver;
typedef basic_ios_locale_saver<char>         ios_locale_saver;
typedef basic_ios_locale_saver<wchar_t>     wios_locale_saver;
typedef basic_ios_all_saver<char>            ios_all_saver;
typedef basic_ios_all_saver<wchar_t>        wios_all_saver;

class ios_iword_saver;
class ios_pword_saver;
class ios_all_word_saver;
```

Restore `ios_base` or `basic_ios` flags on destruction.

------
### Quoted string for `<<` and `>>`.

Header `<boost/io/quoted.hpp>`

```c++
struct detail::quoted_proxy<Str,C> { Str string; C escape, delim; };
basic_ostream<C,Tr>& operator<< (basic_ostream<C,Tr>&, detail::quoted_proxy<> const&);
basic_istream<C,Tr>& operator>> (basic_istream<C,Tr>&, detail::quoted_proxy<> const&);

template <class Char, class Traits, class Alloc>
quoted(const std::basic_string<Char, Traits, Alloc>& s, Char escape='\\', Char delim='\"')
  -> quoted_proxy<std::basic_string<Char, Traits, Alloc> const &, Char>;

template <class Char, class Traits, class Alloc>
quoted(std::basic_string<Char, Traits, Alloc>& s, Char escape='\\', Char delim='\"')
  -> quoted_proxy<std::basic_string<Char, Traits, Alloc> &, Char>;

template <class Char>
quoted(const Char* s, Char escape='\\', Char delim='\"') -> quoted_proxy<const Char*, Char>;
```

* The `quoted_proxy<String, Char>` class is defined with `<<` and `>>`, serving
as the manipulator type. `>>` is only provided for non-`const` version of `basic_string`.

### `ostream_joiner` Iterator Wrapper

Header <boost/io/ostream_joiner.hpp>`

```c++
template<class Delim, class Char=char, class Tr=char_traits<Char>>
class ostream_joiner {
    ostream_type* s_; Delim d_;
public:
    using char_type = Char; using traits_type = Tr; using ostream_type = std::basic_ostream<Char,Tr>;
    ostream_joiner(ostream_type&, [const]Delim&[&]); // ctor
    ostream_joiner& operator=<T>(const T&); // assign, write delim (not first time) before value.
    // iterator interface: operator *, ++, value_type, different_type, pointer, reference, iterator_category
};
make_ostream_joiner<C,Tr,Delim>(ostream_type&, Delim&&) -> ostream_joiner<Delim, C, Tr>;
```

Wraps an output stream and a delimiter. write delimiter between assignments.

### `ostream_put`

Header `<boost/io/ostream_put.hpp>`

```c++
std::basic_ostream<C,Tr>& ostream_put<C,Tr>(std::basic_ostream<C,Tr>&, const C*, size_t);
```

Formatted inserter function. Handle string buffer just like `string_view`.

### Null Output Stream

Header `<boost/io/nullstream.hpp>`

```c++
struct basic_nullbuf<Ch,Tr=std::char_traits<Ch>> : std::basic_streambuf<Ch,Tr> { /*...*/ };
struct basic_onullstream<Ch,Tr=std::char_traits<Ch>> : std::basic_ostream<Ch,Tr> {};
using  onullstream = basic_onullstream<char>;
using wonullstream = basic_onullstream<wchar_t>;
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`.

------
### Standard Facilities

* Standard Library: n3654 Quoted Strings Library Proposal, `std::quoted` (C++14).
