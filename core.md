# Boost.Core

* lib: `boost/libs/core`
* repo: `boostorg/core`
* commit: `c6b098d`, 2025-08-26

------
### `address_of`

Header `<boost/core/addressof.hpp>`

```c++
constexpr inline addressof<T>(T&) -> T* noexcept
```

Use `__builtin_addressof` when available.

Standard: `addressof` (C++11)

------
### `alignof`

Header `<boost/core/alignof.hpp>`

```c++
BOOST_CORE_ALIGNOF(Type)
```

Standard: `alignof` (C++11)

------
### Allocator Access

Header `<boost/core/allocator_access.hpp>`

Access `allocator_traits` members seperately.

```c++
template<class A> struct allocator_XXXX; // A::XXXX
template<class A> using allocator_XXXX_t = allocator_XXXX::type;
template<class A, class T> struct allocator_rebind;
template<class A, class T> using allocator_rebind_t = allocator_rebind<A, T>::type;

template<class A, ...> RET allocator_FUNC(ARGS...);
```

* `XXXX` includes `value_type`, `pointer`, `const_pointer`, `void_pointer`,
`const_void_pointer`, `difference_type`, `size_type`,
`propagate_on_container_copy_assignment`, `propagate_on_container_move_assignment`,
`propagate_on_container_swap`, `is_always_equal`.
* `FUNC` includes `allocate`, `deallocate`, `construct`, `construct_n`,
`destroy`, `destroy_n`, `max_size`, `select_on_container_copy_construction`.

------
### Allocator Traits

Header `<boost/core/allocator_traits.hpp>`

Integrate `allocator_access.hpp` content into `boost::allocator_traits`, for migration purpose.

------
### Bit

Header `<boost/core/bit.hpp>`

```c++
template<class To, class From> To bit_cast(From const&) noexcept;
template<class T> constexpr T byteswap(T) noexcept;
template<class T> constexpr bool has_single_bit(T x) noexcept;
template<class T> constexpr T bit_ceil(T) noexcept;
template<class T> constexpr T bit_floor(T) noexcept;
template<class T> constexpr int bit_width(T) noexcept;
template<class T> constexpr T rotl(T, int) noexcept;
template<class T> constexpr T rotr(T, int) noexcept;
template<class T> constexpr T countl_zero(T) noexcept;
template<class T> constexpr T countl_one(T) noexcept;
template<class T> constexpr T countr_zero(T) noexcept;
template<class T> constexpr T countr_one(T) noexcept;
template<class T> constexpr int popcount(T) noexcept;
```

Standard: `<bit>` (C++20)

------
### Checked Delete

Header `<boost/core/checked_delete.hpp>`

```c++
template<class T> void checked_delete(T*) noexcept; 
template<class T> void checked_array_delete(T*) noexcept; 
template<class T> struct checked_deleter;
template<class T> struct checked_array_deleter;
```

Force `T` to be complete when deleting.
Standard: `default_delete` (C++11)

------
### C Math

Header `<boost/core/cmath.hpp>`

```c++
// return values
int const fp_zero, fp_subnormal, fp_normal, fp_infinite, fp_nan;
template<class T> bool isfinite(T);
template<class T> bool isnan(T);
template<class T> bool isinf(T);
template<class T> bool isnormal(T);
template<class T> int fpclassify(T);

template<class T> bool signbit(T);
template<class T> T copysign(T, T);
```

Standard: `<cmath>` C++11

------
### `data()`

Header `<boost/core/data.hpp>`

```c++
template<class C> constexpr auto data(C& c) noexcept(noexcept(c.data())) -> decltype(c.data());
template<class C> constexpr auto data(C const& c) noexcept(noexcept(c.data())) -> decltype(c.data());
template<class T, std::size_t N> constexpr T* data(T(&a)[N]) noexcept;
template<class T> constexpr const T* data(std::initializer_list<T> l) noexcept;
```

Standard: `data()` function (C++17)

------
### Default Allocator

Header `<boost/core/default_allocator.hpp>`

```c++
template<class T>
struct default_allocator;
```

* Does not provide functions other than `allocate`, `deallocate` and `max_size`
* Supports `BOOST_NO_EXCEPTIONS`

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
### Empty Value

Header `<boost/core/empty_value.hpp>`

```c++
struct empty_init_t {};
inline constexpr empty_init_t empty_init{};

template<class T, unsigned Index=0, bool Empty>
struct empty_value {
  typedef T type;
  empty_value() = default;
  template<class... Args> constexpr empty_value(empty_init_t, Args&&...);

  constexpr const T& get() const noexcept;
  constexpr T& get() noexcept;
};
```

Used for EBO usage, serve as CRTP base.

------
### `exchange`

Header `<boost/core/exchange.hpp>`

```c++
template<class T, class U=T> constexpr T exchange(T&, U&&);
```

Standard: `exchange` (C++14)

------
### Explicit Operator `bool` Simulation

Header `<boost/core/explicit_operator_bool.hpp>`

Macros `BOOST_EXPLICIT_OPERATOR_BOOL()`, `BOOST_EXPLICIT_OPERATOR_BOOL_NOEXCEPT()`,
and `BOOST_CONSTEXPR_EXPLICIT_OPERATOR_BOOL()`. If C++11 explicit conversion operator
is not available, fallback to **safe bool** idiom implementation.

Standard: `explicit operator bool()` (C++11)

------
### `first_scalar`

Header `<boost/core/first_scalar.hpp>`

```c++
template<class T>
constexpr T* first_scalar(T* p) noexcept;
template<class T, std::size_t N>
constexpr auto first_scalar(T (*p)[N]) noexcept;
```

Returns pointer to the first scalar element of an array.

------
### `functor`

Header `<boost/core/functor.hpp>`

```c++
template<auto Func>
struct functor {
  template<class...Args>
  decltype(auto) operator(Args&&...) const noexcept(...);
};
```

Wrap a function into a type. Useful in cases where a type is required, such as smartptr's deleter.  And it generates less types than lambda.

------
### `identity`

Header `<boost/core/identity.hpp>`

```c++
struct identity {
  using is_transparent = ...;
  template<class T> T&& operator(T&&) const noexcept;
};
```

Standard: `identity` in C++20.

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
### `launder`

Header `<boost/core/launder.hpp>`

Standard: `launder` in C++17.

------
### Lightweight Test Framework

Header `<boost/core/lightweight_test.hpp>`

```c++
void lwt_init(); // one-time init
int report_errors(); // error count
```

* `BOOST_TEST(expr)`, `BOOST_ERROR(msg)`, `BOOST_TEST_NOT(expr)`
* `BOOST_TEST_EQ(expr1,expr2)`, `BOOST_TEST_NE(expr1,expr2)`, `BOOST_TEST_LT(expr1,expr2)`, `BOOST_TEST_LE(expr1,expr2)`, `BOOST_TEST_GT(expr1,expr2)`, `BOOST_TEST_GE(expr1,expr2)`
* `BOOST_TEST_CSTR_EQ(expr1,expr2)`, `BOOST_TEST_CSTR_NE(expr1,expr2)`
* `BOOST_TEST_ALL_EQ(begin1, end1, begin2, end2)`, `BOOST_TEST_ALL_WITH(begin1, end1, begin2, end2, predicate)`
* `BOOST_TEST_WITH(expr1,expr2,pred)`, `BOOST_TEST_ALL_WITH(begin1,end1,begin2,end2,pred)`
* `BOOST_TEST_THROWS(expr,excep)`, `BOOST_TEST_NO_THROW(expr)`

Header `<boost/core/lightweight_test_trait.hpp>`

* `BOOST_TEST_TRAIT_TRUE(type)`, `BOOST_TEST_TRAIT_FALSE(type)`, `BOOST_TEST_TRAIT_SAME(type1,type2)`
* `BOOST_LIGHTWEIGHT_TEST_OSTREAM`, by default is `std::cerr`
* Force to call `int report_errors()` before `main` exit.

------
### `make_span`

Header `<boost/core/make_span.hpp>`

```c++
template<class I> constexpr span<I> make_span(I*, size_t) noexcept;
template<class I> constexpr span<I> make_span(I*, I*) noexcept;
template<class T, size_t N> constexpr span<T,N> make_span(T(&)[N]) noexcept;
template<class T, size_t N> constexpr span<T,N> make_span(std::array<T,N>&) noexcept;
template<class T, size_t N> constexpr span<const T,N> make_span(const std::array<T,N>&) noexcept;
template<class R> constexpr span<remove_pointer_t<iterator_t<R>>>
  make_span(R&&) noexcept;
```

To aid when CTAD is not available.

------
### `max_align`

Header `<boost/core/max_align.hpp>`

```c++
union max_align_t;
constexpr size_t max_align = alignof(max_align_t);
```

Standard `max_align_t` (C++11).

------
### Memory Resource

Header `<boost/core/memory_resource.hpp>`

Standard: `pmr::memory_resource` (C++17).

------
### Exception Handling Layer

Header `<boost/core/no_exceptions_support.hpp>`

* `BOOST_TRY`, `BOOST_CATCH(x)`, `BOOST_RETHROW`, `BOOST_CATCH_END`

When exception handling is disabled by compiler, expands to no-op.

------
### `noinit_adaptor`

Header `<boost/core/noinit_adaptor.hpp>`

```c++
template<class A> struct noinit_adaptor;
template<class A>
noinit_adaptor<A> noinit_adapt(const A&) noexcept;
```

Adapts an allocator and change `construct`/`destroy`'s behavior to use placement-new and dtor.

------
### `noncopyable` Base class

Header `<boost/core/noncopyable.hpp>`

Default ctor and dtor are protected and `default`-ed, copy-ctor and copy-assign are `delete`-ed.

------
### `null_deleter` Functor

Header `<boost/core/null_deleter.hpp>`

Just a no-op functor accepting a pointer argument.

### `fclose_deleter` Functor

Header `<boost/core/fclose_deleter.hpp>`

Deleter that calls `fclose` on `FILE*`.

------
### Name-Value Pair

Header `<boost/core/nvp.hpp>`

```c++
template<class T>
struct nvp {
  nvp(const char* name, T& value) noexcept;
  const char* name() const noexcept;
  T& value() const noexcept;
  const T& const_value() const noexcept;
};
template<class T>
const nvp<T> make_nvp(const char*, T&) noexcept;
```

* Wraps a name and a value reference.
* Macro `BOOST_NVP(obj)` give `make_nvp(#obj, obj)`

------
### `pointer_in_range`

Header `<boost/core/pointer_in_range.hpp>`

```c++
template<class T>
constexpr bool pointer_in_range(const T* p, const T* b, const T* e);
```

------
### Pointer Traits

Header `<boost/core/pointer_traits.hpp>`

```c++
template<class T>
struct pointer_traits {...};
template<class T>
struct pointer_traits<T*> {...};
template<class T> constexpr T* to_address(T*) noexcept;
template<class T> auto to_address(const T&) noexcept;
```

Standard: `pointer_traits` C++11, `to_address` C++20.

------
### `quick_exit`

Header `<boost/core/quick_exit.hpp>`

```c++
[[noreturn]] void quick_exit(int code) noexcept;
```

Standard: `quick_exit` (C++11)

------
### Scoped `enum` Emulation

Header `<boost/core/scoped_enum.hpp>`<br>
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
### Serialization Support

Header `<boost/core/serialization.hpp>`

```c++
struct core_version_type;

template<class Ar, class T>
void split_free( Ar& ar, T& t, unsigned int v );
template<class Ar, class T>
void split_member( Ar& ar, T& t, unsigned int v );
template<class Ar, class T>
void load_construct_data_adl( Ar& ar, T* t, unsigned int v );
template<class Ar, class T>
void save_construct_data_adl( Ar& ar, T const* t, unsigned int v );
```

Copied from Boost.Serialization, useful when adding serialization support without depending on Boost.Serialization.

------
### `size`

Header `<boost/core/size.hpp>`

Standard: `std::size` (C++17).

------
### Span

Header `<boost/core/span.hpp>`

```c++
constexpr size_t dynamic_extent = -1;
template<class T, size_t E=dynamic_extent> class span;
// deduction guides

template<class T, size_t N>
span<const byte, E == dynamic_extent ? dynamic_extent : sizeof(T) * E>
as_bytes(span<T,E>) noexcept;
template<class T, size_t N>
span<byte, E == dynamic_extent ? dynamic_extent : sizeof(T) * E>
as_writable_bytes(span<T,E>) noexcept;
```

Standard: `<span>` (C++20)

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
### `type_name`

Header `<boost/core/type_name.hpp>`

```c++
template<class T> std::string type_name();
```

Simplified, readable demangled name for a type.

------
### `snprintf`

Header `<boost/core/snprintf.hpp>`

Provides `snprintf`, `vsnprintf`, `swprintf`, `vswprintf` etc.

Standard: `snprintf` etc. in C++11

------
### `uncaught_exceptions`

Header `<boost/core/uncaught_exceptions.hpp>`

Can't return correct number on pre-C++17 compilers. And `BOOST_CORE_UNCAUGHT_EXCEPTIONS_EMULATED` is defined in this case.

Standard: C++17.

------
### `use_default`

Header `<boost/core/use_default.hpp>`

An empty class, can be used as default template argument to serve as placeholder value.

------
### `verbose_terminate_handler`

Header `<boost/core/verbose_terminate_handler.hpp>`

Can be set by `set_terminate` to print info about uncaught exception.

------
### Yield Primitives

Header `<boost/core/yield_primitives.hpp>`

```c++
void sp_thread_pause() noexcept;
void sp_thread_yield() noexcept;
void sp_thread_sleep() noexcept;
```

Low level primitives.

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
### `alloc_construct` et al.

Header `<boost/core/alloc_construct.hpp>`

```c++
template<class A, class T> void alloc_construct(A&, T*);
template<class A, class T, class...Args> void alloc_construct(A&, T*, Args&&...);
template<class A, class T> void alloc_construct_n(A&, T*, size_t);
template<class A, class T> void alloc_destroy(A&, T*);
template<class A, class T> void alloc_destroy_n(A&, T*, size_t);
```

Deprecated by `<boost/core/allocator_access.hpp>`

------
### String View

Header `<boost/core/detail/string_view.hpp>`

Standard: `string_view` (C++17)

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`
* `<boost/cstdint.hpp>`
* `<boost/config/header_deprecated.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`
* `<boost/current_function.hpp>`

### Boost.StaticAssert

* `<boost/static_assert.hpp>`

### Boost.ThrowException

* `<boost/throw_exception.hpp>` by `<boost/core/verbose_terminate_handler.hpp>`
