# Boost.Ratio

* lib: `boost/libs/ratio`
* repo: `boostorg/ratio`
* commit: `6eceada`, 2024-12-14

------
### Compile-Time Rational Number

#### Header

* `<boost/ratio.hpp>`
* `<boost/ratio/ratio.hpp>`
* `<boost/ratio/ratio_fwd.hpp>`
* `<boost/ratio/ratio_io.hpp>`

#### Class Template `ratio` API

```c++
using std::ratio;
using std::ratio_add; // and `ratio_subtract`, `ratio_multiply`, `ratio_divide`
using std::ratio_equal; // and `ratio_not_equal`, `ratio_less`, `ratio_less_equal`, `ratio_greater`, `ratio_greater_equal`

template<class R1, class R2>
using ratio_gcd = ratio<gcd<R1::num,R2::num>::value, lcm<R1::den,R2::den>::value>::type;
```

#### SI types

```c++
using std::atto; // femto, pico, nano, micro, milli, centi, deci, deca, hecto, kilo, mega, giga, tera, peta, exa
```

#### IO Supports

```c++
struct ratio_string<Ratio, CharT> {
  static std::basic_string<CharT> symbol() { return prefix(); }
  static std::basic_string<CharT> prefix() { return format("[{}/{}]", Ratio::num, Ratio::den)}
};
struct ratio_string<atto,char> { // also for every SI symbol and every char type:
  static std::string symbol() { return "a"; } // f, p, n, '\xB5' (mu), m, c, d, da, h, k, N, M, G, T, P, E
  static std::string prefix() { return "atto"; }
};
```

* The `ratio_string` symbol and prefix are specialized for all SI typedefs

#### *Boost.Chrono* Integration

Header `<boost/ratio/detail/*rational_constant*.hpp>`

```c++
template<class T>                 struct is_ratio : false_type {};
template<intmax_t A, intmax_t B>  struct is_ratio<ratio<A,B>> : true_type {};

template<intmax_t A, intmax_t B>  struct is_evenly_divisible_by_ : bool_constant< A % B == 0> {};
template<intmax_t A>              struct is_evenly_divisible_by_<A, 0> : false_type {}; // no divide by zero
template<class R1, class R2>      struct is_evenly_divisible_by : // R1::num/R2::num, R2::den/R1::den
  bool_constant<is_evenly_divisible_by_<R1::num, R2::num>::value && is_evenly_divisible_by_<R2::den, R1::den>::value> {};

template<intmax_t A, intmax_t B> struct gcd;  // euclid algorithm
template<intmax_t A, intmax_t B> struct lcm;  // euclid algorithm
```

------
### Dependency

None.

------
### Standard Facilities

* Standard Library: `<ratio>` (C++11)
