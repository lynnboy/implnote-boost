# Boost.Align

* lib: `boost/libs/align`
* repo: `boostorg/align`
* commit: `bd7f7f5`, 2025-06-27

------
### Alignment Library

#### Function `align`

Header `<boost/align/align.hpp>`

```c++
void* align(std::size_t alignment, std::size_t size, void*& ptr, std::size_t& space);
```

Use `std::align` if available (C++11).

------
#### Function `align_up`

Header `<boost/align/align_up.hpp>`

```c++
void* align_up(void* ptr, std::size_t alignment);
template<class /*notpointer*/ T>
  constexpr T align_up(T value, std::size_t alignment) noexcept;
```

------
#### Function `align_down`

Header `<boost/align/align_down.hpp>`

```c++
void* align_down(void* ptr, std::size_t alignment);
template<class /*notpointer*/ T>
  constexpr T align_down(T value, std::size_t alignment) noexcept;
```

------
#### Trait `alignment_of`

Header `<boost/align/alignment_of.hpp>`

```c++
template <class X> struct alignment_of;
template <class X> constexpr std::size_t alignment_of_v;
```

C++11 Standard library provides one.

------
#### Function `is_aligned`

Header `<boost/align/is_aligned.hpp>`

```c++
bool is_aligned(const volatile void* ptr, std::size_t alignment) noexcept;
template<class /*notpointer*/ T>
  constexpr bool is_aligned(T value, std::size_t alignment) noexcept;
bool is_aligned(std::size_t alignment, const volatile void* ptr) noexcept;
```

Effect: `(ptr & (alignment-1) == 0)`.

------
#### Macro `BOOST_ALIGN_ASSUME_ALIGNED(ptr, alignment)`

Header `<boost/align/assume_aligned.hpp>`.

Compiler optimization hint.

------
#### Function `aligned_alloc` and `aligned_free`

Header `<boost/align/aligned_alloc.hpp>`

```c++
void* aligned_alloc(std::size_t alignment, std::size_t size) noexcept;
void aligned_free(void* ptr) noexcept;
```

Use `memalign`, `posix_memalign`, or `_aligned_malloc` provided by platform.
Fallback implementation allocates a block with extra space before the returned
block, and store the actual block's address.

------
#### Functor `aligned_delete`

Header `<boost/align/aligned_delete.hpp>`

```c++
struct aligned_delete {
template <class T> void operator()(T* ptr) const noexcept(noexcept(ptr->~T()));
};
```

Similar to (C++11) `default_delete`, call `ptr->~T()` then `aligned_free`.

------
#### `aligned_allocator`

Header `<boost/align/aligned_allocator.hpp>`

```c++
template<class T, std::size_t Alignment>
struct aligned_allocator;
```

Provide over alignment allocation on type `T` objects.

------
#### `aligned_allocator_adaptor`

Header `<boost/align/aligned_allocator_adaptor.hpp>`

```c++
template<class Allocator, std::size_t Alignment>
struct aligned_allocator_adaptor : public Allocator { /*...*/ };
```

Override `allocate` and friends members of base allocator.

------
#### Customization

Instead of default `malloc` and `free` functions for `aligned_alloc` and `aligned_free`:
* User can define `BOOST_ALIGN_USE_ALIGN` and define
`boost::alignment::allocate` and `boost::alignment::deallocate` for customized allocation
* Or define `BOOST_ALIGN_USE_NEW` to use global `new`/`delete`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Core

* `<boost/core/pointer_traits.hpp>`, for `to_address()`

------
### Standard Facilities

* Language
  * `alignof` operator, `alignas` attribute. (C++11)
  * `operator new`/`operator delete` with alignment argument (C++17).
* Standard Library
  * `alignment_of` trait (C++11), `align` function (C++11).
  * `aligned_alloc` (C11/C++17)
  * `is_sufficiently_aligned` (C++26), `assume_aligned` (C++20).
