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

class file_parser_error : public ptree_error { // f"{file|<unspecified file>} ({l}): {msg}"
  std::string m_message, m_filename; unsigned long m_line;
public: ctor(const std::string& msg, const std::string& file, unsigned long l);
  std::string message() const; std::string filename() const; unsigned long line() const;
};

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

------
### Property Tree

* Header `<boost/property_tree/ptree.hpp>`

```c++
class basic_ptree<Key,Data,KeyComp> {
  struct stubs {
    struct by_name{};
    using base_container = multi_index_container<value_type, index_by<sequenced<>, ordered_non_unique<tag<by_name>, member<value_type, const key_type, &value_type::first>, key_compare>>>;
    using by_name_index = base_container::index<by_name>::type;
    static base_container <const>& ch(self <const>* s) { return *(base_container*)s->m_children; }
    static by_name_index <const>& assoc(self <const>* s) { return ch(s).get<by_name>(); }
  };
  data_type m_data; void* m_children{new subs::base_container};
  self* walk_path(path_type& p) const {
    if (p.empty()) return (self*)this;
    const_assoc_iterator el = find(p.reduce());
    if (el == not_found()) return nullptr; else return el->second.walk_path(p);
  }
  self& force_path(path_type& p) {
    if (p.single()) return *this;
    key_type frag = p.reduce(); assoc_iterator el = find(frag);
    self& child = el == not_found() ? push_back(value_type{frag, self{}})->second : el->second;
    return child.force_path(p);
  }
public: using key_type = Key; using data_type = Data; using key_compare = KeyComp;
  using value_type = std::pair<const Key, self>; using size_type = size_t; using path_type = path_of<Key>::type;
  struct iterator : iterator_adaptor<self, subs:base_container::iterator, value_type> {
    ctor(); explicit ctor(base_type b);
    reference dereference() const { return (reference)(*base_reference()); }
  };
  struct const_iterator : iterator_adaptor<self, subs::base_container::const_iterator>
  { ctor(); explicit ctor(base_type b); ctor(iterator b); };
  struct <const>_reverse_iterator : reverse_iterator<<const>_iterator> {};
  struct assoc_iterator : iterator_adaptor<self, subs::by_name_index::iterator, value_type> {
    ctor(); explicit ctor(base_type b);
    reference dereference() const { return (reference)(*base_reference()); }
  };
  class const_assoc_iterator : iterator_adaptor<self, subs::by_name_index::const_iterator>
  { ctor(); explicit ctor(base_type b); ctor(assoc_iterator b); };

  ctor(); explicit ctor(const data_type& d); ctor(const self& r); self& operator=(const self& r);
  ~dtor() { delete &subs::ch(this); }
  void swap(self& r);

  size_type size() const; size_type max_size() const; bool empty() const;
  <const>_iterator begin()<const>; <const>_iterator end()<const>;
  <const>_reverse_iterator rbegin()<const>; <const>_reverse_iterator rend()<const>;
  iterator insert(iterator where, const value_type& v);
  void insert<It>(iterator where, It first, It last);
  iterator erase(iterator p); iterator erase(iterator first, iterator last);
  iterator push_front(const value_type &v); iterator push_back(const value_type& v);
  void pop_front(); void pop_back();
  void reverse();
  void sort<Comp>(Comp comp); void sort();
  bool operator==(const self& r) const; bool operator!=(const self& r) const;

  <const>_assoc_iterator ordered_begin() <const>;
  <const>_assoc_iterator not_found() <const>;
  <const>_assoc_iterator find(const key_type& k) <const>;
  std::pair<<const>_assoc_iterator,<const>_assoc_iterator> equal_range(const key_type& k) <const>;
  size_type count(const key_type& k) const;
  size_type erase(const key_type& k);
  <const>_iterator to_iterator(<const>_assoc_iterator it) <const>;
  <const> data_type& data() <const>;
  void clear();

  <const> self& get_child(const path_type& path, <const self& default_value>) <const>;
  void get_child(const path_type& path, const self&& default_value) = delete;
  optional<<const> self&> get_child_optional(const self& path) <const>;
  self& put_child(const path_type& path, const self_type& value);
  self& add_child(const path_type& path, const self_type& value);
  Type get_value<Type,[Trans]>(<const Type& default_value>, <Trans tr>) const; // distinguish Type/Translator by is_translator
  std::basic_string<Ch> get_value<Ch,[Trans]>(const Ch* default_value, <Trans tr>) const requires is_character<Ch>::value;
  optional<Type> get_value_optional<Type,[Trans]>(<Trans tr>) const;
  void put_value<Type,[Trans]>(const Type& value, <Trans tr>);
  Type get<Type,[Trans]>(const path_type& path, <const Type& default_value>, <Trans tr>) const;
  std::basic_string<Ch> get<Ch,[Trans]>(const path_type& path, const Ch* default_value, <Trans tr>) const requires is_character<Ch>::value;
  optional<Type> get_optional<Type,[Trans]>(const path_type& path, <Trans tr>) const;
  self& put<Type,[Trans]>(const path_type& path, const Type& value, <Trans tr>);
  self& add<Type,[Trans]>(const path_type& path, const Type& value, <Trans tr>);
};
```

------
### Serialization of `ptree`

* Header `<boost/property_tree/ptree_serialization.hpp>`

```c++
namespace bsl = boost::serialization;
void save<Archive,K,D,C>(Archive &ar, const basic_ptree<K,D,C>& t, unsigned) {
  using bsl; stl::save_collection(ar, t); ar << make_nvp("data", t.data());
}
void detail::load_children<Archive,K,D,C>(Archive& ar, basic_ptree<K,D,C>& t) {
  using tree = basic_ptree<K,D,C>; using value_type = tree::value_type;
  bsl::collection_size_type count; ar >> NVP(count);
  bsl::item_version_type item_version{0}; const auto library_version{ar.get_library_version()};
  if (3 < library_version) ar >> NVP(item_version);
  t.clear();
  while (count-->0) {
    bsl::detail::stack_construct u<Archive,value_type>{ar, item_version};
    ar >> bsl::make_nvp("item", u.reference());
    t.push_back(u.reference());
    ar.reset_object_address(&t.back(), &u.reference());
  }
}
void load<Archive,K,D,C>(Archive& ar, basic_ptree<K,D,C>& t, unsigned) {
  load_children(ar, t); ar >> bsl::make_nvp("data", t.data());
}
void serialize<Archive,K,D,C>(Archive& ar, basic_ptree<K,D,C>& t, unsigned file_version) {
  using namespace bsl; split_free(ar, t, file_version);
}
```

------
### INFO file parser

#### INFO file format

```
key1 value1  ; comment
key2 "special: {};#\n\t\"\0"
{ ; allow both value and children
  subkey "multi " \
        "lines"
  no_value ""
  "special key \n{}" value
  "" "" ; empty key, empty value
}
#include "file.info" ; inclusion
```

#### Parser

* Header `<boost/property_tree/info_parser.hpp>`

```c++
namespace boost::property_tree::info_parser;

struct info_parser_error : file_parser_error {};
struct info_writer_settings<Ch> { Ch indent_char; int indent_count; ctor(Ch=Ch(' '), unsigned=4); };
info_writer_settings<Ch> info_writer_make_settings(Ch=Ch(' '), unsigned indent_count=4);

std::basic_string<ChDest> convert_chtype<ChDest,ChSrc>(const ChSrc* text);
std::basic_string<Ch> expand_escapes<It>(It b, It e);
bool is_ascii_space<Ch>(Ch c);
void skip_whitespace<Ch>(const Ch*& text);
std::basic_string<Ch> read_word<Ch>(const Ch*& text);
std::basic_string<Ch> read_line<Ch>(const Ch*& text);
std::basic_string<Ch> read_string<Ch>(const Ch*& text, bool* need_more_lines);
std::basic_string<Ch> read_key<Ch>(const Ch*& text);
std::basic_string<Ch> read_data<Ch>(const Ch*& text, bool* need_more_lines);
void read_info_internal<Ptree,Ch>(std::basic_istream<Ch>& stream, Ptree& pt, const std::string& filename, int include_depth);

void write_info_indent<Ch>(std::basic_ostream<Ch>& stream, int indent, cnost info_writer_settings<Ch>& settings);
std::basic_string<Ch> create_escapes(const std::basic_string<Ch>& s);
bool is_simple_key<Ch>(const std::basic_string<Ch>& key);
bool is_simple_data<Ch>(const std::basic_string<Ch>& key);
void write_info_helpers<Ptree>(std::basic_ostream<Ch>& stream, const Ptree& pt, int indent, const info_writer_settings<Ch>& settings);
void write_info_internal<Ptree>(std::basic_ostream<Ch>& stream, const Ptree& pt, const std::string& filename, const info_writer_settings<Ch>& settings);

void read_info<Ptree,Ch>(std::basic_istream<Ch>& stream, Ptree& pt);
void read_info<Ptree,Ch>(std::basic_istream<Ch>& stream, Ptree& pt, const Ptree& default_ptree);
void read_info<Ptree>(const std::string& filename, Ptree& pt, const std::locale& loc={});
void read_info<Ptree>(const std::string& filename, Ptree& pt, const Ptree& default_ptree, const std::locale& loc={});

void write_info<Ptree,Ch>(std::basic_ostream<Ch>& stream, const Ptree& pt, const info_writer_settings<Ch>& setting={});
void write_info<Ptree>(const std::string& filename, const Ptree& pt, const std::locale& loc={}, const info_writer_settings<Ch>& setting={});

using info_parser::info_parser_error;
using info_parser::info_writer_settings; using info_parser::info_writer_make_settings;
using info_parser::read_info; using info_parser::write_info;
```

------
### INI file parser

* Header `<boost/property_tree/ini_parser.hpp>`

```c++
namespace boost::property_tree::ini_parser;
bool validate_flags(int flags) { return flags == 0; } // no flags supported
struct ini_parser_error : file_parser_error {};

void read_ini<Ptree>(std::basic_istream<Ch>& stream, Ptree& pt);
void read_ini<Ptree>(const std::string& filename, Ptree& pt, const std::locale& loc={});

void detail::check_dupes<Ptree>(const Pree& pt);
void detail::write_keys<Ptree>(std::basic_ostream<Ch>& stream, const Ptree& pt, bool throw_on_children);
void detail::write_top_level_keys<Ptree>(std::basic_ostream<Ch>& stream, const Ptree& pt) { write_keys(stream, pt, false); }
void detail::write_sections<Ptree>(std::basic_ostream<Ch>& stream, const Ptree& pt)

void write_ini<Ptree>(std::basic_ostream<Ch>& stream, const Ptree& pt, int flags=0);
void write_ini<Ptree>(const std::string& filename, const Ptree& pt, int flags=0, const std::locale& loc={});

using ini_parser::ini_parser_error;
using ini_parser::read_ini; using ini_parser::write_ini;
```

------
### JSON file parser

* Header `<boost/property_tree/json_parser.hpp>`

```c++
namespace boost::property_tree::json_parser;
struct json_parser_error : file_parser_error {};

class detail::source<Enc,It,S> {
  struct DoNothing{ void operator()(code_unit) const{} };
  Enc& encoding; It cur; S end; std::string filename; int line, offset;
public: using code_unit = std::iterator_traits<It>::value_type; using encoding_predicate = bool(Enc::*)(code_unit c) const;
  explicit ctor(EncodinEnc& encoding) : encoding{encoding}{}
  void set_input<Range>(const std::string& filename, const Range& r)
  { this->filename=filename; cur=r.begin(); end=r.end(); encoding.skip_introduction(cur,end); line=1; offset=0; }
  bool done() const { return cur==end; }
  void parse_error(const char* msg) { throw_exception(json_parser_error{msg, filename, line}); }
  void next() { if (encoding.is_nl(*cur)) { ++line; offset=0; } else ++offset; ++cur; }
  bool have<Action>(encoding_predicate p, Action& a)
  { bool found = cur!=end && (encoding.*p)(*cur); if(found) { a(*cur); next(); } return found; }
  bool have(encoding_predicate p) { return have(p, DoNothing{}); }
  void expect<Action>(encoding_predicate p, const char* msg, Action& a) { if (!have(p,a)) parse_error(msg); }
  void expect(encoding_predicate p, const char* msg) { expect(p, msg, DoNothing{}); }
  code_unit need_cur(const char* msg) { if (cur==end) parser_error(msg); return *cur; }
  It& raw_cur() { return cur; } S raw_end() { return end; }
};

class detail::number_callback_adapter<Cb,Enc,It,_=std::iterator_traits<It>::iterator_category> { // no copy
  Cb& callbacks; Enc& encodings; It first; It& cur;
public: ctor(Cb& callbacks, Enc& encoding, It& cur);
  void operator()(Enc::external_char){}
  void finish() const { callbacks.on_number(encoding.to_internal(first, cur)); }
};
class detail::number_callback_adapter<Cb,Enc,It,std::input_iterator_tag> { // no copy
  Cb& callbacks; Enc& encodings; bool first{true};
public: ctor(Cb& callbacks, Enc& encoding, It&);
  void operator()(Enc::external_char c){
    if (first) {callbacks.on_begin_number(); first=false;}
    callbacks.on_digit(encoding.to_internal_trivial(c));
  }
  void finish() const { callbacks.on_end_number(); }
};
class detail::string_callback_adapter<Cb,Enc,It,_=std::iterator_traits<It>::iterator_category> { // no copy
  Cb& callbacks; Enc& encodings; It& cur; It run_begin;
public: ctor(Cb& callbacks, Enc& encoding, It& cur);
  void start_run(){run_begin=cur;}
  void finish_run() { callbacks.on_code_units(encoding.to_internal(run_begin, cur)); }
  void process_codepoint<S,EncErrFn>(S end, EncErrFn error_fn) { encoding.skip_codepoint(cur, end, error_fn); }
};
class detail::string_callback_adapter<Cb,Enc,It,std::input_iterator_tag> { // no copy
  Cb& callbacks; Enc& encodings; It& cur;
public: ctor(Cb& callbacks, Enc& encoding, It& cur);
  void start_run(){} void finish_run(){}
  void process_codepoint<S,EncErrFn>(S end, EncErrFn error_fn)
  { encoding.transcode_codepoint(cur, end, bind(&Cb::on_code_unit, ref(callbacks), _1), error_fn); }
};

class detail::parser<Cb,Enc,It,S> {
  using number_adapter = number_adapter<Cb,Enc,It>; using string_adapter = string_callback_adapter<Cb,Enc,It>;
  using source = source<Enc,It,S>; using code_unit = source::code_unit; using enc_pred = source::encoding_predicate;
  Callbacks& callbacks; Enc& encodings; source src;
  // forward `parse_error`, `next`, `have`, `expect`, `need_cur` to `src`
  void skip_ws() { while (have(&Enc::is_ws)); }
  bool parse_int_part(number_adapter& action)
  { if (!have(&Enc::is_digit0, action)) return false; parse_digits(action); return true; }
  void parse_frac_part(number_adapter& action)
  { if (!have(&Enc::is_dot, action)) return; expect(&Enc::is_digit, "...", action); parse_digits(action); }
  void parse_exp_part(number_adapter& action)
  { if (!have(&Enc::is_eE, action)) return; have(&Enc::is_plusnimus, action); expect(&Enc::is_digit, "...", action); parse_digits(action); }
  void parse_digits(number_adapter& action) { while (have(&Enc::is_digit, action)); }
  void parse_escape() {
    if (have(&Enc::is_quote)) feed(0x22); else if (have(&Enc::XX)) feed(0xAA); // all escapes
    else if (have(&Enc::is_u)) parse_codepoint_ref(); else parse_error("..."); }
  unsigned parse_hex_quad() { unsigned cp = 0;
    for (int i=0; i<4; ++i) { int v = encoding.decode_hexdigit(need_cur("..."));
      if (v<0) parse_error("..."); cp*=16; cp+=v; next(); }
    return cp; }
  static bool is_surrogate_high(unsigned cp) { return (cp&0xfc00)==0xd800; }
  static bool is_surrogate_low(unsigned cp) { return (cp&0xfc00)==0xdc00; }
  static unsigned combine_surrogates(unsigned high, unsigned low) { return 0x010000+((high&0x3ff)<<10) | (low&0x3ff); }
  void parse_codepoint_ref() { unsigned cp = parse_hex_quad();
    if (is_surrogate_low(cp)) parse_error("...");
    if (is_surrogate_high(cp)) { expect(&Enc::is_backslash, "..."); expect(&Enc::is_u, "...");
      int low=parse_hex_quad(); if (!is_surrogate_low(low)) parse_error("...");
      cp = combine_surrogates(cp, low); }
    feed(cp); }
  void feed(unsigned cp) { encoding.feed_codepoint(cp, bind(&Cb::on_code_unit, ref(callbacks), _1)); }
public: ctor(Cb& callbacks, Enc& encodings);
  void set_input<Range>(const std::string& filename, const Range& r) { src.set_input(filename, r); }
  void finish() { skip_ws(); if (!src.done()) parse_error("..."); }
  void parse_value() { if (!parse_object() || !parse_array() || !parse_string() || !parse_boolean() || !parse_null() || !parse_number()) parse_error("..."); }
  bool parse_null() { skip_ws(); if (!have(&Enc::is_n)) return false; 'ull'; callbacks.on_null(); return true; }
  bool parse_boolean() { skip_ws(); if (have(&Enc::is_t)) {'rue'; callbacks.on_boolean(true); return true; }
    if (have(&Enc::is_f)) {'alse'; callbacks.on_boolean(false); return true; } return false; }
  bool parse_number() { skip_ws();
    number_adapter adapter{callbacks, encoding, src.raw_cur()};
    bool started{false}; if (have(&Enc::is_minus, adapter)) started = true;
    if (!have(&Enc::is_0, adapter) && !parse_int_part(adapter)) { if (started) parse_error(".."); return false; }
    parse_frac_part(adapter); parse_exp_part(adapter); adapter.finish(); return true; }
  bool parse_string() { skip_ws(); if (!have(&Enc::is_quote)) return false;
    callbacks.on_begin_string(); string_adapter adapter{callbacks, encoding, src.raw_cur()};
    while (!encoding.is_quote(need_cur("..."))) {
      if (encoding.is_backslash(*src.raw_cur())) { adapter.finish_run(); next(); parse_escape(); adapter.start_run(); }
      else { adapter.process_codepoint(src.raw_end(), bind(&parser.parse_error, this, "...")); } }
    adapter.finish_run(); callbacks.on_end_string(); next(); return true; }
  bool parse_array() { skip_ws(); if (!have(&Enc.is_open_bracket)) return false;
    callbacks.on_begin_array(); skip_ws(); if (have(&Enc::is_close_bracket)) { callbacks.on_end_array(); return true; }
    do { parse_value(); skip_ws(); } while (have(&Enc::is_comma));
    expect(&Enc::is_close_bracket, "..."); callbacks.on_end_array(); return true; }
  bool parse_object() { skip_ws(); if (!have(&Enc.is_open_brace)) return false;
    callbacks.on_begin_object(); skip_ws(); if (have(&Enc::is_close_brace)) { callbacks.on_end_object(); return true; }
    do { if (!parse_string()) parse_error("...");
      skip_ws(); expect(&Enc::is_colon, "..."); parse_value(); skip_ws();
    } while (have(&Enc::is_comma));
    expect(&Enc::is_close_brace, "..."); callbacks.on_end_object(); return true; }
};

struct detail::external_ascii_superset_encoding {
  using extenral_char = char;
  bool is_nl(char c) const; // ws, minus, plusminus, dot, eE, 0, digit, digit0, quote, backslash, slash, comma, {open|close}_{bracket|brace}, colon, a,b,e,f,l,n,r,s,t,u
  int decode_hexdigit(char c);
};
struct detail::utf8_utf8_encoding : external_ascii_superset_encoding {
  struct DoNothing;
  iterator_range<It> to_internal<It>(It f, It l) const { return make_iterator_range(f,l); }
  char to_internal_trivial(char c) const { return c; }
  void skip_codepoint<It,S,EncErrFn>(It& cur, S end, EncErrFn error_fn) const { transcode_codepoint(cur, end, DoNothing{}, error_fn); }
  void transcode_codepoint<It,S,TranscodedFn,EncErrFn>(It& cur, S end, TranscodedFn transcoded_fn, EncErrFn error_fn) const;
  void feed_codepoint<TranscodedFn>(unsigned cp, TranscodedFn transcoded_fn);
  void skip_introduction<It,S>(It& cur, S end) const;
};
struct detail::external_wide_encoding {
  using extenral_char = wchar_t;
  bool is_nl(wchar_t c) const; // ws, minus, plusminus, dot, eE, 0, digit, digit0, quote, backslash, slash, comma, {open|close}_{bracket|brace}, colon, a,b,e,f,l,n,r,s,t,u
  int decode_hexdigit(wchar_t c);
};
struct detail::wide_wide_encoding : external_wide_encoding {
  using internal_char = wchar_t; struct DoNothing;
  iterator_range<It> to_internal<It>(It f, It l) const { return make_iterator_range(f,l); }
  wchar_t to_internal_trivial(wchar_t c) const { return c; }
  void skip_codepoint<It,S,EncErrFn>(It& cur, S end, EncErrFn error_fn) const { transcode_codepoint(cur, end, DoNothing{}, error_fn); }
  void transcode_codepoint<It,S,TranscodedFn,EncErrFn>(It& cur, S end, TranscodedFn transcoded_fn, EncErrFn error_fn) const;
  void feed_codepoint<TranscodedFn>(unsigned cp, TranscodedFn transcoded_fn);
  void skip_introduction<It,S>(It& cur, S end) const;
};

const Ch* detail::constants::null_value<Ch>(); // "null"
const Ch* detail::constants::true_value<Ch>(); // "true"
const Ch* detail::constants::false_value<Ch>(); // "false"
class detail::standard_callbacks<Ptree> {
  enum kind {array, object, key, leaf}; struct layer {kind k; Ptree* t; };
  Ptree root; string key_buffer; std::vector<layer> stack;
  Ptree& new_tree();
  string& new_value();
protected: bool is_key() const { return stack.back().key == key; }
  string& current_value() { auto l = stack.back(); if (l.k == key) return key_buffer; else return l.t->data(); }
public: using string = Ptree::data_type; using char_type = string::value_type;
  void on_null() { new_value() = constants::null_value<char_type>(); }
  void on_boolean(bool b) { new_value() = b ? constants::true_value<char_type>() : false_value(); }
  void on_number<Range>(Range code_units) { new_value().assign(code_units.begin(), code_units.end()); }
  void on_begin_number(){ new_value(); } void on_digit(char_type d){ current_value()+=d; } void on_end_number(){}
  void on_begin_string(){ new_value(); } void on_code_unit(char_type c) { current_value()+=c; } void on_end_string(){}
  void on_code_units<Range>(Range code_units){ current_value().append(code_units.begin(), code_units.end()); }
  void on_begin_array() { new_tree(); stack.back().k = array; } void on_end_array() { if (stack.back().k == leaf) stack.pop_back(); stack.pop_back(); }
  void on_begin_object() { new_tree(); stack.back().k = object; } void on_end_object() { if (stack.back().k == leaf) stack.pop_back(); stack.pop_back(); }
  Ptree& output() { return root; }
};

class detail::minirange<It,S>{ It first; S last; public: ctor(It f, S l); It begin() const; S end() const; };
minirange<It,S> detail::make_minirange<It,S>(It f, S l);
struct detail::encoding<Ch>; // utf8/wide

void detail::read_json_internal<It,S,Enc,Cb>(It f, S l, Enc& encoding, Cb& callbacks, const std::string& filename);
void detail::read_json_internal<Ptree>(std::basic_istream<Ch>& stream, Ptree& pt, const std::string& filename);
std::basic_string<Ch> create_escapes<Ch>(const std::basic_string<Ch>& s);
void detail::write_json_helper<Ptree>(std::basic_ostream<Ch>& stream, const Ptree& pt, int indent, bool pretty);
bool detail::verify_json<Ptree>(const Ptree& pt, int depth);
void detail::write_json_internal<Ptree>(std::basic_ostream<Ch>& stream, const Ptree& pt, const std::string& filename, bool pretty);

void read_json<Ptree>(std::basic_istream<Ch>& stream, Ptree& pt);
void read_json<Ptree>(const std::string& filename, Ptree& pt, const std::locale& loc={});
void write_json<Ptree>(std::basic_ostream<Ch>& stream, const Ptree& pt, bool pretty=true);
void write_json<Ptree>(const std::string& filename, const Ptree& pt, const std::locale& loc={}, bool pretty=true);

using json_parser::json_parser_error;
using json_parser::read_json; using json_parser::write_json;
```

------
### XML file parser

* Header `<boost/property_tree/xml_parser.hpp>`

```c++
namespace boost::property_tree::xml_parser;

struct xml_parser_error : file_parser_error {};
Str widen<Str>(const char* text);
struct xml_writer_settings<Str> { using Ch = Str::value_type;
  Ch indent_char; Str::size_type indent_count; Str encoding;
  ctor(Ch=Ch(' '), Str::size_type incount=0, const Str& enc=widen<Str>("utf-8"));
};
xml_writer_settings<Str> xml_writer_make_settings<Str>(Ch=Ch(' '), Str::size_type=4, const Str&=widen<Str>("utf-8"));
Str condense<Str>(const Str& s);
Str encode_char_entities<Str>(const Str& s);
Str decode_char_entities<Str>(const Str& s);
const Str& xmldecl<Str>(); // "<?xml>"
const Str& xmlattr<Str>(); // "<xmlattr>"
const Str& xmlcomment<Str>(); // "<xmlcomment>"
const Str& xmltext<Str>(); // "<xmltext>"

void write_xml_indent<Str>(std::basic_ostream<Ch>& stream, int indent, const xml_writer_settings<Str>& settings);
void write_xml_comment<Str>(std::basic_ostream<Ch>& stream, const Str& s, int indent, bool separate_line, const xml_writer_settings<Str>& settings);
void write_xml_text<Str>(std::basic_ostream<Ch>& stream, const Str& s, int indent, bool separate_line, const xml_writer_settings<Str>& settings);
void write_xml_element<Ptree>(std::basic_ostream<Ch>& stream, const Ptree::key_type& key, const Ptree &pt, int indent, bool separate_line, const xml_writer_settings<Ptree::key_type>& settings);
void write_xml_internal<Ptree>(std::basic_ostream<Ch>& stream, const Ptree& pt, const std::string& filename, const xml_writer_settings<Str>& settings);

static const int no_concat_text = 1, no_comments = 2, trim_whitespace = 4;
bool validate_flags(int flags);

void read_xml_node<Ptree,Ch>(xml_node<Ch>* node, Ptree& pt, int flags);
void read_xml_internal<Ptree>(std::basic_istream<Ch>& stream, Ptree& pt, int flags, const std::string& filename);

void read_xml<Ptree>(std::basic_istream<Ch>& stream, Ptree& pt, int flags=0);
void read_xml<Ptree>(const std::string& filename, Ptree& pt, int flags=0, const std::locale& loc={});
void write_xml<Ptree>(std::basic_ostream<Ch>& stream, const Ptree& pt, const xml_writer_settings<Ptree::key_type>& setting={});
void write_xml<Ptree>(const std::string& filename, const Ptree& pt, const std::locale& loc={}, const info_writer_settings<Ptree::key_type>& setting={});

using xml_parser::xml_parser_error;
using xml_parser::xml_writer_settings; using xml_parser::xml_writer_make_settings;
using xml_parser::read_info; using xml_parser::write_xml;
```

#### Embedded RapidXML Library

```c++
namespace boost::property_tree::detail::rapidxml;
class parser_error : public std::exception;
#define STATIC_POOL_SIZE 64*1024
#define DYNAMIC_POOL_SIZE 64*1024
#define ALIGHMENT sizeof(void*)

enum node_type { node_document, node_element, node_data, node_cdata, node_comment, node_declaration, node_doctype, node_pi };
const int parse_no_data_nodes=1, parse_no_element_values=2, parse_no_string_terminators=4, parse_no_entity_translation=8,
  parse_no_utf8=16, parse_declaration_node=32, parse_comment_nodes=64, parse_doctype_node=128, parse_pi_nodes=256,
  parse_validate_closing_tags=512,parse_trim_whitespace=1024,parse_normalize_whitespace=2048, parse_default=0;
const int parse_non_destructive = parse_no_string_terminators | parse_no_entity_translation;
const int parse_fastest = parse_non_destructive | parse_no_data_nodes;
const int parse_full = parse_declaration_node | parse_comment_nodes | parse_doctype_node | parse_pi_nodes | parse_validate_closing_tags;

class memory_pool<Ch=char>;
class xml_base<Ch=char>;
class xml_attribute<Ch=char>: public xml_base<Ch>;
class xml_node<Ch=char>: public xml_base<Ch>;
class xml_document<Ch=char>: public xml_node<Ch>, public memory_pool<Ch>;
```

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
* `<boost/multi_index_container.hpp>`

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
