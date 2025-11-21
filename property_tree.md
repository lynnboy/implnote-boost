# Boost.PropertyTree

* lib: `boost/libs/property_tree`
* repo: `boostorg/property_tree`
* commit: `2669458`, 2025-06-30

------
### Common Parts

```c++
struct ptree_error : std::runtime_error { ctor(const std::string& what); };
class ptree_bad_data : public ptree_error { any m_data;
  ctor<T>(const std::string& what, const T& data); T data<T>() const { return any_cast<T>(m_data); } };
class ptree_bad_path : public ptree_error { any m_path; // f"{what} ({path.dump()})"
  ctor<T>(const std::string& what, const T& path); T path<T>() const { return any_cast<T>(m_path); } };

struct detail::less_nocase<T> { using Ch = T::value_type;
  std::locale m_locale;
  bool operator()(Ch c1, Ch c2) const { return std::toupper(c1, m_locale) < std::toupper(c2, m_locale); }
  bool operator()(const T& t1, const T& t2) const { return std::lexicographical_compare(t1.begin(), t1.end(), t2.begin(), t2.end(), *this); }
};
struct detail::is_character<Ch>; // trait
struct detail::has_internal_type<T>;
struct detail::has_external_type<T>;
struct detail::is_translator<T> : mpl::and_<has_internal_type<T>, has_external_type<T>>{};
Str detail::widen<Str>(const char* text);
Str detail::narrow<Str,Ch>(const Ch* text);
Str detail::trim<Str>(const Str& s, const std::locale& loc={});

class basic_ptree<Key,Data,KeyComp=std::less<Key>>;
void swap<K,D,C>(basic_ptree<K,D,C>&, basic_ptree<K,D,C>&);

struct translator_between<Internal,External>;
struct translator_between<T,T> { using type = id_translator<T>; };
struct translator_between<std::basic_string<Ch,Tr,A>, std::basic_string<Ch,Tr,A>> { using type = id_translator<std::basic_string<Ch,Tr,A>>; };

class string_path<Str,Trans> {
  Str m_value; char_type m_separator; Trans m_tr; Str::iterator m_start;
  Str::const_iterator cstart() const { return m_start; }
public: using key_type = Trans::external_type; using char_type = Str::value_type;
  explicit ctor(char_type separator={'.'});
  ctor(const {Str&|char_type*} value, char_type separator={'.'}, Trans tr={});
  ctor(const self&); self& operator=(const self&);
  key_type reduce();
  bool empty() const; bool single() const;
  char_type separator() const { return m_separator; }
  std::string dump() const { return detail::dump_sequence(m_value); }
  friend self operator/ (self p1, const self& p2) { p1 /= p2; return p1; }
  self& operator/= (cosnt self& o);
};
struct path_of<std::basic_string<Ch,Tr,A>>
{ using _string = std::basic_string<Ch,Tr,A>; using type = string_path<_string, id_translator<_string>>; };

using path = string_path<std::string, id_translator<std::string>>;
using ptree = basic_ptree<std::string, std::string>;
using iptree = basic_ptree<std::string, std::string, less_nocase<std::string>>;
using wpath = string_path<std::wstring, id_translator<std::wstring>>;
using wptree = basic_ptree<std::wstring, std::wstring>;
using wiptree = basic_ptree<std::wstring, std::wstring, less_nocase<std::wstring>>;

struct id_translator<T> { using internal_type = T; using external_type = T;
  optional<T> get_value(const T& v) { return v; } optional<T> put_value(const T& v) { return v; }
};

struct customize_stream<Ch,Tr,E,En=void> {
  static void insert(std::basic_ostream<Ch,Tr>& s, const E& e) { s << e; }
  static void extract(std::baic_istream<Ch,Tr>& s, E& e) {
    if constexpr (std::is_same_v<En,void>) { s >> e; if (!s.eof()) s >> std::ws; }
    else { s.unsetf(std::ios_base::skipws); s >> e; } }
};
struct detail::is_inexact<F> { using type = /*!std::numeric_limits<T>::is_exact*/; static constexpr bool value = type::value; };
struct customize_stream<Ch,Tr,F,enable_if<is_inexact<F>>::type> {
  static void insert(std::basic_ostream<Ch,Tr>& s, const F& e)
  { s.precision(std::numeric_limits<F>::max_digits10); s << e; }
  static void extract(std::baic_istream<Ch,Tr>& s, F& e) { s >> e; if (!s.eof()) s >> std::ws; }
};
struct customize_stream<Ch,Tr,bool> {
  static void insert(std::basic_ostream<Ch,Tr>& s, bool e) { s.setf(boolalpha); s << e; }
  static void extract(std::baic_istream<Ch,Tr>& s, bool& e) {
    s >> e;
    if (s.fail()) { s.clear(); s.setf(boolalpha); s >> e; }
    if (!s.eof()) s >> std::ws; }
};
struct customize_stream<Ch,Tr,signed char> {
  static void insert(std::basic_ostream<Ch,Tr>& s, signed char e) { s << (int)e; }
  static void extract(std::baic_istream<Ch,Tr>& s, signed char& e) {
    int i; s >> i;
    if (i > std::numeric_limits<signed char>::max() || i < min()) { s.clear(); e=0; s.setstate(badbit); return; }
    e = (signed char)i; if (!s.eof()) s >> std::ws; }
};
struct customize_stream<Ch,Tr,unsigned char> {
  static void insert(std::basic_ostream<Ch,Tr>& s, unsigned char e) { s << (unsigned)e; }
  static void extract(std::baic_istream<Ch,Tr>& s, unsigned char& e) {
    unsigned i; s >> i;
    if (i > std::numeric_limits<unsigned char>::max()) { s.clear(); e=0; s.setstate(badbit); return; }
    e = (unsigned char)i; if (!s.eof()) s >> std::ws; }
};

class stream_translator<Ch,Tr,A,E> { using customized = customize_stream<Ch,Tr,E>;
  std::locale m_loc;
public: using internal_type = std::basic_string<Ch,Tr,A>; using external_type = E;
  explicit ctor(std::locale loc={}) : m_loc{loc}{}
  optional<E> get_value(const internal_type& v) {
    std::basic_istringstream<Ch,Tr,A> iss{v}; iss.imbue(m_loc); E e; customized::extract(iss, e);
    if (iss.fail() || iss.bad() || iss.get() != Tr::eof()) return {}; else return e; }
  optional<internal_type> put_value(const E& v) {
    std::basic_ostringstream<Ch,Tr,A> oss; oss.imbue(m_loc); customized::insert(oss, v);
    if (oss) return oss.str(); else return {}; }
};
struct translator_between<std::basic_string<Ch,Tr,A>,E> { using type = stream_translator<Ch,Tr,A,E>; };
```

detail/: file_parser_error, info_parser_{error,read,utils,write,writer_settings},
	ptree_implementation, rapidxml, xml_parser_{error,flags,read_rapidxml,utils,write,writer_settings}
json_parser/: error
	detail/: narrow_encoding, parser, read, standard_callbacks, wide_encoding, write
info_parser, ini_parser, json_parser, ptree_serialization, ptree, xml_parser

------
### Dependency

#### Boost.Any

* `<boost/any.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Bind

* `<boost/bind/bind.hpp>`
* `<boost/bind/placeholders.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/limits.hpp>`

#### Boost.Core

* `<boost/core/enable_if.hpp>`
* `<boost/core/invoke_swap.hpp>`
* `<boost/core/ref.hpp>`
* `<boost/core/type_name.hpp>`

#### Boost.Iterator

* `<boost/next_prior.hpp>`

#### Boost.MPL

* `<boost/mpl/**/*.hpp>`

#### Boost.MultiIndex

* `<boost/multi_index/**/*.hpp>`
* `<boost/multi_index_countainer.hpp>`

#### Boost.Optional

* `<boost/optional/optional.hpp>`, `<boost/optional/optional_fwd.hpp>`
* `<boost/optional/optional_io.hpp>`

#### Boost.Range

* `<boost/range/iterator_range_core.hpp>`

#### Boost.Serialization

* `<boost/serialization/**/*.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**/*.hpp>`

------
### Standard Facilities
