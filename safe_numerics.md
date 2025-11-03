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

struct is_safe<T> : std::false_type {};
struct base_type<T> { using type = T; };
const base_type<T>::type& base_value<T>(const T& t) { return (base_type<T>::type&)t; }
struct get_promotion_policy<T> { using type = void; };
struct get_exception_policy<T> { using type = void; };

struct exception_policy<AE,IDB,UB,UV> {
    static void on_arithmetic_error(const safe_numerics_error& e, const char* msg) { AE{}(e, msg); }
    static void on_implementation_defined_behavior(const safe_numerics_error& e, const char* msg) { IDB{}(e, msg); }
    static void on_undefined_behavior(const safe_numerics_error& e, const char* msg) { UB{}(e, msg); }
    static void on_uninitialized_value(const safe_numerics_error& e, const char* msg) { UV{}(e, msg); }
};

struct ignore_exception { void operator()(const safe_numerics_error&, const char*) {} };
struct trap_exception { };
struct throw_exception { void operator()(const safe_numerics_error& e, const char* msg) { throw std::system_error(std::error_code{e}, msg); } };
safe_numerics_actions make_safe_numerics_action(const safe_numerics_error& e) {
    if (e <= underflow_error) return arithmetic_error;
    if (e <= shift_too_large) return implementation_defined_behavior;
    if (e == uninitialized_value) return uninitialized_value;
    if (e == success) return no_action;
    assert(false);
}
using loose_exception_policy = exception_policy<throw_exception, ignore_exception, ignore_exception, ignore_exception>;
using loose_trap_policy = exception_policy<trap_exception, ignore_exception, ignore_exception, ignore_exception>;
using strict_exception_policy = exception_policy<throw_exception, throw_exception, throw_exception, ignore_exception>;
using strict_trap_policy = exception_policy<trap_exception, trap_exception, trap_exception, trap_exception>;
using default_exception_policy = strict_exception_policy;

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

struct interval<R> {
    const R l = std::numeric_limits<R>::min(); // lowest() for float/double
    const R u = std::numeric_limits<R>::max();
    ctor<T>(const T& lower, const T& uppwer) : l{lower}, u{upper}{}
    ctor<T>(const std::pair<T,T>& p) : l{p.first}, u{p.second} {}
    ctor<T>(const self<T>& rhs) : l{rhs.l}, u{rhs.u} {}
    tribool includes(const R& t) const { return l <= t && t <= u; }
    tribool includes<const self<R>& t> const { return u >= t.u && l <= t.l; }
    tribool excludes(const R& t) const { return t < l || t > u; }
    tribool excludes<const self<R>& t> const { return t.u < l || u < t.l; }
};
interval<R> make_interval<R>(<const R&>) { return {}; }

interval<T> operator+ <T>(const interval<T>& t, const interval<T>& u) { return {t.l + u.l, t.u + u.u}; }
interval<T> operator- <T>(const interval<T>& t, const interval<T>& u) { return {t.l - u.u, t.u - u.l}; }
interval<T> operator* <T>(const interval<T>& t, const interval<T>& u) { return minmax<T>({t.l*u.l, t.l*u.u, t.u*u.l, t.u*u.u}); }
interval<T> operator/ <T>(const interval<T>& t, const interval<T>& u) { return minmax<T>({t.l/u.l, t.l/u.u, t.u/u.l, t.u/u.u}); }
interval<T> operator% <T>(const interval<T>& t, const interval<T>& u) { return minmax<T>({t.l%u.l, t.l%u.u, t.u%u.l, t.u%u.u}); }
interval<T> operator<< <T>(const interval<T>& t, const interval<T>& u) { return minmax<T>({t.l<<u.l, t.l<<u.u, t.u<<u.l, t.u<<u.u}); }
interval<T> operator>> <T>(const interval<T>& t, const interval<T>& u) { return minmax<T>({t.l>>u.l, t.l>>u.u, t.u>>u.l, t.u>>u.u}); }
interval<T> operator| <T>(const interval<T>& t, const interval<T>& u) { return {std::min(t.l,u.l), std::max(t.u,u.u)}; }
interval<T> operator& <T>(const interval<T>& t, const interval<T>& u) { return {std::max(t.l,u.l), std::min(t.u,u.u)}; }
logic::tribool intersect<T>(const interval<T>& t, const interval<T>& u) { return t.u >= u.l || t.l <= u.u; }
logic::tribool operator< <T>(const interval<T>& t, const interval<T>& u) { return t.u<u.l?true : t.l>u.u?false : logic::indeterminate; }
logic::tribool operator> <T>(const interval<T>& t, const interval<T>& u) { return t.l>u.u?true : t.u<u.l?false : logic::indeterminate; }
bool operator== <T>(const interval<T>& t, const interval<T>& u) { return t.l==u.l && t.u==u.u; }
bool operator!= <T>(const interval<T>& t, const interval<T>& u) { return !(t==u); }
bool operator<= <T>(const interval<T>& t, const interval<T>& u) { return !(t>u); }
bool operator>= <T>(const interval<T>& t, const interval<T>& u) { return !(t<u); }
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,T> (std::basic_ostream<Ch,Tr>& os, const interval<T>& i) { return os << '[' << i.l << ',' << i.u << ']'; }
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, const interval<unsigned char>& i) { return os << '[' << (unsigned)i.l << ',' << (unsigned)i.u << ']'; }
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, const interval<signed char>& i) { return os << '[' << (int)i.l << ',' << (int)i.u << ']'; }
```

------
### Checked Result

* Header `<boost/safe_numerics/checked_result_operations.hpp>`

```c++
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

struct heterogeneous_checked_operation<R,min,max,T,F=make_checked_result<R>,Default=void> { checked_result<R> cast(const T& t) { return (R)t; } };
struct checked_operation<R> {
    static checked_result<R> minus(const R& t) noexcept { return -t; } // bitwise_not
    static checked_result<R> add(const R& t, const R& u) noexcept { return t + u; }
    // subtract, multiply, divide, modulus, left_shift, right_shift, bitwise_{or|xor|and}
    static logic::tribool equal(const R& t, const R& u) noexcept { return t == u; } // less_than, greater_than
};
checked_result<R> checked::cast<R,T>(const T& t) { return heterogeneous_checked_operation<R,std::numeric_limits<R>::min(),max(),T>::cast(t); }
checked_result<R> checked::minus<R>(const R& t) noexcept { checked_operation<R>::minus(t); } // bitwise_not
checked_result<R> checked::add<R>(const R& t, const R& u) noexcept { checked_operation<R>::add(t, u); } // subtract, multiply, divide, modulus, {less|greater}_than_<equal>, equal, {left|right}_shift, bitwise_{or|xor|and}

using bool_type<tf> = std::bool_constant<tf>;
struct heterogeneous_checked_operation<R,min,max,T,F> requires std::is_integral_v<R> && std::is_integral_v<T> {
    static checked_result<R> cast(const T& t) {
        if constexpr (std::is_signed_v<R> && std::is_signed_v<T>)
            return safe_compare::greater_than(t, max) ? F::invoke<positive_overflow_error>("...")
                : safe_compare::less_than(t, min) ? F::invoke<negative_overflow_error>("...") : {(R)t};
        else if constexpr (!std::is_signed_v<R> && std::is_signed_v<T>)
            return safe_compare::less_than(t, min) ? F::invoke<domain_error>("...")
                : safe_compare::greater_than(t, max) ? F::invoke<positive_overflow_error>("...") : {(R)t};
        else
            return safe_compare::greater_than(t, max) ? F::invoke<positive_overflow_error>("...")
                : safe_compare::less_than(t, min) ? F::invoke<positive_overflow_error>("...") : {(R)t};
    }
};
struct heterogeneous_checked_operation<R,min,max,T,F> requires std::is_integral_v<R> && std::is_floating_point_v<T>
{ static checked_result<R> cast(const T& t) { return (R)t; } };
struct heterogeneous_checked_operation<R,min,max,T,F> requires std::is_floating_point_v<R> && std::is_integral_v<T> {
    static checked_result<R> cast(const T& t) {
        if (std::numeric_limits<R>::digits < std::numeric_limits<T>::digits &&
            utility::significant_bits(t) > std::numeric_limits<R>::digits) return F::invoke(precision_overflow_error, "");
        return t;
    }
};

struct checked_operation<R,F> requires std::is_integral_v<R> { using l = std::numeric_limits<R>;
    static checked_result<R> add(const R& t, const R& u) {
        if constexpr (std::is_signed_v<R>)
            return (u>0 && t>l::max()-u) ? F::invoke<positive_overflow_error>("...")
                : (u<0 && t<l::min()-u) ? F::invoke<negative_overflow_error>("...") : {t+u};
        else return (l::max()-u < t) ? F::invoke<positive_overflow_error>("...") : {t+u};
    }
    static checked_result<R> subtract(const R& t, const R& u) {
        if constexpr (std::is_signed_v<R>)
            return (u>0 && t<l::min()+u) ? F::invoke<negative_overflow_error>("...")
                : (u<0 && t>l::max()+u) ? F::invoke<positive_overflow_error>("...") : {t-u};
        else return (t<u) ? F::invoke<negative_overflow_error>("...") : {t-u};
    }
    static checked_result<R> minus(const R& t) {
        if constexpr (std::is_signed_v<R>)
            return (t==l::min()) ? F::invoke<positive_overflow_error>("...") : {-t};
        else return (t>0) ? F::invoke<negative_overflow_error>("...") : {0};
    }
    static checked_result<R> multiply(const R& t, const R& u) {
        constexpr bool rsigned = std::is_signed_v<R>, rbig = sizeof(R) > sizeof(uintmax_t)/2;
        if constexpr (rsigned && rbig)
            if (t > 0) if (u > 0) return t > l::max()/u ? F::invoke<positive_overflow_error>("...") : {t*u};
                        else return u < l::min()/t ? F::invoke<negative_overflow_error>("...") : {t*u};
            else if (u > 0) return t < l::min()/u ? F::invoke<negative_overflow_error>("...") : {t*u};
                    else return t!=0 && u < l::max()/t ? F::invoke<positive_overflow_error>("..") : {t*u};
        else if constexpr (rsigned && !rbig)
            return intmax_t(t) * intmax_t(u) > int_max_t(l::max()) ? F::invoke<positive_overflow_error>("...")
                : intmax_t(t) * intmax_t(u) < int_max_t(l::min()) ? F::invoke<negative_overflow_error>("...") : {t*u};
        else if constexpr (!rsigned && big)
            return u > 0 && t > l::max()/u ? F::invoke<positive_overflow_error>("...") : {t*u};
        else return uintmax_t(t) * uintmax_t(u) > l::max() ? F::invoke<positive_overflow_error>("...") : {t*u};
    }
    static checked_result<R> divide(const R& t, const R& u) {
        if (u == 0) return F::invoke<domain_error>("...");
        if constexpr (std::is_signed_v<R>)
            return u==-1 && t==l::min() ? F::invoke<positive_overflow_error>("..") : {t/u};
        else return {t/u};
    }
    static checked_result<R> modulus(const R& t, const R& u) {
        if (u == 0) return F::invoke<domain_error>("...");
        if constexpr (std::is_signed_v<R>) {
            if (u >= 0) return t % u;
            auto ux = checked::minus(u); return ux.exception() ? t : t % (R)ux;
        } else return {t%u};
    }
    static checked_result<R> left_shift(const R& t, const R& u) {
        if (u == 0) return t;
        if (u < 0) return F::invoke<negative_shift>("...");
        if (u > l::digits) return F::invoke<shift_too_large>("...");
        if constexpr (std::is_signed_v<R>) {
            if (t >= 0) if (safe_compare::greater_than(u, l::digits-significant_bits(t))) return F::invoke<shift_too_large>("...");
                        else return t << u;
            else return F::invoke<negative_shift>("...");
        } else if (safe_compare::greater_than(u, l::digits-significant_bits(t))) return F::invoke<shift_too_large>("...");
                else return t << u;
    }
    static checked_result<R> right_shift(const R& t, const R& u) {
        if (u < 0) return F::invoke<negative_shift>("...");
        if (u > l::digits) return F::invoke<shift_too_large>("...");
        if constexpr (std::is_signed_v<R>) return (t < 0) ? F::invoke<negative_value_shift>("...") : (t >> u);
        else return t >> u;
    }
    static checked_result<R> bitwise_{or|xor|and}(const R& t, const R& u) {
        if (std::max(significant_bits(t),significant_bits(u)) > bits_type<R>::value)
            return F::invoke<positive_overflow_error>("...");
        return t | u; // t ^ u; t & u
    }
    static checked_result<R> bitwise_not(const R& t) {
        if (significant_bits(t) > bits_type<R>::value) return F::invoke<positive_overflow_error>("...");
        return ~t; // t ^ u; t & u
    }
};

void display<T>(checked_result<T>& c); // just terminate()
struct sum_value_type {
    enum flag { known_value, less_than_min, greater_than_max, indeterminate, count } const m_flag;
    static flag to_flat<T>(const checked_result<T>& t) const {
        switch(t.m_e) {
            case success: return known_value;
            case negative_overflow_error: return less_than_min;
            case positive_overflow_error: return greater_than_max;
            default: return indeterminate;
        }
    }
    ctor(const checked_result<T>& t) : m_flag{to_flag(t)} {}
    operator uint8_t() const{ return {m_flag}; }
};
checked_result<T> operator+<T>(const checked_result<T>& t, const checked_result<T>& u) requires std::is_integral_v<T> {
    const enum safe_numerics_error result[4*4] = {s,n,p,r, n,n,r,r, p,r,p,r, r,r,r,r};
    auto e = result[ sum_value_type{t} * 4 + sum_value_type{u} ];
    return e == success ? checked::add<T>(t, u) : {e, ""};
};
checked_result<T> operator+<T>(const checked_result<T>& t) requires std::is_integral_v<T> { return t; }
checked_result<T> operator-<T>(const checked_result<T>& t, const checked_result<T>& u) requires std::is_integral_v<T> {
    const enum safe_numerics_error result[4*4] = {s,p,n,r, n,r,n,r, p,p,r,r, r,r,r,r};
    auto e = result[ sum_value_type{t} * 4 + sum_value_type{u} ];
    return e == success ? checked::subtract<T>(t, u) : {e, ""};
};
checked_result<T> operator-<T>(const checked_result<T>& t) requires std::is_integral_v<T> { return checked_result<T>{0} - t; }

struct product_value_type {
    enum flag { less_than_min, less_than_zero, zero, greater_than_zero, greater_than_max, indeterminate, count, t_value, u_value, z_value } const m_flag;
    static flag to_flat<T>(const checked_result<T>& t) const {
        switch(t.m_e) {
            case success: return t < 0 ? less_than_zero : t > 0 : greater_than_zero : zero;
            case negative_overflow_error: return less_than_min;
            case positive_overflow_error: return greater_than_max;
            default: return indeterminate;
        }
    }
    ctor(const checked_result<T>& t) : m_flag{to_flag(t)} {}
    operator uint8_t() const{ return {m_flag}; }
};
checked_result<T> operator*<T>(const checked_result<T>& t, const checked_result<T>& u) requires std::is_integral_v<T> {
    const enum safe_numerics_error result[6*6] = {gm,gm,z,lm,lm,i, gm,gz,z,lz,lm,i, z,z,z,z,z,i, lm,lz,z,gz,gm,i, lm,lm,z,gm,gm,i, i,i,i,i,i,i};
    switch(result[ product_value_type{t} * 6 + product_value_type{u} ]) {
        case less_than_min: return negative_overflow_error;
        case zero: return T{0};
        case greater_than_max: return positive_overflow_error;
        case less_than_zero: case greater_than_zero: return checked::multiply<T>(t, u);
        case indeterminate: return range_error;
        default: assert(false);
    }
};
checked_result<T> operator/<T>(const checked_result<T>& t, const checked_result<T>& u) requires std::is_integral_v<T> {
    const enum safe_numerics_error result[6*6] = {i,gm,lm,lm,lm,i, z,gz,lm,lz,z,i, z,z,i,z,z,i, z,lz,gm,gz,z,i, lm,lm,gm,gm,i,i, i,i,i,i,i,i};
    switch(result[ product_value_type{t} * 6 + product_value_type{u} ]) {
        case less_than_min: return negative_overflow_error;
        case zero: return T{0};
        case greater_than_max: return positive_overflow_error;
        case less_than_zero: case greater_than_zero: return checked::divide<T>(t, u);
        case indeterminate: return range_error;
        default: assert(false);
    }
};
checked_result<T> operator%<T>(const checked_result<T>& t, const checked_result<T>& u) requires std::is_integral_v<T> {
    const enum safe_numerics_error result[6*6] = {i,zv,i,zv,i,i, tv,gz,i,lz,t,i, z,z,i,z,z,i, tv,lz,i,gz,tv,i, i,uv,i,uv,i,i, i,i,i,i,i,i};
    switch(result[ product_value_type{t} * 6 + product_value_type{u} ]) {
        case zero: return T{0};
        case less_than_zero: case greater_than_zero: return checked::modulus<T>(t, u);
        case indeterminate: return range_error;
        case t_value: return t; case u_value: return checked::subtract<T>(u, 1); case z_value: return checked::subtract<T>(1, u)
        default: assert(false);
    }
};

logic::tribool operator< <T>(const checked_result<T>& t, const checked_result<T>& u) {
    enum class result_type : uint8_t { runtime, false_value, true_value, indeterminate };
    const result_type resultx[4*4] = {r,f,t,i, t,i,t,i, f,f,i,i, i,i,i,i};
    switch (resultx[sum_value_type{t}*4 + sum_value_type{u}]) {
        case runtime: return (const T&)t < (const T&)u;
        case false_value: return false; case true_value: return true;
        case indeterminate: return logic::indeterminate;
        default: assert(false);
    }
}
logic::tribool operator>= <T>(const checked_result<T>& t, const checked_result<T>& u) { return !(t<u); }
logic::tribool operator> <T>(const checked_result<T>& t, const checked_result<T>& u) { return u < t; }
logic::tribool operator<= <T>(const checked_result<T>& t, const checked_result<T>& u) { return !(u<t); }
logic::tribool operator== <T>(const checked_result<T>& t, const checked_result<T>& u) {
    enum class result_type : uint8_t { runtime, false_value, true_value, indeterminate };
    const result_type resultx[4*4] = {r,f,f,i, f,i,f,i, f,f,i,i, i,i,i,i};
    switch (resultx[sum_value_type{t}*4 + sum_value_type{u}]) {
        case runtime: return (const T&)t == (const T&)u;
        case false_value: return false; case true_value: return true;
        case indeterminate: return logic::indeterminate;
        default: assert(false);
    }
}
logic::tribool operator!= <T>(const checked_result<T>& t, const checked_result<T>& u) { return !(t==u); }

checked_result<T> operator~ <T>(const checked_result<T>& t) requires std::is_integral_v<T> { return ~t.m_r; }
checked_result<T> operator<< <T>(const checked_result<T>& t, const checked_result<T>& u) requires std::is_integral_v<T> {
    const uint8_t result[6*6] = { 1,2,2,2,2,1, 3,4,5,6,2,1, 3,3,3,3,3,3, 3,7,5,8,9,1, 1,9,9,9,9,1, 1,1,1,1,1,1};
    switch(result[ product_value_type{t} * 6 + product_value_type{u} ]) {
        case 1: return range_error;
        case 2: return negative_overflow_error;
        case 3: return {0};
        case 4: return t >> -u;
        case 5: return t;
        case 6: { checked_result<T> temp_t = t*2, temp_u = u-1; return -(-tempt << temp_u); }
        case 7: return t >> -u;
        case 8: { checked_result<T> r = checked::left_shift<T>(t,u); return r.m_e == shift_too_large ? positive_overflow_error : r; }
        case 9: return positive_overflow_error;
        default: assert(false);
    }
}
checked_result<T> operator>> <T>(const checked_result<T>& t, const checked_result<T>& u) requires std::is_integral_v<T> {
    const uint8_t result[6*6] = {2,2,2,2,1,1, 2,4,5,6,3,1, 3,3,3,3,3,3, 9,7,5,8,3,1, 9,9,9,9,1,1, 1,1,1,1,1,1};
    switch(result[ product_value_type{t} * 6 + product_value_type{u} ]) {
        case 1: return range_error;
        case 2: return negative_overflow_error;
        case 3: return {0};
        case 4: return t << -u;
        case 5: return t;
        case 6: { checked_result<T> temp_t = t/2, temp_u = u-1; return -(-tempt >> temp_u); }
        case 7: return t << -u;
        case 8: { checked_result<T> r = checked::left_shift<T>(t,u); return r.m_e == shift_too_large ? 0 : r; }
        case 9: return positive_overflow_error;
        default: assert(false);
    }
}
checked_result<T> operator| <T>(const checked_result<T>& t, const checked_result<T>& u) requires std::is_integral_v<T>
{ return t.exception() || u.exception() ? range_error : checked::bitwise_or<T>((T)t, (T)u); }
checked_result<T> operator^ <T>(const checked_result<T>& t, const checked_result<T>& u) requires std::is_integral_v<T>
{ return t.exception() || u.exception() ? range_error : checked::bitwise_xor<T>((T)t, (T)u); }
checked_result<T> operator& <T>(const checked_result<T>& t, const checked_result<T>& u) requires std::is_integral_v<T>
{ return t.exception() || u.exception() ? range_error : checked::bitwise_and<T>((T)t, (T)u); }

std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,R> (std::basic_ostream<Ch,Tr>& os, const checked_result<R>& r) {
    bool e = r.exception(); os << e;
    if (!e) os << (R)r; else os << std::error_code(r.m_e).message() << ':' << (char const*)r;
    return os;
}
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, const checked_result<signed char>& r) {
    bool e = r.exception(); os << e;
    if (!e) os << (int16_t)r; else os << std::error_code(r.m_e).message() << ':' << (char const*)r;
    return os;
}
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr,R> (std::basic_istream<Ch,Tr>& os, checked_result<R>& r) {
    bool e; is >> e;
    if (!e) is >> (R)r; else is >> std::error_code(r.m_e).message() >> ':' >> (char const*)r;
    return is;
}
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr> (std::basic_istream<Ch,Tr>& os, checked_result<signed char>& r) {
    bool e; is >> e;
    if (!e) { int16_t i; is >> i; r.m_contents.m_r = (signed char)i; }
    else is >> std::error_code(r.m_e).message() >> ':' >> (char const*)r;
    return is;
}

struct std::numeric_limits<checked_result<R>> : std::numeric_limits<R> {
    static checked_result<R> min() noexcept { return {base::min()}; }
    static checked_result<R> max() noexcept { return {base::max()}; }
};

```

------
### Safe Integer

* Header `<boost/safe_numerics/safe_integer.hpp>`

```c++
struct is_safe<safe_base<T,min,max,P,E>> : std::true_type {};
struct get_promotion_policy<safe_base<T,min,max,P,E>> { using type = P; };
struct get_exception_policy<safe_base<T,min,max,P,E>> { using type = E; };
struct base_type<safe_base<T,min,max,P,E>> { using type = T; };
T base_value(const safe_base<T,min,max,P,E>& st) { return (T)st; }

class safe_base<Stored,min,max,P,E> { Stored m_t; using l = std::numeric_limits<R>;
    Stored validated_cast<T>(const T& t) const { return validate_detail<Stored,min,max,E>::return_value(t); }
    void output<Ch,Tr>(std::basic_ostream<Ch,Tr>& os) const
    { os << ((std::is_same_v<T,signed char> || is_same_v<T,unsigned char> || is_same_v<T,wchar_t>) ? (int)m_t : m_t); }
    friend std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr>(std::basic_ostream<Ch,Tr>& os, const self& t) { t.output(os); return os; }
    void input<Ch,Tr>(std::basic_istream<Ch,Tr>& is) {
        if (std::is_same_v<T,signed char> || is_same_v<T,unsigned char> || is_same_v<T,wchar_t>)
        { is >> std::ws; if (is.peek() == '-') is.setstate(is.failbit); }
        is >> m_t; if (is.fail()) dispatch<E,domain_error>("..."); else validated_cast(m_t);
    }
    friend std::basic_istream<Ch,Tr>& operator>> <Ch,Tr>(std::basic_istream<Ch,Tr>& is, self& t) { t.input(is); return is; }
public: struct skip_validation{};
    ctor() {dispatch<E,uninitialized_value>("...");}
    explicit ctor(const Stored& rhs, skip_validation) : m_t{rhs} {}
    ctor<T>(const T& t) requires std::is_convertible_v<T,Stored> : m_t{validated_cast(t)} {}
    ctor<T, T n, Px,Ex>(const safe_literal_impl<T,n,Px,Ex>& t) : m_t{validated_cast(t)} {}
    ~dtor()=default; // copy/move ctor and assig =default
    operator R <R> () const requires is_safe<R>::value { return validate_detail<R,l::min(),l::max(),E>::return_value(m_t); }
    self& operator= <T> (const T& rhs) { m_t = validated_cast(rhs); return *this; }
    self& operator++() { return *this = *this + 1; } // and --
    self& operator++(int) { self old_t = *this; ++(*this); return old_t; } // and --
    auto operator+() const { return *this; } auto operator-() const { return 0 - *this;} auto operator~()const { return ~Stored{0u} ^ *this; }
};
struct std::numeric_limits<safe_base<T,min,max,P,E>> : std::numeric_limits<T> {
    static safe_base lowest() noexcept { return {min, skip_validation{}}; }
    static safe_base min() noexcept { return {min, skip_validation{}}; }
    static safe_base max() noexcept { return {max, skip_validation{}}; }
};

struct dispatch_switch::dispatch_case<EP,act> {};
struct dispatch_switch::dispatch_case<EP,uninitialized_value>
{ static void invoke(const safe_numerics_error& e, const char* msg) { EP::on_uninitialized_value(e,msg); } };
struct dispatch_switch::dispatch_case<EP,arithmetic_error>
{ static void invoke(const safe_numerics_error& e, const char* msg) { EP::on_arithmetic_error(e,msg); } };
struct dispatch_switch::dispatch_case<EP,implementation_defined_behavior>
{ static void invoke(const safe_numerics_error& e, const char* msg) { EP::on_implementation_defined_behavior(e,msg); } };
struct dispatch_switch::dispatch_case<EP,undefined_behavior>
{ static void invoke(const safe_numerics_error& e, const char* msg) { EP::on_undefined_behavior(e,msg); } };
void dispatch<EP,err>(const char* msg) { dispatch_switch::dispatch_case<EP,make_safe_numerics_action(err)>::invoke(err, msg); }
struct dispatch_and_return<EP,R> { static checked_result<R> invoke<err>(const char*& msg) { dispatch<EP,err>(msg); return {err, msg}; } }

struct validate_detail<R,min,max,E> {
    using r_type = checked_result<R>; using l = std::numeric_limits<T>;
    struct exception_possible { static R return_value<T>(const T& t)
    { return heterogeneous_checked_operation<R,min,max,base_type<T>::type, dispatch_and_return<E,R>>::cast(t); } };
    struct exception_not_possible { static R return_value<T>(const T& t){ return (R)base_value(t); } };
    static R return_value<T>(const T& t) {
        constexpr interval<r_type> t_interval{checked::cast<R>(base_value(l::min())), checked::cast<R>(base_value(l::max()))},
                r_interval{r_type{min}, r_type{max}};
        return std::conditional_t<r_interval.includes(t_interval), exception_not_possible, exception_policy>::return_value(t);
    }
};

struct common_exception_policy<T,U> {
    using t_ep = get_exception_policy<T>::type; using u_ep = get_exception_policy<U>::type;
    using type = std::conditional_t<!is_same_v<void,u_ep>, u_ep, conditional_t<!is_same_v<void,t_ep>, t_ep, void>>;
};
struct common_promotion_policy<T,U> {
    using t_pp = get_promotion_policy<T>::type; using u_pp = get_promotion_policy<U>::type;
    using type = std::conditional_t<!is_same_v<void,u_pp>, u_pp, conditional_t<!is_same_v<void,t_pp>, t_pp, void>>;
};

std::pair<R,R> casting_helper<EP,R,T,U> (const T& t, const U& u) {
    using r_type = checked_result<R>; using l = std::numeric_limits<R>;
    const r_type tx = heterogeneous_checked_operation<R,l::min(),l::max(),base_type<T>::type,dispatch_and_return<EP,R>>::cast(base_value(t));
    const r_type ux = heterogeneous_checked_operation<R,l::min(),l::max(),base_type<U>::type,dispatch_and_return<EP,R>>::cast(base_value(u));
    const R tr = tx.exception ? (R)t : tx.m_conetnts.m_r, ur = ux.exception ? (R)u : ux.m_conetnts.m_r; return {tr,ur};
}
using anon::legal_overload<F<...>,T,U> = mp_and< mp_or<is_safe<T>,is_safe<U>>, mp_valid<F,base_type<T>::type,base_type<U>::type>>;

class addition_result<T,U> {
    using promotion_policy = common_promotion_policy<T,U>::type; using exception_policy = common_exception_policy<T,U>::type;
    using result_base_type = promotion_policy::addition_result<T,U>::type; using r_type = checked_result<result_base_type>;
    using lt = std::numeric_limits<T>; using lu = std::numeric_limits<U>; using l = std::numeric_limits<result_base_type>;
    constexpr interval<r_type> ri = interval<r_type>{checked::cast<result_base_type>(base_value(lt::min())),cast(base_value(lt::max()))}
                                + interval<r_type>{checked::cast<result_base_type>(base_value(lu::min())),cast(base_value(lu::max()))};
    constexpr interval<result_base_type> reti{ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u};
public: using type = safe_base<result_base_type, reti.l, reti.u, promotion_policy, exception_policy>;
    static type return_value(const T& t, const U& u) {
        if constexpr (ri.l.exception() || ri.u.exception() || !reti.includes(ri)) {
            auto r = casting_helper<exception_policy,result_base_type>(t,u);
            auto rx = checked_operation<result_base_type,dispatch_and_return<exception_policy,result_base_type>>::add(r.first,r.second);
            return {rx.exception()?r.first+r.second:rx.m_contents.m_r, skip_validation{}};
        } else return {(result_base_type)base_value(t) + (result_base_type)base_value(u), skip_validation{}};
    }
};
using addition_operator<T,U> = decltype(std::declval<T const&>()+std::declval<U const&>());
addition_result<T,U>::type operator+ <T,U> (const T& t, const U& u) requires legal_overload<addition_operator,T,U>::value { return addition_result<T,U>::return_value(t,u); }
T operator+= <T,U> (T& t, const U& u) requires legal_overload<addition_operator,T,U>::value { t = (T)(t + u); return t; }

class subtraction_result<T,U> {
    using promotion_policy = common_promotion_policy<T,U>::type; using exception_policy = common_exception_policy<T,U>::type;
    using result_base_type = promotion_policy::subtraction_result<T,U>::type; using r_type = checked_result<result_base_type>;
    using lt = std::numeric_limits<T>; using lu = std::numeric_limits<U>; using l = std::numeric_limits<result_base_type>;
    constexpr interval<r_type> ri = interval<r_type>{checked::cast<result_base_type>(base_value(lt::min())),cast(base_value(lt::max()))}
                                - interval<r_type>{checked::cast<result_base_type>(base_value(lu::min())),cast(base_value(lu::max()))};
    constexpr interval<result_base_type> reti{ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u};
public: using type = safe_base<result_base_type, reti.l, reti.u, promotion_policy, exception_policy>;
    static type return_value(const T& t, const U& u) {
        if constexpr (ri.l.exception() || ri.u.exception() || !reti.includes(ri)) {
            auto r = casting_helper<exception_policy,result_base_type>(t,u);
            auto rx = checked_operation<result_base_type,dispatch_and_return<exception_policy,result_base_type>>::subtract(r.first,r.second);
            return {rx.exception()?r.first+r.second:rx.m_contents.m_r, skip_validation{}};
        } else return {(result_base_type)base_value(t) - (result_base_type)base_value(u), skip_validation{}};
    }
};
using subtraction_operator<T,U> = decltype(std::declval<T const&>() - std::declval<U const&>());
subtraction_result<T,U>::type operator- <T,U> (const T& t, const U& u) requires legal_overload<subtraction_operator,T,U>::value { return subtraction_result<T,U>::return_value(t,u); }
T operator-= <T,U> (T& t, const U& u) requires legal_overload<subtraction_operator,T,U>::value { t = (T)(t - u); return t; }

class multiplication_result<T,U> {
    using promotion_policy = common_promotion_policy<T,U>::type; using exception_policy = common_exception_policy<T,U>::type;
    using result_base_type = promotion_policy::multiplication_result<T,U>::type; using r_type = checked_result<result_base_type>;
    using lt = std::numeric_limits<T>; using lu = std::numeric_limits<U>; using l = std::numeric_limits<result_base_type>;
    constexpr interval<r_type> ri = interval<r_type>{checked::cast<result_base_type>(base_value(lt::min())),cast(base_value(lt::max()))}
                                * interval<r_type>{checked::cast<result_base_type>(base_value(lu::min())),cast(base_value(lu::max()))};
    constexpr interval<result_base_type> reti{ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u};
public: using type = safe_base<result_base_type, reti.l, reti.u, promotion_policy, exception_policy>;
    static type return_value(const T& t, const U& u) {
        if constexpr (ri.l.exception() || ri.u.exception() || !reti.includes(ri)) {
            auto r = casting_helper<exception_policy,result_base_type>(t,u);
            auto rx = checked_operation<result_base_type,dispatch_and_return<exception_policy,result_base_type>>::multiply(r.first,r.second);
            return {rx.exception()?r.first*r.second:rx.m_contents.m_r, skip_validation{}};
        } else return {(result_base_type)base_value(t) * (result_base_type)base_value(u), skip_validation{}};
    }
};
using multiplication_operator<T,U> = decltype(std::declval<T const&>() * std::declval<U const&>());
multiplication_result<T,U>::type operator* <T,U> (const T& t, const U& u) requires legal_overload<multiplication_operator,T,U>::value { return multiplication_result<T,U>::return_value(t,u); }
T operator*= <T,U> (T& t, const U& u) requires legal_overload<multiplication_operator,T,U>::value { t = (T)(t * u); return t; }

class division_result<T,U> {
    using promotion_policy = common_promotion_policy<T,U>::type; using exception_policy = common_exception_policy<T,U>::type;
    using result_base_type = promotion_policy::division_result<T,U>::type; using r_type = checked_result<result_base_type>;
    using L<T> = std::numeric_limits<T>; using lt = L<T>; using lu = L<U>; using l = L<result_base_type>;
    constexpr interval<r_type> ti{checked::cast<result_base_type>(base_value(lt::min())),cast(base_value(lt::max()))},
                                ui{checked::cast<result_base_type>(base_value(lu::min())),cast(base_value(lu::max()))};
    static auto get_r_type_interval() {
        if (ui.u < (r_type)0 || ui.l > (r_type)0) return ti % ui;
        return utility::minmax<r_type>({ti.l/ui.l, ti.l/r_type(-1), ti.l/r_type(1), ti.l/ui.u, ti.u/ui.l, ti.u/r_type(-1), ti.u/r_type(1), ti.u/ui.u});
    }
    constexpr interval<r_type> ri = get_r_type_interval();
    constexpr interval<result_base_type> reti{ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u};
public: using type = safe_base<result_base_type, reti.l, reti.u, promotion_policy, exception_policy>;
    static type return_value(const T& t, const U& u) {
        if constexpr (ri.l.exception() || ri.u.exception() || !reti.includes(ri)) {
            constexpr int bits = std::min(L<uintmax_t>::digits, std::max({l::digits, L<base_type<T>::type>::digits, L<base_type<U>::type>::digits})) + l::is_signed?1:0;
            using temp_base = std::conditional_t<l::is_signed, boost::int_t<bits>::least, boost::uint_t<bits>::least>;
            auto r = casting_helper<exception_policy,temp_base>(t,u);
            auto rx = checked_operation<temp_base,dispatch_and_return<exception_policy,temp_base>>::divide(r.first,r.second);
            return {rx.exception()?r.first/r.second:rx, skip_validation{}};
        } else return {(result_base_type)base_value(t) / (result_base_type)base_value(u), skip_validation{}};
    }
};
using division_operator<T,U> = decltype(std::declval<T const&>() / std::declval<U const&>());
division_result<T,U>::type operator/ <T,U> (const T& t, const U& u) requires legal_overload<division_operator,T,U>::value { return division_result<T,U>::return_value(t,u); }
T operator/= <T,U> (T& t, const U& u) requires legal_overload<division_operator,T,U>::value { t = (T)(t / u); return t; }

class modulus_result<T,U> {
    using promotion_policy = common_promotion_policy<T,U>::type; using exception_policy = common_exception_policy<T,U>::type;
    using result_base_type = promotion_policy::modulus_result<T,U>::type; using r_type = checked_result<result_base_type>;
    using L<T> = std::numeric_limits<T>; using lt = L<T>; using lu = L<U>; using l = L<result_base_type>;
    constexpr interval<r_type> ti{checked::cast<result_base_type>(base_value(lt::min())),cast(base_value(lt::max()))},
                                ui{checked::cast<result_base_type>(base_value(lu::min())),cast(base_value(lu::max()))};
    static auto get_r_type_interval() {
        if (ui.u < (r_type)0 || ui.l > (r_type)0) return ti % ui;
        return utility::minmax<r_type>({ti.l/ui.l, ti.l/r_type(-1), ti.l/r_type(1), ti.l/ui.u, ti.u/ui.l, ti.u/r_type(-1), ti.u/r_type(1), ti.u/ui.u});
    }
    constexpr interval<r_type> ri = get_r_type_interval();
    constexpr interval<result_base_type> reti{ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u};
public: using type = safe_base<result_base_type, reti.l, reti.u, promotion_policy, exception_policy>;
    static type return_value(const T& t, const U& u) {
        if constexpr (ri.l.exception() || ri.u.exception() || !reti.includes(ri)) {
            constexpr int bits = std::min(L<uintmax_t>::digits, std::max({l::digits, L<base_type<T>::type>::digits, L<base_type<U>::type>::digits})) + l::is_signed?1:0;
            using temp_base = std::conditional_t<l::is_signed, boost::int_t<bits>::least, boost::uint_t<bits>::least>;
            auto r = casting_helper<exception_policy,temp_base>(t,u);
            auto rx = checked_operation<temp_base,dispatch_and_return<exception_policy,temp_base>>::divide(r.first,r.second);
            return {rx.exception()?r.first%r.second:rx, skip_validation{}};
        } else return {(result_base_type)base_value(t) % (result_base_type)base_value(u), skip_validation{}};
    }
};
using modulus_operator<T,U> = decltype(std::declval<T const&>() % std::declval<U const&>());
modulus_result<T,U>::type operator% <T,U> (const T& t, const U& u) requires legal_overload<modulus_operator,T,U>::value { return modulus_result<T,U>::return_value(t,u); }
T operator%= <T,U> (T& t, const U& u) requires legal_overload<modulus_operator,T,U>::value { t = (T)(t % u); return t; }

class less_than_result<T,U> {
    using promotion_policy = common_promotion_policy<T,U>::type; using exception_policy = common_exception_policy<T,U>::type;
    using result_base_type = promotion_policy::comparison_result<T,U>::type; using r_type = checked_result<result_base_type>;
    using L<T> = std::numeric_limits<T>; using lt = L<T>; using lu = L<U>; using l = L<result_base_type>;
    constexpr interval<r_type> ti{checked::cast<result_base_type>(base_value(lt::min())),cast(base_value(lt::max()))},
                                ui{checked::cast<result_base_type>(base_value(lu::min())),cast(base_value(lu::max()))};
public: static bool return_value(const T& t, const U& u) {
        if constexpr (ti < ui) return true; if constexpr (ti > ui) return false;
        if constexpr (ti.l.exception() || ti.u.exception() || ui.l.exception() || ui.u.exception()) {
            auto r = casting_helper<exception_policy,result_base_type>(t,u);
            return safe_compare::less_than(r.first, r.second);
        } else return (result_base_type)base_value(t) < (result_base_type)base_value(u);
    }
};
using {less_than|greater_than}_<or_equal>_operator<T,U> = decltype(std::declval<T const&>() {<|>|<=|>=} std::declval<U const&>());
bool operator< <T,U> (const T& t, const U& u) requires legal_overload<less_than_operator<T,U>,T,U>::value { return less_than_result<T,U>::return_value(t,u); }
bool operator> <T,U> (const T& t, const U& u) requires legal_overload<greater_than_operator<T,U>,T,U>::value { return u < t; }
bool operator<= <T,U> (const T& t, const U& u) requires legal_overload<less_than_or_equal_operator<T,U>,T,U>::value { return !(t < u); }
bool operator>= <T,U> (const T& t, const U& u) requires legal_overload<greater_than_or_equal_operator<T,U>,T,U>::value { return !(u < t); }

class equal_result<T,U> {
    using promotion_policy = common_promotion_policy<T,U>::type; using exception_policy = common_exception_policy<T,U>::type;
    using result_base_type = promotion_policy::comparison_result<T,U>::type; using r_type = checked_result<result_base_type>;
    using L<T> = std::numeric_limits<T>; using lt = L<T>; using lu = L<U>; using l = L<result_base_type>;
    constexpr interval<r_type> ti{checked::cast<result_base_type>(base_value(lt::min())),cast(base_value(lt::max()))},
                                ui{checked::cast<result_base_type>(base_value(lu::min())),cast(base_value(lu::max()))};
public: static bool return_value(const T& t, const U& u) {
        if constexpr (!intersect(ti,ui)) return false;
        if constexpr (ti.l.exception() || ti.u.exception() || ui.l.exception() || ui.u.exception()) {
            auto r = casting_helper<exception_policy,result_base_type>(t,u);
            return safe_compare::equal(r.first, r.second);
        } else return (result_base_type)base_value(t) == (result_base_type)base_value(u);
    }
};
using <not>_equal_to_operator<T,U> = decltype(std::declval<T const&>() {==|!=} std::declval<U const&>());
bool operator== <T,U> (const T& t, const U& u) requires legal_overload<equal_to_operator<T,U>,T,U>::value { return equal<T,U>::return_value(t,u); }
bool operator!= <T,U> (const T& t, const U& u) requires legal_overload<not_equal_to_operator<T,U>,T,U>::value { return !(t == u); }

class {left|right}_shift_result<T,U> {
    using promotion_policy = common_promotion_policy<T,U>::type; using exception_policy = common_exception_policy<T,U>::type;
    using result_base_type = promotion_policy::{left|right}_shift_result<T,U>::type; using r_type = checked_result<result_base_type>;
    using L<T> = std::numeric_limits<T>; using lt = L<T>; using lu = L<U>; using l = L<result_base_type>;
    constexpr interval<r_type> ri = interval<r_type> ri{checked::cast<result_base_type>(base_value(lt::min())),cast(base_value(lt::max()))}
                                {<<|>>} interval<r_type> ri{checked::cast<result_base_type>(base_value(lu::min())),cast(base_value(lu::max()))};
    constexpr interval<result_base_type> reti{ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u};
public: using type = safe_base<result_base_type, reti.l, reti.u, promotion_policy, exception_policy>;
    static type return_value(const T& t, const U& u) {
        if constexpr (ri.l.exception() || ri.u.exception() || !reti.includes(ri)) {
            auto r = casting_helper<exception_policy,result_base_type>(t,u);
            auto rx = checked_operation<result_base_type,dispatch_and_return<exception_policy,result_base_type>>::{left|right}_shift(r.first,r.second);
            return {rx.exception() ? r.first {<<|>>} r.second : rx.m_contents.m_r, skip_validation()};
        } else return {(result_base_type)base_value(t) {<<|>>} (result_base_type)base_value(u), skip_validation()};
    }
};
using {left|right}_shift_operator<T,U> = decltype(std::declval<T const&>() {==|!=} std::declval<U const&>());
{left|right}_shift_result<T,U> operator{<<|>>} <T,U> (const T& t, const U& u) requires Numeric<T>() && legal_overload<{left|right}_shift_operator<T,U>,T,U>::value { return {left|right}_shift_result<T,U>::return_value(t,u); }
T operator{<<|>>}= <T,U> (T& t, const U& u) requires Numeric<T>() && legal_overload<{left|right}_shift_operator,T,U>::value { t = (T)(t << u); return t; }

using stream_{output|input}_operator<T,Ch,Tr> = decltype(std::declval<std::basic_{o|i}stream<Ch,Tr>&>() {>>|<<} std::declval<T[const]&>());
std::basic_ostream<Ch,Tr>& operator<< <T,Ch,Tr>(std::basic_ostream<Ch,Tr>& os, const T& t) requires mp_valid<stream_output_operator,T,Ch,Tr>::value { t.output(os); return os; }
std::basic_istream<Ch,Tr>& operator>> <T,Ch,Tr>(std::basic_istream<Ch,Tr>& is, T& t) requires mp_valid<stream_input_operator,T,Ch,Tr>::value { t.input(is); return is; }

class bitwise_{or|xor|and}_result<T,U> {
    using promotion_policy = common_promotion_policy<T,U>::type; using exception_policy = common_exception_policy<T,U>::type;
    using result_base_type = promotion_policy::bitwise_{or|xor|and}_result<T,U>::type; using r_type = std::make_unsigned_t<result_base_type>;
    constexpr r_type min{0}, max = round_out(std::max(r_type{base_value{std::numeric_limits<T>::max()}}, r_type{base_value{std::numeric_limits<U>::max()}}));
public: using type = safe_base<result_base_type, min, max, promotion_policy, exception_policy>;
    static type return_value(const T& t, const U& u) { return {(result_base_type)base_value(t) {|/^/&} (result_base_type)base_value(u), skip_validation()}; }
};
using bitwise_{or|xor|and}_operator<T,U> = decltype(std::declval<T const&>() {|/^/&} std::declval<U const&>());
bitwise_{or|xor|and}_result<T,U> operator{|/^/&} <T,U> (const T& t, const U& u) requires legal_overload<bitwise_{or|xor|and}_operator<T,U>,T,U>::value { return bitwise_{or|xor|and}_result<T,U>::return_value(t,u); }
T operator{|/^/&}= <T,U> (T& t, const U& u) requires legal_overload<bitwise_{or|xor|and}_operator,T,U>::value { t = (T)(t {|/^/&} u); return t; }
```

#### Safe Integer Policy

```c++
struct native { // promotion policy
    struct addition_result<T,U> { using type = decltype(base_type<T>::type() + base_type<U>::type()); };
    struct subtraction_result<T,U> { using type = decltype(base_type<T>::type() - base_type<U>::type()); };
    struct multiplication_result<T,U> { using type = decltype(base_type<T>::type() * base_type<U>::type()); };
    struct division_result<T,U> { using type = decltype(base_type<T>::type() / base_type<U>::type()); };
    struct modulus_result<T,U> { using type = decltype(base_type<T>::type() % base_type<U>::type()); };
    struct comparison_result<T,U> { using type = decltype(base_type<T>::type() + base_type<U>::type()); };
    struct {left|right}_shift_result<T,U> { using type = decltype(base_type<T>::type() {<<|>>} base_type<U>::type()); };
    struct bitwise_{or|and|xor}_result<T,U> { using type = decltype(base_type<T>::type() {|/&/^} base_type<U>::type()); };
};

using safe<T,P=native,E=default_exception_policy> = safe_base<T,std::numeric_limits<T>::min(),std::numeric_limits<T>::max(),P,E>;
```

#### Automatic Policy

```c++
struct automatic {
private:
    struct defer_stored_signed_lazily { using type = utility::signed_stored_type<min,max>; };
    struct defer_stored_unsigned_lazily { using type = utility::unsigned_stored_type<min,max>; };
    struct result_type<T,min,max> { using type = conditional_t<numeric_limits<T>::is_signed, defer_stored_signed_lazily, defer_stored_unsigned_lazily>::type<min,max>; };
public:
    struct addition_result<T,U> {
        using lt = std::numeric_limits<T>; using lu = std::numeric_limits<U>; using l = std::numeric_limits<temp_base_type>;
        using temp_base_type = std::conditional_t<!lt::is_signed && !lu::is_signed, uintmax_t, intmax_t>; using r_type = checked_result<temp_base_type>;
        constexpr static interval<r_type> ri = interval<r_type>{checked::cast<temp_base_type>(base_value{lt::min()}), checked::cast(base_value{lt::max()})}
                                            + interval<r_type>{checked::cast<temp_base_type>(base_value{lu::min()}), checked::cast(base_value{lu::max()})};
        using type = result_type<temp_base_type, ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u>;
    };
    struct subtraction_result<T,U> {
        using lt = std::numeric_limits<T>; using lu = std::numeric_limits<U>; using l = std::numeric_limits<temp_base_type>;
        using temp_base_type = intmax_t; using r_type = checked_result<temp_base_type>;
        constexpr static interval<r_type> ri = interval<r_type>{checked::cast<temp_base_type>(base_value{lt::min()}), checked::cast(base_value{lt::max()})}
                                            - interval<r_type>{checked::cast<temp_base_type>(base_value{lu::min()}), checked::cast(base_value{lu::max()})};
        using type = result_type<temp_base_type, ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u>;
    };
    struct multiplication_result<T,U> {
        using lt = std::numeric_limits<T>; using lu = std::numeric_limits<U>; using l = std::numeric_limits<temp_base_type>;
        using temp_base_type = std::conditional_t<!lt::is_signed && !lu::is_signed, uintmax_t, intmax_t>; using r_type = checked_result<temp_base_type>;
        constexpr static interval<r_type> ri = interval<r_type>{checked::cast<temp_base_type>(base_value{lt::min()}), checked::cast(base_value{lt::max()})}
                                            * interval<r_type>{checked::cast<temp_base_type>(base_value{lu::min()}), checked::cast(base_value{lu::max()})};
        using type = result_type<temp_base_type, ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u>;
    };
    struct division_result<T,U> {
        using lt = std::numeric_limits<T>; using lu = std::numeric_limits<U>; using l = std::numeric_limits<temp_base_type>;
        using temp_base_type = std::conditional_t<!lt::is_signed && !lu::is_signed, uintmax_t, intmax_t>; using r_type = checked_result<temp_base_type>;
        constexpr static interval<r_type> rl = {checked::cast<temp_base_type>(base_value{lt::min()}), checked::cast(base_value{lt::max()})},
                                        ru = {checked::cast<temp_base_type>(base_value{lu::min()}), checked::cast(base_value{lu::max()})};
        constexpr static interval<r_type> rx() { if (ul.u < 0 || ul.l > 0) return ti/ui;
            return utility::minmax({ti.l/ui.l, ti.l/r_type(-1), ti.l/r_type(1), ti.l/ui.u, ti.u/ui.l, ti.u/r_type(-1), ti.u/r_type(0), ti.u/ui.u}); }
        constexpr static interval<r_type> ri = rx();
        using type = result_type<temp_base_type, ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u>;
    };
    struct modulus_result<T,U>; // same as division_result
    struct comparison_result<T,U> {
        using lt = std::numeric_limits<T>; using lu = std::numeric_limits<U>; using l = std::numeric_limits<temp_base_type>;
        using temp_base_type = std::conditional_t<!lt::is_signed && !lu::is_signed, uintmax_t, intmax_t>; using r_type = checked_result<temp_base_type>;
        constexpr static interval<r_type> rl = {checked::cast<temp_base_type>(base_value{lt::min()}), checked::cast(base_value{lt::max()})},
                                        ru = {checked::cast<temp_base_type>(base_value{lu::min()}), checked::cast(base_value{lu::max()})};
        constexpr static interval<r_type> ri = {min(ti.l,ui.l,max(ti.u,ui.u))};
        using type = result_type<temp_base_type, ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u>;
    };
    struct left_shift_result<T,U> {
        using lt = std::numeric_limits<T>; using lu = std::numeric_limits<U>; using l = std::numeric_limits<temp_base_type>;
        using temp_base_type = std::conditional_t<!lt::is_signed && !lu::is_signed, uintmax_t, intmax_t>; using r_type = checked_result<temp_base_type>;
        constexpr static interval<r_type> ri = interval<r_type>{checked::cast<temp_base_type>(base_value{lt::min()}), checked::cast(base_value{lt::max()})}
                                            << interval<r_type>{checked::cast<temp_base_type>(base_value{lu::min()}), checked::cast(base_value{lu::max()})};
        using type = result_type<temp_base_type, ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u>;
    };
    struct right_shift_result<T,U> {
        using lt = std::numeric_limits<T>; using lu = std::numeric_limits<U>; using l = std::numeric_limits<temp_base_type>;
        using temp_base_type = std::conditional_t<!lt::is_signed && !lu::is_signed, uintmax_t, intmax_t>; using r_type = checked_result<temp_base_type>;
        constexpr static r_type u_min = checked::cast<temp_base_type>(base_value{lu::min()});
        constexpr static interval<r_type> ti = {checked::cast<temp_base_type>(base_value{lt::min()}), checked::cast(base_value{lt::max()})},
                                        ui = {u_min.exception()?r_type(0):u_min, checked::cast<temp_base_type>(base_value{lu::max()})};
        using type = result_type<temp_base_type, ri.l.exception()?l::min():ri.l, ri.u.exception()?l::max():ri.u>;
    };
    struct bitwise_and_result<T,U> { using type = decltype(base_type<T>::type() & base_type<U>::type()); };
    struct bitwise_or_result<T,U> { using type = decltype(base_type<T>::type() | base_type<U>::type()); };
    struct bitwise_xor_result<T,U> { using type = decltype(base_type<T>::type() ^ base_type<U>::type()); };
};
```

#### C++ Policy

```c++
struct cpp<CharBits,ShortBits,IntBits,LongBits,LongLongBits> {
    using local_char_type = int_t<CharBits>::exact;
    using local_short_type = int_t<ShortBits>::exact;
    using local_int_type = int_t<IntBits>::exact;
    using local_long_type = int_t<LongBits>::exact;
    using local_long_long_type = int_t<LongLongBits>::exact;

    using rank<T> = conditional_t<is_same_v<local_char_type,make_signed_t<T>>, integral_constant<int,1>,
                    conditional_t<is_same_v<local_short_type,make_signed_t<T>>, integral_constant<int,2>,
                    conditional_t<is_same_v<local_int_type,make_signed_t<T>>, integral_constant<int,3>,
                    conditional_t<is_same_v<local_long_type,make_signed_t<T>>, integral_constant<int,4>,
                    conditional_t<is_same_v<local_long_long_type,make_signed_t<T>>, integral_constant<int,5>, 6>>>>>;
    using higher_ranked_type<T,U> = conditional_t<(rank<T>::value < rank<U>::value), U, T>;
    using copy_sign<T,U> = conditional_t<std::is_signed_v<U>, make_signed_t<T>, make_unsigned_t<T>>;
    using integral_promotion<T> = copy_sign<higher_ranked_type<local_int_type, T>, T>;
    using select_signed<T,U> = conditional_t<std::numeric_limits<T>::is_signed, T, U>;
    using select_unsigned<T,U> = conditional_t<std::numeric_limits<T>::is_signed, U, T>;
    using usual_arithmetic_conversions<T,U> = conditional_t<is_same_v<T,U>,T,
                    conditional_t<numeric_limits<T>::is_signed == conditional_t<numeric_limits<U>::is_signed, higher_ranked_type<T,U>,
                    conditional_t<rank<select_unsigned<T,U>>::value >= select_signed<T,U>>::value, select_unsigned<T,U>,
                    conditional_t<numeric_limits<select_signed<T,U>>::digits >= numeric_limits<select_unsigned<T,U>>::digits, select_signed<T,U>, make_signed<selected_signed<T,U>>>>>>;
    using result_type<T,U> = usual_arithmetic_conversions<integral_promotion<base_type<T>::type>, integral_promotion<base_type<U>::type>>::type;

    struct addition_result<T,U> { using type = result_type<T,U>; };
    struct subtraction_result<T,U> { using type = result_type<T,U>; };
    struct multiplication_result<T,U> { using type = result_type<T,U>; };
    struct division_result<T,U> { using type = result_type<T,U>; };
    struct modulus_result<T,U> { using type = result_type<T,U>; };
    struct comparison_result<T,U> { using type = result_type<T,U>; };
    struct left_shift_result<T,U> { using type = result_type<T,U>; };
    struct right_shift_result<T,U> { using type = result_type<T,U>; };
    struct bitwise_and_result<T,U> { using type = result_type<T,U>; };
    struct bitwise_or_result<T,U> { using type = result_type<T,U>; };
    struct bitwise_xor_result<T,U> { using type = result_type<T,U>; };
};
```

#### Safe Literal Integer

```c++
struct is_safe<safe_literal_impl<T,n,P,E>> : std::true_type {};
struct get_promotion_policy<safe_literal_impl<T,n,P,E>> { using type = P; };
struct get_exception_policy<>safe_literal_impl<T,n,P,E>> { using type = E; };
struct base_type<safe_literal_impl<T,n,P,E>> { using type = T; };
T base_value<T,n,P,E>(const safe_literal_impl<T,n,P,E>&) { return n; }
std::basic_ostream<Ch,Tr>& operator << <Ch,Tr,T,n,P,E> (std::basic_ostream<Ch,Tr>& os, const safe_literal_impl<T,n,P,E>&)
{ return os << (std::is_same_v<T,signed char> || std::is_same_v<T,unsigned char> ? (int)n : n); }

struct safe_literal_impl<T,n,P,E> {
    ctor(){}
    operator R <R>() const requires !is_safe<R>::value { using l = std::numeric_limits<R>;
        return validate_detail<R,l::min(),l::min(),E>::return_value(*this); }
    self operator+() const { return {}; } // unary +
    auto operator-() const requires !checked::minus(n).exception() { return self<T,-n,P,E>{}; }
    auto operator~() const requires !checked::bitwise_not(n).exception() { return self<T,~n,P,E>{}; }
};
using safe_signed_literal<n,P=void,E=void> = safe_literal_impl<signed_stored_type<n,n>,n,P,E>;
using safe_unsigned_literal<n,P=void,E=void> = safe_literal_impl<unsigned_stored_type<n,n>,n,P,E>;
auto make_safe_literal_impl<T,n,P=void,E=void> () requires std::is_signed_v<T> { return safe_signed_literal<n,P,E>{}; }
auto make_safe_literal_impl<T,n,P=void,E=void> () requires !std::is_signed_v<T> { return safe_unsigned_literal<n,P,E>{}; }
auto make_safe_literal<T n, P, E> { return make_safe_literal_impl<T,n,P,E>(); }

struct std::numeric_limits<safe_literal_impl<T,n,P,E>> : numeric_limits<T> {
    using SL = safe_literal_impl<T,n,P,E>;
    static SL {lowest|min|max}() constexpr { return {}; }
};
```

#### Range Value

```c++
struct range_value<T> { const T& m_t; ctor(const T& t): m_t{t} {} };
range_value<T> make_range_value<T>(const T& t) { return {t}; }
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,T> (std::basic_ostream<Ch,Tr>& os, const range_value<T>& t)
{ return os << make_interval<T>() << t.m_t; }
struct result_display<T> { const T& m_t; ctor(const T& t): m_t{t} {} }; // same as range_value
range_value<T> make_result_display<T>(const T& t) { return {t}; }
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,T> (std::basic_ostream<Ch,Tr>& os, const result_display<T>& r)
{ return os << std::hex << make_range_value(r.m_t) << '(' << std::dec << r.m_t << ')'; }

using safe_signed_range<min,max,P=native,E=default_exception_policy> = safe_base<signed_stored_type<min,max>,min,max>;
using safe_unsigned_range<min,max,P=native,E=default_exception_policy> = safe_base<unsigned_stored_type<min,max>,min,max>;
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
