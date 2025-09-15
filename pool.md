# Boost.Pool

* lib: `boost/libs/pool`
* repo: `boostorg/pool`
* commit: `571909a`, 2025-07-04

------
### Simple Segregated Storage (Free-List Pool)

Header `<boost/pool/simple_segregated_storage.hpp>`

```c++
class simple_segregated_storage<SizeType=size_t> { // noncopyable
protected:
  void * first;   // head of free list

  static void *& nextof(void * const ptr);    // access the 'next' ptr in block
  void * find_prev(void * ptr);               // find insert position for new block in free-list
public:
  using size_type = SizeType;

  simple_segregated_storage();  // empty, no blocks yet
  bool empty() const { return first == 0; }

  static void* segregate(void * block, size_type blocksize, size_type chunksize, void * end = 0);
  void add_block(void * block, size_type blocksize, size_type chunksize)
    { first = segregate(block, blocksize, chunksize, first); }   // new free-list head
  void add_ordered_block(void * block, size_type blocksize, size_type chunksize) {
    void * loc = find_prev(block);                        // insert in free-list at proper position
    if (loc == 0) add_block(block, blocksize, chunksize);
    else nextof(loc) = segregate(block, blocksize, chunksize, nextof(loc);
  }

  void * malloc() { void * ret = first; first = nextof(first); return ret; }  // get head
  void free(void * chunk) { nextof(chunk) = first; first = chunk; }           // put at head
  void ordered_free(void * chunk);                  // find insert position and link in free-list
  void * malloc_n(size_type n, size_type partition_size);       // try to find n contiguous free chunks
  void free_n(void * chunks, size_type blocksize, size_type partition_size);          // add_block
  void ordered_free_n(void * chunks, size_type blocksize, size_type partition_size);  // add_ordered_block
};
```

* Free list is single-list, stored in each block as `void*` pointer.
* `chunksize` should be `sizeof(void*) * n`, `blocksize` should be at least `chunksize`.
* `segregate()` will split block by `chunksize`, and build link within it, returning list head.

------
### Pools and Allocators

* Header `<boost/pool/pool.hpp>`

```c++
struct default_user_allocator_new_delete {
  // size_type, difference_type
  static char* malloc(size_type bytes) { return new (std::nothrow) char[bytes]; }
  static void free(char* block) { return delete[] block; }
};
struct default_user_allocator_malloc_free {
  // size_type, difference_type
  static char* malloc(size_type bytes) { return ::malloc(bytes); }
  static void free(char* block) { return ::free(block); }
};

struct details::PODptr<SizeType> { // used internally to refer full buffer, store n chunks and ptr & size of next buffer
  char* ptr; size_type sz;    // sz = n * chunksize + sizeof(void*) + sizeof(size_type)
  ctor(char* const, size_type); // and def-ctor
  bool valid() const; void invalidate();    // invalid when ptr==nullptr
  // begin(), end(), total_size(), element_size(), next_size(), next_ptr()
  PODptr next() const { return PODptr(next_ptr(), next_size()); }
  void next(PODptr const& arg) const { next_ptr() = arg.begin(); next_size() = arg.total_size(); }
};

class pool<UserAllocator = default_user_allocator_new_delete>
  : protected simple_segregated_storage<UserAllocator::size_type> {
  static const size_type min_alloc_size, min_align;
protected:
  PODptr<size_type> list;                 // holds head to block buffer list
  const size_type requested_size;
  size_type next_size, start_size, max_size;

  PODptr find_POD(void * chunk) const;    // find the buffer containing chunk
  [const] simple_segregated_storage<size_type> & store() [const]; // self (base-obj)
public:
  using user_allocator = UserAllocator; // and size_type, difference_type

  explicit pool(size_type requested_size, size_type next_size = 32, size_type max_size = 0);
  ~pool() { purge_memory(); }
  bool release_memory();  // order required, release empty blocks
  bool purge_memory();    // release all blocks

  // {get|set}_next_size(), {get|set}_max_size(), get_requested_size()
  void * [ordered_]malloc();
  void * ordered_malloc(size_type n);
  void [ordered_]free(void*);
  void [ordered_]free(void*, size_type n);

  bool is_from(void * chunk) const;   // whether chunk is allocated from one of the blocks
};
```

* Allocation is started at `next_size`, and `next_size` is growing in a doubling scheme.

* Header `<boost/pool/singleton_pool.hpp>`
* Header `<boost/pool/object_pool.hpp>`

```c++
class singleton_pool<Tag, RequestedSize,
  UserAllocator = default_user_allocator_new_delete,
  Mutex = default_mutex, NextSize = 32, MaxSize = 0> {
  struct pool_type : Mutex, pool<UserAllocator> {
    pool_type() : pool(RequestedSize, NextSize, MaxSize) {} // fixed by template arguments
  };
  static aligned_storage<sizeof(pool_type), alignment_of_v<pool_type>> storage; // the singleton instance
  static pool_type& get_pool(); // initialize 'storage' as a 'pool_type' on first access
public:
  // malloc(), free() and friends, is_from, release_memory, purge_memory
  //   forwarding to stored pool, but guarded by mutex
};

class object_pool<T, UserAllocator=default_user_allocator_new_delete>
  : protected pool<UserAllocator> {
public:
  using element_type = T;

  explicit object_pool(size_type next_size = 32, size_type max_size = 0)
    : pool(sizeof(T), next_size, max_size) {}
  ~object_pool();           // iterate all blocks, call dtor for each allocated trunks, deallocate each block

  // malloc(), free(), is_from(), call 'ordered_malloc' and 'ordered_free', casted to 'element_type *'
  // {get|set}_next_size

  element_type * construct<...Args>(Args&&...args); // malloc and placement new with forwarding
  void destroy(element_type *);       // call dtor and free
};
```

* `singleton_pool` wraps a `pool` and protect it with a customizable mutex, the pool is forced create before `main`.

* Header <boost/pool/pool_alloc.hpp>

```c++
struct pool_allocator_tag; struct fast_pool_allocator_tag;
class pool_allocator<T, UserAllocator = default_user_allocator_new_delete,
  Mutex = default_mutex, NextSize = 32, MaxSize = 0>;
class fast_pool_allocator<T, UserAllocator = default_user_allocator_new_delete,
  Mutex = default_mutex, NextSize = 32, MaxSize = 0>;
```

* `pool_allocator` and `fast_pool_allocator` provide standard 'Allocator' concept, upon `singleton_pool`
* `pool_allocator` call `ordered` version, suitable for array-allocation scenario
* `fast_pool_allocator` call non-`ordered` version, suitable for single-allocation scenario

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`

#### Boost.Integer

* `<boost/integer/common_factor_ct.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>` - by allocators

#### Boost.TypeTraits

* `<boost/type_traits/alignment_of.hpp>`
* `<boost/type_traits/aligned_storage.hpp>`

#### Boost.WinAPI

* `<boost/winapi/critical_section.hpp>` - by singleton_pool and allocators for Windows and no stdlib `<mutex>`

------
### Standard Facilities

* Standard Library: `<memory>`, standard allocator
