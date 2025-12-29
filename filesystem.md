# Boost.Filesystem

* lib: `boost/libs/filesystem`
* repo: `boostorg/filesystem`
* commit: `ce7a835`, 2025-08-29

------
#### Common Parts

```c++
// string types and traits
system::error_category const& codecvt_error_category() noexcept;
namespace detail::path_traits {
using path_native_char_type = char; // wchar_t for Windows
#define IS_CHAR_NATIVE true
#define IS_WCHAR_T_NATIVE false
using codecvt_type = std::codecvt<wchar_t,char,mbstate_t>;

struct unknown_type_tag {};
struct ntcts_type_tag {};
struct char_ptr_tag : ntcts_type_tag {}; struct char_array_tag : ntcts_type_tag {};
struct string_class_tag {};
struct std_string_tag : string_class_tag {}; struct boost_container_string_tag : string_class_tag {};
struct std_string_view_tag : string_class_tag {}; struct boost_string_view_tag : string_class_tag {};
struct range_type_tag {};
struct directory_entry_tag {};

struct path_source_traits<T> { using tag_type = unknown_type_tag; using char_type = void; bool is_native = false;};
// spec for <const>char*, <const>wchar_t*, <const> char[<N>], <const> wchar_t[<N>],
// std::<w>string_<view>, boost::container::basic_string_<view>, and directory_entry
struct is_known_path_source_tag<Tag>; // unknown_type_tag is true_type
struct is_path_source<T> : is_known_path_source_tag<path_source_traits<T>::tag_type>::type{};
struct is_native_path_source<T> : bool_constant<path_source_traits<T>::is_native>{};
struct is_path_char_type<T>; // char and wchar_t
struct is_iterator_to_path_chars<It> : is_path_char_type<std::iterator_traits<It>::value_type>::type{};
struct is_path_source_iterator<It> : bool_constant<is_iterator<It>::value && is_iterator_to_path_chars<It>::value>{};
struct is_native_char_ptr<T>; // <const> path_native_char_type*

void convert(const char* from, const char* from_end, std::wstring& to, const codecvt_type* cvt=nullptr);
void convert(const wchar_t* from, const wchar_t* from_end, std::string& to, const codecvt_type* cvt=nullptr);

Cb::result_type dispatch<Src,Cb>(Src const& source, Cb cb, const codecvt_type* cvt=nullptr); // call dispatch with path_traits<>::tag_type of Src
Cb::result_type dispatch<Cb>(const {char|wchar_t}* source, Cb cb, const codecvt_type* cvt, ntcts_type_tag); // call cb on source
Cb::result_type dispatch<Src,Cb>(Src const& source, Cb cb, const codecvt_type* cvt, {string_class_tag|range_type_tag}); // call cb on string(source)
Cb::result_type dispatch<Cb>(directory_entry const& de, Cb cb, const codecvt_type* cvt, directory_entry_tag);

struct check_is_convertible_to_path_source<T>;
struct make_dependent<T,_> { using type = T; };
Cb::result_type dispatch_convertible<Src,Cb>(Src const& source, Cb cb, const codecvt_type* cvt=nullptr);
}

// exceptions
struct filesystem_error : public system_error {
  ctor({const char*|std::string const&} what_arg, <path const& path1_arg>, <path const& path2_arg>, system::error_code ec);
  ctor(self const& that); self& operator=(self const& that); ~dtor() noexcept;
  path const& path1() const noexcept; path const& path2() const noexcept;
  const char* what() const noexcept override;
private:
  struct impl : boost::intrusive_ref_counter<impl> { path m_path1, m_path2; std::string m_what; };
  boost::intrusive_ptr<impl> m_imp_ptr;
};

// file status types
enum file_type { status_error, file_not_found, regular_file, directory_file,
  symlink_file, block_file, character_file, fifo_file, socket_file, reparse_file, type_unknown };
enum perms { no_perms =0,
  owner_read=0400, owner_write=0200, owner_exe=0100, owner_all=0700,
  group_read=040, group_write=020, group_exe=010, group_all=070,
  others_read=04, others_write=02, others_exe=01, others_all=07,
  all_all=0777,
  set_uid_on_exe=04000, set_gid_on_exe=02000, sticky_bit=01000,
  perms_mask=07777, perms_not_known=0xFFFF, add_perms=0x1000, remove_perms=0x2000, symlink_perms=0x4000,
};
BITMASK(perms)

struct file_status { // all constexpr
  ctor() noexcept; explicit ctor(file_type v, <perms prms>) noexcept;
  ctor(self {const&|&&} rhs) noexcept; self operator=(self {const&|&&} rhs) noexcept;
  file_type type() const noexcept; perms permissions() const noexcept;
  void type(file_type v) noexcept; void permissions(perms prms) noexcept;
  bool operator{==|!=}(self const& rhs) const noexcept;
private: file_type m_value={status_error}; perms m_perms={perms_not_known};
};
bool type_present(file_status f) noexcept;
bool permissions_present(file_status f) noexcept;
bool status_known(file_status f) noexcept;
bool exists(file_status f) noexcept;
bool is_regular_file(file_status f) noexcept;
bool is_directory(file_status f) noexcept;
bool is_symlink(file_status f) noexcept;
bool is_block_file(file_status f) noexcept;
bool is_character_file(file_status f) noexcept;
bool is_fifo(file_status f) noexcept;
bool is_socket(file_status f) noexcept;
bool is_reparse_file(file_status f) noexcept;
bool is_other(file_status f) noexcept;
```

#### Path

```c++
struct path_detail::path_constants<Ch,sep,prefSep,dot> { using path_constants_base=self; using value_type = Ch;
  static constexpr value_type separator=sep, preferred_separator=prefSep, dot=dot;
};

struct detail::path_algorithms { // mostly _v3 and _v4 versions
  struct substring { size_t pos, size; };
  using value_type = path_traits::path_native_char_type; using string_type = std::basic_string<value_type>;
  static bool has_filename(path const& p);
  static path filename(path const& p);
  static path stem(path const& p);
  static path extension(path const& p);
  static void remove_filename(path& p);
  static void replace_extension(path& p, path const& new_extension);
  static path lexically_normal(path const& p);
  static path generic_path(path const& p);
  static void make_preferred(path& p);
  static int compare(path const& left, path const& right);
  static void append(path& p, const value_type* b, const value_type* e);
  static void append(path& left, path const& right);
  static string_type::size_type append_separator_if_needed(path& p);
  static void erase_redundant_separator(path& p, string_type::size_type sep_pos);
  static string_type::size_type file_root_name_size(path const& p);
  static string_type::size_type file_root_path_size(path const& p);
  static substring find_root_directory(path const& p);
  static substring find_relative_path(path const& p);
  static string_type::size_type file_parent_path_size(path const& p);
  static string_type::size_type file_filename_size(path const& p);
  static string_type::size_type file_extension_size(path const& p);
  static int lex_compare(path_iterator f1, path_iterator l1, path_iterator f2, path_iterator l2);
  static void increment(path_iterator& it);
  static void decrement(path_iterator& it);
};

struct path : public path_constants<path_native_char_type, '/', '/', '.'> { // L'/', L'\\', L'.' on Windows
  using value_type = path_algorithms::value_type; using string_type = path_algorithms::string_type;
  using codecvt_type = path_traits::codecvt_type;
  using <const>_iterator = path_iterator; using <const>_reverse_iterator = path_reverse_iterator;
  ctor(path {const&|&&} p, <codecvt_type const&>) <noexcept>;
  ctor(const value_type* s, <codecvt_type const&>);
  ctor(const value_type* begin, const value_type* end, <codecvt_type const&>);
  ctor({string_type|basic_string_view<value_type>} const& s, <codecvt_type const&>) <noexcept>;
  ctor<Src>(Src const& source, <codecvt_type const&>) const requires is_path_source<Src> && is_native_path_source<Src>;
  ctor<InIt>(InIt begin, InIt end, <codecvt_type const&>) requires is_path_source<Src> && is_native_path_source<Src>;

  ctor(nullptr)=delete; self& operator=(nullptr_t)=delete;

  self& operator=(path {const&|&&} p) <noexcept>;
  self& operator=(string_type&& p) noexcept;
  self& operator=<Src>(Src const& source) requires is_path_source<Src> && is_native_path_source<Src>;
  path& assign(path {const&|&&} p, <codecvt_type const&>) <noexcept>;
  path& assign(string_type {const&|&&} p, <codecvt_type const&>) <noexcept>;
  path& assign<Src>(Src const& p, <codecvt_type const&>);
  path& assign(const value_type* begin, const value_type* end, <codecvt_type const&>);
  path& assign<InIt>(InIt begin, InIt end, <codecvt_type const&>);

  path& operator+=(self const& p);
  path& operator+=<Src>(Src const& source) requires is_convertible_to_path_source<Src>;
  path& operator+=(value_type c);
  path& operator+=<Ch>(Ch c);
  path& concat(path const& p, <codecvt_type const&>);
  path& concat<Src>(Src const& source, <codecvt_type const&>) requires is_convertible_to_path_source<Src>;
  path& concat(value_type c);
  path& concat(const value_type* begin, const value_type* end, <codecvt_type const&>);
  path& concat<Ch>(Ch c);
  path& concat<InIt>(InIt begin, InIt end, <codecvt_type const&>);

  path& operator/=(self const& p);
  path& operator/=<Src>(Src const& source) requires is_convertible_to_path_source<Src>;
  path& append(path const& p, <codecvt_type const&>);
  path& append<Src>(Src const& source, <codecvt_type const&>) requires is_convertible_to_path_source<Src>;
  path& append(const value_type* begin, const value_type* end, <codecvt_type const&>);

  void clear() noexcept;
  path& make_preferred();
  path& remove_filename();
  path& remove_filename_and_trailing_separators();
  path& remove_trailing_separators();
  path& replace_filename(path const& replacement);
  path& replace_extension(path const& new_extension={});

  void swap(self& rhs) noexcept;

  string_type const& native() const noexcept;
  const value_type* c_str() const noexcept;
  string_type::size_type size() const noexcept;
  Str string<Str>(<codecvt_type> const& cvt) const;
  std::string const& string(<codecvt_type> const& cvt) const; // by val on WIN
  std::wstring wstring(<codecvt_type> const& cvt) const; // by const& on WIN

  path generic_path() const;
  Str generic_string<Str>(<codecvt_type> const& cvt) const;
  std::<w>string generic_<w>string(<codecvt_type> const& cvt) const;

  int compare(path const& p) const;
  int compare<Src>(Src const& source, <codecvt_type> const& cvt) const;

  path root_path() const; path root_name() const; path root_directory() const;
  path relative_path() const; path parent_path() const;
  path filename() const; path stem() const; path extension() const;

  bool empty() const noexcept;
  bool filename_is_dot() const; bool filename_is_dot_dot() const;
  bool has_root_path() const; bool has_root_name() const; bool has_root_directory() const;
  bool has_relative_path() const; bool has_parent_path() const;
  bool has_filename() const; bool has_stem() const; bool has_extension() const;
  bool is_relative() const; bool is_absolute() const;

  path lexically_normal() const;
  path lexically_relative(path const& base) const;
  path lexically_proximate(path const& base) const;

  iterator begin() const; iterator end() const;
  reverse_iterator rbegin() const; reverse_iterator rend() const;

  static std::locale imbue(std::locale const& loc);
  static codecvt_type const& codecvt();

private: struct assign_op; struct concat_op; struct append_op; struct compare_op;
  string_type m_pathname;
};

path const& detail::dot_path(); path const& detail::dot_dot_path();

class path_detail::path_iterator : iterator_facade<self, const path, bidirectional_traversal_tag> {
  path const& dereference() const { return m_element; }
  bool equal(self const& rhs) const noexcept;
  void increment(); void decrement();
  path m_element; const path* m_path_ptr; path::string_type::size_type m_pos;
};
class path_detail::path_reverse_iterator : iterator_facade<self, const path, bidirectional_traversal_tag> {
  path const& dereference() const { return m_element; }
  bool equal(self const& rhs) const noexcept;
  void increment(); void decrement();
  path m_element; path_iterator m_iter;
};
bool path_detail::lexicographical_compare(path_iterator f1, path_iterator const& l1, path_iterator f2, path_iterator const& l2);

bool operator{==|!=|<|<=|>|>=}(path const& lhs, path const& rhs);
bool operator{==|!=|<|<=|>|>=}<Src>(path const& lhs, Src const& rhs);
bool operator{==|!=|<|<=|>|>=}<Src>(Src const& lhs, Path const& rhs);
size_t hash_value(path const& p) noexcept;
void swap(path& lhs, path& rhs) noexcept;
path operator/(path lhs, path const& rhs);
path operator/<Src>(path lhs, Src const& rhs);
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, path const& p);
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr> (std::basic_istream<Ch,Tr>& is, path& p);

bool portable_posix_name(std::string const& name);
bool windows_name(std::string const& name);
bool portable_name(std::string const& name);
bool portable_directory_name(std::string const& name);
bool portable_file_name(std::string const& name);
bool native(std::string const& name);

bool detail::is_directory_separator(path::value_type c) noexcept; // /, \ (win)
bool detail::is_element_separator(path::value_type c) noexcept; // /, \ (win), : (win)
```

#### Directory

```c++
enum class directory_options : unsigned {
  none=0, skip_permission_denied=1, follow_directory_symlink=2, skip_dangling_symlinks=4, pop_on_error=8,
  _detail_no_follow=16, _detail_no_push=32 };
BITMASK(directory_options)

struct detail::directory_iterator_params;
void detail::directory_iterator_construct(directory_iterator& it, path const& p,
  directory_options opts, directory_iterator_params* params, system::error_code* ec);
void detail::directory_iterator_increment(directory_iterator& it, system::error_code* ec);
struct detail::recur_dir_itr_imp;
void detail::recursive_directory_iterator_construct(recursive_directory_iterator& it, path const& dir_path,
  directory_options opts, system::error_code* ec);
void detail::recursive_directory_iterator_increment(recursive_directory_iterator& it, system::error_code* ec);
void detail::recursive_directory_iterator_pop(recursive_directory_iterator& it, system::error_code* ec);

struct directory_entry {  using value_type = path::value_type;
  ctor() noexcept{}  ctor(self {const&|&&} rhs) <noecxept>;  self& operator==(self {const&|&&} rhs) <noecxept>;
  explicit ctor(path const& p);   ctor(path const&, system::error_code& ec);

  void assign(path {const&|&&} p, <system::error_code& ec>);
  void replace_filename(path const& p, <system::error_code& ec>);
  path const& path() const noexcept; operator path const& () const noexcept;
  void refresh(<system::error_code& ec>) <noexcept>;
  file_status status(<system::error_code& ec>) const <noexcept>;
  file_status symlink_status(<system::error_code& ec>) const <noexcept>;
  file_type file_type(<system::error_code& ec>) const <noexcept>;
  file_type symlink_file_type(<system::error_code& ec>) const <noexcept>;
  bool exists(<system::error_code& ec>) const <noexcept>;
  bool is_regular_file(<system::error_code& ec>) const <noexcept>;
  bool is_directory(<system::error_code& ec>) const <noexcept>;
  bool is_symlink(<system::error_code& ec>) const <noexcept>;
  bool is_block_file(<system::error_code& ec>) const <noexcept>;
  bool is_character_file(<system::error_code& ec>) const <noexcept>;
  bool is_fifo(<system::error_code& ec>) const <noexcept>;
  bool is_socket(<system::error_code& ec>) const <noexcept>;
  bool is_reparse_file(<system::error_code& ec>) const <noexcept>;
  bool is_other(<system::error_code& ec>) const <noexcept>;
  bool operator{==|!=|<|<=|>|>=}(self const& rhs) const;
private: path m_path; mutable file_status m_status, m_symlink_status;
};

file_status status(directory_entry const& e, <system::error_code& ec>) noexcept;
file_status symlink_status(directory_entry const& e, <system::error_code& ec>) noexcept;
bool type_present(directory_entry const& e, <system::error_code& ec>) noexcept;
bool status_known(directory_entry const& e, <system::error_code& ec>) noexcept;
bool exists(directory_entry const& e, <system::error_code& ec>) noexcept;
bool is_regular_file(directory_entry const& e, <system::error_code& ec>) noexcept;
bool is_directory(directory_entry const& e, <system::error_code& ec>) noexcept;
bool is_symlink(directory_entry const& e, <system::error_code& ec>) noexcept;
bool is_block_file(directory_entry const& e, <system::error_code& ec>) noexcept;
bool is_character_file(directory_entry const& e, <system::error_code& ec>) noexcept;
bool is_fifo(directory_entry const& e, <system::error_code& ec>) noexcept;
bool is_socket(directory_entry const& e, <system::error_code& ec>) noexcept;
bool is_reparse_file(directory_entry const& e, <system::error_code& ec>) noexcept;
bool is_other(directory_entry const& e, <system::error_code& ec>) noexcept;

struct detail::dir_itr_imp : intrusive_ref_counter<self> { directory_entry dir_entry; void* handle; /*other data for Win*/};
class directory_iterator : public iterator_facade<self, directory_entry, single_pass_traversal_tag> {
public:
  ctor() noexcept{} ctor(self const&) =default; self& operator=(self {const&|&&}) <noexcept>;
  explicit ctor(path const&, directory_options opts=none, <system::error_code& ec>) <noexcept>;
  self& increment(<system::error_code& ec>) <noexcept>;
private: intrusive_ptr<dir_itr_imp> m_imp;
  reference dereference() const;
  bool equal(self const& rhs) const noexcept;
};
directory_iterator const& <c>begin(directory_iterator const& iter) noexcept;
directory_iterator  <c>end(directory_iterator const& iter) noexcept;
directory_iterator<&> range_begin(directory_iterator <const>& iter) noexcept;
directory_iterator range_end(directory_iterator <const>&) noexcept;

struct detail::recur_dir_itr_imp : intrusive_ref_counter<self>
{ using element_type=directory_iterator; std::vector<element_type> m_stack; directory_options m_options; };
class recursive_directory_iterator : public iterator_facade<self, directory_entry, single_pass_traversal_tag> {
public:
  ctor() noexcept{} ctor(self const&) =default; self& operator=(self {const&|&&}) <noexcept>;
  explicit ctor(path const&, directory_options opts=none, <system::error_code& ec>) <noexcept>;
  self& increment(<system::error_code& ec>) <noexcept>;
  int depth() const noexcept;
  bool recursion_pending() const noexcept;
  void pop(<system::error_code& ec>) <noexcept>;
  void disable_recursion_pending(bool value=true) noexcept;
  file_status <symlink>_status() const;
private: intrusive_ptr<recur_dir_itr_imp> m_imp;
  reference dereference() const;
  bool equal(self const& rhs) const noexcept;
};
recursive_directory_iterator const& <c>begin(recursive_directory_iterator const& iter) noexcept;
recursive_directory_iterator  <c>end(recursive_directory_iterator const& iter) noexcept;
recursive_directory_iterator<&> range_begin(recursive_directory_iterator <const>& iter) noexcept;
recursive_directory_iterator range_end(recursive_directory_iterator <const>&) noexcept;
```

#### Operations

```c++
struct space_info { uintmax_t capacity, free, available; };
enum class copy_options : unsigned { none=0,
  skip_existing=1, overwrite_existing=2, update_existing=4, synchronize_data=8, synchronize=16, ignore_attribute_errors=32,
  recursive=256, copy_symlinks=512, skip_symlinks=1024, directories_only=2048, create_symlinks=4096, create_hard_links=8192,
  _detail_recursing=16384,
};
BITMASK(copy_options)

file_status status(path const& p, <system::error_code& ec>) <noexcept>;
file_status symlink_status(path const& p, <system::error_code& ec>) <noexcept>;
bool exists(path const& p, <system::error_code& ec>) <noexcept>;
bool is_regular_file(path const& p, <system::error_code& ec>) <noexcept>;
bool is_directory(path const& p, <system::error_code& ec>) <noexcept>;
bool is_symlink(path const& p, <system::error_code& ec>) <noexcept>;
bool is_block_file(path const& p, <system::error_code& ec>) <noexcept>;
bool is_character_file(path const& p, <system::error_code& ec>) <noexcept>;
bool is_fifo(path const& p, <system::error_code& ec>) <noexcept>;
bool is_socket(path const& p, <system::error_code& ec>) <noexcept>;
bool is_reparse_file(path const& p, <system::error_code& ec>) <noexcept>;
bool is_other(path const& p, <system::error_code& ec>) <noexcept>;
bool is_empty(path const& p, <system::error_code& ec>) <noexcept>;

path initial_path<_>(<system::error_code& ec>);
path current_path<_>(<system::error_code& ec>);
void copy(path const& from, path const& to, copy_options options=none, <system::error_code& ec>) <noexcept>;
void copy_file(path const& from, path const& to, copy_options options=none, <system::error_code& ec>) <noexcept>;
void copy_symlink(path const& existing_symlink, path const& new_symlink, <system::error_code& ec>) <noexcept>;
bool create_directories(path const& p, <system::error_code& ec>) <noexcept>;
bool create_directory(path const& p, <path const& existing>, <system::error_code& ec>) <noexcept>;
bool create_directory_symlink(path const& to, path const& from, <system::error_code& ec>) <noexcept>;
bool create_hard_link(path const& to, path const& new_hard_link, <system::error_code& ec>) <noexcept>;
bool create_symlink(path const& to, path const& new_symlink, <system::error_code& ec>) <noexcept>;
uintmax_t file_size(path const& p, <system::error_code& ec>) <noexcept>;
uintmax_t hard_link_count(path const& p, <system::error_code& ec>) <noexcept>;
time_t creation_time(path const& p, <system::error_code& ec>) <noexcept>;
time_t last_write_time(path const& p, <system::error_code& ec>) <noexcept>;
void last_write_time(path const& p, const time_t new_time, <system::error_code& ec>) <noexcept>;
void permissions(path const& p, perms prms, <system::error_code& ec>) <noexcept>;
path read_symlink(path const& p, <system::error_code& ec>) <noexcept>;
bool remove(path const& p, <system::error_code& ec>) <noexcept>;
uintmax_t remove_all(path const& p, <system::error_code& ec>) <noexcept>;
void rename(path const& old_p, path const& new_p, <system::error_code& ec>) <noexcept>;
void resize_file(path const& p, uintmax_t size, <system::error_code& ec>) <noexcept>;
path relative(path const& p, path const& base=current_path(), <system::error_code& ec>);
space_info space(path const& p, <system::error_code& ec>) <noexcept>;
path system_complete(path const& p, <system::error_code& ec>) <noexcept>;
path temp_directory_path(<system::error_code& ec>) <noexcept>;
path unique_path(path const& p="%%%%-%%%%-%%%%-%%%%", <system::error_code& ec>);

path absolute(path const& p, path const& base=current_path(), <system::error_code& ec>);
path canonical(path const& p, path const& base=current_path(), <system::error_code& ec>);
bool equivalent(path const& p1, path const& p2, <system::error_code& ec>);
path weakly_canonical(path const& p, path const& base=current_path(), <system::error_code& ec>);

bool detail::possible_large_file_size_support();
```

#### I/O Streams

```c++
class basic_filebuf<Ch,Tr=std::char_traits<Ch>> : public std::basic_filebuf<Ch,Tr> {
public: ctor()=default; // no copy, allow move
  base* open(path const& p, std::ios_base::openmode mode);
};
class basic_ifstream<Ch,Tr=std::char_traits<Ch>> : public std::basic_ifstream<Ch,Tr> {
public: ctor()=default; // no copy, allow move
  explicit ctor(path const& p); ctor(path const& p, std::ios_base::openmode mode);
  void open(path const& p);
  void open(path const& p, std::ios_base::openmode mode);
};
class basic_ofstream<Ch,Tr=std::char_traits<Ch>> : public std::basic_ofstream<Ch,Tr> {
public: ctor()=default; // no copy, allow move
  explicit ctor(path const& p); ctor(path const& p, std::ios_base::openmode mode);
  void open(path const& p);
  void open(path const& p, std::ios_base::openmode mode);
};
class basic_fstream<Ch,Tr=std::char_traits<Ch>> : public std::basic_ifstream<Ch,Tr> {
public: ctor()=default; // no copy, allow move
  explicit ctor(path const& p); ctor(path const& p, std::ios_base::openmode mode);
  void open(path const& p);
  void open(path const& p, std::ios_base::openmode mode);
};
using filebuf=basic_filebuf<char>; using wfilebuf=basic_filebuf<wchar_t>;
using ifstream=basic_ifstream<char>; using wifstream=basic_ifstream<wchar_t>;
using ofstream=basic_ofstream<char>; using wofstream=basic_ofstream<wchar_t>;
using fstream=basic_fstream<char>; using wfstream=basic_fstream<wchar_t>;
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Atomic

* `<boost/atomic/atomic_ref.hpp>`
* `<boost/memory_order.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/workaround.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/config/abi_prefix.hpp>`, `<boost/config/abi_suffix.hpp>`
* `<boost/config/auto_link.hpp>`
* `<boost/config/warning_disable.hpp>`
* `<boost/cstdint.hpp>`

#### Boost.ContainerHash

* `<boost/functional/hash_fwd.hpp>`

#### Boost.Core

* `<boost/core/bit.hpp>`

#### Boost.Detail

* `<boost/detail/bitmask.hpp>`
* `<boost/detail/utf8_codecvt_facet.hpp>`

#### Boost.IO

* `<boost/io/quoted.hpp>`

#### Boost.Iterator

* `<boost/iterator/is_iterator.hpp>`
* `<boost/iterator/iterator_categories.hpp>`
* `<boost/iterator/iterator_facade.hpp>`

#### Boost.PreDef

* `<boost/predef/library/c/cloudabi.h>`
* `<boost/predef/os/bsd/free.h>`
* `<boost/predef/os/bsd/open.h>`
* `<boost/predef/platform.h>`

#### Boost.Scope

* `<boost/scope/unique_fd.hpp>`
* `<boost/scope/unique_resource.hpp>`

#### Boost.SmartPtr

* `<boost/smart_ptr/intrusive_ptr.hpp>`
* `<boost/smart_ptr/intrusive_ref_counter.hpp>`

#### Boost.System

* `<boost/system/api_config.hpp>`
* `<boost/system/error_category.hpp>`
* `<boost/system/error_code.hpp>`
* `<boost/system/system_error.hpp>`

#### Boost.TypeTraits -- for old stdlib

* `<boost/type_traits/conjunction.hpp>`
* `<boost/type_traits/disjunction.hpp>`
* `<boost/type_traits/negation.hpp>`

#### Boost.WinAPI

* `<boost/winapi/basic_types.hpp>`
* `<boost/winapi/bcrypt.hpp>`
* `<boost/winapi/config.hpp>`
* `<boost/winapi/crypt.hpp>`
* `<boost/winapi/dll.hpp>`
* `<boost/winapi/error_codes.hpp>`
* `<boost/winapi/get_last_error.hpp>`

------
### Standard Facilities

Library: `<filesystem>` (C++11)
