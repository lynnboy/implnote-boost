# Boost.Numeric/Interval

* lib: `boost/libs/numeric/interval`
* repo: `boostorg/interval`
* commit: `26800d3`, 2025-07-02

------
### Interval Library

The *Interval* is a value range (value set) from the value domain of a numeric type.

Header `<boost/numeric/interval.hpp>`

------
#### Interval Class

```c++
class comparison_error: public std::runtime_error;

concept bool Checking<Tr, T> = requires (T const& t) {
  { Tr::is_nan(t) } -> bool; { Tr::nan() } -> T;
  { Tr::is_empty(t, t) } -> bool; { Tr::empty_lower() } -> T; { Tr::empty_upper() } -> T;
  { Tr::neg_inf() } -> T; { Tr::pos_inf() } -> T;
};

concept bool ArithRounding<Tr, T, T2> = default_constructible<Tr> && destructible<Tr> &&
  requires (Tr r, T const& t, T2 const& t2) {
  { r.conv_{down|up}(t2) } -> T;                // convert
  { r.{add|sub|mul|div}_{down|up}(t, t) } -> T; // basic arithmetic
  { r.median(t, t) } -> T;
  { r.sqrt_{down|up}(t, t) } -> T;              // sqrt
  { r.int_{down|up}(t, t) } -> T;               // floor/ceil
};

concept bool TranscRounding<Tr, T> = ArithRounding<Tr, T> &&
  requires (Tr r, T const& t) {
  { r.{exp|log|sin|cos|tan|asin|acos|atan|sinh|cosh|tanh|asinh|acosh|atanh}_{down|up}(t) } -> T;
};

concept bool ProtectedRounding<Tr, T> =
    ArithRounding<Tr, T> && ArithRounding<Tr::unprotected_rounding, T>;

struct policies<Rounding, Checking> { using rounding=Rounding; using checking = Checking; };

struct change_rounding<Interval, NewRounding> =
    interval<Interval::base_type, policies<NewRounding, Interval::traits_type::checking>;
struct change_checking<Interval, NewChecking> =
    interval<Interval::base_type, policies<Interval::traits_type::rounding, NewChecking>;
struct unprotect<Interval> =
    change_rounding<Interval, Interval::traits_type::rounding::unprotected_rounding>;

class interval<T, Policies> requires Checking<Policies::checking> && ArithRounding<Policies::rounding> {
  T low, up;
public:
  using base_type = T; using traits_type = Policies;
  T const & lower() const; T const& upper() const;

  static interval empty();                  // [empty_lower,empty_upper]
  static interval whole();                  // [neg_inf,pos_inf]
  static interval hull(const T&, const T&); // [min(x,y),max(x,y)], any NaN is excluded

  interval(); // [0,0]
  interval(T const&); interval<T1>(T1 const&);
  interval(T const& l, T const& u); interval<T1,T2>(T1 const& l, T2 const& u);  // empty if l > u
  interval(interval const&); interval<T1,Policies1>(interval<T1,Policies1> const&);
  // operator= for T, <T1>, interval, interval<T1,Polices1>
  void assign(T const&, T const&);
  // operators '+=', '-=', '*=', '/='
  // operators '<', '>', '<=', '>=', '==', '!='
};
```

* `checking` define what is empty `[empty_lower,empty_upper]` and what is whole `[neg_inf,pos_inf]`,
  also test NaN value and empty interval.
* On construction and assignment, test `is_nan` for single value, `is_empty` for pair or interval, if result is NaN or empty
  then the resulting interval is empty, the incoming interval's empty check is taken by its own policy.
* On construction and assignment from a different type, the `rounding` policy is used to round values.
* Interval default comparison is defined only for non-overlaped non-empty intervals, otherwise will throw `comparison_error`
* Intervals are equal only when both are singulars

------
#### Basic Access And Constants

Header `<boost/numeric/interval/utility_fwd.hpp>`, `<boost/numeric/interval/utility.hpp>`

```c++
Interval pi<Interval>(); Interval pi_half<Interval>(); Interval pi_twice<Interval>(); // PI constants

T const& lower(interval<T,P> const&); T const& upper(interval<T,P> const&);
T const& checked_lower(interval<T,P> const&);  // checked version return 'nan' for 'empty'

T width(interval<T,P> const&);  // return 0 for 'empty'
T median(interval<T,P> const&);  // return 'nan' for 'empty'
interval<T,P> widen(interval<T,P> const&, T const&);  // enclose value in interval, return 'empty' for 'empty'

bool empty(interval<T,P> const&); bool singleton(interval<T,P> const&);
bool zero_in(interval<T,P> const&); bool in(T const&, interval<T,P> const&);
bool [proper_]subset(interval<T,P1> const&, interval<T,P2> const&);
bool {overlap|equal}(interval<T,P1> const&, interval<T,P2> const&);    // equal is range/set equal, not '=='
interval<T,P> intersect(interval<T,P> const&, interval<T,P> const&);
interval<T,P> hull({interval<T,P>|T} const&, {interval<T,P>|T} const&);         // union of sets
auto bisect(interval<T,P> const&) -> std::pair<interval<T,P>, interval<T,P>>;   // cut set

T norm(interval<T,P> const&);  // 'nan' for empty, and return max(abs(lower),abs(upper))
interval<T,P> abs(interval<T,P> const&);  // return [0,norm] if zero in range
interval<T,P> {max|min}({interval<T,P>|T} const&, {interval<T,P>|T} const&);    // range min/max
```

------
#### Arithmetics

```c++
interval<T,P> operator {+|-}(interval<T,P> const&);
interval<T,P> operator {+|-|*|/}({interval<T,P>|T} const&, {interval<T,P>|T} const&);

interval<T,P> {add|sub|mul|div}(T const&, T const&);

interval<T,P> fmod({interval<T,P>|T} const&, {interval<T,P>|T} const&);
interval<T,P> division_part1(interval<T,P> const&, interval<T,P> const&, bool&);
interval<T,P> division_part2(interval<T,P> const&, interval<T,P> const&, bool=true);
interval<T,P> multiplicative_inverse(interval<T,P> const&);

interval<T,P> pow(interval<T,P> const&, int);
interval<T,P> sqrt(interval<T,P> const&);
interval<T,P> square(interval<T,P> const&);
interval<T,P> nth_root(interval<T,P> const&, int);
```

* Any `nan` or `empty` cause result be `empty`.
* Use `P::rounding` to perform rounding.
* Some functions in namespace `boost::numeric::interval_lib`, others in `boost::numeric`.

------
#### Transcend

```c++
interval<T,P> exp(interval<T,P> const&) requires TranscRounding<P::rounding, T>;
interval<T,P> log(interval<T,P> const&) requires TranscRounding<P::rounding, T>;
interval<T,P> [a]{cos|sin|tan}[h](interval<T,P> const&) requires TranscRounding<P::rounding, T>;
```

* User needs to explicit use a `TranscRounding` policy, the default policies is just `ArithRounding`.

------
#### Comparisons

```c++
namespace ::boost::numeric::interval_lib::compare::certain {
  bool operator {<|<=|>|>=|==|!=} (interval<T,P1> const&, {interval<T,P2>|T} const&);
}
namespace ::boost::numeric::interval_lib::compare::possible {
  bool operator {<|<=|>|>=|==|!=} (interval<T,P1> const&, {interval<T,P2>|T} const&);
}
namespace ::boost::numeric::interval_lib::compare::lexicographic {
  bool operator {<|<=|>|>=|==|!=} (interval<T,P1> const&, {interval<T,P2>|T} const&);
}
namespace ::boost::numeric::interval_lib::compare::set {
  bool operator {<|<=|>|>=|==|!=} (interval<T,P1> const&, {interval<T,P2>|T} const&);
}
namespace ::boost::numeric::interval_lib::compare::tribool {
  boost::logic::tribool operator {<|<=|>|>=|==|!=} (interval<T,P1> const&, {interval<T,P2>|T} const&);
}
namespace ::boost::numeric::interval_lib {  // explicit comparisons
  boost::logic::tribool {cer|pos}{lt|le|gt|ge|eq|ne} ({interval<T,P1>|T} const&, {interval<T,P2>|T} const&);
}
```

* Operators always throws `comparison_error` when input is `nan` or `empty`, except some cases of `set::`
* Exlicit comparisons don't test for emptyness

------
#### Predefined Checking Policy

```c++
struct exception_create_empty;    // throw 'std::runtime_error'
struct exception_invalid_number;  // throw 'std::invalid_argument'

struct checking_base<T> {
  static T {pos|neg}_inf();   // get 'std::numeric_limits<T>::infinity' by default
  static T nan();                 static bool is_nan(T const&); // get quiet_NaN by default
  static T empty_{lower|upper}(); static bool is_empty(T const&, T const&); // [-inf,inf], or [1,0]
};

struct checking_no_empty<T, Checking=checking_base<T>, Exception=exception_create_empty> : Checking {
  static T nan() { assert(false); return Checking::nan() }
  static T empty_{lower|upper}() { Exception()(); return Checking::empty_{lower|upper}(); }
  static bool is_empty(T const&, T const&) { return false; }
};
struct checking_no_nan<T, Checking=checking_base<T>> : Checking {
  static bool is_nan(T const&) { return false; }
};
struct checking_catch_nan<T, Checking=checking_base<T>, Exception=exception_invalid_number> : Checking {
  static bool is_nan(T const&) { if (Checking::is_nan(x)) Exception()(); return false; }
};
struct checking_strict<T> : checking_no_nan<T, checking_no_empty<T>> {};
```

* `checking_strict` will throw whenever empty `interval` appears.

------
#### Predefined Rounding Policy

```c++
struct rounding_control<T> {  // define API to perform rounding
  using rounding_mode = int;
  static void {get|set}_rounding_mode(rounding_mode{&|}) {} // fetch/apply rounding mode of environment
  static void {upward|downward|to_nearest} {}               // update environment's rounding mode
  static const T& to_int(T const& x) { return x; }          // round to integer according to mode
  static const T& force_rounding(T const& x) { return x; }  // round according to current rounding mode
};

struct rounded_arith_{exact|std|opp}<T, Rounding=rounding_control<T>> : Round {
  void init();
  T conv_{down|up}<U> (U const&);
  T {add|sub|mul|div}_{down|up} (T const&, T const&);
  T median (T const&, T const&);
  T {sqrt|int}_{down|up} (T const&);
};
struct rounded_transc_{exact|std|opp}<T, Rounding=rounded_arith_{exact|std|opp}<T>> : Round {
  T {exp|log}_{down|up} (T const&);
  T [a]{sin|cos|tan}[h]_{down|up} (T const&);
};

struct save_state_unprotected<Rounding> : Rounding {
  using unprotected_rounding = save_state_unprotected<Rounding>;  // unprotected of unprotected
};
struct save_state<Rounding> : Rounding {
  rounding_mode mode;
  save_state() { get_rounding_mode(mode); init(); }
  ~save_state() { set_rounding_mode(mode); }
  using unprotected_rounding = save_state_unprotected<Rounding>;
};
struct save_state_nothing<Rounding> : Rounding {    // don't save state
  using unprotected_rounding = save_state_nothing<Rounding>;
};

struct rounded_math<T> : save_state_nothing<rounded_arith_exact<T>> {};
struct rounded_math<FP> : save_state<rounded_arith_opp<FP>> {}; // for float/double/long double
```

* `_exact` roundings don't use rounding mode at all.
* `_std` roundings change rounding mode before each calculation.
* `_opp` roundings try keeps `upward` mode, only change mode when necessary and will restore back the mode.
* `rounded_math` by default (integers) use exact arith, while fp types use `arith_opp` and supports state saving.
* `rounding_control` is specialized for various compilers/architectures.

------
#### Misc Utilities

Header `<boost/numeric/interval/limits.hpp>` and `<boost/numeric/interval/io.hpp>`

```c++
class std::numeric_limits<boost::numeric::interval<T, Policies>>; // infinity is whole, quiet_NaN is empty

auto operator<<(std::basic_ostream<C,Tr> &, interval<T,P> const&); // '[l,u]', empty is '[]'
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/limits.hpp>`
* `<boost/config/no_tr1/cmath.hpp>`

#### Boost.Detail

* `<boost/detail/fenv.hpp>` - when C99 hardware rounding is used

#### Boost.Tribool

* `<boost/logic/tribool.hpp>` - required by `tribool` interval comparison.

------
### Standard Facilities
