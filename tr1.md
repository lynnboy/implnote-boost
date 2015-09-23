# Boost.TR1

* lib: `boost/libs/tr1`
* repo: `boostorg/tr1`
* commit: `9f28b876`, 2014-08-18

------
### Boost Implementation of TR1

#### Header

* `<boost/tr1/*.hpp>`

Deprecated.

Automatic forward to platform provided TR1 implementation if available.



##### Constructors & Destructor

`any() noexcept`
`any(const ValueType& value)`
`any(const any& other)`
`any(any&& other) noexcept`
`any(ValueType&& value)`
`~any() noexcept`

##### Methods

* `swap(any& rhs) noexcept -> any&`
* `operator=(const any& rhs) -> any&`
* `operator=(any&& rhs) noexcept -> any&`
* `operator=(ValueType&& rhs) -> any&`
* `empty() const noexcept -> bool`
* `clear() noexcept`
* `type() const noexcept -> const boost::typeindex::type_info &`

#### Class `bad_any_cast`

```c++
class bad_any_cast : std::bad_cast;
```

#### Function `swap`

`swap(any&, any&) noexcept`

#### `any_cast`s

* `any_cast<ValueType>(any*) noexcept -> ValueType*`
* `any_cast<ValueType>(const any*) noexcept -> const ValueType*`
* `any_cast<ValueType>(any&) -> ValueType`
* `any_cast<ValueType>(const any&) -> ValueType`
* `any_cast<ValueType>(any&&) -> ValueType`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`
* `<boost/config/no_tr1/cmath.hpp>`
* `<boost/config/no_tr1/utility.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Core

* `<boost/utility/enable_if.hpp>`
* `<boost/ref.hpp>` - `functional`

#### Boost.Array

* `<boost/array.hpp>` - `array`

#### Boost.TypeTraits

* `<boost/type_traits/is_convertible.hpp>`, `<boost/type_traits/is_same.hpp>`
* `<boost/type_traits/is_floating_point.hpp>`, `<boost/type_traits/is_fundamental.hpp>`
* `<boost/type_traits/integral_constant.hpp>`
* `<boost/type_traits/add_const.hpp>`, `<boost/type_traits/add_reference.hpp>` - `utility`
* `<boost/type_traits.hpp>`, `<boost/type_traits/is_base_of_tr1.hpp>` - `type_traits`

#### Boost.MPL

* `<boost/mpl/if.hpp>`

#### Boost.Math

* `<boost/math/tr1.hpp>` - `cmath`
* `<boost/math/complex.hpp>` - `complex`

#### Boost.Iterator

* `<boost/iterator/iterator_facade.hpp>` - `random`

#### Boost.Bind

* `<boost/mem_fn.hpp>`, `<boost/bind.hpp>` - `functional`

#### Boost.Function

* `<boost/function.hpp>` - `functional`

#### Boost.Functional/Hash

* `<boost/functional/hash.hpp>` - `functional`

#### Boost.Utility

* `<boost/utility/result_of.hpp>` - `functional`

#### Boost.SmartPtr

* `<boost/shared_ptr.hpp>`, `<boost/weak_ptr.hpp>`, `<boost/enable_shared_from_this.hpp>` - `memory`

#### Boost.Random

* `<boost/random.hpp>`, `<boost/nondet_random.hpp>` - `random`

#### Boost.Regex

* `<boost/regex.hpp>` - `regex`

#### Boost.Tuple

* `<boost/tuple/tuple.hpp>`, `<boost/tuple/tuple_comparison.hpp>` - `tuple`, old style

#### Boost.Fusion

* `<boost/fusion/include/tuple.hpp>`, `<boost/fusion/include/std_pair.hpp>` - `tuple`, new style

#### Boost.Unordered

* `<boost/unordered_map.hpp>`, `<boost/unordered_set.hpp>` - `unordered_map`, `unordered_set`

------
### Standard Facilities

* C++ TR1
