# Boost.Ratio

* lib: `boost/libs/ratio`
* repo: `boostorg/ratio`
* commit: `3be1f312`, 2017-02-18

------
### Compile-Time Rational Number

#### Header

* `<boost/ratio.hpp>`
* `<boost/ratio/ratio.hpp>`
* `<boost/ratio/ratio_fwd.hpp>`
* `<boost/ratio/ratio_io.hpp>`

#### Class Template `ratio` API

```c++
template <intmax_t N, intmax_t D=1>
class ratio {
public:
  static constexpr int_max_t num, den;  // normalized
  using type = ratio<num, den>;         // normalized

  // extensions:
  ratio();  // empty
  ratio(const ratio<N2,D2>&);  // normalized N2/D2 should match this type
  ratio& operator=(const ratio<N2,D2>&);

  using value_type = boost::rational;
  // tag, num_type, den_type
  static value_type value(); value_type operator()() const; // 'rational'
};
ratio_add<R1,R2>;       ratio_subtract<R1,R2>;
ratio_multiply<R1,R2>;  ratio_divide<R1,R2>;

ratio_equal<R1,R2>;   ratio_not_equal<R1,R2>;
ratio_less<R1,R2>;    ratio_less_equal<R1,R2>;
ratio_greater<R1,R2>; ratio_greater_equal<R1,R2>;

// extensions
ratio_power<R,int P>; // R*R*...*R
ratio_negate<R>;      ratio_abs<R>;       ratio_sign<R>;    ratio_inverse<R>;
ratio_gcd<R1,R2>;     ratio_lcm<R1,R2>;   ratio_modulo<R1,R2>;
ratio_min<R1,R2>;     ratio_max<R1,R2>;
```

* Overflow is reported by static assert.
* Can use `static_assert`, `BOOST_STATIC_ASSERT` or `BOOST_MPL_ASSERT_MSG`
* By extension, `ratio` behave as *Boost.MPL* numeric constant.

#### SI types

* Defined SI typedefs from `atto` (1/1000^6) to `exa' (1000^6).
* IEC binary amounts `kibi` (1024) to `exbi` (1024^6).

#### IO Supports

```c++
struct ratio_string<Ratio, CharT> {
  static std::basic_string<CharT> symbol(); // default is 'prefix()'
  static std::basic_string<CharT> prefix(); // default is '[<num>/<den>]'
};
```

* The `ratio_string` symbol and prefix are specialized for all SI and IEC typedefs

#### *Boost.MPL* Integration

Header `<boost/ratio/mpl/rational_constant.hpp>`

* When extensions are enabled, the `ratio` type can be used just like builtin MPL numeric types.
* Arithmetics: `mpl::plus`, `mpl::times`, `mpl::gcd`, etc.
* Comparisons: `mpl::equal_to`, `mpl::less`, etc.
* `mpl::numeric_cast` from integer to `ratio`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`.
* `<boost/cstdint.hpp>`.
* `<boost/detail/workaround.hpp>`.

#### Boost.StaticAssert

* `<boost/static_assert.hpp>` - when configured to use.

#### Boost.Core

* `<boost/core/enable_if.hpp>`

#### Boost.Integer

* `<boost/integer_traits.hpp>`

#### Boost.MPL

* `<boost/mpl/*>`

#### Boost.TypeTraits

* `<boost/type_traits/integral_constant.hpp>`

#### Boost.Rational

* `<boost/rational.hpp>` - when extensions are enabled.

------
### Standard Facilities

* Standard Library: `<ratio>` (C++11)
