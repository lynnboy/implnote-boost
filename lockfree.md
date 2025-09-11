# Boost.Lockfree

* lib: `boost/libs/lockfree`
* repo: `boostorg/lockfree`
* commit: `5493605`, 2025-08-25

------
### Lockfree Data Structures

#### Header

* `<boost/lockfree/lockfree_forward.hpp>`
* `<boost/lockfree/policies.hpp>`
* `<boost/lockfree/queue.hpp>`
* `<boost/lockfree/stack.hpp>`
* `<boost/lockfree/spsc_queue.hpp>`
* `<boost/lockfree/spsc_value.hpp>`

------
#### Customization Policies and Details

```c++
// `template_keyword` from Boost.Parameter
template <bool> fixed_sized;
template <size_t> capacity;
template <class Alloc=std::allocator> allocator;
template <bool> allow_multiple_reads;

using std::atomic; using std::memory_order_[aquire|consume|release|relaxed];

struct uses_optional_t {}; inline cosntexpr uses_optional_t uses_optional;

// extract for keyword parameter
using detail::extract_fixed_size<bound_args, def=false> = ... || def;
struct detail::extract_capacity<bound_args> { static constexpr size_t capacity; bool has_capacity; };
struct detail::extract_allocator<bound_args, T> {
  static constexpr bool has_allocator;
  using type = allocator_rebind<..., T>;
};
using detail::extract_allocator_t<bound_args, T> = detail::extract_allocator<bound_args, T>::type;
using detail::extract_allow_multiple_reads<bound_args, def=false> = ... || def;

void detail::copy_payload<T,U>(T& t, U& u) {
  if constexpr (is_convertible_v<T, U>) u = t; else u = U(t);
}
inline constexpr size_t detail::cacheline_bytes = 256; // 128, 64, depends on CPU arch
#define BOOST_LOCKFREE_PTR_COMPRESSION 1 // for x86_64, or ARM >= 8.0 except Android

template<class T>
class alignas(2*sizeof(void*)) tagged_ptr { // generic implementation
  T* ptr; tag_t tag; // keep ptr and tag in the same cache line
public:
  using tag_t = size_t;
  tagged_ptr(T*p, tag_t t=0) : ptr(p), tag(t) {}
  // def-ctor, copy-ctor, copy-ass, operator==, operator!=
  void set(T*, tag_t); T* get_ptr() const; void set_ptr(T*);
  tag_t get_tag() const; void set_tag(tag_t); tag_t get_next_tag() const; // tag+1, wrap unsigned
  T& operator*() const; T* operator->() const; operator bool() const; // pointer API
};
template<class T>
class tagged_ptr { // compressed pointer implementation
  uint64_t ptr; // high 16-bits is the tag, low 48-bits is the pointer
public:
  using tag_t = uint16_t;
  // same API as above
};
```

* Based on **Boost.Parameter** named template parameter style.
* For `queue` and `stack`
  * Provide `capacity` makes it compile-time sized, storage is embedded.
  * Provide `fixed_sized` without `capacity` means allocated fixed sized storage on construction
  * `allocator` is used by `fixed_sized` or runtime allocated storage.
* For `spsc_queue`, `fixed_sized` is unused and ignored, either can be compile-time sized or fixed sized.
* `allow_multiple_reads` is used for `spsc_value`.

* Define `BOOST_LOCKFREE_FORCE_BOOST_ATOMIC` to use Boost.Atomic instead of stdlib's.

------
#### Queue and Stack

```c++
template<typename T, typename... Options>
  requires is_copy_assignable_v<T> && is_trivially_copy_assignable_v<T> && is_trivially_destructible_v<T>
class queue {
public:
  typedef T value_type;     // also 'allocator' and 'size_type'
  bool is_lock_free() const;          // whether underlying storage and atomic are all lock_free

  queue([allocator const&]);          // for 'capacity' is specified
  explicit queue(size_type n[, allocator const&]); // either 'fixed_sized' or dynamic sized
  ~queue(); // comsume_all

  void reserve[_unsafe](size_type);   // only for dynamic sized
  bool empty() const;
  bool [unsynchronized_]push(T const &);  // allocation might occur
  bool bounded_push(T const &);           // force no allocation
  bool [unsynchronized_]pop(T &);
  template <typename U> bool [unsynchronized_]pop(U & u);   // 'U' should allow 'u = t' or 'u = U(t)'
  template <typename F> bool consume_one(F [const] & f);    // invoke 'f(t)'
  template <typename F> size_t consume_all(F [const] & f);  // loops 'consume_one'
};

template<typename T, typename... Options>
  requires is_copy_assignable_v<T> || is_move_assignable_v<T>
class stack {
public:
  typedef T value_type;     // also 'allocator' and 'size_type'
  bool is_lock_free() const;          // whether underlying storage and atomic are all lock_free

  stack([allocator const&]);          // for 'capacity' is specified
  explicit stack(size_type n[, allocator const&]); // either 'fixed_sized' or dynamic sized
  ~stack(); // comsume_all

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
  template <typename F> size_t consume_all_atomic(F [const] & f);  // off stack then consume
  template <typename F> size_t consume_all_atomic_reversed(F [const] & f);
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
#### SPSC (Single Producer Single Comsumer) Queue and Value

```c++
template<typename T, typename... Options>
  requires is_default_constructible_v<T> && is_move_assignable_v<T> || is_copy_assignable_v<T>
class spsc_queue {
public:
  typedef T value_type;     // also 'allocator' and 'size_type'

  spsc_queue([allocator const&]);     // for 'capacity' is specified
  explicit spsc_queue(size_type n[, allocator const&]); // for dynamic sized
  ~spsc_queue(); // comsume_all
  
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

template<typename T, typename... Options>
struct spsc_value;
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

#### Boost.Align

* `<boost/align/align_up.hpp>`
* `<boost/align/aligned_allocator.hpp>`
* `<boost/align/aligned_allocator_adaptor.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Atomic

* `<boost/atomic.hpp>` - fallback version when `<atomic>` is not available

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Core

x* `<boost/utility/enable_if.hpp>`
* `<boost/core/allocator_access.hpp>`
* `<boost/core/span.hpp>`
* `<boost/core/no_exceptions_support.hpp>`

#### Boost.Parameter

* `<boost/parameter/binding.hpp>`
* `<boost/parameter/parameters.hpp>`
* `<boost/parameter/optional.hpp>`
* `<boost/parameter/template_keyword.hpp>`

#### Boost.Predef

x* `<boost/predef.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

x* `<boost/type_traits/is_convertible.hpp>`
x* `<boost/type_traits/is_copy_constructible.hpp>`
* `<boost/aligned_storage.hpp>` - used by `spsc_queue`

------
### Standard Facilities


