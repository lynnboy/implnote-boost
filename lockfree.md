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
class alignas(2*sizeof(void*)) detail::tagged_ptr { // generic implementation
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
class detail::tagged_ptr { // compressed pointer implementation
  uint64_t ptr; // high 16-bits is the tag, low 48-bits is the pointer
public:
  using tag_t = uint16_t;
  // same API as above
};

template<typename T, typename Alloc=std::allocator<T>>
class alignas(cacheline_bytes) detail::freelist_stack : Alloc { // priv impl for convenient
  using tagged_node_ptr = tagged_ptr<struct freelist_node>; // T**
  struct freelist_node { tagged_node_ptr next; };
  atomic<tagged_node_ptr> pool_; // head pointer
public:
  using index_t = T*; using tagged_node_handle = tagged_ptr<T>; // T*
  freelist_stack<Alloc2>(Alloc2 const& alloc, size_t n=0); // put initial n nodes to list
  void reserve<bool ThreadSafe>(size_t count);
  T* construct<bool ThreadSafe, bool Bounded, ...Args>(Args &&...args);
  void destruct<bool ThreadSafe>(T* node); void destruct<bool ThreadSafe>(tagged_node_handle const& node);
  ~freelist_stack(); // dealloc all nodes to underlying allocator
  bool is_lock_free() const;
  // `get_handle`/`get_pointer`, convert between T* and handle; `null_handle()` return nullptr
protected: // above API calls these funcs
  T* allocate<bool ThreadSafe, bool Bounded>(); // get node from linked list
  void deallocate<bool ThreadSafe>(T* node); // put back node to linked list
};

class detail::tagged_index { // similar to tagged_ptr
protected: index_t index; tag_t tag;
public:
  using index_t = tag_t = uint16_t;
  // ctor, copy-ctor, ctor(index_t, tag_t=0), operator==, operator!=
  // get_index, set_index, get_tag, set_tag, get_next_tag
};

template<class T, size_t size>
struct detail::compiletime_sized_freelist_storage; // wrap `std::array<char, size*sizeof(T) + 64>` as buffer
template<class T, class Alloc=std::allocator<T>>
struct detail::runtime_sized_freelist_storage; // allocate buffer by ctor(alloc, count), deallocate buffer at dtor
template<class T, class NodeStorage=runtime_sized_freelist_storage>
class detail::fixed_size_freelist : NodeStorage { // priv impl for convenient
  struct freelist_node { tagged_index next; }; // just store index is enough
  atomic<tagged_index> pool_; // head index
public:
  using index_t = tagged_index::index_t; using tagged_node_handle = tagged_index;
  fixed_size_freelist<Alloc2>(Alloc2 const& alloc, size_t n=0); // initialize buffer of n nodes
  void reserve<bool ThreadSafe>(size_t count);
  T* construct<bool ThreadSafe, bool Bounded, ...Args>(Args &&...args);
  void destruct<bool ThreadSafe>(T* node); void destruct<bool ThreadSafe>(tagged_node_handle const& node);
  ~fixed_size_freelist(); // dealloc all nodes to underlying allocator
  bool is_lock_free() const;
  // `get_handle`/`get_pointer`, convert between T*, index_t and handle; `null_handle()` return end index
protected: // above API calls these funcs
  index_t allocate<bool ThreadSafe>(); // get node from linked list
  void deallocate<bool ThreadSafe>(index_t index); // put back node to linked list
};

using detail::select_freelist_t<T,Alloc,isCompSized,isFixedSize,size> =
  isCompSized ? fixed_size_freelist<compiletime_sized_freelist_storage<T,size>> :
  isFixeSized ? fixed_size_freelist<runtime_sized_freelist_storage<T,Alloc>> :
                freelist_stack<T,Alloc>;
struct select_tagged_handle<T,isNodeBased> {
  using tagged_handle_type = isNodeBased ? tagged_ptr<T> : tagged_index;
  using handle_type = isNodeBased ? T* : tagged_index::index_t; // T* or uint16_t
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
* Free lists are lock-free singlely linked list.
* `freelist_stack` implementation based on tagged pointer into dynamic allocated nodes.
  - `ThreadSafe` version run `compare_exchange_weak` in loop until success.
  - `Bounded` disable alloc of new node from underlying allocator.
  - `allocate` increments head pointer's tag value.
* `fixed_size_freelist` implementation based on tagged index into a fixed (array or malloced) buffer.
  - `ThreadSafe` version run `compare_exchange_weak` in loop until success.
  - `allocate` increments head index's tag value.

------
#### Queue and Stack (Tagged value based)

```c++
template<typename T, typename... Options>
  requires is_copy_assignable_v<T> && is_trivially_copy_assignable_v<T> && is_trivially_destructible_v<T>
class queue {
  static constexpr bool has_capacity, fixed_sized; static constexpr size_t capacity // extract from Options
  static constexpr bool node_based=!(has_capacity||fixed_sized), compile_time_sized=has_capacity;
  struct alignas(cacheline_bytes) node {atomic<tagged_node_handle> next; T data; };
  using pool_t = select_freelist_t<>; using tagged_node_handle = ...; using handle_type = ...;

  atomic<tagged_node_handle> head_, tail_; // Doubly linked list, padded to align at cache line
  pool_t pool; // free node list
public:
  using allocator = Options/* extract from Options */;
  using value_type = T; using size_type = size_t;
  bool is_lock_free() const;          // whether underlying storage and atomic are all lock_free

  queue([allocator const&]);  queue<U>(allocator<U>const&); // has_capacity (comptime)
  explicit queue(size_type n[, allocator<U> const&]); // !has_capacity (runtime/fixed)
  ~queue(); // comsume_all
  // not copyable, not moveable

  void reserve(size_type); void reserve_unsafe(size_type);  // only for dynamic sized
  bool empty() const;
  bool push(T&&); bool bounded_push(T&&); // w/wo allocation, also `T const&`
  bool unsynchronized_push(T&&); // not atomic, no compare_exchange_weak loop
  bool pop(T&); bool pop<U>(U&); std::optional<U> pop<U>(uses_optional_t);
  bool unsynchronized_pop<U>(T&); // not atomic, no compare_exchange_weak loop
  template <typename F> bool consume_one(F [const] & f);    // invoke 'f(t)'
  template <typename F> size_t consume_all(F [const] & f);  // loops 'consume_one'
};
```

```c++
template<typename T, typename... Options>
  requires is_copy_assignable_v<T> || is_move_assignable_v<T>
class stack {
  static constexpr bool has_capacity, fixed_sized; static constexpr size_t capacity // extract from Options
  static constexpr bool node_based=!(has_capacity||fixed_sized), compile_time_sized=has_capacity;
  struct node {atomic<tagged_node_handle> next; T data; };
  using pool_t = select_freelist_t<>; using tagged_node_handle = ...; using handle_type = ...;

  atomic<tagged_node_handle> tos; // Singlely linked list
  pool_t pool; // free node list
public:
  using allocator = Options/* extract from Options */;
  using value_type = T; using size_type = size_t;
  bool is_lock_free() const;          // whether underlying storage and atomic are all lock_free

  stack([allocator const&]);  stack<U>(allocator<U>const&); // has_capacity (comptime)
  explicit stack(size_type n[, allocator<U> const&]); // !has_capacity (runtime/fixed)
  ~stack(); // comsume_all
  // not copyable, not moveable

  void reserve(size_type); void reserve_unsafe(size_type);  // only for dynamic sized
  bool empty() const;
  bool push(T&&); bool bounded_push(T&&); // w/wo allocation, also `T const&`
  ConstIter push<ConstIter>(ConstIter b, ConstIter e); // for range, also `bounded_`
  size_type push<Extend>(span<const T, Extent>); // for span, also `bounded_`
  bool unsynchronized_push(T&&); // not atomic, no compare_exchange_weak loop, also `T const&`, Iter pair, `span`
  bool pop(T&); bool pop<U>(U&); std::optional<U> pop<U>(uses_optional_t);
  bool unsynchronized_pop<U>(T&); // not atomic, no compare_exchange_weak loop

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
template<typename T>
class detail::ringbuffer_base {
  atomic<size_t> write_index_, read_index_; // padded to align cacheline
protected: // API that operate on `T* buffer, size_t max`
  T& front(T*); const T& front(const T*) const;
  size_t next_index(size_t i, size_t max); // ringbuffer calculate API
  static size_t read_available(size_t w, size_t r, size_t max); size_t read_available(size_t max) const;
  static size_t write_available(size_t w, size_t r, size_t max); size_t write_available(size_t max) const;
  bool push(T&&, T*buffer, size_t max);
  bool push(T const* p, size_t c, T*buffer, size_t max);
  bool push<ConstIter>(ConstIter b, ConstIter e, T*buffer, size_t max);
  size_t pop(T* p, size_t c, T*buffer, size_t max);
  size_t pop_to_output_iterator<Iter>(Iter, T*buffer, size_t max);
  bool consume_one<Fun>(Fun&&, T*buffer, size_t max);
  size_t consume_all<Fun>(Fun&&, T*buffer, size_t max);
public: // API provided for `spsc_queue`
  void reset(); // consume_all, reset indexes to 0
  bool empty(); bool is_lock_free() const;
};

template<typename T, size_t Max>
class detail::compile_time_sized_ringbuffer : public ringbuffer_base<T> {
  constexpr size_t max_size = Max + 1
  boost::aligned_storage<max_size*sizeof(T), alignment_of_v<T>> storage_;
public: // API, implemented on base API
  size_t max_number_of_elements() const;
  T& front(); const T& front() const;
  bool push(T&&); bool push(const T&);
  size_t push(T const*, size_t); size_t push<size>(T const(&)[size]); ConstIter push(ConstIter, ConstIter);
  size_t pop(T*, size); size_t pop_to_output_iterator(Iter it);
  bool consume_one<Fun>(Fun&&); size_t consume_all<Fun>(Fun&&);
};

template<typename T, typename Alloc> // fixed sized
class detail::runtime_sized_ringbuffer : public ringbuffer_base<T>, private Alloc {
  using Tr = std::allocator_traits<ALloc>;
  Tr::pointer array_; // buffer pointer
  size_t max_elements_; // buffer size
public: // API, implemented on base API
  ctor(size_t max_elements); ~dtor(); // allocate buffer on ctor; deallocate on dtor
  ctor<U>(allocator_rebind<Alloc,U> const& alloc, size_t max_elements);
  // other API just the same as compile_time_sized_ringbuffer
};

using detail::make_ringbuffer::ringbuffer_type<T, ...Options> = // extract from Options
  !has_capacity ? runtime_sized_ringbuffer<T, allocator> : compile_time_sized_ringbuffer<T, capacity>;

template<typename T, typename... Options>
  requires is_default_constructible_v<T> && is_move_assignable_v<T> || is_copy_assignable_v<T>
class spsc_queue : public make_ringbuffer<T, Options> {
public:
  using allocator = Options/* extract from Options */;
  using value_type = T; using size_type = size_t;

  spsc_queue([allocator const&]);  spsc_queue<U>(allocator<U>const&); // has_capacity (comptime)
  explicit spsc_queue(size_type n[, allocator<U> const&]); // !has_capacity (runtime/fixed)
  ~spsc_queue(); // comsume_all
  // not copyable, not moveable

  // API implemented forward to buffer API:
  // push, pop, front, consume_one, consume_all, read_available, write_available, reset
  optional<U> pop<U>(uses_optional_t);
  size_t pop(Iter); // call pop_to_output_iterator, enabled when convertible
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

```c++
template<typename T, typename... Options>
class spsc_value {
  constexpr bool allow_multiple_reads = Options;// extract
  struct tagged_index { uint8_t byte; }; // bits 0-2 is index (range 0-7), bits 3 is tag (true/false, means `is_consumable`)
  struct alignas(cacheline_bytes) cache_aligned_value { T value; };

  std::array<cache_aligned_value, 3> m_buffer; //
  tagged_index m_write_index{0};
  atomic<tagged_index> m_available_index{1};
  tagged_index m_read_index{2};

  void swap_write_buffer(); bool swap_read_buffer();
public:
  explicit spsc_value(); // if `allow_multiple_reads`, write default value at buf[0]
  explicit spsc_value(T v); // write initial v at buf[0]
  void write(const T&); void write (T&&);
  bool read(T&); optional<T> read(uses_optional_t);
  bool consume<Fun>(Fun&&);
};
```

* Implemented on three slots: reading, writing and free_available.
  * reading and writing are never conflict
  * just swap with free slot on read/write.
* if not `allow_multiple_reads`, check slot index tag for readed state, then move the value instead of copying it.

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


