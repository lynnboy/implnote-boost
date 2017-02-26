# Boost.IO

* lib: `boost/libs/io`
* repo: `boostorg/io`
* commit: `00ef24c8`, 2017-01-29

------
### IO State Savers

#### Header

* `<boost/io_fwd.hpp>`
* `<boost/io/ios_state.hpp>`

#### IOS State Savers

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

#### Rationale

Restore `ios_base` or `basic_ios` flags on destruction.

------
### Quoted string for `<<` and `>>`.

#### Header

* `<boost/io/detail/quoted_manip.hpp>`

#### IO Manipulator `quoted`

```c++
template <class Char, class Traits, class Alloc>
quoted(const std::basic_string<Char, Traits, Alloc>& s, Char escape='\\', Char delim='\"')
  -> quoted_proxy<std::basic_string<Char, Traits, Alloc> const &, Char>;

template <class Char, class Traits, class Alloc>
quoted(std::basic_string<Char, Traits, Alloc>& s, Char escape='\\', Char delim='\"')
  -> quoted_proxy<std::basic_string<Char, Traits, Alloc> &, Char>;

template <class Char>
quoted(const Char* s, Char escape='\\', Char delim='\"') -> quoted_proxy<const Char*, Char>;
```

#### Rationale

* The `quoted_proxy<String, Char>` class is defined with `<<` and `>>`, serving
as the manipulator type. `>>` is only provided for non-`const` version of `basic_string`.

------
### Dependency

#### Boost.Config

* `<boost/detail/workaround.hpp>`.

------
### Standard Facilities

* Standard Library: n3654 Quoted Strings Library Proposal, (C++14).
