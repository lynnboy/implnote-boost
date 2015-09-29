# Boost.Format

* lib: `boost/libs/format`
* repo: `boostorg/format`
* commit: `8023d13d`, 2015-09-23

------
### Format String Based Upon Stream

Header `<boost/format.hpp>`

* Supports `%` argument feeding, using internal stream to make step1 formating
* `printf` like format specification, with extensions
* Arguments can be `bind` at specific position.
* Optionally report errors via exceptions.

```c++
template <typename CharT, typename Trait, typename Alloc>
class basic_format {
  enum style_values { ordered = 1, special_needs = 4 } style;
  vector<format_item_t>        items_;  // all items
  vector<bool>                 bound_;  // map for which items are bound
  int             cur_arg_, num_args_;  // arg count, and parsed arg slot count (expected)
  string_type                 prefix_;  // substring of format before first item
  uchar                   exceptions_;  // exception mask
  internal_streambuf_t           buf_;  // stream buffer, reused throughout lifetime, to reduce allocation
  bool    dumped_;  // output flag
  optional<locale_t>  loc_; // locale, if supported

public:
  using string_type = basic_string<CharT, Trait, Alloc>;
  using format_item_t = format_item<CharT, Trait, Alloc>;
  using internal_streambuf_t = basic_altstringbuf<CharT, Trait, Alloc>;

  explicit basic_format(CharT const*=NULL [, std::locale const&]);
  explicit basic_format(string_type const&[, std::locale const&]);
  // copy-ctor, copy-assign, swap
  // accessors: getloc(), expected_args(), bound_args(), fed_args(), cur_arg(), remaining_args()
  // exception flag: exceptions() return flag, exception(unsigned char) swap flag

  basic_format& clear();        // clear except bind
  basic_format& clear_binds();  // clear all items
  basic_format& parse(string_type const&);  // parse format, prepare and clear all arg items

  string_type str() const;    // get resulting formatted string, set after str() first time called
  size_type size() const;     // the size sumed from each item, will be the resulting string's size

  basic_format& operator % (T const&);        // format argument feed
  basic_format& modify_item(int argN, T manipulator);   // modify stream format by std::manip before feed
  basic_format& bind_arg(int argN, T const&); // explicitly bind N-th arg
  basic_format& clear_bind(int argN);
};
auto operator <<(basic_ostream<CharT, Trait> const&, basic_format const&); // os << format.str();
string_type str(basic_format const&); // call format.str();

using format = basic_format<char>; using wformat = basic_format<wchar_t>;
enum format_error_bits { bad_format_string_bit = 1, ... all_error_bits = 255, no_error_bits=0 };

class format_error : public std::exception; // base exception
// bad_format_string, too_few_args, too_many_args, out_of_range are derived classes

template<class Ch, class Tr=::std::char_traits<Ch>, class Alloc=::std::allocator<Ch> >
class basic_altstringbuf; // implemented using a shared_ptr<string> for storage, at least allocate 256 chars
template<class Ch, class Tr =::std::char_traits<Ch>, class Alloc=::std::allocator<Ch> >
class basic_oaltstringstream;

template<class Ch, class Tr> 
struct stream_format_state {
    std::streamsize width_, precision_;
    Ch fill_; 
    std::ios_base::fmtflags flags_;
    std::ios_base::iostate  rdstate_, exceptions_;
    optional<locale_t>  loc_;

    stream_format_state(Ch fill) { reset(fill); }
    void reset(Ch fill);                                //- sets to default state.
    void set_by_stream(const basic_ios& os);            //- sets to os's state.
    void apply_on(basic_ios & os, locale_t * = 0) const;//- applies format_state to the stream
    template<class T> void apply_manip(T manipulator);  //- modifies state by applying manipulator
};
template<class Ch, class Tr, class Alloc>  
struct format_item {     
    enum pad_values { zeropad = 1, spacepad =2, centered=4, tabulation = 8 };
    enum arg_values { argN_no_posit   = -1, argN_tabulation = -2, argN_ignored  = -3 };

    format_item(Ch fill);
    void reset(Ch fill);
    void compute_states(); // sets states  according to truncate and pad_scheme.

    int         argN_;  // arg_values
    string_type  res_, appendix_;
    stream_format_state fmtstate_;// set by parsing, is only affected by modify_item
    std::streamsize truncate_;  //- is set for directives like %.5s that ask truncation
    unsigned int pad_scheme_;   // pad_values
};

tuple<T...> group(T...a, Var cosnt&);
tuple<T...> group_head(tuple<T..., Val> const&); // all except last, all should be std manipulators
tuple<Val> group_last(tuple<T..., Val> const&); // last, the value
```

* When parsed, the text before first argument is stored as `prefix`, while other literal are stored as
  `appendix` in each item data.
* Parsing code translates recognized `printf`-style format specification into corresponding stream state.
* When an argument is feed to a format, it is internally converted for each `format_item` with the expected position
  by use of a `altstringstream` instance and gets the result string cached as result string.
* `group` is provided to combine std manipulators with an argument as a whole.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Optional

* `<boost/optional.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Utility

* `<boost/utility/base_from_member.hpp>`

#### Boost.SmartPtr

* `<boost/shared_ptr.hpp>`

------
### Standard Facilities
