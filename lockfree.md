# Boost.Lockfree

* lib: `boost/libs/lockfree`
* repo: `boostorg/lockfree`
* commit: `f0bda4bd`, 2015-09-28

------
### Lockfree Data Structures

#### Header

* `<boost/lockfree/policies.hpp>`
* `<boost/lockfree/queue.hpp>`
* `<boost/lockfree/stack.hpp>`
* `<boost/lockfree/spsc_queue.hpp>`

------
#### Customization Policies

```c++
template <bool> fixed_sized;
template <size_t> capacity;
template <class Alloc=std::allocator> allocator;
```

* Based on **Boost.Parameter** named template parameter style.
* For `queue` and `stack`
  * Provide `capacity` makes it compile-time sized, storage is embedded.
  * Provide `fixed_sized` without `capacity` means allocated fixed sized storage on construction
  * `allocator` is used by `fixed_sized` or runtime allocated storage.
* For `spsc_queue`, `fixed_sized` is unused and ignored, either can be compile-time sized or fixed sized.

------
#### Queue and Stack

```c++
template<typename T, ... Options>
class queue {
public:
  typedef T value_type;     // also 'allocator' and 'size_type'
  bool is_lock_free() const;          // whether underlying storage and atomic are all lock_free

  queue([allocator const&]);          // for 'capacity' is specified
  explicit queue(size_type n[, allocator const&]); // either 'fixed_sized' or dynamic sized
  ~queue();

  void reserve[_unsafe](size_type);   // only for dynamic sized
  bool empty() const;
  bool [unsynchronized_]push(T const &);  // allocation might occur
  bool bounded_push(T const &);           // force no allocation
  bool [unsynchronized_]pop(T &);
  template <typename U> bool [unsynchronized_]pop(U & u);   // 'U' should allow 'u = t' or 'u = U(t)'
  template <typename F> bool consume_one(F [const] & f);    // invoke 'f(t)'
  template <typename F> size_t consume_all(F [const] & f);  // loops 'consume_one'
};

template<typename T, ... Options>
class stack {
public:
  typedef T value_type;     // also 'allocator' and 'size_type'
  bool is_lock_free() const;          // whether underlying storage and atomic are all lock_free

  stack([allocator const&]);          // for 'capacity' is specified
  explicit stack(size_type n[, allocator const&]); // either 'fixed_sized' or dynamic sized
  ~stack();

  void reserve[_unsafe](size_type);   // only for dynamic sized
  bool empty() const;
  bool [unsynchronized_]push(T const &);  // allocation might occur
  bool bounded_push(T const &);           // force no allocation
  ConstIterator [unsynchronized_]push(ConstIterator, ConstIterator);  // allocation might occur
  ConstIterator bounded_push(ConstIterator, ConstIterator);           // force no allocation
  bool [unsynchronized_]pop(T &);
  template <typename U> bool [unsynchronized_]pop(U & u);   // 'U' should allow 'u = t' or 'u = U(t)'
  template <typename F> bool consume_one(F [const] & f);    // invoke 'f(t)'
  template <typename F> size_t consume_all(F [const] & f);  // loops 'consume_one'
};
```

* `unsynchronized` and `unsafe` version use the same algorithm with normal version, except
  they use ordinary assignment and relaxed atomic loads
* `stack` have just one "Top Of Stack" pointer, thus synchronization is simpler
  * `stack` provides range version push, because its underlying operation is the same with one-node push
* `queue` have one `head` and a `tail` pointer, and use a pre-allocated dummy node.
* Each member are stored at different cacheline (hopefully 64 bytes)

* Dynamic sized pool use `tagged_ptr` type storing a pointer value and a version tag
  * General version stores tag as `size_t`, hopefully `atomic<tagged_ptr>` can be lock-free.
  * On x86-64 like arch, it is compressed into a single 64bit word, tag use the (unused) high 16bit
* Static sized or fixed sized pools use `tagged_index` type, both index and tag are 16bit
* Pool type provides translating between tagged value and pointer/tag values

------
#### SPSC (Single Producer Single Comsumer) Queue

```c++
template<typename T, ... Options>
class spsc_queue {
public:
  typedef T value_type;     // also 'allocator' and 'size_type'

  spsc_queue([allocator const&]);     // for 'capacity' is specified
  explicit spsc_queue(size_type n[, allocator const&]); // for dynamic sized
  ~spsc_queue();
  
  void reset();

  size_type read_available() const;   // size
  size_type write_available() const;  // free space
  [const] T & front() [const];

  bool push(T const &);
  size_type push(T const *, size_type); size_type push<size>(T const (&)[size]); // array version
  ConstIterator push(ConstIterator, ConstIterator); // range version
  bool pop();
  template <typename U> bool pop(U & u);  // 'U' should allow 'u = t' or 'u = U(t)'
  size_type pop(T *, size_type); size_type pop<size>(T (&)[size]);  // array version
  size_type pop(OutputIterator);          // range (generator) version
  template <typename F> bool consume_one(F [const] & f);    // invoke 'f(t)'
  template <typename F> size_t consume_all(F [const] & f);  // loops 'consume_one'
};
```

* Implemented upon a _ring buffer_
  * remembering the read and write index, `atomic<size_t>`
  * use one extra space for distinguishing empty/full
* Reader and writer both are single-threaded
  * Index values don't require tagging
  * `push` and `pop` don't require loop and tag checking
  * `spsc_queue` is *lockfree* and *waitfree* - no waiting at all.
* Loading owned value is `relaxed`, loading the other is `acquire`; storing is `release`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/cstdint.hpp>`

#### Boost.Array

* `<boost/array.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Core

* `<boost/utility/enable_if.hpp>`

#### Boost.MPL

* `<boost/mpl/if.hpp>`, `<boost/mpl/bool.hpp>`, `<boost/mpl/size_t.hpp>`, `<boost/mpl/void.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_convertible.hpp>`
* `<boost/type_traits/has_trivial_assign.hpp>`, `<boost/type_traits/has_trivial_destructor.hpp>`
* `<boost/aligned_storage.hpp>` - used by `spsc_queue`

#### Boost.Parameter

* `<boost/parameter.hpp>`

#### Boost.Atomic

* `<boost/atomic.hpp>` - fallback version when `<atomic>` is not available

#### Boost.Tuple

* `<boost/tuple/tuple.hpp>` - internally used by `stack`

#### Boost.Integer

* `<boost/integer_traits.hpp>` - internally used by `stack`

#### Boost.Utility

* `<boost/utility.hpp>` - `next` used by `spsc_queue`

------
### Standard Facilities


