# Boost.Rational

* lib: `boost/libs/rational`
* repo: `boostorg/rational`
* commit: `894bd14`, 2025-07-04

------
### Rational Number

#### Header

* `<boost/rational.hpp>`

#### Class `bad_rational`

```c++
class bad_rational : public std::domain_error;
```

#### Class `rational`

```c++
using detail::is_comp<Int1,Int2>::value = // nl = std::numeric_limits<>
  !is_array<Int1> && // exclude array type
  (nl<Int1>::is_specialized && nl<Int1>::is_integer && // case 1: Int1 is a built-in integer type
    nl<Int1>::digits <= nl<Int2>::digits && nl<Int1>::radix == nl<Int2>::radix && // Int1's rank not larger than Int2's
    (!nl<Int1>::is_signed || nl<Int2>::is_signed)) || // and either Int1 is unsigned or both are signed
  is_same_v<Int1,Int2> || // case 2: just the same type
  (is_class_v<Int1> && is_class_v<Int2> && is_convertible_t<Int1,Int2>); // case 3: convertible class types

template <typename IntType>
class rational {
  int_type num{0}, den{1};
  constexprvoid normalize();
  constexprbool test_invariant() const; constexpr static bool is_normalized(n, d); // den > 0 && gcd(n,d) == 1
  constexprstatic bool is_safe_nconv<T>(T const& val); // val is representable by IntType
public:
  using int_type = IntType;
  constexpr ctor() {}
  constexpr ctor<T>(T const& n) requires is_comp<T,IntType> : num(n) {}
  constexpr ctor<T,U>(T const& n, U const& d) requires is_comp<T,IntType> && is_comp<U,IntType>
   : num(n), den(d) { normalize(); }
  constexpr ctor<T>(rational<T> const& r) requires is_comp<T,IntType> :num(r.numerator()), den(r.denominator()) {
    if (!is_normalized(num, den)) throw bad_rational(...);
  }
  constexpr ctor<T>(rational<T> const& r) requires !is_comp<T,IntType> :num(r.numerator()), den(r.denominator()) {
    if (!is_normalized(num, den) && !(is_safe_nconv(r.denominator()) && is_safe_nconv(r.numerator())))
      throw bad_rational(...);
  }

  constexpr rational& operator= <T> (const T& n) requires is_comp<T,IntType>;
  constexpr rational& assign <T,U> (const T& n, const U& d) requires is_comp<T,IntType> && is_comp<U,IntType>;

  constexpr IntType const& numerator() const { return num; }
  constexpr IntType const& denominator() const { return den; }

  constexpr rational& operator += (rational const&); // also `-=`, `*=`, `/=`
  constexpr rational& operator += <T> (T const&) requires is_comp<T,IntType>; // also `-=`, `*=`, `/=`
  constexpr rational const& operator ++(); // and `--`, and postfix `++`/`--`
  constexpr bool operator! () const; explicit constexpr operator bool () const;
  constexpr bool operator< (rational const&) const; // and also `>`, `==`
  constexpr bool operator< <T> (T const&) const requires is_comp<T,IntType>; // and also `>`, `==`
};
constexpr rational<IntType> operator + (rational<IntType> const&); // unary +, also -
constexpr rational<IntType,Arg> operator + (rational<IntType> const&, Arg const&); // binary `+`, also `-`, `*`, `/`, and also inversed and both `rational` args
constexpr bool operator < (rational<IntType> const&, Arg const&); // also `>`, `<=`, `>=`, `==`, `!=`, and also inversed and both `rational` args

// IO support
std::istream& operator>> (std::istream&, rational<IntType>&);
std::ostream& operator<< (std::ostream&, rational<IntType> const&);

T rational_cast<T, IntType>(rational<IntType> const& r) { return static_cast<T>(r.numerator()/r.denominator());}
rational<IntType> abs(rational<IntType> const&);
```

Keep normalized if possible.
Throw `bad_rational` whenever encounter error.

#### Specialization for Boost.Integer

`gcd` / `lcm` support for `rational`.

```c++
template<typename IntType> struct gcd_evaluator<rational<IntType>>;
template<typename IntType> struct lcm_evaluator<rational<IntType>>;
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`.
* `<boost/detail/workaround.hpp>`.

#### Boost.Core

* `<boost/utility/enable_if.hpp>`

#### Boost.Integer

* `<boost/integer/common_factor_rt.hpp>` - Run-time GCD/LCM functions

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_convertible.hpp>`, `<boost/type_traits/is_same.hpp>`
* `<boost/type_traits/is_class.hpp>`, `<boost/type_traits/is_array.hpp>`

#### Boost.Utility

* `<boost/call_traits.hpp>`

------
### Standard Facilities

* Proposals
  * n3611 - A Rational Number Library for C++ (not accepted).
