# Boost.Any

* lib: `boost/libs/any`
* repo: `boostorg/any`
* commit: `4ca14647`, 2017-02-23

------
### Any

#### Header

* `<boost/any.hpp>`

#### Class `any`

```c++
class any;
```

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

* `<boost/config.hpp>`.

#### Boost.TypeIndex

* `<boost/type_index.hpp>`, for RTTI.

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`, several type traits.

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Core

* `<boost/utility/enable_if.hpp>`
* `<boost/core/addressof.hpp>`

#### Boost.MPL

* `<boost/mpl/if.hpp>`, would be `<boost/type_traits/conditional.hpp>` better.

------
### Standard Facilities

* Proposals:
  * N3804 - Any library proposal.
  * N4335 - C++ Extensions for Library Fundamentals v1.
