# Boost.Integer

* lib: `boost/libs/integer`
* repo: `boostorg/integer`
* commit: `13b153c6`, 2016-10-07

------
### Integer Traits

Header `<boost/integer_traits.hpp>`

* Inherites from `std::numeric_limits`
* Provide static value `const_min` and `const_max`

------
#### Integer Type Selection

Header `<boost/integer.hpp>`

Trait classes
* `int_fast_t<LeastType>`
  * `fast`, and `type` to be fast type to store `LeastType`
* `int_t<Bits>`
* `uint_t<Bits>`
  * `least` is type at least with required size.
  * `fast` is fastest type supports required size.
  * `exact` is exact sized type.
* `int_max_value_t<MaxValue>`
* `int_min_value_t<MinValue>`
* `uint_value_t<MaxValue>`
  * `least` is type with size at least support required value.
  * `fast` is fastest type supports required value.

------
#### Integer Bit Masks

Header `<boost/integer/integer_mask.hpp>`

* `high_bit_mask_t<Bit>` - Single bit mask at position `Bit`
  * `least` is _least_ type for mask value.
  * `fast` is _fast_ type for mask value.
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

Function `integer_log2<T>(T value) -> int`.

#### Compile-Time Integer Min/Max

Header `<boost/integer/static_min_max.hpp>`

Trait class `static_signed_min<V1,V2>`, `static_signed_max<V1,V2>`, `static_unsigned_min<V1,V2>`,
and `static_unsigned_max<V1,V2>`.

#### Integer GCD/LCM

Header `<boost/integer/common_factor.hpp>`

Trait class `static_gcd<V1,V2>`, `static_lcm<V1,V2>` (Compile time).
Functors `gcd_evaluator<IntType>` and `lcm_evaluator<IntType>` (Runtime).
Functions `gcd<IntType>(IntType const&, IntType const&) -> IntType` and `lcm`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`, `<boost/cstdint.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

------
### Standard Facilities

Language: `constexpr` (C++11)
Standard Library: `<cstdint>` (C++11)
