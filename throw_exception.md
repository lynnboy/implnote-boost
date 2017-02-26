# Boost.ThrowException

* lib: `boost/libs/throw_exception`
* repo: `boostorg/throw_exception`
* commit: `56d65d5f`, 2017-02-20

------
### Boost Exception Base Class

#### Header

* `<boost/exception/exception.hpp>`

#### Class `exception` and `enable_error_info()`

```c++
class exception {
public:
  exception();
  virtual ~exception() noexcept = 0;
private:
  mutable refcount_ptr<error_info_container> data_;
  mutable char const* throw_function_;
  mutable char const* throw_file_;
  mutable int throw_line_;
  friend set_info(E const &, error_info<Tag, T> const &);
  friend struct get_info<error_info<Tag, T>>;
  friend struct set_info_rv<error_info<Tag, T>>;
  friend get_diagnostic_information(exception const &, char const *) -> char const*;
  friend copy_boost_exception(exception *, exception const *);
};

template <class E> // E is std::exception derived
struct error_info_injector : public E, public exception {
  explicit error_info_injector(E const & x) : E(x) {}
};

enable_error_info(E const & x) -> E or error_info_injector<E>;
```

##### Rationale

`boost::exception` serves as base class for custom exception classes, it contains
builtin error info for `file`, `line`, and `function`, and a `error_info_container`
for arbitrary error info storage.

If the custom exception is not derived from `boost::exception`, `enable_error_info()`
will inject it as a base class to support error info.

For detail of error info support, see **Boost::Exception** library.

#### `enable_current_exception()`

```c++
class clone_base {
public:
  virtual clone_base const* clone() const = 0;
  virtual void rethrow() const = 0;
  virtual ~clone_base() noexcept {}
}

copy_boost_exception( exception*, exception const *);
template <class E>
class clone_impl : public E, public virtual clone_base { /* ... */ };

enable_current_exception(E const & x) -> clone_impl<E>;
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

#### Usage

##### Function `throw_exception`

```c++
throw_exception(E const&);
```

Add error-info and current-exception support, by converting the exception by
`enable_error_info()` and `enable_current_exception()`, then throw the resulting
exception.

If compiler don't support _exception-handling_ (`BOOST_NO_EXCEPTIONS` is defined),
the function becomes `throw_exception(std::exception const&)` and requires user
definition.

##### Macro `BOOST_THROW_EXCEPTION(x)`

Add the throwing position (file, line, function) to throwing exception.

#### Switches

* Macro `BOOST_EXCEPTION_DISABLE`, don't add error-info and current-exception support,
just throw the exception in `throw_exception()`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`.
* `<boost/detail/workaround.hpp>`.

#### Boost.Assert

* `<boost/current_function.hpp>`.

------
### Standard Facilities

* Core Language: `throw`
* Standard Library: `<exception>`
* Proposals:
  * N2179 - Language Support for Transporting Exceptions between Threads (C++11).
