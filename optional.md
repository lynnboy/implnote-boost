# Boost.Optional

* lib: `boost/libs/optional`
* repo: `boostorg/optional`
* commit: `bfdf0c7`, 2024-12-09

------
### Boost Optional

* Header `<boost/optional_fwd.hpp>`
* Header `<boost/optional.hpp>`
* Header `<boost/optional/optional_io.hpp>`

```c++
struct none_t{} none;
struct in_place_init_t{} in_place_init;
struct in_place_init_if_t{} in_place_init_if;

class bad_optional_access : public std::logic_error;

template<class T> class optional {
public:
  using value_type = T;
  using reference_type = T&; using reference_const_type = T const&; using rval_reference_type = T&&;
  using pointer_type = T*; using pointer_const_type = T const*;

  // def-ctor, copy-ctor, move-ctor
  ctor(none_t); noexcept;
  ctor(T const&); ctor(T&&); ctor(bool cond, T const&); // (cond) init from value
  explicit ctor<U>(optional<U> const&); explicit ctor<U>(optional<U> &&);
  explicit ctor<...Args>(in_place_init_t, Args&&...args); explicit ctor<...Args>(in_place_init_if_t, bool cond, Args&&...args);
  explicit ctor<InPlaceFactory>(InPlaceFactory const&); explicit ctor<TypedInPlaceFactory>(TypedInPlaceFactory const&);

  operator=(none_t) noexcept;
  void reset() noexcept;

  // copy-assign, move-assign
  optional& operator=<U>(optional<U> const&); optional& operator=<U>(optional<U> &&);
  void emplace<...Args>(Args&&...args);
  optional& operator=<InPlaceFactory>(InPlaceFactory const&); optional& operator=<TypedInPlaceFactory>(TypedInPlaceFactory const&);

  T const& operator*() const; T& operator*()&; T&& operator*() &&;
  T const& value() const; T& value() &; T&& value() &&;
  T const& get() const; T& get();

  T const* operator->() const; T* operator->();
  T const* get_ptr() const; T* get_ptr();

  bool has_value() const noexcept;
  explicit operator bool() const noexcept;
  bool operator!() const noexcept;

  T value_or<U>(U&&) const&; T value_or<U>(U&&) &&;
  T value_or_eval<F>(F) const&; T value_or_eval<F>(F) &&;
  auto map<F>(F) const&; auto map<F>(F) &; auto map<F>(F) &&; // -> optional<result_of<F>>
  auto flat_map<F>(OF) const&; auto flat_map<F>(OF) &; auto flat_map<OF>(OF) &&; // -> result_of<OF>
};
template<class T> class optional<T&> {
  using value_type = T&;
  using reference_type = T&; using reference_const_type = T&; using rval_reference_type = T&;
  using pointer_type = T*; using pointer_const_type = T*;

  // def-ctor, copy-ctor
  ctor(none_t) noexcept;
  ctor<R>(R&&) noexcept; ctor<R>(bool cond, R&&) noexcept;
  explicit ctor<U>(optional<U&> const&) noexcept;

  optional& operator=(none_t) noexcept;
  void reset() noexcept;

  // copy-assign
  optional& operator=<U>(optional<U&> const&) noexcept;
  optional& operator=<R>(R&&) noexcept;
  void emplace<R>(R&&) noexcept;

  T& operator*() const; T& value() const; T& get() const;
  T* operator->() const; T* get_ptr() const noexcept;
  bool has_value() const noexcept; explicit operator bool() const noexcept; bool operator!() const noexcept;

  T& value_or<R>(R&&) const noexcept; T& value_or_eval<F>(F) const;
  auto map<F>(F) const; auto flat_map<OF>(OF) const;
};

// operator==, !=, <, > <=, >=
// operator==, != with none_t
template<class T> optional<decay_t<T>> make_optional([bool cond], T [const]&&);
template<class T> void swap(opitonal<T>&, optional<T>&);
template<class T> void swap(optional<T&>&, optional<T&>&) noexcept;
template<class T> auto get_optional_value_or(optional<T> const&, optional::reference) -> optional::reference; // also const_reference
template<class T> T [const]& get(optional<T> [const]&);
template<class T> T [const]* get(optional<T> [const]*);
template<class T> auto get_pointer(optional<T> [const]&);
// std::hash support

template<class Ch, class Tr> std::basic_ostream<Ch,Tr>& operator <<(std::basic_ostream<Ch,Tr>&, none_t);
template<class Ch, class Tr, T> std::basic_ostream<Ch,Tr>& operator <<(std::basic_ostream<Ch,Tr>&, optional<T> const&);
template<class Ch, class Tr, T> std::basic_istream<Ch,Tr>& operator >>(std::basic_istream<Ch,Tr>&, optional<T> const&);
```

#### Class Template `optional`

Allow bind to reference type (but not rvalue-reference).

Optimized for trivally-copyable types.

Differences from STD proposal:

* Supports bind to lvalue-reference, specialized for reference and storing pointer for them. (C++26 included)
* Constructors/assignments for `optional<U>`, when `T` can construct from `U`
* Constructors/assignments for _InPlaceFactory_, and `in_place_init_if_t` constructors.
* Member and non-member `get()` and `get_ptr()`, to get reference/pointer to stored data.
* Supports conditional init/make (`cond, val`).
* `boost::none` vs. `std::nullopt`

#### Exception `bad_optional_access`

Thrown by `optional::value()` when it contains nothing.

#### I/O of `optional`

* Empty `optional` becomes `"--"`.
* Non-empty `optional` having a leading space and the content value's representation.

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.Core

* `<boost/core/addressof.hpp>`
* `<boost/core/enable_if.hpp>`
* `<boost/core/invoke_swap.hpp>`
* `<boost/core/launder.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/alignment_of.hpp>`, `<boost/type_traits/type_with_alignment.hpp>`
* `<boost/type_traits/*.hpp>` for const/reference, decay, etc.
* `<boost/type_traits/*.hpp>` for nothrow_xxx detection
* `<boost/type_traits/is_same.hpp>`, `<boost/type_traits/is_base_of.hpp>`

------
### Standard Facilities

Standard library: `<optional>` (C++17), `optional<T&>` (C++26).

Proposals:
  Library Fundamentals v1 - `<optional>`
