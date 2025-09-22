# Boost.ThrowException

* lib: `boost/libs/throw_exception`
* repo: `boostorg/throw_exception`
* commit: `f3d4d55`, 2024-12-13

------
### Boost Exception Base Class

#### Header

* `<boost/exception/exception.hpp>`

#### Class `exception` and `enable_error_info()`

```c++
class detail::refcount_ptr<T> { // requires T::add_ref, T::release
  T* px_{nullptr};
  void add_ref() { if (px_) px_->add_ref(); }
  void release() { if (px_ && px_->release()) px_ = nullptr; }
public:
  ctor();
  ~dtor(); {release();}
  ctor(refcount_ptr const& x) :px_(x.px_) { add_ref(); }
  refcount_ptr& operator= (refcount_ptr const& x) { adopt(x.px_); return *this; }
  void adopt(T* px); { release(); px_=px; add_ref(); }
  T* get() const;
};

struct error_info<Tag,T> { // primary template is not enabled by default
  using value_type = T,
  value_type v_;
  explicit ctor(value_type v);
};
// specializations:
using throw_function = error_info<struct throw_function_, char const*>;
using throw_file = error_info<struct throw_file_, char const*>;
using throw_line = error_info<struct throw_line, int>;
using throw_column = error_info<struct throw_column, int>;

struct detail::error_info_container { // interface for `error_info` containers
  virtual char const* get_diagnostic_information(char const*) const =0;
  virtual shared_ptr<error_info_base> get(type_info_ const&) const =0; // lookup by type
  virtual void set(shared_ptr<error_info_base> const&, type_info_ const&) = 0; // set by type
  virtual void add_ref() const = 0;
  virtual void release() const = 0;
  virtual refcount_ptr<error_info_container> clone() const = 0;
};

class exception {
protected:
  exception(); virtual ~exception() noexcept = 0;
private:
  mutable refcount_ptr<error_info_container> data_; // for other error_info
  mutable char const* throw_function_{nullptr}, throw_file_{nullptr};
  mutable int throw_line_{-1}, throw_column_{-1};
public:
  void set<Tag>(Tag::type const&);
  Tag::type const* get<Tag>() const;
};

struct get_info<T>; struct set_info_rv<T>; // specialize for the 4 throw_xxx error_infos
E const& set_info<E,Tag,T> (E const&, error_info<Tag,T> const&); // and overloads for the 4 throw_xxx
source_location get_exception_throw_location(exception const&); // fill with the 4 info

char const* get_diagnostic_information(exception const&, char const*);
void copy_boost_exception(exception*, exception const*);

template <class E> // E is std::exception derived
struct detail::error_info_injector : public E, public exception {
  explicit error_info_injector(E const & x) : E(x) {}
};

auto enable_error_info<E>(E const & x) -> is_base_v<exception,E> ? E : error_info_injector<E>;
```

##### Rationale

`boost::exception` serves as base class for custom exception classes, it contains
builtin error info for `file`, `line`, `column`, and `function`, and a `error_info_container`
for arbitrary error info storage.

If the custom exception is not derived from `boost::exception`, `enable_error_info()`
will inject it as a base class to support error info.

For detail of error info support, see **Boost::Exception** library.

#### `enable_current_exception()`

```c++
void detail::copy_boost_exception( exception*, exception const *); // copy 4 member, and clone the ei container

struct detail::clone_base {
  virtual clone_base const* clone() const = 0;
  virtual void rethrow() const = 0;
  virtual ~clone_base() noexcept {}
}
// requires is_base<E,exception>, call copy_boost_exception to copy data
class detail::clone_impl<E> : public E, public virtual clone_base { /* ... */ };

auto enable_current_exception<E>(E const & x) -> clone_impl<E>; // wrap exception into a clone_impl
```

##### Rationale

`enable_current_exception` will inject `clone_impl` as base class, if exception is
derived from `boost::exception`, the copy-ctor and `clone` will automatically invoke
`copy_boost_exception()` to copy all error_info to new instance.

For detail of `current_exception()` support, see **Boost::Exception** library.

------
### `BOOST_THROW_EXCEPTION`

#### Header

* `<boost/throw_exception.hpp>`

#### Function `throw_exception`

```c++
struct wrapexcept<E>
  : public (is_base_v<clone_base,E> ? empty : clone_base), public E,
  , public (is_base_v<exception,E> ? empty : exception) {
  explicit ctor(E const&); // if E is exception, call op= for it
  explicit ctor(E const&, source_location const& loc); // copy E, then fill 4 errorinfo
  clone_base const* clone() const override; // copyctor then copy_boost_exception
  void rethrow() const override;
};

[[noreturn]] void throw_exception<E>(E const&); // throw wrapexcept<E>
[[noreturn]] void throw_exception<E>(E const&, source_location const&);

#define BOOST_THROW_EXCEPTION(x) ::boost::throw_exception(x, BOOST_CURRENT_LOCATION)

struct detail::with_throw_location<E> : public E { source_location location_; };
[[noreturn]] void throw_with_location<E>(E&& e, source_location const& loc=BOOST_CURRENT_LOCATION);

source_location get_throw_location<E>(E const& e) {
  if (auto pl = dynamic_cast<throw_location const*>(&e)) return pl->location_;
  else if (auto px = dynamic_cast<exception const*>(&e)) return get_exception_throw_location(*px);
  else return {};
}
```

Add error-info and current-exception support, by converting the exception by
`enable_error_info()` and `enable_current_exception()`, then throw the resulting
exception.

If compiler don't support _exception-handling_ (`BOOST_NO_EXCEPTIONS` is defined),
the function becomes `throw_exception(std::exception const&)` and requires user
definition.

Source location support: `throw_with_location(ex);`, wraps any exception with a thin wrapper.

#### Switches

* Macro `BOOST_EXCEPTION_DISABLE`, don't add error-info and current-exception support,
just throw the exception in `throw_exception()`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`.
* `<boost/config/workaround.hpp>`.

#### Boost.Assert

* `<boost/assert/source_location.hpp>`.

------
### Standard Facilities

* Core Language: `throw`
* Standard Library: `<exception>`
* Proposals:
  * N2179 - Language Support for Transporting Exceptions between Threads (C++11).
