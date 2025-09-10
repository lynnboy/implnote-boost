# Boost.Integer

* lib: `boost/libs/integer`
* repo: `boostorg/integer`
* commit: `4d702c7`, 2025-08-26

------
### Synopsis

Header `<boost/integer_fwd.hpp>`

```c++
struct integer_traits<T>; // integer_traits

// integer.hpp
struct int_fast_t<LeastInt>;
struct int_t<Bits>; struct uint_t<Bits>;
struct int_max_value_t<MaxV>; struct int_min_value_t<MinV>; struct uint_value_t<MaxV>;

struct high_bit_mask_t<Bit>; struct low_bits_mask_t<Bits>; // integer_mask.hpp

using static_log2_argument_type = boost::uintmax_t;
using static_log2_result_type = int;
struct static_log2<V>;  // static_log2.hpp

using static_min_max_[un]signed_type = boost[u]intmax_t;
struct static_[un]signed_(min|max)<V1, V2>; // static_min_max.hpp

namespace integer {
  using static_gcd_type = boost::uintmax_t;
  struct static_gcd<V1, V2>; struct static_lcm<V1, V2>; // common_factor_ct.hpp

  struct gcd_evaluator<IntegerType>; struct lcm_evaluator<IntegerType>; // common_factor_rt.hpp
}
```

### Integer Traits

Header `<boost/integer_traits.hpp>`

* Inherites from `std::numeric_limits`
* Provide static value `is_integral`, `const_min` and `const_max`

------
#### Integer Type Selection

Header `<boost/integer.hpp>`

Trait classes
* `int_fast_t<LeastType>`:
  * `fast`, and `type` to be fast type to store `LeastType`
* `int_t<Bits>`, `uint_t<Bits>`:
  * `least` is type at least with required size.
  * `fast` is fastest type supports required size.
  * `exact` is exact sized type.
* `int_max_value_t<MaxValue>`, `int_min_value_t<MinValue>`, `uint_value_t<MaxValue>`:
  * `least` is type with size at least support required value.
  * `fast` is fastest type supports required value.

------
#### Integer Bit Masks

Header `<boost/integer/integer_mask.hpp>`

* `high_bit_mask_t<Bit>` - Single bit mask at position `Bit`
  * type `least` is _least_ type for mask value.
  * type `fast` is _fast_ type for mask value.
  * `high_bit`, mask value with `least` type.
  * `high_bit_fast`, mask value with `fast` type.
  * `bit_position`, value of `Bit`.
* `low_bits_mask_t<Bits>` - Bit mask of all low `Bits` positions
  * `least` is _least_ type for mask value.
  * `fast` is _fast_ type for mask value.
  * `sig_bits`, mask value with `least` type.
  * `sig_bits_fast`, mask value with `fast` type.
  * `bit_count`, value of `Bits`.

------
### Integer Helpers

#### Compile-Time Integer log2

Header `<boost/integer/static_log2.hpp>`

Trait class `static_log2<Value>`.

#### Run-Time Integer log2

Header `<boost/integer/integer_log2.hpp>`

```c++
int integer_log2<T>(T);
```

#### Compile-Time Integer Min/Max

Header `<boost/integer/static_min_max.hpp>`

Trait class `static_signed_min<V1,V2>`, `static_signed_max<V1,V2>`, `static_unsigned_min<V1,V2>`,
and `static_unsigned_max<V1,V2>`.

#### Integer GCD/LCM

Header `<boost/integer/common_factor.hpp>`:
* Header `<boost/integer/common_factor_ct.hpp>` (compile time)
  - Trait class `static_gcd<V1,V2>`, `static_lcm<V1,V2>`.
* Header `<boost/integer/common_factor_rt.hpp>` (run time)
  - Functions `gcd<IntType>(IntType const&, IntType const&) -> IntType` and `lcm`.
  - Functions `gcd_range(I, I) -> pair<I::value_type, I>` and `lcm_range`, loop on `gcd`/`lcm`
  - Functors `gcd_evaluator<IntType>` and `lcm_evaluator<IntType>` (call `gcd`/`lcm`).

Header `<boost/integer/extended_euclidean.hpp>`:

```c++
struct euclidean_result_t<Z> { Z gcd, x, y; };
extended_euclidean<Z>(Z, Z) -> euclidean_result_t<Z>;
```

Header `<boost/integer/mod_inverse.hpp>`:

```c++
mode_inverse<Z>(Z a, Z modulus) -> Z; // modular multiplicative inverse, 0 for not-found
```

------
### Dependency

#### Boost.Assert

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`, `<boost/cstdint.hpp>`

#### Boost.Core

* `<boost/core/bit.hpp>`
* `<boost/core/invoke_swap.hpp>`
* `<boost/core/enable_if.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_integral.hpp>`
* `<boost/type_traits/make_unsigned.hpp>`

------
### Standard Facilities

Language: `constexpr` (C++11)
Standard Library: `<cstdint>` (C++11)
