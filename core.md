# Boost.Core

* lib: `boost/libs/core`
* repo: `boostorg/core`
* commit: `f8165366`, 2016-01-06

------
### `address_of`

Header `<boost/core/addressof.hpp>`

`addressof<T>(T&) -> T*`

Standard: `addressof` (C++11)

------
### Checked Delete

Header `<boost/core/checked_delete.hpp>`

```c++
template<class T> void checked_delete(T*); 
template<class T> void checked_array_delete(T*); 
template<class T> struct checked_deleter;
template<class T> struct checked_array_deleter;
```

Force `T` to be complete when deleting.
Standard: `default_delete` (C++11)

------
### Demangle C++ Symbol

Header `<boost/core/demangle.hpp>`

```c++
demangle_alloc(char const*) noexcept -> char const*;
demangle_free(char const*) noexcept;

class scoped_demangle_name { // wraps demangle_alloc/demangle_free
public:
  explicit scoped_demangle_name(char const*) noexcept;
  ~scoped_demangle_name() noexcept;
  get() noexcept -> char const*
};

demangle(const char*) -> std::string;
```

Depends on `abi::__cxa_demangle`.

------
### Explicit Operator `bool` Simulation

Header `<boost/core/explicit_operator_bool.hpp>`

Macros `BOOST_EXPLICIT_OPERATOR_BOOL()`, `BOOST_EXPLICIT_OPERATOR_BOOL_NOEXCEPT()`,
and `BOOST_CONSTEXPR_EXPLICIT_OPERATOR_BOOL()`. If C++11 explicit conversion operator
is not available, fallback to **safe bool** idiom implementation.

Standard: `explicit operator bool()` (C++11)

------
### `ignore_unused`

Header `<boost/core/ignore_unused.hpp>`

```C++
template<typename ...Ts> constexpr ignore_unused(Ts const&...) ->void;
template<typename ...Ts> constexpr ignore_unused() ->void;
```

Can both touch unused variables and unused typedefs.

------
### `is_same` Trait

Header `<boost/core/is_same.hpp>`

Standard: `is_same` trait (C++11)

------
### Lightweight Test Framework

Header `<boost/core/lightweight_test.hpp>`
Header `<boost/core/lightweight_test_trait.hpp>`

* `BOOST_TEST(expr)`, `BOOST_ERROR(msg)`
* `BOOST_TEST_EQ(expr1,expr2)`, `BOOST_TEST_NE(expr1,expr2)`
* `BOOST_TEST_THROWS(expr,excep)`
* `BOOST_TEST_TRAIT_TRUE(type)`, `BOOST_TEST_TRAIT_FALSE(type)`
* `BOOST_LIGHTWEIGHT_TEST_OSTREAM`, by default is `std::cerr`
* Force to call `int report_errors()` before `main` exit.

------
### Exception Handling Layer

Header `<boost/core/no_exceptions_support.hpp>`

* `BOOST_TRY`, `BOOST_CATCH(x)`, `BOOST_RETHROW`, `BOOST_CATCH_END`

When exception handling is disabled by compiler, expands to no-op.

------
### `noncopyable` Base class

Header `<boost/core/noncopyable.hpp>`

Default ctor and dtor are protected and `default`-ed, copy-ctor and copy-assign are `delete`-ed.

------
### `null_deleter` Functor

Header `<boost/core/null_deleter.hpp>`

Just a no-op functor accepting a pointer argument.

------
### Scoped `enum` Emulation

Header `<boost/core/scoped_enum.hpp>`
Header `<boost/core/underlying_type.hpp>`

* `BOOST_SCOPED_ENUM_UT_DECLARE_BEGIN(EnumType,UnderlyingType)`
  `enum class EnumType : UnderlyingType`
* `BOOST_SCOPED_ENUM_DECLARE_BEGIN(EnumType)` - `enum class EnumType`
* `BOOST_SCOPED_ENUM_DECLARE_END(EnumType)` - `;`
* `BOOST_SCOPED_ENUM_NATIVE(EnumType)` - `EnumType`
* `BOOST_SCOPED_ENUM_FORWARD_DECLARE(EnumType)` - `enum class EnumType`

When `enum class` is not available, declare `EnumType` tobe a wrapper `struct`
and inject comparison operators `==`, `<`, etc. and explicit convertion from/to
the underlying type.

* Trait `native_type<EnumType>`, `underlying_type<EnumType>`
* Cast `underlying_cast<UnderlyingType>(EnumType)`
* Function `native_value(EnumType)` get the real or wrapped `enum` value.

Standard: `enum class`, `underlying_type` (C++11)

------
### `typeinfo` Simulation

Header `<boost/core/typeinfo.hpp>`
Header `<boost/detail/sp_typeinfo.hpp>`

* Class `typeinfo` is defined to be `std::type_info` if RTTI is available
  Otherwise it defines a compile-time simulation type.
* Macro `BOOST_CORE_TYPEID(T)` is `typeid(T)` or instantiate of `typeinfo<T>`
* Function `demangled_name(typeinfo const&)`
* `sp_typeinfo` and `BOOST_SP_TYPEID(T)` is used by Boost.SmartPtr etc.

------
### `get_pointer` Function

Header `<boost/get_pointer.hpp>`

Overloaded for `T*`, and `std::auto_ptr`, `std::unique_ptr`, `std::shared_ptr`, etc.
Boost smart pointers and wrapper types usually provide overload for it.

------
### `visit_each` Algorithm

Header `<boost/visit_each.hpp>`

```c++
template <typename Visitor, typename T>
inline void visit_each(Visitor& visitor, const T& t, long) { visitor(t); }          // fallback

template <typename Visitor, typename T>
inline void visit_each(Visitor& visitor, const T& t) { visit_each(visitor, t, 0); } // entrance
```
* Overload `visit_each` to access sub-objects.
* Expanded by **Boost.Bind** and **Boost.Phoenix**.
* Used by **Boost.Signals** and **Boost.Signals2**.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`
* `<boost/current_function.hpp>`

------
### Standard Facilities

Standard Library: `<system_error>` (C++11)
