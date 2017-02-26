# Boost.Numeric/Conversion

* lib: `boost/libs/numeric/conversion`
* repo: `boostorg/numeric_conversion`
* commit: `c9d7a49b`, 2016-07-06

------
### Numeric Conversion Traits

Header `<boost/numeric/conversion/conversion_traits.hpp>`

```c++
struct bounds<N> {
  using limits = std::numeric_limits<N>;
  static N lowest() { return limits::is_integer ? limits::min() : -limits::max(); }
  static N highest() { return limits::max(); }
  static N smallest() { return limits::is_integer ? 1 : limits::min(); }
};

enum sign_mixture_enum { {signed|unsigned}_to_{signed|unsigned} };
struct sign_mixture<T, S> : get_sign_mixture<remove_cv<T>, remove_cv<S>>::type {};

enum int_float_mixture_enum { {integral|float}_to_{integral|float} };
struct int_float_mixture<T, S> : get_int_float_mixture<remove_cv<T>, remove_cv<S>>::type {};

enum udt_builtin_mixture_enum { {builtin|udt}_to_{builtin|udt} };
struct udt_builtin_mixture<T, S> : get_udt_builtin_mixture<remove_cv<T>, remove_cv<S>>::type {};

struct get_is_subranged<T, S> {
  using exp<N> = numeric_limits<N>::max_exponent; using dig<N> = numeric_limits<N>::digits;
  using type =
    if_ <is_same<T,S>> .then <false_>
    .else_<
      switch_<get_udt_builtin_mixture<T,S>>
      .case_<builtin_to_builtin>.then<
        switch_<get_int_float_mixture<T,S>>
        .case_<integral_to_integral>.then<        // int -> int
          switch_<get_sign_mixture<T,S>>
          .case_<signed_to_unsigned>.then<true_>  // signed -> unsigned, always subrange, loss negative
          .case_<unsigned_to_signed>.then< dig<T>+1 < dig<S>*2 >  // assume integers always have 2^N size
          .default< dig<T> < dig<S> >             // same sign, less digits cause subrange
        >
        .case_<integral_to_float>.then<false_>    // int -> float not subranged
        .case_<float_to_integral>.then<true_>     // float -> int subranged
        .case_<float_to_float>.then< exp<T> < exp<S> || (exp<T> == exp<S> && dig<T> < dig<S>) >
      >
      .case_<udt_to_builtin>.then<true_>          // assume UDT types are always supertypes of builtins
      .default_<false_>
    >;
};
struct is_subranged<T, S> : get_is_subranged<remove_cv<T>, remove_cv<S>>::type {};

struct get_conversion_traits<T, S> {
  using target_type = T; using source_type = S;

  using trivial = is_same<S,T> ? true_ : false_;    // trivial means no conversion
  using result_type = trivial ? T const& : T; using argument_type = is_arithmetic<S> ? S : S const&;

  using subranged = trivial ? false_ : get_is_subranged<T,S>::type;
  using supertype = subranged ? S : T; using subtype = subranged ? T : S;

  using int_float_mixture = get_int_float_mixture<T,S>;
  using sign_mixture = get_sign_mixture<T,S>;
  using udt_builtin_mixture = get_udt_builtin_mixture<T,S>;
};
struct conversion_traits<T, S> : get_conversion_traits<remove_cv<T>,remove_cv<S>>::type {};

concept bool ConversionTraits<Tr> = requires {
  typename Tr::source_type, Tr::target_type, Tr::result_type, Tr::argument_type;
  { typename Tr::int_float_mixture::value } -> int_float_mixture_enum;
  { typename Tr::udt_builtin_mixture::value } -> udt_builtin_mixture_enum;
  { typename Tr::sign_mixture::value } -> sign_mixture_enum;
  mpl::if_<Tr::trivial,void,void>::type;
  mpl::if_<Tr::subranged,void,void>::type;
};
```

* `conversion_traits` is the default traits used by `converter`
* Trait should state whether the conversion is subranged, and the types of conversion argument/result.

------
### Converter

Header `<boost/numeric/conversion/converter.hpp>`

```c++
struct Trunc<S> {
  static source_type nearbyint(argument_type s) { return s < 0 ? std::ceil(s) : std::floor(s); }
  using round_style = integral_c<std::float_round_style, std::round_toward_zero>;
};
struct Floor<S> {
  static source_type nearbyint(argument_type s) { return std::floor(s); }
  using round_style = integral_c<std::float_round_style, std::round_toward_neg_infinity>;
};
struct Ceil<S> {
  static source_type nearbyint(argument_type s) { return std::ceil(s); }
  using round_style = integral_c<std::float_round_style, std::round_toward_infinity>;
};
struct RoundEven<S> {
  static source_type nearbyint(argument_type s) {
    S prev = floor(s); S next = ceil(s); S rt = (s-prev) - (next-s);
    if (rt < 0.0) return prev; if (rt > 0.0) return next;   // nearest
    bool is_prev_even = floor(prev/2)*2 == prev;
    return is_prev_even ? prev : next;                      // even one
  }
  using round_style = integral_c<std::float_round_style, std::round_to_nearest>;
};

concept bool Float2IntRounder<R,S> = requires(S const& s) {
  { R::round_style::value } -> std::float_round_style;
  { R::nearbyint(s) } -> S;
};

enum range_check_result { cInRange = 0, cNegOverflow = 1, cPosOverflow = 2 };
class bad_numeric_cast : public std::bad_cast;
class negative_overflow : public bad_numeric_cast;
class positive_overflow : public bad_numeric_cast;
struct def_overflow_handler {
  void operator() (range_check_result); // throw(negative_overflow,positive_overflow)
};
struct silent_overflow_handler; // do nothing

concept bool OverflowHandler<H> = DefaultConstructible<H> && requires (H h, range_check_result r) {
  h(r);
};

struct raw_converter<Traits> {
  static result_type low_level_convert(argument_type s) { return static_cast<result_type>(s); }
}

concept bool RawConverter<C,T,S> = requires(S const& s) {
  { C::low_level_convert(s) } -> T;
};

struct UseInternalRangeChecker{}; // tag type

concept bool UserRangeChecker<C,S> = is_same<C, UseInternalRangeChecker> || requires (S const& s) {
  { C::out_of_range(s) } -> range_check_result;
  C::validate_range(s);
};

struct GetRC<Traits, OverflowHandler, Float2IntRounder> { // internal used
  auto LT_LoT(s) { return s < bounds<T>::lowest(); }
  auto LT_Zero(s) { return s < 0; }
  auto LE_PrevLoT(s) { return s <= bounds<T>::lowest() - 1.0; }
  auto LT_HalfPrevLoT(s) { return s <= bounds<T>::lowest() - 0.5; }
  auto GT_HiT(s) { return s > bounds<T>::highest(); }
  auto GE_SuccHiT(s) { return s >= bounds<T>::highest() + 1.0; }
  auto GT_HalfSuccHiT(s) { return s >= bounds<T>::highest() + 0.5; }

  static range_check_result out_of_range(argument_type s) {
    bool n = false; bool p = false;
    if_ (Traits::udt_builtin_mixture != builtin_to_builtin) return cInRange;  // dummy
    switch_ (Traits::int_float_mixture) {
    case_ integral_to_float:  return cInRange;
    case_ float_to_float:
      if_ (Traits::subranged) { n = LT_LoT(s); p = GT_HiT(s); }       // [l, h]
      else_ return cInRange;
    case_ float_to_integral:
      switch_ (Float2IntRounder::round_style) {
      case_ std::round_toward_zero: { n = LE_PrevLoT(s); p = GE_SuccHiT(s); } // (l-1, h+1)
      case_ std::round_to_nearest: { n = LT_HalfPrevLoT(s); p = GT_HalfSuccHiT(s); } // [l-0.5,h+0.5]
      case_ std::round_toward_infinity: { n = LE_PrevLoT; p = GT_HiT; } // (l-1, h]
      case_ std::round_toward_neg_infinity: { n = LT_LoT; p = GE_SuccHiT; } // [l, h+1)
      }
    case_ integral_to_integral:
      switch_ (Traits::sign_mixture) {
      case_ signed_to_unsigned: { n = LT_Zero; p = GT_HiT; } // [0, h]
      case_ unsigned_to_signed: { p = GT_HiT; } // [0, h]
      default_:                 { n = LT_LoT; p = GT_HiT; } // [l, h]
      }
    }
    return n ? cNegOverflow : p ? cPosOverflow : cInRange;
  }
  static void validate_range(argument_type s) {
    if_ (Traits::udt_builtin_mixture != builtin_to_builtin) return;           // dummy
    OverflowHandler()( out_of_range(s) );
  }
};

struct converter<T, S, Traits = conversion_traits<T,S>,
    OverflowHandler = def_overflow_handler, Float2IntRounder = Trunc<Traits::source_type>,
    RawConverter = raw_converter<Traits>, UserRangeChecker = UseInternalRangeChecker>
{
  using traits = Traits;  // and argument_type, result_type

  using RangeChecker = is_same<UserRangeChecker, UseInternalRangeChecker> ?
    GetRC<Traits,OverflowHandler,Float2IntRounder>::type : UserRangeChecker;

  // Range Checker API
  static range_check_result out_of_range(argument_type s) {
    if_ (traits::trivial) return cInRange;                    // dummy
    return RangeChecker::out_of_range(s);
  }
  static void validate_range(argument_type s) {
    if_ (traits::trivial) return;                             // dummy
    RangeChecker::validate_range(s);
  }

  // Converter API
  static result_type low_level_convert(argument_type s) {
    if_ (traits::trivial) return s;
    return RawConverter::low_level_convert(s);
  }
  static source_type nearbyint(argument_type s) {
    if_ (traits::trivial) return s;
    if_ (traits::int_float_mixture != float_to_integer) return s;
    return Float2IntRounder::nearbyint(s);
  }
  static result_type convert(argument_type s) {
    if_ (traits::trivial) return s;
    validate_range(s);
    if_ (traits::int_float_mixture == float_to_integer) s = nearbyint(s);
    return low_level_convert(s);
  }

  result_type operator() (argument_type s) const { return convert(s); }
};

struct make_converter_from<S,
    OverflowHandler = def_overflow_handler, Float2IntRounder = Trunc<S>,
    UserRangeChecker = UseInternalRangeChecker>
{
  struct to<T, Traits = conversion_traits<T,S>, RawConverter = raw_converter<Traits>> {
    using type = converter<T,S,Traits,OverflowHandler,Float2IntRounder,RawConverter,UserRangeChecker>;
  };
};
```

* Actually implemented by template dispatching, meta if-else-switch-case.
* User can provide different policies to support non-builtin logic.

------
### Numeric Cast

Header `<boost/numeric/conversion/cast.hpp>`

```c++
struct numeric_cast_traits<Target,Source> {
  using overflow_policy = def_overflow_handler;
  using range_checking_policy = UseInternalRangeChecker;
  using rounding_policy = Trunc<Source>;
};
Target numeric_cast<Target, Source>(Source arg) {
  using conv_traits = conversion_traits<Target, Source>;
  using cast_traits = numeric_cast_traits<Target, Source>;
  using converter = converter<Target, Source, conv_traits,
    cast_traits::overflow_policy, casts_traits::rounding_policy,
    raw_converter<conv_traits>, cast_traits::range_checking_policy>;
  return converter::convert(arg);
}
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`
* `<boost/cstdint.hpp>` - by `numeric_cast`
* `<boost/config/no_tr1/cmath.hpp>`

#### Boost.Core

* `<boost/type.hpp>` - by `numeric_cast`

#### Boost.TypeTraits

* `<boost/type_traits/is_arithmetic.hpp>`, `<boost/type_traits/is_same.hpp>`
* `<boost/type_traits/remove_cv.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/seq/elem.hpp>`, `<boost/preprocessor/seq/size.hpp>` - by `numeric_cast`
* `<boost/preprocessor/iteration/iterate.hpp>` - by `numeric_cast`

#### Boost.MPL

* `<boost/mpl/if.hpp>`, `<boost/mpl/eval_if.hpp>`, `<bool/mpl/identity.hpp>`
* `<boost/mpl/not.hpp>`, `<boost/mpl/and.hpp>`, `<bool/mpl/bool.hpp>`
* `<boost/mpl/integral_c.hpp>`, `<boost/mpl/int.hpp>`
* `<boost/mpl/multiplies.hpp>`, `<boost/mpl/less.hpp>`, `<boost/mpl/equal_to.hpp>`

------
### Standard Facilities
