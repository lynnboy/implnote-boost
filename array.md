# Boost.Array

* lib: `boost/libs/array`
* repo: `boostorg/array`
* commit: `e988ef9`, 2025-04-03

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
// size(), empty(), max_size(), static_size (constant)
// swap(), operator=(), assign() (deprecated), fill()
// data(), c_array() (deprecated), T[N] elems (data member)
};

template<class T>
class array<T, 0>; // fail on all element access operations
```

##### Non-member functions

* `==`, `!=`, `<`, `>`, `<=`, `>=`, `<=>`
* `swap(array<T,N>&, array<T,N>&)`.
* `hash_value(const array<T,N>&) -> std::size_t`.
* `get<size_t Idx>(array<T,N>&) noexcept -> T&`.
* `get<size_t Idx>(array<T,N> const&) noexcept -> const T&`.
* `to_array<T, N>(T const(&)[N]) -> array<T,N>`.
* `to_array<T, N>(T (&&)[N]) -> array<T,N>`.
* `get_c_array(array<T,N>&) -> T(&)[N]` (deprecated).
* `get_c_array(array<T,N> const&) -> const T(&)[N]` (deprecated).

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/workaround.hpp>`.

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
