# Boost.Numeric/Interval

* lib: `boost/libs/numeric/interval`
* repo: `boostorg/interval`
* commit: `538299d4`, 2015-01-09

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

concept bool ProtectedRounding<Tr, T> = ArithRounding<Tr, T> && ArithRounding<Tr::unprotected_rounding, T>;

struct policies<Rounding, Checking> { using rounding=Rounding; using checking = Checking; };

struct change_rounding<Interval, NewRounding> =
    interval<Interval::base_type, policies<NewRounding, Interval::traits_type::checking>;
struct change_checking<Interval, NewChecking> =
    interval<Interval::base_type, policies<Interval::traits_type::rounding, NewChecking>;
struct unprotect<Interval> = change_rounding<Interval, Interval::traits_type::rounding::unprotected_rounding>;

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

```c++
Interval pi<Interval>(); Interval pi_half<Interval>(); Interval pi_twice<Interval>(); // PI constants

T const& [checked_]{lower|upper}(interval<T,P> const&);  // checked version return 'nan' for 'empty'

T width(interval<T,P> const&);  // return 0 for 'empty'
T median(interval<T,P> const&);  // return 'nan' for 'empty'
interval<T,P> widen(interval<T,P> const&, T const&);  // enclose value in interval, return 'empty' for 'empty'

bool empty(interval<T,P> const&); bool singleton(interval<T,P> const&);
bool zero_in(interval<T,P> const&); bool in(T const&, (interval<T,P> const&);
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
#### Misc Utilities

Header `<boost/numeric/interval/limits.hpp>` and `<boost/numeric/interval/io.hpp>`

```c++
class std::numeric_limits<boost::numeric::interval<T, Policies>>; // infinity is whole, quiet_NaN is empty

auto operator<<(std::basic_ostream<C,Tr> &, interval<T,P> const&); // '[l,u]', empty is '[]'
```

------
### Dependency

#### Boost::Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`.

#### Boost::Assert

* `<boost/assert.hpp>`.

#### Boost::StaticAssert

* `<boost/static_assert.hpp>`.

#### Boost::ThrowException

* `<boost/throw_exception.hpp>`.

#### Boost::Core

* `<boost/swap.hpp>`.
* `<boost/detail/iterator.hpp>`. (May be redundant.)

------
### Standard Facilities

* Standard Library: `<array>`.
