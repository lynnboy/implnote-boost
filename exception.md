# Boost.Exception

* lib: `boost/libs/exception`
* repo: `boostorg/exception`
* commit: `7599ec73`, 2017-03-30

------
### Exception Storing Arbitrary Error Info

------
#### Class `error_info`

Header `<boost/exception/info.hpp>`

```c++
template <class Tag, class T>
class error_info {
public:
  typedef T value_type;
  error_info(value_type const & value);   error_info(value_type && value);
  error_info(error_info const & x);       error_info(error_info && x);
  virtual ~error_info();
  value() const -> value_type const &;    value() -> value_type &;
private:
  virtual std::string name_value_string() const;
  virtual error_info* clone() const;
  operator=(error_info const &) = delete; operator=(error_info &&) = delete;
};
to_string(error_info<Tag,T> const &) -> std::string;
```

* `error_info` bundles a tag type and a value.
* Default stringize format is `"[<tag type>] = <stringize of value>\n"`.
  Customized `error_info` derived class can overload `to_string` to provide alternate dump.

------
#### Putting and Getting of Error Info

Header `<boost/exception/info.hpp>`, `<boost/exception/get_error_info.hpp>`, and `<boost/exception/info_tuple.hpp>`

```c++
operator<<(boost::exception const & x, error_info<Tag,T> const & v);
operator<<(boost::exception const & x, error_info<Tag,T> && v);

template <class ErrorInfo>
get_error_info(boost::exception const & x) -> ErrorInfo::value_type const *;
template <class ErrorInfo>
get_error_info(boost::exception & x) -> ErrorInfo::value_type *;

operator<<(boost::exception const & x, tuple<error_info<Tag,T>...> const & v);
```

* `error_info` are stored in `boost::exception` dynamically, memory allocation occurred on-demand.
* `error_info` are stored by `shared_ptr`, thus shared when exception is copied/cloned.
* `error_info` are indexed by its type, thus each error kept unique.
* `throw_function`, `throw_file`, `throw_line` are builtin in `boost::exception`.
* Supports up to 4-tuple for `tuple` version.
* `get_error_info` returns `nullptr` if not found.

------
### Predefined Error Info Types

Header `<boost/exception/errinfo_*.hpp>`

```c++
// builtins:
typedef error_info<struct throw_function_,char const *> throw_function;
typedef error_info<struct throw_file_,char const *> throw_file;
typedef error_info<struct throw_line_,int> throw_line;
// others:
typedef error_info<struct errinfo_api_function_,char const *> errinfo_api_function;
typedef error_info<struct errinfo_at_line_,int> errinfo_at_line;
typedef error_info<struct errinfo_errno_,int> errinfo_errno;
typedef error_info<struct errinfo_file_handle_,weak_ptr<FILE> > errinfo_file_handle;
typedef error_info<struct errinfo_file_name_,std::string> errinfo_file_name;
typedef error_info<struct errinfo_file_open_mode_,std::string> errinfo_file_open_mode;
typedef error_info<struct errinfo_nested_exception_,exception_ptr> errinfo_nested_exception;
typedef error_info<struct errinfo_type_info_name_,std::string> errinfo_type_info_name;

to_string( errinfo_errno const & e ) -> std::string; // "<value>, \"<strerror>\""

// used by `current_exception` when exception type changed
typedef error_info<struct tag_original_exception_type,std::type_info const *> original_exception_type;
to_string( original_exception_type const & e ) -> std::string; // demangled name of original exception type
```

------
### Exception Propagating Support

Header `<boost/exception_ptr.hpp>` and `<boost/exception/current_exception_cast.hpp>`

```c++
class exception_ptr;
rethrow_exception(exception_ptr const&);
current_exception() -> exception_ptr;
template <class T>
copy_exception(T const & e) -> exception_ptr; // called 'make_exception_ptr' in STD

template <class E>
current_exception_cast() -> E*;

class unknown_exception : public boost::exception, public std::exception { /*...*/ };
template <class T>
class current_exception_std_exception_wrapper : public T, public boost::exception { /*...*/ };
```

* Similar to C++ 11 API.
* Only works exceptions thrown with `enable_current_exception`, or `BOOST_THROW_EXCEPTION`.
* Type `exception_ptr` wraps a `shared_ptr<exception_detail::clone_base const>`.
* If the captured exception isn't `clone_base`, auto detect it is one of known STD exceptions:
  * Recognized STD exception are injected with a `boost::exception` base class
  * Unrecognized exception are copied as `unknown_exception`
* When cloning of exception is failed due to `bad_alloc` or other exception thrown by the allocation
  or other operation during constructing the new exception object,
  a stored static `exception_ptr` for `bad_alloc` or `bad_exception` is returned.

------
### Stringize Facilities

Header `<boost/exception/to_string.hpp>` and `<boost/exception/to_string_stub.hpp>`

```c++
template <typename T, class Char=char, class Traits=std::char_traits<Char>>
concept bool is_output_streamable =
  requires (std::basic_ostream<Char, Traits> & os, T const & t) { os << t; };
template <typename T>
concept bool has_to_string =
  requires (T const & t) { to_string(t); };

std::string to_string(is_output_streamable const &); // using `<<` to get string
std::string to_string(std::pair<T,U> const &); // "(<first>,<second>)"
std::string to_string(std::exception const &); // "<what>"

std::string to_string_stub(has_to_string const & x) { return to_string(x); }
std::string to_string_stub(auto const & x); // "[ type: <type>, size: <sizeof>, dump: XX XX ... ]"
std::string to_string_stub(has_to_string const & x, auto) { return to_string(x); }
std::string to_string_stub(auto const &, std::string stub) { return stub; }
std::string to_string_stub(auto const &, const char* stub) { return stub; }
std::string to_string_stub(auto const & x, auto stub) { return stub(x); }

std::string to_string_stub(std::pair<T,U> const &, auto stub); // "(<first, stub>,<second, stub>)"
```

------
### Diagnostic Information Dumping

Header `<boost/exception/diagnostic_information.hpp>` and `<boost/exception_ptr.hpp>`

```c++
std::string diagnostic_information(auto const& ex, bool verbose=true);
char const* diagnostic_information_what(boost::exception const& ex, bool verbose=true) noexcept;

std::string current_exception_diagnostic_information(bool verbose=true);
std::string diagnostic_information(exception_ptr const& p, bool verbose=true);
std::string to_string(exception_ptr const& p); // indent each line of diagnostic information with two spaces
```

* Firstly, detect base instance of `boost::exception` and `std::exception`.
* If the `boost::exception` base is available, put throw location line at verbose.
* Put exception's dynamic type name line at verbose.
* If the exception doesn't use `diagnostic_information_what` as `what()`, put `"std::exception::what: <what>"` line at verbose.
* Then follows dumping of every `error_info` item stored in the `boost::exception`.
* The generated diagnostic information string is cached in `boost::exception` instance along with `error_info` list.
* `diagnostic_information_what` is called by custom exception to be used as implementation for `what()`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`
* `<boost/current_function.hpp>`

#### Boost.ThrowException

* `<boost/exception/exception.hpp>`

#### Boost.Core

* `<boost/core/demangle.hpp>`
* `<boost/core/typeinfo.hpp>`
* `<boost/utility/enable_if.hpp>`

#### Boost.SmartPtr

* `<boost/shared_ptr.hpp>` - when `BOOST_EXCEPTION_MINI_BOOST` is not defined.

#### Boost.Tuple

* `<boost/tuple/tuple.hpp>` - to support `tuple`-ed `error_info` setting.

------
### Standard Facilities

* Standard Library: `exception_ptr` and `current_exception` etc. (C++11)
* Proposals:
  * N3757 - Support for user-defined exception information and diagnostic information in std::exception
  * N3758 - Standard exception information types for std::exception
