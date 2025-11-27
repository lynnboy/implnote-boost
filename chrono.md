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

struct common_type<time_point<C,D1>,time_point<C,D2>> { using type = time_point<C,common_type_t<D1,D2>>; };

class time_point<Clock,Duration> { duration d_{duration::zero()};
public: using clock = Clock; using duration = Duration; using rep = duration::rep; using period = Duration::period; using difference_type = Duration;
    ctor(); explicit ctor(const duration& d); ctor<D2>(const time_point<clock,D2>& t) requires (...)
    duration time_since_epoch() const { return d_; }
    self operator+() const; // and -, ++, --, +=, -=; extensions
    self& operator+=(const duration& d) {d_ += d; return *this;}
    self& operator-=(const duration& d) {d_ -= d; return *this;}
    static time_point min() { return {duration::min()}; }
    static time_point max() { return {duration::max()}; }
};
time_point<C,common_type_t<D1,duration<R2,P2>>> operator+<C,D1,R2,P2>(const time_point<C,D1>& lhs, const duration<R2,P2>& rhs)
{ using CD = common_type_t<D1,duration<R2,P2>>; return {lhs.time_since_epoch() + CD(rhs)}; }
time_point<C,common_type_t<duration<R1,P1>,D2>> operator+<R1,P1,C,D2>(const duration<R1,P1>& lhs, const time_point<C,D2>& rhs) { return rhs + lhs; }
time_point<C,common_type_t<D1,duration<R2,P2>>> operator-<C,D1,R2,P2>(const time_point<C,D1>& lhs, const duration<R2,P2>& rhs) { lhs + (-rhs); }
time_point<C,common_type_t<duration<R1,P1>,D2>> operator-<R1,P1,C,D2>(const duration<R1,P1>& lhs, const time_point<C,D2>& rhs) { return lhs.time_since_epoch() - rhs.time_since_epoch(); }

bool operator== <C,D1,D2>(const time_point<C,D1>& lhs, const time_point<C,D2>& rhs) { return lhs.time_since_epoch() == rhs.time_since_epoch(); }
bool operator!= <C,D1,D2>(const time_point<C,D1>& lhs, const time_point<C,D2>& rhs) { return !(lhs == rhs); }
bool operator<  <C,D1,D2>(const time_point<C,D1>& lhs, const time_point<C,D2>& rhs) { return lhs.time_since_epoch() < rhs.time_since_epoch(); }
bool operator>  <C,D1,D2>(const time_point<C,D1>& lhs, const time_point<C,D2>& rhs) { return rhs < lhs; }
bool operator<= <C,D1,D2>(const time_point<C,D1>& lhs, const time_point<C,D2>& rhs) { return !(rhs < lhs); }
bool operator>= <C,D1,D2>(const time_point<C,D1>& lhs, const time_point<C,D2>& rhs) { return !(lhs < rhs); }

time_point<C,ToD> time_point_cast<ToD,C,D>(const time_point<C,D>& t) { return {duration_cast<ToD>(t.time_since_epoch())}; }

To floor<To,Rep,Period>(const duration<Rep,Period>& d) { To t = duration_cast<To>(d); if (t>d) --t; return t; }
To ceil<To,Rep,Period>(const duration<Rep,Period>& d) { To t = duration_cast<To>(d); if (t<d) ++t; return t; }
To round<To,Rep,Period>(const duration<Rep,Period>& d) { using D = common_type_t<To,duration<Rep,Period>>;
    To t0 = duration_cast<To>(d); To t1 = t0; D diff0, diff1;
    if (t0>d) { --t1; diff0 = t0-d; diff1 = d-t1; } else { ++t1; diff0 = d-t0; diff1 = t1-d; }
    if (diff0==diff1) return t0.count()&1 ? t1 : t0;
    return (diff0<diff1) ? t0 : t1;
}
```

------
### Clocks

```c++
struct clock_string<Clock,Ch>;

struct system_clock {
    using duration = duration<int_least64_t, ratio<1LL,10000000LL>>; // nanoseconds on WIN
    using rep = duration::rep; using period = duration::period; using time_point = time_point<self>;
    static constexpr bool is_steady = false;
    static self now(<error_code& ec>) <noexcept>; // win: GetSystemTimeAsFileTime, posix: clock_gettime(CLOCK_REALTIME), mac: gettimeofday
    static time_t to_time_t(const self& t) noexcept; // win: /=10000000, posix: 1000000000, mac: 1
    static self from_time_t(time_t) noexcept; // win: *= 10000000, posix: 1000000000, mac: 1
};
struct clock_string<system_clock,Ch> {
    static std::basic_string<Ch> name(); // "system_clock"
    static std::basic_string<Ch> since(); // " since Jan 1, 1970"
};

struct steady_clock {
    using duration = nanoseconds;
    using rep = duration::rep; using period = duration::period; using time_point = time_point<self>;
    static constexpr bool is_steady = true;
    static self now(<error_code& ec>) <noexcept>; // win: QueryPerformanceFrequency, posix: clock_gettime(CLOCK_MONOTONIC), mac: mach_absolute_time
};
struct clock_string<steady_clock,Ch> {
    static std::basic_string<Ch> name(); // "steady_clock"
    static std::basic_string<Ch> since(); // " since boot"
};

using high_resolution_clock = steady_clock;

struct process_real_cpu_clock {
    using duration = nanoseconds;
    using rep = duration::rep; using period = duration::period; using time_point = time_point<self>;
    static constexpr bool is_steady = true;
    static self now(<error_code& ec>) <noexcept>; // win: clock, posix: times, mac: times
};
struct clock_string<process_real_cpu_clock,Ch> {
    static std::basic_string<Ch> name(); // "process_real_cpu_clock"
    static std::basic_string<Ch> since(); // " since process start-up"
};

struct process_user_cpu_clock {
    using duration = nanoseconds;
    using rep = duration::rep; using period = duration::period; using time_point = time_point<self>;
    static constexpr bool is_steady = true;
    static self now(<error_code& ec>) <noexcept>; // win: GetProcessTimes, posix: times, mac: times
};
struct clock_string<process_user_cpu_clock,Ch> {
    static std::basic_string<Ch> name(); // "process_user_cpu_clock"
    static std::basic_string<Ch> since(); // " since process start-up"
};

struct process_system_cpu_clock {
    using duration = nanoseconds;
    using rep = duration::rep; using period = duration::period; using time_point = time_point<self>;
    static constexpr bool is_steady = true;
    static self now(<error_code& ec>) <noexcept>; // win: GetProcessTimes, posix: times, mac: times
};
struct clock_string<process_system_cpu_clock,Ch> {
    static std::basic_string<Ch> name(); // "process_system_cpu_clock"
    static std::basic_string<Ch> since(); // " since process start-up"
};

struct process_times<Rep> : arithmetic<self, multiplicative<self,Rep, less_than_comparable<self>>> {
    using rep = Rep;
    rep real{0}, user{0}, system{0};
    ctor()=default; explicit ctor<Rep2>(self<Rep2> const& rhs);
    self& operator+=(self const& rhs); // and -=, *=, /=
    self& operator*=(rep const& rhs); // and /=
    bool operator== <Rep2> (self<Rep2> const& rhs)=default;
    bool operator< (self const& rhs) const;
    void print<Ch,Tr>(std::basic_ostream<Ch,Tr>& os) const; // "{<real>;<user>;<system>}"
    void read<Ch,Tr>(std::basic_istream<Ch,Tr>& is);
};
struct common_type<process_times<R1>, process_times<R2>> { using type = process_times<common_type_t<R1,R2>>; };
struct common_type<process_times<R1>, R2> { using type = process_times<common_type_t<R1,R2>>; };
struct common_type<R1, process_times<R2>> { using type = process_times<common_type_t<R1,R2>>; };
bool operator== <R1,P1,R2,P2> (const duration<process_times<R1>,P1>& lhs, const duration<process_times<R2>,P2>& rhs)
{ return duration_eq<duration<R1,P1>,duration<R2,P2>>{}(lhs.count().real, rhs.count().real); }
bool operator== <R1,P1,R2,P2> (const duration<process_times<R1>,P1>& lhs, const duration<R2,P2>& rhs)
{ return duration_eq<duration<R1,P1>,duration<R2,P2>>{}(lhs.count().real, rhs); }
bool operator== <R1,P1,R2,P2> (const duration<R1,P1>& lhs, const duration<process_times<R2>,P2>& rhs) { rhs == lhs; }
bool operator< <R1,P1,R2,P2> (const duration<process_times<R1>,P1>& lhs, const duration<process_times<R2>,P2>& rhs)
{ return duration_lt<duration<R1,P1>,duration<R2,P2>>{}(lhs.count().real, rhs.count().real); }
bool operator< <R1,P1,R2,P2> (const duration<process_times<R1>,P1>& lhs, const duration<R2,P2>& rhs)
{ return duration_lt<duration<R1,P1>,duration<R2,P2>>{}(lhs.count().real, rhs); }
bool operator< <R1,P1,R2,P2> (const duration<R1,P1>& lhs, const duration<process_times<R2>,P2>& rhs)
{ return duration_lt<duration<R1,P1>,duration<R2,P2>>{}(lhs, rhs.count().real); }
struct std::numeric_limits<process_times<Rep>> {
    using Res = process_times<Rep>; using l = std::numeric_limits<Rep>;
    static const bool is_specialized = true;
    static Res min() { return Res{l::min(), l::min(); l::min()}; }
    static Res max() { return Res{l::max(), l::max(); l::max()}; }
    static Res lowest() { return min(); }
    static const int digits = l::digits*3, digits10 = l::digits10*3, radix = 0;
    static const bool is_signed = Rep::is_signed, is_integer = Rep::is_integer, is_exact = Rep::is_exact;
};
struct duration_values<process_times<Rep>> {
    using Res = process_times<Rep>; using l = std::numeric_limits<Rep>;
    static Res zero() { return Res{l::max(), l::max(), l::max()}; }
    static Res max() { return Res{l::min(), l::min(), l::min()}; }
};

std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,Rep> (std::basic_ostream<Ch,Tr>& os, process_times<Rep> const& rhs) { rhs.print(os); return os; }
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr,Rep> (std::basic_istream<Ch,Tr>& is, process_times<Rep>& rhs) { rhs.read(is); return is; }

using process_cpu_clock_times = process_times<nanoseconds::rep>;
struct process_cpu_clock {
    using times = process_cpu_clock_times; using duration = duration<times,nano>;
    using rep = duration::rep; using period = duration::period; using time_point = time_point<self>;
    static constexpr bool is_steady = true;
    static self now(<error_code& ec>) <noexcept>; // win: GetProcessTimes, posix: times, mac: times
};
struct clock_string<process_cpu_clock,Ch> {
    static std::basic_string<Ch> name(); // "process_cpu_clock"
    static std::basic_string<Ch> since(); // " since process start-up"
};

struct thread_clock {
    using duration = nanoseconds;
    using rep = duration::rep; using period = duration::period; using time_point = time_point<self>;
    static constexpr bool is_steady = true;
    static self now(<error_code& ec>) <noexcept>; // win: GetThreadTimes, posix: clock_gettime(CLOCK_THREAD_CPUTIME_ID), mac: thread_info
};
struct clock_string<thread_clock,Ch> {
    static std::basic_string<Ch> name(); // "thread_clock"
    static std::basic_string<Ch> since(); // " since thread start-up"
};
```

------
### IO

#### Common

```c++
std::basic_string<Ch> to_basic_string<Ch,T>(T const& v) { std::basic_stringstream<Ch> s; s<<v; return s.str(); }
std::string to_string<T>(T const& v) { return to_basic_string<char>(v); }
std::string to_wstring<T>(T const& v) { return to_basic_string<wchar_t>(v); }
std::string to_u16string<T>(T const& v) { return to_basic_string<char16_t>(v); }
std::string to_u32string<T>(T const& v) { return to_basic_string<char32_t>(v); }
```

#### Version 1

```c++
class duration_punct<Ch> {
    bool use_short_;
    string_type long_seconds_{"seconds"}, long_minutes_{"minutes"}, long_hours_{"hours"}, short_seconds_{"s"}, short_minutes_{"m"}, short_hours_{"h"};
    string_type short_name<P>(P) const { return ratio_string<P,Ch>::symbol() + short_seconds_; }
    string_type short_name(ratio<1>) const { return short_seconds_;}
    string_type short_name(ratio<60>) const { return short_minutes_;}
    string_type short_name(ratio<3600>) const { return short_hours_;}
    string_type long_name<P>(P) const { return ratio_string<P,Ch>::prefix() + long_seconds_; }
    string_type long_name(ratio<1>) const { return long_seconds_;}
    string_type long_name(ratio<60>) const { return long_minutes_;}
    string_type long_name(ratio<3600>) const { return long_hours_;}
public: using string_type = std::basic_string<Ch>;
    enum {use_long, use_short};
    static std::locale::id id;
    explicit ctor(int use=use_long) : use_short_{use==use_short}{}
    ctor(int use, const string_type& ls, const string_type& lm, const string_type& lh, const string_type& ss, const string_type& sm, const string_type& sh);
    ctor(int use, const self& d);
    string_type short_name<Period>() const { return short_name(Period::type{}); }
    string_type long_name<Period>() const { return long_name(Period::type{}); }
    string_type plural<Period>() const { return long_name(Period::type{}); }
    string_type singular<Period>() const { return {long_name(), 0, long_name().size()-1}; }
    string_type name<Period>() const { return use_short_? short_name<Period>() : long_name<Period>(); }
    string_type name<Period,D>(D v) const { return use_short_? short_name<Period>() : (v==-1||v==1) ? singular<Period>() : plural<Period>(); }
    bool is_short_name() const { return use_short_; } bool is_long_name() const { return !use_short_; }
};

std::basic_ostream<Ch,Tr>& duration_short <Ch,Tr> (std::basic_ostream<Ch,Tr>& os); // make duration_punct(use_short) a facet
std::basic_ostream<Ch,Tr>& duration_long <Ch,Tr> (std::basic_ostream<Ch,Tr>& os); // make duration_punct(use_long) a facet
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,Rep,Period> (std::basic_ostream<Ch,Tr>& os, const duration<Rep,Period>& d);

bool detail::reduce(Inter& r, unsigned long long& den, std::ios_base::iostate& err); // (r/den) is integral
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr,Rep,Period> (std::basic_istream<Ch,Tr>& is, duration<Rep,Period>& d);

std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,Clock,Duration> (std::basic_ostream<Ch,Tr>& os, const time_point<Clock,Duration>& tp);
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr,Clock,Duration> (std::basic_istream<Ch,Tr>& is, time_point<Clock,Duration>& tp);
```

#### Version 2

##### Common Parts

```c++
// ios flags / facets
struct detail::xalloc_key_holder<T> { static int value; static bool initialized; };
struct xalloc_key_initializer<T> { ctor(){ if (!xalloc_key_holder<T>::initialized) { xalloc_key_holder<T>::value = std::ios_base::xalloc(); xalloc_key_holder<T>::initialized = true; } } };
class ios_state_ptr<Final,T> { // no copy
    static bool is_registered(std::ios_base& ios) { long iw = ios.iword(index()); return iw == 1; }
    static bool set_registered(std::ios_base& ios) { long& iw = ios.iword(index()); iw = 1; }
    static void callback(std::ios_base::event evt, std::ios_base& ios, int index) {
        switch (evt) {
        case erase_event: { void*& pw = ios.pword(index); if (pw) { delete (T*)pw; pw = nullptr; } break; }
        case copyfmt_event: { void*& pw = ios.pword(index); if (pw) pw = new T{*(T*)pw}; break; }
        }
    }
    static int index() { return xalloc_key_holder<Final>::value; }
    static void register_once(int indx, std::ios_base& ios) { if (!is_registered(ios)) { set_registered(ios); ios.register_callback(callback, indx); } }
protected: std::ios_base& ios_;
public: using element_type = T;
    explicit ctor(std::ios_base& ios); ~dtor();
    T <const>* get() <const> noexcept { register_once(index(), ios_); void*& pw = ios_.pword(index()); return (const T*)pw; }
    T <const>* operator->() <const> noexcept { return get(); }
    T <const>& operator*() <const> noexcept { return *get(); }
    T* release() noexcept() { void*& pw = ios_.pword(index()); T* ptr = (T*)pw; pw = nullptr; return ptr; }
    void reset(T* new_ptr=nullptr) noexcept { register_once(index(), ios_); void*& pw = ios_.pword(index()); delete (T*)pw; pw = new_ptr; }
    explicit operator bool() const noexcept { return get() != nullptr; }
    std::ios_base& getios() <const> noexcept { return ios_; }
    operator std::ios_base& () <const> noexcept { return ios_; }
};
struct ios_state_not_null_ptr<Final,T> : public ios_state_ptr<Final,T> {
    explicit ctor(std::ios_base& ios) : base{ios}{ if (get()==nullptr) base::reset(new T{}); }
    void reset(T* new_value) noexcept { assert(new_value != nullptr); base::reset(new_value); }
};
class ios_flags<Final> { // no copy
    std::ios_base& ios_;
    long value() const noexcept { return ios_.iword(index()); }
    long& ref() noexcept { return ios_.iword(index()); }
    static int index() { return xalloc_key_holder<Final>::value; }
public: explicit ctor(std::ios_base& ios) : ios_{ios}{}  ~dtor(){}
    long flags() const noexcept { return value(); }
    long flags(long v) noexcept { long tmp = flags(); ref() = v; return tmp; }
    long setf(long v) { long tmp = value(); ref() |= v; return tmp; }
    void unsetf(long mask) { ref() &= ~mask; }
    long setf(long v, long mask) { long tmp = value(); unsetf(mask); ref() |= v&mask; return tmp; }
    operator std::ios_base <const>& () <const> noexcept { return ios_; }
};

struct manip<Final> { void operator()(std::ios_base& ios) const { (*(const Final*)this)(ios); } };
Out_stream& operator<< <Out_stream,Manip_type> (Out_stream& out, const manip<Manip_type>& op) { if (out.good()) op(out); return out; }
In_stream& operator>> <In_stream,Manip_type> (In_stream& in, const manip<Manip_type>& op) { if (in.good()) op(in); return in; }

enum class duration_style { prefix, symbol };
enum class timezone { utc, local };
struct fmt_masks : public ios_flags<fmt_masks> { // no copy
    ctor(std::ios_base& ios) : base{ios} {}
    enum type { uses_symbol = 1; uses_local = 2 };
    duration_style get_duration_style() { return flags()&uses_symbol ? symbol : prefix; }
    void set_duration_style(duration_style style) { if (style == symbol) setf(uses_symbol); else unsetf(uses_symbol); }
    timezone get_timezone() { return flags()&uses_local ? local: utc; }
    void set_timezone(timezone tz) { if (tz == local) setf(uses_local) else unsetf(uses_local); }
};
namespace{ xalloc_key_initializer<fmt_masks> init; }
duration_style get_duration_style(std::ios_base& ios) { return fmt_masks(ios).get_duration_style(); }
void set_duration_style(std::ios_base& ios, duration_style style) { fmt_masks(ios).set_duration_style(style); }
std::ios_base& symbol_format(std::ios_base& ios) { fmt_masks(ios).setf(uses_symbol); return ios; }
std::ios_base& name_format(std::ios_base& ios) { fmt_masks(ios).unsetf(uses_symbol); return ios; }
timezone get_timezone(std::ios_base& ios) { return fmt_masks(ios).get_timezone(); }
void set_timezone(std::ios_base& ios, timezone tz) { fmt_masks(ios).set_timezone(tz); }
std::ios_base& local_timezone(std::ios_base& ios) { fmt_masks(ios).setf(uses_local); return ios; }
std::ios_base& utc_timezone(std::ios_base& ios) { fmt_masks(ios).unsetf(uses_local); return ios; }
struct detail::ios_base_data_aux<Ch> { std::basic_string<Ch> time_fmt, duration_fmt; };
struct detail::ios_base_data<Ch>{};
namespace{ xalloc_key_initializer<ios_base_data<char>> init; xalloc_key_initializer<ios_base_data<wchar_t>> winit;
    xalloc_key_initializer<ios_base_data<char16_t>> u16init; xalloc_key_initializer<ios_base_data<char32_t>> u32init; }
std::basic_string<Ch> get_time_fmt<Ch>(std::ios_base& ios) {}

FwdIt detail::scan_keyword<InIt,FwdIt>(InIt& b, InIt e, FwdIt kb, FwdIt ke, std::ios_base::iostate& err);
```

##### Duration IO

```c++
// duration units
struct rt_ratio { intmax_t num, den; ctor<Period>(Period const&); ctor(intmax_t n=0, intmax_t d=0); };
class duration_units<Ch=char> : public std::locale::facet {
public: using char_type = Ch; using string_type = std::basic_string<Ch>;
    static std::locale::id id;
    explicit ctor(size_t refs=0) : base{refs}{}
    virtual const string_type* get_<n_d>_valid_units_{start|end}() const=0;
    virtual bool match_<n_d>_valid_unit(const string_type* k, rt_ratio& rt) const=0;
    virtual string_type get_pattern() const=0;
    string_type get_<n_d>_unit<Rep,Period>(duration_style style, duration<Rep,Period> const& d) const
    { return do_get_<n_d>_unit(style, rt_ratio{Perod{}}, (intmax_t)d.count()); }
    bool is_named_unit<Period>() const { return do_is_named_unit(rt_ratio{Period{}}); }
protected: virtual ~dtor();
    virtual string_type do_get_<n_d>_unit(duration_style style, rt_ratio rt, intmax_t v) const=0;
    virtual bool do_is_named_unit(rt_ratio rt) const=0;
};

struct detail::duration_units_default_holder<Ch> { using string_type = std::basic_string<Ch>;
    static string_type *n_d_valid_units_, *valid_units_; static bool initialized_;
};
class duration_units_default<Ch=char> : public duration_units<Ch> { using Holder = duration_units_default_holder<Ch>;
public: using char_type = Ch; using string_type = std::basic_string<Ch>;
    explicit ctor(size_t refs=0) : base{refs}{}  ~dtor(){}
    bool match_n_d_valid_unit(const string_type* k) const
    { size_t index = (k-get_n_d_valid_units_start()) / (pfs_+1); return index == 0; }
    bool match_valid_unit(const string_type* k, rt_ratio& rt) const {
        size_t index = (k-get_n_d_valid_units_start()) / (pfs_+1);
        switch (index){
            case 0~15: rt = rt_ratio{std::XX{}}; break; // std::atto ~ std::exa
            case 16~18: rt = rt_ratio{ratio<1|60|3600>} break;
        };
        return index <= 18;
    }
    const string_type* get_n_d_valid_units_start() const override { return Holder::n_d_valid_units_; }
    const string_type* get_n_d_valid_units_end() const override { return Holder::n_d_valid_units_ + (pfs_+1); }
    const string_type* get_valid_units_start() const override { return Holder::valid_units_; }
    const string_type* get_valid_units_end() const override { return Holder::valid_units_ + 19*(pfs_+1); }
    string_type get_pattern() const override { static const string_type pattern{"%v %u"}; return pattern; }
protected: static const size_t pfs_=2;
    bool do_is_named_unit(rt_ratio rt) const override; // recognize rt as std::atto~std::exa, ratio<1/1>, ratio<60/1>, ratio<3600/1>
    string_type do_get_n_d_unit(duration_style style, rt_ratio, intmax_t v) const override
    { return do_get_unit(style, ratio<1>{}, do_get_plural_form(v)); }
    string_type do_get_unit(duration_style style, rt_ratio rt, intmax_t v) const override
    { return do_get_unit(style, ratio<>{}, do_get_plural_form(v)); } // recognize rt as std::atto~std::exa, ratio<1/1>, ratio<60/1>, ratio<3600/1>

    virtual size_t do_get_plural_forms() const { return static_get_plural_forms(); }
    static size_t static_get_plural_forms() { return pfs_; }
    virtual size_t do_get_plural_form(int_least64_t value) const { return static_get_plural_form(value); }
    static size_t static_get_plural_form(int_least64_t value) { return (value==-1||value==1) ? 0 : 1; }
    virtual string_type do_get_unit(duration_style style, ratio<1|60|3600> u, size_t pf) const { return static_get_unit(style,u,pf); }
    static string_type static_get_unit(duration_style style, ratio<1|60|3600>, size_t pf); // recognize "s/m/h", "second/minute/second", "seconds/minutes/seocnds"
    virtual string_type do_get_unit(duration_style style, {atto|...|exo} u, size_t pf) const { return do_get_ratio_prefix(style,u) + do_get_unit(style, ratio<1>{}, pf); }
    static string_type static_get_unit(duration_style style, {atto|...|exo} u, size_t pf) { return static_get_ratio_prefix(style,u) + static_get_unit(style,ratio<1>{},pf); }
    virtual string_type do_get_ratio_prefix(duration_style style, {atto|...|exo} u) const { return static_get_ratio_prefix(style, u); }
    static string_type static_get_ratio_prefix(duration_style style, {atto|...|exo})
    { return style == symbol ? ratio_string<{atto|...|exo},Ch>::symbol() : ratio_string<{atto|...|exo},Ch>::prefix(); }

    string_type* fill_units<Period>(string_type* it, Period) const; // fill all plural prefixes and symbol prefix to string array
public: static string_type* static_fill_units<Period>(string_type* it, Period); // fill all plural prefixes and symbol prefix to string array
    static string_type* static_init_valid_units(string_type* it); // call static_fill_units on atto~exa, and ratio<1>,ratio<60>,ratio<3600>
};
struct detail::duration_units_default_initializer_t { using Holder = duration_units_default_holder<Ch>; using string_type = Holder<Ch>::string_type;
    ctor() { if (!Holder::initialized) {
        Holder::n_d_valid_units_ = new string_type[3]; Holder::valid_units_ = new string_type[19*3];
        string_type* it = Holder::n_d_valid_units_; duration_units_default<Ch>::static_fill_units(it, ratio<1>{});
        it = Holder::valid_units_; duration_units_default<Ch>::static_init_valid_units(it);
        Holder::initialized = true;
    } }
    ~dtor() { if (Holder::initialized) { delete[] Holder::n_d_valid_units_; delete[] Holder::valid_units_; Holder::initialized = false; } }
};
namespace{ duration_units_default_initializer_t<char> init; duration_units_default_initializer_t<wchar_t> winit; }

// duration_put facet
struct duration_put<Ch,OutIt=std::ostreambuf_iterator<Ch>> : public std::locale::facet {
    using char_type = Ch; using string_type = std::basic_string<Ch>; using iter_type = OutIt;
    static std::locale::id id;
    explicit ctor(size_t refs=0) : base{refs}{} ~dtor(){}
    iter_type put<Rep,Period>(duration_units<Ch> const& units_facet, iter_type s, std::ios_base& ios, char_type fill, duration<Rep,Period> const& d, const Ch* pattern, const Ch* pat_end, const char_type* val=nullptr) const;
    iter_type put<Rep,Period>(iter_type s, std::ios_base& ios, char_type fill, duration<Rep,Period> const& d, const Ch* pattern, const Ch* pat_end, const char_type* val=nullptr) const;
    iter_type put<Rep,Period>(iter_type s, std::ios_base& ios, char_type fill, duration<Rep,Period> const& d, const char_type* val=nullptr) const; // use facet's pattern
    iter_type put_value<Rep,Period>(iter_type s, std::ios_base& ios, char_type fill, duration<Rep,Period> const& d, const char_type* val=nullptr) const;
    iter_type put_value<Rep,Period>(iter_type s, std::ios_base& ios, char_type fill, duration<process_times<Rep>,Period> const& d, const char_type* val=nullptr) const;
    iter_type put_unit<Rep,Period>(duration_units<Ch> const& facet, iter_type s, std::ios_base& ios, char_type fill, duration<Rep,Period> const& d) const;
    iter_type put_unit<Rep,Period>(duration_units<Ch> const& facet, iter_type s, std::ios_base& ios, char_type fill, duration<process_times<Rep>,Period> const& d) const;
    iter_type put_unit<Rep,Period>(iter_type s, std::ios_base& ios, char_type fill, duration<Rep,Period> const& d) const;
};

// duration_get facet
struct duration_get<Ch,InIt=std::istreambuf_iterator<Ch>> : public std::locale::facet {
    using char_type = Ch; using string_type = std::basic_string<Ch>; using iter_type = InIt;
    static std::locale::id id;
    explicit ctor(size_t refs=0) : base{refs}{} ~dtor(){}
    iter_type get<Rep,Period>(duration_units<Ch> const& facet, iter_type s, iter_type end, std::ios_base& ios, std::ios_base::iostate& err, duration<Rep,Period>& d, const Ch* pattern, const Ch* pat_end) const;
    iter_type get<Rep,Period>(iter_type s, iter_type end, std::ios_base& ios, std::ios_base::iostate& err, duration<Rep,Period>& d, const Ch* pattern, const Ch* pat_end) const;
    iter_type get<Rep,Period>(iter_type s, iter_type end, std::ios_base& ios, std::ios_base::iostate& err, duration<Rep,Period>& d) const; // use facet's pattern
    iter_type get_value<Rep>(iter_type s, iter_type end, std::ios_base& ios, std::ios_base::iostate& err, Rep& r) const;
    iter_type get_value<Rep>(iter_type s, iter_type end, std::ios_base& ios, std::ios_base::iostate& err, process_times<Rep>& r) const;
    iter_type get_unit(duration_units<Ch> const& facet, iter_type i, iter_type e, std::ios_base& ios, std::ios_base::iostate& err, rt_ratio& rt) const;
    iter_type get_unit(iter_type i, iter_type e, std::ios_base& ios, std::ios_base::iostate& err, rt_ratio& rt) const;
protected:
    iter_type do_get_n_d_valid_unit(duration_units<Ch> const& facet, iter_type i, iter_type e, std::ios_base&, std::ios_base::iostate& err) const;
    iter_type do_get_valid_unit(duration_units<Ch> const& facet, iter_type i, iter_type e, std::ios_base&, std::ios_base::iostate& err, rt_ratio& rt) const;
};

// duration format manipulator
class duration_fmt: public manip<duration_fmt> {
    duration_style style_;
public: explicit ctor(duration_style style) noexcept : style_{style}{}
    void operator()(std::ios_base& ios) const { set_duration_style(ios, style_); }
};

class duration_style_io_saver { // no copy
    state_type& s_save_; aspect_type a_save_;
public:
    using state_type = std::ios_base; using aspect_type = duration_style;
    explicit ctor(state_type& s) :s_save_{s}, a_save_{get_duration_style(s)}{}
    ctor(state_type& s, aspect_type new_value) :self{s}{ set_duration_style(s, new_value); }
    ~dtor() { restore(); }
    void restore() { set_duration_style(s_save_, a_save_); }
};

struct duration_put_enabled<Rep> : bool_constant<is_integral_v<Rep> || is_floating_point_v<Rep>>{};
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,Rep,Period> (std::basic_ostream<Ch,Tr>& os, const duration<Rep,Period>& d); // use duration_put facet
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr,Rep,Period> (std::basic_istream<Ch,Tr>& is, duration<Rep,Period>& d); // use duration_get facet
```

##### Timepoint IO

```c++
// time_point units
std::basic_string<Ch> get_epoch_custom<Ch,Clock,TPUFacet>(Clock, TPUFacet& f) { return f.do_get_epoch(Clock{}); }
class time_point_units<Ch=char> : public std::locale::facet {
protected: virtual ~dtor();
public: using char_type = Ch; using string_type = std::basic_string<Ch>;
    static std::locale::id id;
    explicit ctor(size_t refs=0) : base{refs}{}
    virtual string_type get_pattern() const=0;
    string_type get_epoch<Clock>() const { return get_epoch_custom<Ch>(Clock{}, *this); }
    virtual string_type do_get_epoch(system_clock) const=0;
    virtual string_type do_get_epoch(steady_clock) const=0;
    virtual string_type do_get_epoch(process_{real|user|system}_cpu_clock) const=0;
    virtual string_type do_get_epoch(process_cpu_clock) const=0;
    virtual string_type do_get_epoch(thread_clock) const=0;
};

class time_point_units_default<Ch=char> : public time_point_units<Ch> {
public: using char_type = Ch; using string_type = std::basic_string<Ch>;
    explicit ctor(size_t refs=0) : base{refs}{}  ~dtor(){}
    string_type get_pattern() const override { static const string_type pattern{"%d%e"}; return pattern; }
    string_type do_get_epoch(system_clock) const override { return clock_string<system_clock,Ch>::since(); }
    string_type do_get_epoch(steady_clock) const override { return clock_string<steady_clock,Ch>::since(); }
    string_type do_get_epoch(process_{real|user|system}_cpu_clock) const override { return clock_string<process_{real|user|system}_cpu_clock,Ch>::since(); }
    string_type do_get_epoch(process_cpu_clock) const override { return clock_string<process_cpu_clock,Ch>::since(); }
    string_type do_get_epoch(thread_clock) const override { return clock_string<thread_clock,Ch>::since(); }
};

// time_point_put facet
struct time_point_put<Ch,OutIt=std::ostreambuf_iterator<Ch>> : public std::locale::facet {
    using char_type = Ch; using string_type = std::basic_string<Ch>; using iter_type = OutIt;
    static std::locale::id id;
    explicit ctor(size_t refs=0) : base{refs}{} ~dtor(){}
    iter_type put<Clock,Duration>(time_point_units<Ch> const& units_facet, iter_type s, std::ios_base& ios, char_type fill, time_point<Clock,Duration> const& tp, const Ch* pattern, const Ch* pat_end) const;
    iter_type put<Clock,Duration>(iter_type i, std::ios_base& ios, char_type fill, time_point<Clock,Duration> const& tp, const Ch* pattern, const Ch* pat_end) const;
    iter_type put<Clock,Duration>(iter_type i, std::ios_base& ios, char_type fill, time_point<Clock,Duration> const& tp) const; // use facet's pattern
    iter_type put_duration<Rep,Period>(iter_type i, std::ios_base& ios, char_type fill, duration<Rep,Period> const& d) const;
    iter_type put_epoch<Clock>(time_point_units<Ch> const& facet, iter_type s, std::ios_base&) const;
    iter_type put_epoch<Clock>(iter_type i, std::ios_base&) const;
};

// time_point_get facet
struct time_point_get<Ch,InIt=std::istreambuf_iterator<Ch>> : public std::locale::facet {
    using char_type = Ch; using iter_type = InIt;
    static std::locale::id id;
    explicit ctor(size_t refs=0) : base{refs}{} ~dtor(){}
    iter_type get<Clock,Duration>(time_point_units<Ch> const& facet, iter_type s, iter_type end, std::ios_base& ios, std::ios_base::iostate& err, time_point<Clock,Duration> &tp, const Ch* pattern, const Ch* pat_end) const;
    iter_type get<Clock,Duration>(iter_type i, iter_type e, std::ios_base& ios, std::ios_base::iostate& err, time_point<Clock,Duration> &tp, const Ch* pattern, const Ch* pat_end) const;
    iter_type get<Clock,Duration>(iter_type i, iter_type e, std::ios_base& ios, std::ios_base::iostate& err, time_point<Clock,Duration> &tp) const;
    iter_type get_duration<Rep,Period>(duration_get<Ch> const& facet, iter_type s, iter_type end, std::ios_base& ios, std::ios_base::iostate& err, duration<Rep,Period>& d) const;
    iter_type get_duration<Rep,Period>(iter_type i, iter_type e, std::ios_base& ios, std::ios_base::iostate& err, duration<Rep,Period>& d) const;
    iter_type get_epoch<Clock>(time_point_units<Ch> const& facet, iter_type i, iter_type e, std::ios_base&, std::ios_base::iostate& err) const;
    iter_type get_epoch<Clock>(iter_type i, iter_type e, std::ios_base& ios, std::ios_base::iostate& err) const;
};

// std facet wrap
struct detail::time_get<Ch,InIt=std::istreambuf_iterator<Ch>> {
    std::time_get<Ch> const& that_;
    ctor(std::time_get<Ch> const& that) :that_{that}{}
    using facet=std::time_get<CharT>; using iter_type = facet::iter_type; using char_type = facet::char_type; using string_type = std::basic_string<char_type>;
    static int get_up_to_n_digits(InIt& b, InIt e, std::ios_base::iostate& err, const std::ctype<Ch>& ct, int n);
    void get_{day|month|year4|hour|minute|second|12_hour|day_year_num|weekday}(int& v, iter_type& b, iter_type e, std::ios_base::iostate& err, const std::ctype<Ch>& ct) const;
    void get_{white_space|percent}(iter_type& b, iter_type e, std::ios_base::iostate& err, const std::ctype<Ch>& ct) const;
    InIt get(iter_type b, iter_type e, std::ios_base& iob, std::ios_base::iostate& err, std::tm* tm, char fmt, char) const;
    InIt get(iter_type b, iter_type e, std::ios_base& iob, std::ios_base::iostate& err, std::tm* tm, const char_type* fmtb, const char_type* fmte) const;
};

// manipulators
class detail::time_manip<Ch> : public manip<self> {
    std::basic_string<Ch> fmt_; timezone tz_;
public: ctor(timezone tz, std::basic_string<Ch> fmt) :fmt_{fmt}, tz_{tz}{}
    void operator()(std::ios_base& ios) const { set_time_fmt<Ch>(ios, fmt_); set_timezone(ios, tz_); }
};
class detail::time_man : public manip<self> {
    timezone tz_;
public: ctor(timezone tz) :tz_{tz}{}
    void operator()(std::ios_base& ios) const { set_timezone(ios, tz_); }
};
time_manip<Ch> time_fmt<Ch>(timezone tz, {const Ch*|std::basic_string<Ch>} fmt) { return {tz,fmt}; }
time_man time_fmt(timezone tz) { return {tz}; }

class time_fmt_io_saver<Ch=char,Tr=std::char_traits<Ch>> {
    state_type& s_save_; aspect_type a_save_;
public: using state_type = std::ios_base; using aspect_type = basic_string<Ch,Tr>;
    explicit ctor(state_type& s) :s_save_{s}, a_save_{get_time_fmt<Ch>(s_save_)}{}
    ctor(state_type& s, aspect_type new_value) :self{s}{ set_time_fmt(s_save_, new_value); }
    ~dtor() { restore(); }
    void restore() { set_time_fmt(s_save_, a_save_); }
};
class timezone_io_saver {
    state_type& s_save_; aspect_type a_save_;
public: using state_type = std::ios_base; using aspect_type = timezone;
    explicit ctor(state_type& s) :s_save_{s}, a_save_{get_timezone(s_save_)}{}
    ctor(state_type& s, aspect_type new_value) :self{s}{ set_timezone(s_save_, new_value); }
    ~dtor() { restore(); }
    void restore() { set_timezone(s_save_, a_save_); }
};

std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,Clock,Duration> (std::basic_ostream<Ch,Tr>& os, const time_point<Clock,Duration>& tp); // use time_point_put facet
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr,Clock,Duration> (std::basic_istream<Ch,Tr>& is, time_point<Clock,Duration>& tp); // use time_point_get facet

int32_t detail::is_leap(int32_t year);
int32_t detail::days_from_0(int32_t year);
int32_t detail::days_from_1970(int32_t year);
int32_t detail::days_from_1jan(int32_t year, int32_t month, int32_t day);
time_t detail::internal_timegm(std::tm const* t);
unsigned detail::days_before_years(int32_t y);
void civil_from_days<Int>(Int z, Int& y, unsigned& m, unsigned& d) noexcept;
std::tm* detail::internal_gmtime(time_t const* t, std::tm* tm);
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,Duration> (std::basic_ostream<Ch,Tr>& os, const time_point<system_clock,Duration>& tp);

minutes detail::extract_z<Ch,InIt>(InIt& b, InIt e, std::ios_base::iostate& err, const std::ctype<Ch>& ct);
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr,Duration> (std::basic_istream<Ch,Tr>& is, time_point<system_clock,Duration>& tp);
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
