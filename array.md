# Boost.Array

* lib: `boost/libs/array`
* repo: `boostorg/array`
* commit: `23f6b27`, 2024-04-12

------
### Class `array`

#### Header

* `<boost/array.hpp>`

#### Class `array`

```c++
template<class T, std::size_t N>
class array
{
public:
  T elems[N];
// Container interface: typedefs, begin/end access reverse iterators
// operator[] and at(), front(), back()
// size(), empty(), max_size(), static_size
// swap(), operator=(), assign(), fill()
// data(), c_array()
};

template<class T>
class array<T, 0>; // fail on all element access operations
```

##### Non-member functions

* Equality & Relational comparasion operators.
* `swap(array<T,N>&, array<T,N>&)`.
* `get_c_array(array<T,N>&) -> T(&)[N]`.
* `get_c_array(array<T,N> const&) -> const T(&)[N]`.
* `hash_value(const array<T,N>&) -> std::size_t`.
* `get<size_t Idx>(array<T,N>&) noexcept -> T&`.
* `get<size_t Idx>(array<T,N> const&) noexcept -> const T&`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`.

#### Boost.Assert

* `<boost/assert.hpp>`.

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`.

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`.

#### Boost.Core

* `<boost/core/invoke_swap.hpp>`.

------
### Standard Facilities

* Standard Library: `<array>`.
