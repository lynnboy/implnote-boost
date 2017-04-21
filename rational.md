# Boost.Rational

* lib: `boost/libs/rational`
* repo: `boostorg/rational`
* commit: `d1cd08fc`, 2017-03-27

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
template <typename IntType>
class rational;
```

##### Constructors & Destructor

* `constexpr rational()` - 0/1
* `constexpr rational(n)` - n/1
* `rational(num, den)` - num/den, normalized
* `constexpr explicit rational(rational<NewType> const &)`

##### Assign

* `operator=(n) -> rational&` - n/1
* `assign(num, den) -> rational&` - num/den

##### Operators

* `+=`, `-=`, `*=`, `/=`, `++`, `--`, `!`, `bool`, `<`, `==`, `>`

##### Accessors

* `constexpr numerator() const -> const IntType&`
* `constexpr denominator() const -> const IntType&`

##### Non-member Operators

* `<=`, `==`, `+`, `-`, `*`, `/`, `++`, `--`
  Both for `rational` and `IntType`.

##### IO Supports

* `<<` and `>>`

##### Non-member Functions

* `constexpr rational_cast<T>(const rational<IntType>&) -> T`
* `abs(const rational<IntType>&) -> rational<IntType>`

#### Rationale

Keep normalized if possible.
Throw `bad_rational` whenever encounter error.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`.
* `<boost/detail/workaround.hpp>`.

#### Boost.Core

* `<boost/utility/enable_if.hpp>`

#### Boost.Utility

* `<boost/operators.hpp>` - Support non-function operators.
* `<boost/call_traits.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Integer

* `<boost/integer/common_factor_rt.hpp>` - Run-time GCD/LCM functions

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_convertible.hpp>`
* `<boost/type_traits/is_class.hpp>`
* `<boost/type_traits/is_same.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

------
### Standard Facilities

* Proposals
  * n3611 - A Rational Number Library for C++ (not accepted).
