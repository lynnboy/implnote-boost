# Boost.Any

* lib: `boost/libs/any`
* repo: `boostorg/any`
* commit: `9b1941c7`, 2025-06-18

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

#### Class `unique_any`

```c++
class boost::anys::unique_any;
```

'any' that internally use `unique_ptr` to hold any type, enable non-copyable & non-movable types.

#### Class Template `basic_any`

```c++
template<std::size_t OptSize = sizeof(void*), std::size_t OptAlign = alignof(void*)>
class boost::anys::basic_any;
```

'any' with small object optimization.

#### Class `bad_any_cast`

```c++
class bad_any_cast : std::bad_cast;
```

#### Function `swap`

`swap(any&, any&) noexcept`

And for `basic_any` and `unique_any`.

#### `any_cast`s

* `any_cast<ValueType>(any*) noexcept -> ValueType*`
* `any_cast<ValueType>(const any*) noexcept -> const ValueType*`
* `any_cast<ValueType>(any&) -> ValueType`
* `any_cast<ValueType>(const any&) -> ValueType`
* `any_cast<ValueType>(any&&) -> ValueType`

And for `basic_any` and `unique_any`.

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

------
### Standard Facilities

* Standard Library
  * class `std::any` (C++17).
  * class `std::experimental::any` Library Fundamentals TS (v1).
