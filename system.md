# Boost.System

* lib: `boost/libs/system`
* repo: `boostorg/system`
* commit: `14c5f52`, 2025-07-03

------
### System Error Code

#### Header `<boost/system/cerrno.hpp>`

Supply missing POSIX `EXXX` macros.

#### Header `<boost/system/error_code.hpp>`

```c++
struct is_error_code_enum<T> { static const bool value = false; };
struct is_error_condition_enum<T> { static const bool value = false; };

static const ulong_long_type detail::generic_category_id = 0xB2AB'117A'257E'DFD0,
    detail::system_category_id, detail::interop_category_id; // +1, +2
class detail::std_category : public std::error_category { // proxy instance to support std types
    error_category const* pc_; // ptr to owner instance
    public: explicit ctor<id>(error_category const* pc, id_wrapper<id>);
    // implements std cate's abstract functions: name, message, default_error_condition, equivalent, forward to `pc_`
};

class error_category { // not copyable
    ulong_long_type id_;
    mutable char stdcat_[4*sizeof(void*)]; // alignas(void*), for a `std_category` instance
    mutable atomic<unsigned> sc_init_; // locker for stdcat_'s initialization
protected: ~dtor(); explicit constexpr ctor(ulong_long_type id); // serve as base class
public:
    virtual const char* name() const noexcept = 0;
    virtual error_condition default_error_condition(int ev) const noexcept;
    virtual bool equivalent(int code, const error_condition& cond) const noexcept;
    virtual bool equivalent(const error_code& code, int cond) const noexcept;
    virtual std::string message(int ev) const = 0;
    virtual const char* message(int ev, char* buf, size_t len) const noexcept; // snprintf into buf
    virtual bool failed(int ev) const noexcept { return ev!=0; }
    operator std::error_category const& () const; // impl conv to std type, auto-construct std_cat_ for non-generic/system cats
};
bool operator==(error_category const& lhs, error_category const& rhs); // and !=, <, >, <=, >=

struct detail::generic_error_category: public error_category {
    constexpr ctor() noexcept : error_category(generic_category_id){}
    const char* name() const noexcept override { return "generic"; }
    // implements message. call `strerror`/`strerror_r`
};
struct detail::system_error_category: public error_category {
    constexpr ctor() noexcept : error_category(system_category_id){}
    const char* name() const noexcept override { return "generic"; }
    // implements message. call `FormatMessage` on Windows, otherwise the same as generic version
    error_condition default_error_condition(int ev) const noexcept override {
#if BOOST_WINDOWS_API
        ev = system_category_condition_win32(ev);
        if (ev == -1) return error_condition(ev, *this);
#endif
        return error_condition(generic_value_tag(ev));
    }
};
int detail::system_category_condition_win32(int ev) noexcept; // convert windows errno to errc;
struct detail::interop_error_category: public error_category {
    constexpr ctor() noexcept : error_category(interop_category_id){}
    const char* name() const noexcept override { return "std:unknown"; }
    // implements message. return "Unknown interop error %d" with errno value
};

constexpr error_category const& generic_category() noexcept; // return instance
constexpr error_category const& system_category() noexcept; // return instance
constexpr error_category const& interop_category() noexcept; // return instance

class error_code {
    struct data {int val_; const error_category* cat_;};
    union{ data d1_; unsigned char d2_[sizeof(std::error_code)]; }; // 
    uintptr_t lc_flags_; // 0 for def-ctored, 1 for d2_, 2 for d1_ and !failed; 3 for d1_ and faled; otherwise ptr to `source_location`
public:
    constexpr ctor() noexcept; // case 0
    constexpr ctor(int ev, error_category const& cat) noexcept; // case 2/3
    constexpr ctor<E>(E e) noexcept requires is_error_code_enum<E> || std::is_error_code_enum<E>; // make_error_code
    ctor(error_code const& ec, source_location const* loc) noexcept;
    ctor(std::error_code const& ec) noexcept; // case 1

    constexpr void assign(int ev, error_category const& cat) noexcept;
    void assign(int ev, error_category const& cat, source_location const* loc) noexcept;
    void assign(error_code const& ec, source_location const* loc) noexcept;
    error_code& operator=<E>(E e) noexcept requires is_error_code_enum<E> || std::is_error_code_enum<E>;

    constexpr void clear() noexcept;
    constexpr int value() const noexcept;
    constexpr error_category const& value() const noexcept;
    error_condition default_error_condition() const noexcept;
    std::string message() const;
    char const* message(char* buf, size_t len) const noexcept;
    constexpr bool failed() const noexcept;
    constexpr explicit operator bool() const noexcept;
    bool has_location() const noexcept;
    source_location const& location() const noexcept;

    operator std::error_code () [const]; // Compatibility with std::error_code
    operator std::error_code& ();

    std::string to_string() const; // "{catname}:{ev}"
};

class error_condition {
    int val_ = 0; error_category const* cat_ = nullptr;
    constexpr ctor() noexcept;
    constexpr ctor(int val, error_category const& cat) noexcept;
    constexpr explicit ctor(generic_value_tag vt)  noexcept : val_(vt.value) {}
    constexpr ctor(errc_t e) noexcept : val_(e) {}
    constexpr ctor<E>(E e) noexcept requires is_error_condition_enum<E> || std::is_error_condition_enum<E>; // make_error_condition

    constexpr void assign(int val, error_category const& cat) noexcept;
    error_code& operator=<E>(E e) noexcept requires is_error_condition_enum<E> || std::is_error_condition_enum<E>;

    constexpr void clear() noexcept;
    constexpr int value() const noexcept;
    constexpr error_category const& value() const noexcept;
    std::string message() const;
    char const* message(char* buf, size_t len) const noexcept;
    constexpr bool failed() const noexcept;
    constexpr explicit operator bool() const noexcept;
};

constexpr bool operator==(error_condition const&, error_condition const&) noexcept; // !=, <, >, <=, >=
constexpr bool operator==(error_code const&, error_code const&) noexcept; // !=, <, >, <=, >=
// and all 6 comparations with E of is_error_condition_enum/is_error_code_enum, == and != with error_condition

size_t hash_value(error_code const&); // hash support

std::basic_ostream<Ch,Tr>& operator<< (std::basic_ostream<Ch,Tr>&, error_code const&)
std::basic_ostream<Ch,Tr>& operator<< (std::basic_ostream<Ch,Tr>&, error_condition const&)

// error code definitions
enum errc::errc_t { success = 0, address_family_not_supported = EAFNOSUPPORT, /*... POSIX errno */ };
is_error_condition_enum<errc::errc_t>::value = true;

error_code errc::make_error_code(err_t e) noexcept;
error_code errc::make_error_code(err_t e, source_location const* loc) noexcept;
error_condition errc::make_error_condition(err_t e) noexcept;
```

* `enum errc::errc_t` - mimic `enum class errc`
* Trait `is_error_code_enum<T>` and `is_error_condition_enum<T>`
* `class error_code` and `class error_condition`, implicit convertible to `std::error_code` and `std::error_condition`
* Function `make_error_code(errc_t)` and `make_error_condition(errc_t)`
* `class error_category`, implicit convertible to `std::error_category`
* Function `system_category()` and `generic_category()`
* System category accept `GetLastError()` on Windows.

#### Header `<boost/system/system_error.hpp>`

```c++
class system_error: public std::runtime_error {
    error_code code_; // wrapped data
public:
    explicit ctor(error_code const& ec);
    explicit ctor(error_code const& ec, {std::string const&|char const*} prefix);
    explicit ctor(int ev, error_category const& ecat);
    explicit ctor(int ev, error_category const& ecat, {std::string const&|char const*} prefix);

    error_code code() const noexcept;
};
```

#### Platform-specific

Header `<boost/system/linux_error.hpp>`, `<boost/system/windows_error.hpp>` and `<boost/system/cygwin_error.hpp>`

Provide platform-specific enum for additional `error_code`, and provide
corresponding `make_error_code()` overload for that enum type.

#### `result<T>`

Header `<boost/system/result.hpp>`

```c++
void throw_exception_from_error(error_code const& e, source_location const& loc);
// and errc::errc_t, std::error_code, std::errc, std::exception_ptr.

using in_place_value_t = variant2::in_place_index_t<0>;
constexpr in_place_value_t in_place_value{};
using in_place_error_t = variant2::in_place_index_t<1>;
constexpr in_place_error_t in_place_error{};

class result<T,E=error_code> {
    variant<T,E> v_; // wrapped value and error_code
public: // all functions are constexpr
    using value_type = T; using error_type = E;
    ctor() noexcept(is_nothrow_def_cons_v<T>) requires is_def_cons_v<T>;
    ctor<A=T> (A&& a) noexcept(is_nothrow_cons_v<T,A>); // requires A -> T, !(A -> E), T not arithmetic, 
    ctor<A=E> (A&& a) noexcept(is_nothrow_cons_v<E,A>); // requires A -> E, !(A -> T)
    explicit ctor<...A> (A&&... a) noexcept(is_nothrow_cons_v<T,A...>); // requires T(A...), !E(A...), T not arithmetic, 
    explicit ctor<...A> (A&&... a) noexcept(is_nothrow_cons_v<E,A...>); // requires E(A...), !T(A...)
    ctor<...A> (in_place_value_t, A&&... a) noexcept(is_nothrow_cons_v<T,A...>); // requires T(A...)
    ctor<...A> (in_place_error_t, A&&... a) noexcept(is_nothrow_cons_v<E,A...>); // requires E(A...)
    ctor<T2,E2>(result<T2,E2> {const&|&&} r2) noexcept(...); // requires T2->T, E2->E, !(result<T2,E2> const& -> T)

    bool has_value() const noexcept; bool has_error() const noexcept;
    explicit operator bool() const noexcept;

    T [const]{&|&&} value<U=T> (source_location const& loc=CURRENT_LOCATION) [const]{&|&&}; // if is_move_const<U>: && version returns T, &&const version is deleted.
    T [const]* operator->() [const] noexcept;
    T [const]& operator*() [const]& noexcept;
    T [const]&& operator* <U=T> () [const]&&; // if is_move_const<U>: && version returns T, &&const version is deleted.

    E error() {const&|&&};

    T& emplace<...A> (A&&...a);
    void swap(result& r) noexcept(noexcept(v_.swap(v_)));
};

class result<void,E> {
    variant<monostate,E> v_;
public: // all functions are constexpr
    using value_type = void; using error_type = E;
    ctor() noexcept;
    ctor<A> (A&& a) noexcept(is_nothrow_cons_v<E,A>); // requires A -> E
    explicit ctor<...A> (A&&... a) noexcept(is_nothrow_cons_v<E,A...>); // requires E(A...), !E(A...)
    ctor(in_place_value_t) noexcept;
    ctor<...A> (in_place_error_t, A&&... a) noexcept(is_nothrow_cons_v<E,A...>); // requires E(A...)
    ctor<E2>(result<void,E2> const& r2) noexcept(...); // requires E2->E

    bool has_value() const noexcept; bool has_error() const noexcept;
    explicit operator bool() const noexcept;

    void value (source_location const& loc=CURRENT_LOCATION) const;
    void [const]* operator->() [const] noexcept;
    void operator*() const noexcept;
    E error() {const&|&&};

    T& emplace();
    void swap(result& r) noexcept(noexcept(v_.swap(v_)));
};

using detail::reference_to_temporary<U,A> = !is_reference_v<A> || !is_convertible_v<remove_reference_t<A>*, U*>;

class result<U&,E> {
    variant<U*,E> v_; // wrapped value and error_code
public: // all functions are constexpr
    using value_type = U&; using error_type = E;
    ctor<A> (A&& a) noexcept(is_nothrow_cons_v<U&,A>); // requires A -> U&, !(A -> E), !ref_to_temp<U,A>
    ctor<A=E> (A&& a) noexcept(is_nothrow_cons_v<E,A>); // requires A -> E, !(A -> T)
    explicit ctor<A> (A&& a) noexcept(is_nothrow_cons_v<U&,A>); // requires U&(A), !E(A...), !ref_to_temp<U,A>, !(A->U&)
    explicit ctor<...A> (A&&... a) noexcept(is_nothrow_cons_v<E,A...>); // requires E(A...), !&&(A...)
    ctor<A> (in_place_value_t, A&& a) noexcept(is_nothrow_cons_v<U&,A>); // requires U&(A), !ref_to_temp<U,A>
    ctor<...A> (in_place_error_t, A&&... a) noexcept(is_nothrow_cons_v<E,A...>); // requires E(A...)
    ctor<U2,E2>(result<U2&,E2> const& r2) noexcept(...); // requires U2&->U&, E2->E, !(result<U2&,E2> const&->U&), !ref_to_temp<U,U2&>

    bool has_value() const noexcept; bool has_error() const noexcept;
    explicit operator bool() const noexcept;

    U& value<U=T> (source_location const& loc=CURRENT_LOCATION) const;
    U* operator->() const noexcept;
    U& operator*() const noexcept;
    E error() {const&|&&};

    U& emplace<A> (A&& a); // requires U&->A, !ref_to_temp<U,A>
    void swap(result& r) noexcept(noexcept(v_.swap(v_)));
};

void swap<T,E>(result<T,E>& r1, result<T,E>& r2) noexcept(noexcept(r1.swap(r2)));
bool operator== <T,E> (result<T,E>& r1, result<T,E>& r2) noexcept(noexcept(r1.v_==r2.v_)); // also !=
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,T,E> (std::basic_ostream<Ch,Tr>&, result<T,E> const&);

using detail::is_value_convertible_to<T,U> = is_lref<T> ? (is_lref<U> && is_conv<remove_ref<T>*, remove_ref<U>*>) : is_conv<T,U>;
// combining operator support
// `r | u` := `r ? *r : u`
T operator| <T,E,U> (result<T,E> {const&|&&} r, U&& u); // requires is_val_conv<T,U>
// `r | f` := `r ? *r : f()`
T operator| <T,E,F> (result<T,E> {const&|&&} r, F&& f); // requires is_val_conv<T,result_of<F>>
U operator| <T,E,F,U=result_of<F>> (result<T,E> {const&|&&} r, F&& f); // requires is_val_conv<T,result_of<F>>, U is result<>
result<T,E>& operator|= <T,E,U> (result<T,E>& r, U&& u); // requires is_val_conv<U,T>
result<T,E>& operator|= <T,E,F> (result<T,E>& r, F&& f); // requires is_val_conv<U,T>, is_val_conv<result_of<F>,T>
// `r & f` := `r ? f(*r) : r`
result<U,E> operator& <T,E,F,U=result_of<F(T const&)>> (result<T,E> {const&|&&} r, F&& f); // requires U !is result<>
U operator& <T,E,F,U=result_of<F(T const&)>> (result<T,E> {const&|&&} r, F&& f); // requires U is result<>
// `r & f` := `r ? f() : r`
result<U,E> operator& <E,F,U=result_of<F()>> (result<void,E> {const&|&&} r, F&& f); // requires U !is result<>
U operator& <E,F,U=result_of<F()>> (result<void,E> {const&|&&} r, F&& f); // requires U is result<>
result<T,E>& operator&= <T,E,F> (result<T,E>& r, F&& f); // requires ...
result<void,E>& operator&= <E,F> (result<void,E>& r, F&& f); // requires ...
```

#### Difference from Standard

* Non allocating `message` overload.
* `failed()`, error code != 0.
* Category id (64-bit value).
* `error_code` can attach source location.
* `result<T>`

#### Configuration

* `BOOST_SYSTEM_DISABLE_THREADS` macro to disable locking (make `mutex` noop).
* `BOOST_SYSTEM_AVOID_STD_GENERIC_CATEGORY` and `BOOST_SYSTEM_AVOID_STD_SYSTEM_CATEGORY` to avoid impl convert to the respective std instance.
* `BOOST_SYSTEM_USE_UTF8` to use `FormatMessageW` instead of `FormatMessageA` for Windows error message.

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`, `<boost/assert/source_location.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`
* `<boost/cstdint.hpp>`
x* `<boost/config/abi_prefix.hpp>`, `<boost/config/abi_suffix.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Variant2

* `<boost/variant2/variant.hpp>`

#### Boost.WinAPI

* `<boost/winapi/config.hpp>`, `<boost/winapi/error_codes.hpp>`

------
### Standard Facilities

Standard Library:
- `<system_error>` (C++11)
- `<expected>` (C++23)
