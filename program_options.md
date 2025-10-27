# Boost.ProgramOptions

* lib: `boost/libs/program_options`
* repo: `boostorg/program_options`
* commit: `902aaed`, 2025-09-13

------
### API

------
#### Errors

```c++
std::string strip_prefixes(const std::string& text)
{ auto i = text.find_first_not_of("-/"); return i==npos ? text : text.substr(i); }
struct error : std::logic_error { ctor(const std::string&); };
struct too_many_positional_options_error : error {}; // too many positional options have been specified on the command line
struct invalid_command_line_style : error { ctor(const std::string&); };
struct reading_file : error { ctor(const char* filename); }; // can not read options configuration file '%filename%'
struct error_with_option_name : error {
protected: int m_option_style;
  using string_pair = std::pair<std::string, std::string>;
  std::map<std::string, std::string> m_substitutions;
  std::map<std::string, string_pair> m_substitution_defaults;
  mutable std::string m_message;

  virtual void substitute_placeholders(const std::string& error_template) const;
  void replace_token(const std::string& from, const std::string& to) const;
  std::string get_canonical_option_{name|prefix}() const;
public: std::string m_error_template;
  ctor(std::string const& template_, std::string const& option_name="", std::string const& original_token="", int option_style=0);
  void set_substitute(std::string const& parameter_name, std::string const& value) { m_substitutions[parameter_name] = value; }
  void set_substitute_default(std::string const& parameter_name, std::string const& from, std::string const& to)
  { m_substitution_defaults[parameter_name] = std::make_pair(from, to); }
  void add_context(const std::string& option_name, const std::string& original_token, int option_style)
  { set_option_name(option_name); set_original_token(original_token); set_prefix(option_style); }
  void set_prefix(int option_style) { m_option_style = option_style; }
  virtual void set_option_name(const std::string& option_name) { set_substitute("option", option_name); }
  std::string get_option_name() const { return get_canonical_option_name(); }
  void set_original_token(const std::string& original_token) { set_substitute("original_token", original_token); }
  virtual const char* what() const noexcept override;
};
struct multiple_views : error_with_option_name { }; // option '%canonical_option%' only takes a single argument
struct multiple_occurrences : error_with_option_name {}; // option '%canonical_option%' cannot be specified more than once
struct required_option : error_with_option_name {}; // the option '%canonical_option%' is required but missing
struct error_with_no_option_name : error_with_option_name {
  ctor(const std::string& template_, const std::string& original_token="");
  virtual void set_option_name(const std::string&) override{}
};
struct unknown_option : error_with_no_option_name {}; // unrecognized option '%canonical_option%'
struct ambiguous_option : error_with_no_option_name { // option '%canonical_option%' is ambiguous
  ctor(const std::vector<std::string>& xalternatives);
  const std::vector<std::string>& alternatives() const noexcept;
};
struct invalid_syntax : error_with_option_name {
protected: std::string get_template(kind_t kind);
  kind_t m_kind;
public: enum kind_t { long_not_allowed=30, long_adjacent_not_allowed, short_adjacent_not_allowed,
                      empty_adjacent_parameter, missing_parameter, extra_parameter, unrecognized_line };
  ctor(kind_t kind, const std::string& option_name="", const std::string& original_token="", int option_style=0);
  kind_t kind() const { return m_kind; }
  virtual std::string tokens() const { return get_option_name(); }
};
struct invalid_config_file_syntax : invalid_syntax {
  ctor(const std::string& invalid_line, kind_t kind);
  virtual std::string tokens() const override { return m_substitutions.find("invalid_line")->second; }
};
struct invalid_command_line_syntax : invalid_syntax {
  ctor(kind_t kind, const std::string& option_name="", const std::string& original_token="", int option_style=0);
};
struct validation_error : error_with_option_name {
protected: std::string get_template(kind_t kind);
  kind_t m_kind;
public: enum kind_t { multiple_values_not_allowed=30, at_least_one_value_required,
                      invalid_bool_value, invalid_option_value, invalid_option };
  ctor(kind_t kind, const std::string& option_name="", const std::string& original_token="", int option_style=0);
  kind_t kind() const { return m_kind; }
};
struct invalid_option_value : validation_error { ctor(const std::<w>string&); };
struct invalid_bool_value : validation_error { ctor(const std::string&); };
```

------
#### Value Semantic Builders & Variables

```c++
struct value_semantic { virtual ~dtor() {}
  virtual std::string name() const = 0;
  virtual unsigned {min|max}_tokens() const = 0;
  virtual bool is_composing() const = 0;
  virtual bool is_required() const = 0;
  virtual void parse(any& value_store, const std::vector<std::string>& new_tokens, bool utf8) const = 0;
  virtual bool apply_default(any& value_store) const = 0;
  virtual void notify(const any& value_store) const = 0;
};
class value_semantic_codecvt_helper<{char|wchar_t}> : public value_semantic {
  void parse(any& value_store, const std::vector<std::string>& new_tokens, bool utf8) const override;
protected: virtual void xparse(any& value_store, const std::vector<std::string>& new_tokens) const = 0;
};
class untyped_value : public value_semantic_codecvt_helper<char> {
  bool m_zero_tokens;
public: ctor(bool zero_tokens=false) : m_zero_tokens{zero_tokens} {}
  std::string name() const override;
  unsigned {min|max}_tokens() const override;
  bool is_composing() const override { return false; }
  bool is_required() const override { return false; }
  void xparse(any& value_store, const std::vector<std::string>& new_tokens) const override;
  bool apply_default(any&) const override { return false; }
  void notify(const any&) const override {}
};
struct typed_value_base { virtual ~dtor() {}
  virtual const std::type_info& value_type() const = 0;
};
class typed_value<T,Ch=char> : public value_semantic_codecvt_helper<Ch>, public typed_value_base {
  T* m_store_to;
  std::string m_value_name, m_default_value_as_text, m_implicit_value_as_text;
  any m_default_value, m_implicit_value;
  bool m_composing{false}, m_implicit{false}, m_multitoken{false}, m_zero_tokens{false}, m_required{false};
  function1<void, const T&> m_notifier;

public: ctor(T* store_to) : m_store_to(store_t) {}
  typed_value* default_value(const T& v, const std::string& textual= lexical_cast<std::string>(v))
  { m_default_value = any(v); m_default_value_as_text = textual; return this; }
  typed_value* implicit_value(const T& v, const std::string& textual= lexical_cast<std::string>(v))
  { m_implicit_value = any(v); m_implicit_value_as_text = textual; return this; }
  typed_value* value_name(const std::string& name) { m_value_name = name; return this; }
  typed_value* notifier(function1<void, const T&> f) { m_notifier = f; return this; }
  typed_value* composing() { m_composing = true; return this; }
  typed_value* multitoken() { m_multitoken = true; return this; }
  typed_value* zero_tokens() { m_zero_tokens = true; return this; }
  typed_value* required() { m_required = true; return this; }

  std::string name() const override;
  bool is_composing() const override { return m_composing; }
  unsigned min_tokens() const override { if (m_zero_tokens || !m_implicit_value.empty()) return 0; else return 1; }
  unsigned max_tokens() const override { if (m_multitoken) return std::numeric_limits<unsigned>::max(); return m_zero_tokens ? 0 : 1; }
  bool is_required() const override { return m_required; }
  void xparse(any& value_store, const std::vector<std::basic_string<Ch>>& new_tokens) const override;
  virtual bool apply_default(any& value_store) const override
  { if (m_default_value.empty()) return false; value_store = m_default_value; return true; }
  void notify(const any& value_store) const override;
  const std::type_info& value_type() const override { return typeid(T); }
};

typed_value<T,{char|wchar_t}>* <w>value<T>(<T* v>);
typed_value<bool>* bool_switch(<bool* v>);

std::string arg;
const std::basic_string<Ch>& validators::get_single_string<Ch>(const std::vector<std::basic_string<Ch>>& v, bool allow_empty=false) {
  if (v.size() > 1) throw_exception(validation_error{multiple_values_not_allowed});
  if (v.size() == 1) v.front();
  else if (!allow_empty) throw_exception(validation_error{at_least_one_value_required});
  static std::basic_string<Ch> empty; return empty;
}
void validate<T,Ch>(any& v, const std::vector<std::basic_string<Ch>>& xs, T* long) {
  check_first_occurrence(v); std::basic_string<Ch> s{get_single_string(xs)};
  try { v = any{lexical_cast<T>(s)}; } catch(const bad_lexical_cast&) { throw_exception(invalid_option_value{x}); }
}
void validate(any& v, const std::vector<std::basic_string<{char|wchar}>>& v, {bool|std::string}*, int);
void validate<T,Ch>(any& v, const std::vector<std::basic_string<Ch>>& s, st::vector<T>*, int) {
  if (v.empty()) v = any{std::vector<T>{}};
  std::vector<T>* tv = any_cast<std::vector<T>>(&v);
  for (unsigned i = 0; i < s.size(); ++i) {
    try { any a; std::vector<std::basic_string<Ch>> cv; cv.push_back(s[i]);
      validate(a, cv, (T*)nullptr, 0); tv->push_back(any_cast<T>(a)); }
    catch(const bad_lexical_cast&) { throw_exception(invalid_option_value{s[i]}); }
  }
}
void validate<T,Ch>(any& v, const std::vector<std::basic_string<Ch>>& s, optional<T>*, int) {
  validators::check_first_occurrence(v); validators::get_single_string(s);
  any a; validate(a, s, (T*)nullptr, 0); v = any{optional<T>{any_cast<T>(a)}}; 
}

void store(const basic_parsed_options<char>& options, variables_map& m, bool utf8=false);
void store(const basic_parsed_options<wchar_t>& options, variables_map& m);
void notify(variables_map& m);
class variable_value {
  any v; bool m_defaulted{false}; shared_ptr<const value_semantic> m_value_semantic;
  void store(const basic_parsed_options<char>& options, variables_map& m, bool);
public: ctor(); ctor(const any& xv, bool xdefaulted) : v{xv}, m_defualted{xdefaulted} {}
  <const> T& as <T> () <const> { return boost::any_cast<T<const>&>(v); }
  bool empty() const { return v.empty(); } bool defaulted() const { return m_defaulted; }
  <const> any& value() <const> { return v; }
};

class abstract_variables_map { const self* m_next;
  virtual const variable_value& get(const std::string& name) const = 0;
public: ctor(); ctor(const self* next); virtual ~dtor(){}
  const variable_value& operator[](const std::string& name) const;
  void next(self* next);
};

class variables_map : public abstract_variables_map, public std::map<std::string, variable_value> {
  std::set<std::string> m_final;
  std::map<std::string, std::string> m_required;

  const variable_value& get(const std::string& name) const override;
  void store(const basic_parsed_options<char>& options, self& xm, bool utf8);
public: ctor(); ctor(const abstract_variables_map* next);
  const variable_value& operator[](const std::string& name) const { return base::operator[](name); }
  void clear(); void notify();
};
```

-----
#### Options Description

```c++
class option_description {
  std::string m_short_name, m_description;
  std::vector<std::string> m_long_names;
  shared_ptr<const value_semantic> m_value_semantic;
  self& set_names(const char* name);
public: ctor(); virtual ~dtor();
  ctor(const char* name, const value_semantic* s, <const char* description>);
  enum match_result { no_match, full_match, approximate_match };
  match_result match(const std::string& option, bool approx, bool long_ignore_case, bool short_ignore_case) const;
  const std::string& key(const std::string& option) const;
  std::string canonical_display_name(int canonical_option_style=0) const;
  const std::string& long_name() const;
  const std::pair<const std::string*, size_t> long_names() const;
  const std::string& description() const;
  shared_ptr<const value_semantic> semantic() const;
  std::string format_name() const;
  std::string format_parameter() const;
};
class options_description_easy_init { options_description* owner;
public: ctor(options_description* owner);
  self& operator(const char* name, const char* description);
  self& operator(const char* name, const value_semantic* s, <const char* description>);
};
class options_description {
  using name2index_iterator = std::map<std::string, int>::const_iterator;
  using approximation_range = std::pair<name2index_iterator, name2index_iterator>;
  std::string m_caption; const unsigned m_line_length, m_min_description_length;
  std::vector<shared_ptr<option_description>> m_options;
  std::vector<char> belong_to_group; std::vector<shared_ptr<self>> groups;
public: static const unsigned m_default_line_length;
  ctor(<const std::string& caption>, unsigned line_length=m_default_line_length, unsigned min_description_length=m_default_line_length/2);
  void add(shared_ptr<option_description> desc);
  self& add(const self& desc);
  unsigned get_option_column_width() const;
  options_description_easy_init add_options();
  const option_description& find(const std::string& name, bool approx, bool long_ignore_case=false, bool short_ignore_case=false) const;
  const option_description* find_nothrow(const std::string& name, bool approx, bool long_ignore_case=false, bool short_ignore_case=false) const;
  const std::vector<shared_ptr<option_description>>& options() const;
  void print(std::ostream& os, unsigned width=0) const;
  friend std::ostream& operator<<(std::ostream& os, const self& desc);
};
struct duplicate_option_error : error {};
class positional_options_description {
  std::vector<std::string> m_names; std::string m_trailing;
public: ctor();
  self& add(const char* name, int max_count);
  unsigned max_total_count() const;
  const std::string& name_for_position(unsigned position) const;
};
```

------
#### Command-line Parsers

```c++
enum command_line_style::style_t { allow_long = 1, allow_short = 2, allow_dash_for_short = 4, allow_slash_for_short = 8,
  long_allow_adjacent = 16, long_allow_next = 32, short_allow_adjacent = 64, short_allow_next = 128,
  allow_sticky = 256, allow_guessing = 512, long_case_insensitive = 1024, short_case_insensitive = 2048, allow_long_disguise = 4096,
  case_insensitive = long_case_insensitive|short_case_insensitive,
  unix_style = allow_short|short_allow_adjacent|short_allow_next|allow_long|long_allow_adjacent|long_allow_next|allow_sticky|allow_guessing|allow_dash_for_short,
  default_style = unix_style };

struct basic_option<Ch> {
  std::string string_key; int position_key;
  std::vector<std::basic_string<Ch>> value, original_tokens;
  bool unregistered, case_insensitive;
  ctor(); ctor(const std::string& xstring_key, const std::vector<std::string>& xvalue) : string_key{xstring_key}, value{xvalue} {};
};

class detail::cmdline {
public: using additional_parser = function1<std::pair<std::string, std::string>, const std::string&>;
  using style_parser = function1<std::vector<option>, std::vector<std::string>&>;
  std::vector<std::string> m_args;
  style_t m_style;
  bool m_allow_unregistered;
  const options_description* m_desc;
  const positional_options_description* m_positional;
  additional_parser m_additional_parser;
  style_parser m_style_parser;

  ctor(const std::vector<std::string>& args); ctor(int argc, const char* const* argv);
  void style(int style);
  int get_canonical_option_prefix();
  void allow_unregistered();
  void set_options_description(const options_description& desc);
  void set_positional_options(const positional_options_description& positional);

  std::vector<option> run();
  std::vector<option> parse_{long|short|dos|disguised_long}_option(std::vector<std::string>& args);
  std::vector<option> parse_terminator(std::vector<std::string>& args);
  std::vector<option> handle_additional_parser(std::vector<std::string>& args);
  void set_additional_parser(additional_parser p);
  void extra_style_parser(style_parser s);
  void check_style(int style) const;
  bool is_style_active(style_t style) const;
  void init(const std::vector<std::string>& args);
  void finish_option(option& opt, std::vector<std::string>& other_tokens, const std::vector<style_parser>& style_parsers);

  void test_cmdline_detail();
};

struct basic_parsed_options<Ch> {
  explicit ctor(const options_description* xdescription, int options_prefix=0);
  std::vector<basic_option<Ch>> options;
  const options_description* description;
  int m_options_prefix;
};
struct basic_parsed_options<wchar_t> {
  explicit ctor(const self<char>& po);
  std::vector<basic_option<Ch>> options;
  const options_description* description;
  basic_parsed_options<char> utf8_encoded_options;
  int m_options_prefix;
};
using <w>parsed_options = basic_parsed_options<{char|wchar_t}>;

using ext_parser = function1<std::pair<std::string, std::string>, const std::string&>;
class basic_command_line_parser<Ch> : cmdline {
  const options_description* m_desc;
public: ctor(const std::vector<std::basic_string<Ch>>& args);
  ctor(int argc, const charT* const argv[]);
  self& options(const options_description& desc);
  self& positional(const positional_options_description& desc);
  self& style(int);
  self& extra_parser(ext_parser);
  basic_parsed_options<Ch> run();
  self& allow_unregistered();
  self& extra_style_parser(cmdline::style_parser s);
};

basic_parsed_options<Ch> parse_command_line<Ch>(int argc, const Ch* const argv[],
  const options_description&, int style=0, ext_parser ext={});
basic_parsed_options<Ch> parse_config_file<Ch>(std::basic_istream<Ch>& const options_description&, bool allow_unregistered=false);
basic_parsed_options<Ch> parse_config_file<Ch=char>(const char* filename, const options_description&, bool allow_unregistered=false)

enum collect_unrecognized_mode { include_positional, exclude_positional };
std::vector<std::basic_string<Ch>> collect_unrecognized(const std::vector<basic_option<Ch>>& options, collect_unrecognized_mode mode);

parsed_options parse_environment(const options_description& const function1<std::string, std::string>& name_mapper);
parsed_options parse_environment(const options_description& const {std::string&|char*} prefix);

std::vector<std::<w>string> split_unix(const std::<w>string& cmdline,
  const std::<w>string& separator=" \t", const std::<w>string& quote="'\"", const std::<w>string& escape="\\");
std::vector<std::<w>string> split_winmain(const std::<w>string& cmdline);
```

------
### Dependency

#### Boost.Any

* `<boost/any.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`
* `<boost/version.hpp>`
* `<boost/config/auto_link.hpp>`

#### Boost.Core

* `<boost/noncopyable.hpp>`

#### Boost.Detail

* `<boost/detail/utf8_codecvt_facet.hpp>`

#### Boost.Function

* `<boost/function.hpp>`, `<boost/function/function1.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_facade.hpp>`

#### Boost.LexicalCast

* `<boost/lexical_cast.hpp>`

#### Boost.SmartPtr

* `<boost/shared_ptr.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

------
### Standard Facilities
