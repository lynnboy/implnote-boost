# Boost.SafeNumerics

* lib: `boost/libs/safe_numerics`
* repo: `boostorg/safe_numerics`
* commit: `c46d7a3`, 2024-08-19

------
### Concepts and Common Parts

* Header `<boost/safe_numerics/concept/numeric.hpp>`, `integer.hpp`, `promotion_policy.hpp`, `exception_policy.cpp`

```c++
using Numeric<T> = std::bool_constant<std::numeric_limits<T>::is_specialized>;
using Integer<T> = std::bool_constant<Numeric<T>() && std::numeric_limits<T>::is_integer>;
concept PromotionPolicy<PP,T,U> = requires {
    typename PP::addition_result<T,U>; typename PP::subtraction_result<T,U>;
    typename PP::multiplication_result<T>; typename PP::division_result<T,U>; typename PP::modulus_result<T,U>;
    typename = PP::{left|right}_shift_result<T,U>; typename PP::comparison_result<T,U>;
    typename = PP::bitwise_{and|or|xor}_result<T,U>;
};
concept ExceptionPolicy<EP> = requires (safe_numerics_error e, const char* message) {
    EP::on_arithmetic_error(e, message);
    EP::on_undefined_behavior(e, message);
    EP::on_implementation_defined_behavior(e, message);
    EP::on_uninitialized_value(e, message);
};

enum class safe_numerics_error : uint8_t { success, // no_action
    positive_overflow_error, negative_overflow_error, domain_error, range_error, precision_overflow_error, underflow_error, // arithmetic_error
    negative_value_shift, negative_shift, shift_too_large, // implementation_defined_behavior
    uninitialized_value, // uninitialized_value
};
constexpr inline const char* literal_string(const safe_numerics_error& e); // identifier of (e)
const uint8_t safe_numerics_casting_error_count = domain_error + 1, safe_numerics_error_count = uninitialized_value + 1;
std::is_error_code_enum<safe_numerics_error> : std::true_type{};
struct : std::error_category{
    const char* name() const noexcept override; //"safe numerics error"
    std::string message(int ev) const override; // literal_string
} const safe_numerics_error_category {}; // global single instance
std::error_code make_error_code(const safe_numerics_error& e) { return {(int)e, safe_numerics_error_category}; }

enum class safe_numerics_actions { no_action, // success
    uninitialized_value, // uninitialized_value
    arithmetic_error, // xxx_error
    implementation_defined_behavior, // shift XXX
    undefined_behavior,
};
std::is_error_condition_enum<safe_numerics_actions> : true_type {};
struct : std::error_category{
    const char* name() const noexcept override; //"safe numerics error group"
    std::string message(int ev) const override; // =name()
    bool equivalent(const std::error_code& code, int condition) const noexcept override; // UB always false
} const safe_numerics_actions_category {}; // global single instance

checked_result<R> { // constexpr
    const safe_numerics_error m_e;
    union contents { R m_r; char const* const m_msg
        ctor(const R&) noexcept; ctor(char const* msg) noexcept;
        operator R() noexcept; operator char const* () noexcept;
    } m_contents;
    ctor()=delete; ctor(self const&)=default; ctor(self&&)=default;
    ctor(const R& r) noexcept: m_e{success}, m_contents{r} {} // value
    ctor(safe_numerics_error const& e, const char* msg="") noexcept: m_e{e}, m_contents{msg} {} // error
    ctor<T>(self<T> const& t) noexcept : m_e{t.m_e} { if (t.m_e == success) m_contents.m_r = t.m_r; else m_contents.m_msg = t.m_msg; }
    bool exception() const { return m_e != success; }
    operator R() const noexcept { return m_cnotents.m_r; }
    operator safe_numerics_error() const noexcept { return m_e; }
    operator const char*() const noexcept { return m_conetnts.m_msg; }
};
struct make_checked_result<R> { static checked_result<R> invoke<e>(char const*& m) noexcept { return {e,m}; } };

struct heterogeneous_checked_operation<R,min,max,T> { checked_result<R> cast(const T& t) { return (R)t; } };
struct checked_operation<R> {
    static checked_result<R> minus(const R& t) noexcept { return -t; } // bitwise_not
    static checked_result<R> add(const R& t, const R& u) noexcept { return t + u; }
    // subtract, multiply, divide, modulus, left_shift, right_shift, bitwise_{or|xor|and}
    static logic::tribool equal(const R& t, const R& u) noexcept { return t == u; } // less_than, greater_than
};
checked_result<R> checked::cast<R,T>(const T& t) { return heterogeneous_checked_operation<R,std::numeric_limits<R>::min(),max(),T>::cast(t); }
checked_result<R> checked::minus<R>(const R& t) noexcept { checked_operation<R>::minus(t); } // bitwise_not
checked_result<R> checked::add<R>(const R& t, const R& u) noexcept { checked_operation<R>::add(t, u); } // subtract, multiply, divide, modulus, {less|greater}_than_<equal>, equal, {left|right}_shift, bitwise_{or|xor|and}

struct utility::print_value<n>{ enum test:char { value = n<0 ? n-256 : n+256 }; }; // CT printing value in warning message
struct utility::static_test<T>{}; struct utility::static_test<std::false_type>{[[deprecated]]ctor(){}}; // show in deprecating message
using utility::static_warning<T> = static_test<T>;

using utility::bits_type<T> = std::integral_constant<int,std::numeric_limits<T>::digits + (std::numeric_limits<T>::is_signed?1:0)>;
unsigned int utility::ilog2<T>(const T& t);
unsigned int utility::significant_bits<T>(const T& t);

using utility::signed_stored_type<min,max> = boost::int_t<std::max(significant_bits(min), significant_bits(max)) + 1>::least;
using utility::unsigned_stored_type<min,max> = boost::int_t<significant_bits(max)>::least;

std::pair<T,T> utility::minmax<T>(const std::initializer_list<T>& l);
T utility::round_out<T>(const T& t);
```

------
### Safe Numeric

* Header `<boost/safe_numerics/safe_integer.hpp>`

```c++
using detail::make_unsigned<T> = std::make_unsigned<T>;
struct detail::less_than<tsigned,usigned> { static bool invoke<T,U>(const T& t, const U& u) {
    if constexpr (tsigned != usigned) return u < 0 ? true : u < 0 ? false : (std::make_unsigned_t<T>&)t < (std::make_unsigned_t<U>&)u;
    else return t < u;
} };
bool less_than<T,U>(const T& lhs, const U& rhs) requires std::is_integral_v<T> && std::is_integral_v<U>
{ return less_than<std::is_signed_v<T>,std::is_signed_v<U>>::invoke(lhs, rhs); }
bool less_than<T,U>(const T& lhs, const U& rhs) requires std::is_floating_point_v<T> && std::is_floating_point_v<U> { return lhs < rhs; }
bool greater_than<T,U>(const T& lhs, const U& rhs) { return less_than(rhs, lhs); }
bool less_than_equal<T,U>(const T& lhs, const U& rhs) { return !greater_than(rhs, lhs); }
bool greater_than_equal<T,U>(const T& lhs, const U& rhs) { return !less_than(rhs, lhs); }

struct detail::equal<tsigned,usigned> { static bool invoke(const T& t, const U& u) {
    if constexpr (tsigned != usigned) return (t < 0 || u < 0) ? false : (std::make_unsigned_t<T>&)t == (std::make_unsigned_t<U>&)u;
    else return t == u;
} };
bool equal(const T& lhs, const U& rhs) requires std::is_integral_v<T> && std::is_integral_v<U>
{ return equal<std::is_signed_v<T>,std::is_signed_v<U>>::invoke(lhs, rhs); }
bool equal<T,U>(const T& lhs, const U& rhs) requires std::is_floating_point_v<T> && std::is_floating_point_v<U> { return lhs == rhs; }
bool not_equal<T,U>(const T& lhs, const U& rhs) { return !equal(rhs, lhs); }
```

------
### Dependency

#### Boost.ConceptCheck

* `<boost/concept/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Core

* `<boost/core/enable_if.hpp>`

#### Boost.Integer

* `<boost/integer.hpp>`

#### Boost.Logic

* `<boost/logic/tribool.hpp>`

#### Boost.MP11

* `<boost/mp11.hpp>`

------
### Standard Facilities
