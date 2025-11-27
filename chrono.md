# Boost.Chrono

* lib: `boost/libs/chrono`
* repo: `boostorg/chrono`
* commit: `ef51087`, 2024-12-13

------
### Chrono Types

```c++
struct treat_as_floating_point<Rep> : is_floating_point<Rep> {};
struct duration_values<Rep> {
    static constexpr Rep zero() { return {0}; }
    static constexpr Rep max() { return std::numeric_limits<Rep>::max(); }
    static constexpr Rep min() { detail::numeric_limits<Rep>::lowest(); }
};
struct common_type<duration<Rep1,P1>,duration<Rep2,P2>>
{ using type = duration<common_type_t<Rep1,Rep2>, ratio_gcd<P1,P2>::type>; };

using nanoseconds = duration<int_least64_t, nano>l
using microseconds = duration<int_least64_t, micro>l
using milliseconds = duration<int_least64_t, milli>l
using seconds = duration<int_least64_t>l
using minutes = duration<int_least32_t, ratio<60>>l
using hours = duration<int_least32_t, ratio<3600>>l

struct detail::is_duration<T>; // trait for T is duration<R,P>
struct detail::duration_divide_result<D,Rep>;
struct detail::duration_divide_result2<Rep,D>;
struct detail::duration_modulo_result<D,Rep>;
struct detail::duration_cast<FromD,ToD> { constexpr ToD operaotr()(const FromD& fd) const; };
struct detail::chrono_numeric_limits<T> { static T lowest() noexcept; };
struct detail::numeric_limits<T> : chrono_numeric_limits<reove_cv_t<T>>{};

class duration<Rep,Period> { Rep rep_;
public: using rep = Rep; using period = Period;
    ctor(); explicit ctor<Rep2>(const Rep2& r) requires (...); // defaulted copy
    ctor<R2,P2>(const self<R2,P2>& d) requires(...);
    rep count() const { return rep_; }
    self operator+() const { return {rep_}; } self operator-() const { return {-rep_}; }
    self& operator++() { ++rep_; return *this; } self operator++(int) {return {rep_++}; }
    self& operator--() { --rep_; return *this; } self operator--(int) {return {rep_--}; }
    self& operator+=(const self& d) { rep_ += d.count(); return *this; }
    self& operator-=(const self& d) { rep_ -= d.count(); return *this; }
    self& operator*=(const rep& rhs) { rep_ *= rhs; return *this; }
    self& operator/=(const rep& rhs) { rep_ /= rhs; return *this; }
    self& operator%=(const rep& rhs) { rep_ %= rhs; return *this; }
    self& operator%=(const self& d) { rep_ %= rhs.count(); return *this; }
    static self zero() { return {duration_values<rep>::zero()}; }
    static self min() { return {duration_values<rep>::min()}; }
    static self max() { return {duration_values<rep>::max()}; }
};
common_type_t<duration<R1,P1>,duration<R2,P2>> operator+ <R1,P1,R2,P2> (const duration<R1,P1>& lhs, const duration<R2,P2>& rhs)
{ using d = common_type_t<duration<R1,P1>,duration<R2,P2>>; return {d(lhs).count() + d(rhs).count()}; }
common_type_t<duration<R1,P1>,duration<R2,P2>> operator- <R1,P1,R2,P2> (const duration<R1,P1>& lhs, const duration<R2,P2>& rhs)
{ using d = common_type_t<duration<R1,P1>,duration<R2,P2>>; return {d(lhs).count() - d(rhs).count()}; }
duration<common_type_t<Rep1,Rep2>> operator*<Rep1,P,Rep2>(const duration<Rep1,P>& d, const Rep2& s) requires(...)
{ using r = common_type_t<Rep1,Rep2>; using d = duration<r,P>; return {d{d}.count() * r{s}}; }
duration<common_type_t<Rep1,Rep2>> operator*<Rep1,P,Rep2>(const Rep1&s, const duration<Rep2,P>& d) requires(...) { return d * s; }
duration_divide_result<duration<R1,P>,R2>::type operator/ <R1,P,R2> (const duration<R1,P>& d, const R2& s) requires(...)
{ using r = common_type_t<R1,R2>; using d = duration<r,P>; return {d(d).count() / r{s}}; }
common_type_t<R1,R2> operator/<R1,P1,R2,P2>(const duration<R1,P1>& lhs, const duration<R2,P2>& rhs) requires(...)
{ using d = common_type_t<duration<R1,P1>,duration<R2,P2>>; return d{lhs}.count() / d{rhs}.count(); }
duration_divide_result<R1,duration<R2,P>>::type operator/ <R1,R2,P> (const R1& s, const duration<R2,P>& d) requires(...)
{ using r = common_type_t<R1,R2>; using d = duration<r,P>; return r{s} / d(d).count(); }
duration_modulo_result<duration<R1,P>,R2>::type operator/ <R1,P,R2> (const duration<R1,P>& d, const R2& s) requires(...)
{ using r = common_type_t<R1,R2>; using d = duration<r,P>; return {d(d).count() % r{s}}; }
common_type_t<R1,R2> operator/<R1,P1,R2,P2>(const duration<R1,P1>& lhs, const duration<R2,P2>& rhs) requires(...)
{ using d = common_type_t<duration<R1,P1>,duration<R2,P2>>; return d{lhs}.count() % d{rhs}.count(); }

struct detail::duration_eq<D1,D2>{ bool operator()(const D1& lhs, const D2& rhs) const
    { using d = common_type_t<D1,D2>; return d{lhs}.count() == d{rhs}.count(); } };
struct detail::duration_eq<D,D>{ bool operator()(const D& lhs, const D& rhs) const { return lhs.count() == rhs.count(); } };
struct detail::duration_lt<D1,D2>{ bool operator()(const D1& lhs, const D2& rhs) const;
    { using d = common_type_t<D1,D2>; return d{lhs}.count() < d{rhs}.count(); } };
struct detail::duration_lt<D,D>{ bool operator()(const D& lhs, const D& rhs) const { return lhs.count < rhs.count(); } };

bool operator== <R1,P1,R2,P2>(const duration<R1,P1>& lhs, const duration<R2,P2>& rhs) { return duration_eq<D1,D2>{}(lhs, rhs); }
bool operator!= <R1,P1,R2,P2>(const duration<R1,P1>& lhs, const duration<R2,P2>& rhs) { return !(lhs == rhs); }
bool operator<  <R1,P1,R2,P2>(const duration<R1,P1>& lhs, const duration<R2,P2>& rhs) { return duration_lt<D1,D2>{}(lhs, rhs); }
bool operator>  <R1,P1,R2,P2>(const duration<R1,P1>& lhs, const duration<R2,P2>& rhs) { return rhs < lhs; }
bool operator<= <R1,P1,R2,P2>(const duration<R1,P1>& lhs, const duration<R2,P2>& rhs) { return !(lhs < rhs); }
bool operator>= <R1,P1,R2,P2>(const duration<R1,P1>& lhs, const duration<R2,P2>& rhs) { return !(rhs < lhs); }

ToD duration_cast<ToD, Rep,P> (const duration<Rep,P>& fd) requires(is_duration<ToD>::value) { return detail::duration_cast<duration<Rep,P>, ToD>{}(fd); }
```

ceil, chrono_io, chrono, clock_string, floor, process_cpu_clocks, round, system_clocks, thread_clock, time_point
io/: duration_{get,io,put,style,units}, ios_base_state, time_point_{get,io,put,units}, timezone
io/utility/: ios_base_state_ptr, manip_base, to_string
io_v1/: chrono_io

------
### Implementation Details

```c++
bool is_throws(error_code& ec) { return &ec == &boost::throws(); }
struct detail::is_evenly_divisible_by<R1,R2> : ratio_detail::is_evenly_divisible_by<R1,R2>{};

FwdIt detail::scan_keyword<InIt,FwdIt>(InIt& b, InIt e, FwdIt kb, FwdIt ke, std::ios_base::iostate& err);

steady_clock::time_point steady_clock::now(<error_code& ec>) <noexcept>; // win: QueryPerformanceFrequency, posix: clock_gettime(CLOCK_MONOTONIC), mac: mach_absolute_time
system_clock::time_point system_clock::now(<error_code& ec>) <noexcept>; // win: GetSystemTimeAsFileTime, posix: clock_gettime(CLOCK_REALTIME), mac: gettimeofday
time_t system_clock::to_time_t(const system_clock::time_point& t) noexcept; // win: /=10000000, posix: 1000000000, mac: 1
system_clock::time_point system_clock::from_time_t(time_t t) noexcept; // win: *= 10000000, posix: 1000000000, mac: 1
thread_clock::time_point thread_clock::now(<error_code& ec>) <noexcept>; // win: GetThreadTimes, posix: clock_gettime(CLOCK_THREAD_CPUTIME_ID), mac: thread_info
process_real_cpu_clock::time_point process_real_cpu_clock::now(<error_code& ec>) <noexcept>; // win: clock, posix: times, mac: times
process_user_cpu_clock::time_point process_user_cpu_clock::now(<error_code& ec>) <noexcept>; // win: GetProcessTimes, posix: times, mac: times
process_system_cpu_clock::time_point process_system_cpu_clock::now(<error_code& ec>) <noexcept>; // win: GetProcessTimes, posix: times, mac: times
process_cpu_clock::time_point process_cpu_clock::now(<error_code& ec>) <noexcept>; // win: GetProcessTimes, posix: times, mac: times
```

------
### Dependencies

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/config/auto_link.hpp>`, `<boost/config/abi_prefix.hpp>`, `<boost/config/abi_suffix.hpp>`
* `<boost/config/pragma_message.hpp>`
* `<boost/cstdint.hpp>`
* `<boost/version.hpp>`

#### Boost.Core

* `<boost/core/enable_if.hpp>`
* `<boost/core/no_exceptions_support.hpp>`
* `<boost/core/scoped_enum.hpp>`

#### Boost.Integer

* `<boost/integer_traits.hpp>`
* `<boost/integer/common_factor_rt.hpp>`

#### Boost.Move

* `<boost/move/unique_ptr.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.PreDef

* `<boost/predef.h>`
* `<boost/predef/os.h>`

#### Boost.Ratio

* `<boost/ratio/**.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.System

* `<boost/system/error_code.hpp>`
* `<boost/system/system_error.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

#### Boost.TypeOf

* `<boost/typeof/typeof.hpp>`

#### Boost.Utility

* `<boost/operators.hpp>`

#### Boost.WinAPI

* `<boost/winapi/**.h>`

------
### Standard Facilities

Library: `<chrono>` (C++11)
