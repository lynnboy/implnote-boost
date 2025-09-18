# Boost.Sort

* lib: `boost/libs/sort`
* repo: `boostorg/sort`
* commit: `1a58a13`, 2025-06-29

------
### Sort Algorithms Overview

Algorithm complexities:

Algorithm | Parallel | Stable? | Additional Memory | Best | Average | Worse
-|-|-|-|-|-|-
`spreadsort` | no | no | $\tt{key\_length}$ | $N$ | $N\cdot\sqrt{\log{N}}$ | $\min({N\cdot\log{N}, N\cdot\tt{key\_length}})$
`pdqsort` | no | no | $\log{N}$ | $N$ | $\log{N}$ | $\log{N}$
`spinsort` | no | yes | $N/2$ | $N$ | $\log{N}$ | $\log{N}$
`flat_stable_sort` | no | yes | $\tt{data\_size}/256 + 8\rm{K}$ | $N$ | $\log{N}$ | $\log{N}$
`block_indirect_sort` | yes | no | $\tt{block\_size} \cdot \tt{num\_threads}$ | $N$ | $\log{N}$ | $\log{N}$
`sample_sort` | yes | yes | $N$ | $N$ | $\log{N}$ | $\log{N}$
`parallel_stable_sort` | yes | yes | $N/2$ | $N$ | $\log{N}$ | $\log{N}$

`block_size` choices:

Object Size (bytes) | 1 - 15 | 16 - 31 | 32 - 63 | 64 - 127 | 128 - 255 | 256 - 511 | 512 -
-|-|-|-|-|-|-|-
`block_size` (number of elements) | 4096 | 2048 | 1024 | 768 | 512 | 256 | 128

-----
### Common utility

```c++
namespace common;
// traits
using util::value_iter<It> = iterator_traits<It>::value_type;
using util::compare_iter<It> = std::less<value_iter<It>>;
using util::enable_if_integral<T> = enable_if_t<is_integral_v<T>>; // and enable_if_not_integral
using util::enable_if_string<T> = enable_if_t<is_same<T,std::string>>; // and enable_if_not_string
struct util::constructor<T> { void operator()(auto&&...args); }; // wrap invoke of T's ctor.

// algorithms
uint32_t util::nbits32(uint32_t n) noexcept; // and `nbits64`. Get required bitwidth for n
void util::construct_object<T, ...Args>(T* p, Args&&... args); // placement-new call ctor with arg forwarding
void util::destroy_object<T>(T* p); // call dtor
void util::initialize<It,T=value_iter<It>> (It f, It l, T& v); // move construct v to *f, and move construct each value to next, the move assign *(l-1) to v
It2 util::move_forward<It1,It2>(It2 dest, It1 f, It1 l); // move assign each of [f,l) into [dest,-)
It2 util::move_backward<It1,It2>(It2 dest, It1 f, It1 l); // move assign each of [f,l) into [-,dest), from l to f
T* util::move_construct<It,T=value_iter<It>> (T* p, It f, It l); // move construct each of [f,l) into [p, -)
void util::destroy<It>(It f, It l); // call destroy_object on each of [f,l)
void util::reverse<It>(It f, It l); // just std::reverse(f,l)

struct range<Iter> {
  Iter first, last; // stored iterator pair
  ctor(Iter const&, Iter const&);
  bool empty() const; bool not_empty() const; bool valid() const; size_t size() const;
  Iter front() const; Iter back() const;
};
range<It> concat(range<It> const&, range<Itr> const&); // pre: contiguous
range<It2> move_forward<It1,It2>(range<It2> const& dest, range<It1> const& src);
range<It2> move_backward<It1,It2>(range<It2> const& dest, range<It1> const& src);
range<T*> move_construct<It,T=value_iter<It>>(range<T*> const& dest, range<It> const& src);
void destroy<It>(range<It>);
range<It> initialize<It,T=value_iter<It>>(range,It> const&, T& v);

// pivot point
It mid3<It,Compare> (It i1, It i2, It i3, Compare comp); // sort the three elements and return iterator to the middle
void pivot3<It,Compare> (It f, It l, Compare comp); // find mid3 of (f[1],f[N/2],l[-1]), swap with first element
It mid9<It,Compare> (It i1, It i2, ..., It i9, Compare comp); // mid3( mid3(i1,i2,i3), mid3(i4,i5,i6), mid3(i7,i8,i9) )
void pivot9<It,Compare> (It f, It l, Compare comp); // find mid9 of (f[1],f[N/8],f[2N/8],...,f[7N/8],l[-1]), swap with first element
```

#### Searching

```c++
struct filter_pass<T> { using key=T; const key& operator()(const T& v) const { return v; } }; // identity value projection
template<class It, class Filter=filter_pass<value_iter<It>>>, class Compare=std::less<Filter::key>>
It util::find_first(It f, It l, Filter::key const& v, Compare const& comp=Compare(), Filter proj=Filter());
It util::find_last(It f, It l, Filter::key const& v, Compare const& comp=Compare(), Filter proj=Filter());
It util::lower_bound(It f, It l, Filter::key const& v, Compare const& comp=Compare(), Filter proj=Filter());
It util::upper_bound(It f, It l, Filter::key const& v, Compare const& comp=Compare(), Filter proj=Filter());
std::pair<It,It> util::equal_range(It f, It l, Filter::key const& v, Compare const& comp=Compare(), Filter proj=Filter());
It util::insert_first(It f, It l, Filter::key const& v, Compare const& comp=Compare(), Filter proj=Filter()); // => lower_bound
It util::insert_last(It f, It l, Filter::key const& v, Compare const& comp=Compare(), Filter proj=Filter()); // => upper_bound
```

#### Circular Buffer

```c++
template <class T, uint32_t Power2=11>
struct util::circular_buffer { // buffer size is 2's power, default 8192
  using value_t = T;
  const size_t NMAX = 1<<Power2, MASK = NMAX-1, BLOCK_SIZE = NMAX / 2, LOG_BLOCK = Power2 - 1;

  T* ptr = nullptr; size_t nelem, first_pos; bool initialized;

  ctor(); dtor(); // malloc / free
  void initialize(T& v); void destroy_all();
  T* get_buffer();
  bool empty() const; bool full() const;
  size_t size() const; size_t capacity() const; size_t free_size() const;
  void clear();
  [const] T& front() [const]; [const] T& back() [const];
  [const] T& operator[](uint32_t) [const];
  void push_front(T{const|&}&); void push_back(T{const|&}&);
  void pop_front(); void pop_back();
  void {push|pop}_{move|copy}_{front|back} <It> (It first, size_t n);
};
```

#### Insert sort supporting

```c++
// Insert sort. pre: [f,m) is sorted. insert each of [m,l) into ordered place within [f,m). post: [f,l) is sorted.
// move [m,l) to [space,-), for each v in [space,-): find pos in [f,m), make room by move_backward, then emplace v
void util::insert_sorted<It1,It2,Compare>(It1 f, It1 m, It1 l, Compare comp, It2 space); // and also `insert_sorted_backward`
void util::insert_sorted_backward<It1,It2,Compare>(It1 f, It1 m, It1 l, Compare comp, It2 space); // similar

void insert_sort<It,Compare=compare_iter<It>> (It f, It l, Compare comp=Compare());

It is_stable_sorted_forward<It,Compare=compare_iter<It>> (It f, It l, Compare comp=Compare());
It is_reverse_stable_sorted_forward<It,Compare=compare_iter<It>> (It f, It l, Compare comp=Compare());
size_t number_stable_sorted_forward<It,Compare=compare_iter<It>> (It f, It l, size_t min_n, Compare comp=Compare());
It is_stable_sorted_backward<It,Compare=compare_iter<It>> (It f, It l, Compare comp=Compare());
It is_reverse_stable_sorted_backward<It,Compare=compare_iter<It>> (It f, It l, Compare comp=Compare());
size_t number_stable_sorted_backward<It,Compare=compare_iter<It>> (It f, It l, size_t min_n, Compare comp=Compare());

// recursively run at two half subranges; at even levels, call `insert_sort` for 32 and below elements; then merge to r2
void internal_sort<It1,It2,Compare> (range<It1> const& r1, range<It2> const& r2, Compare comp, uint32_t level, bool even=true);
// call `internal_sort`, or `insert_sort` for 32 and below elements.
void range_sort_data<It1,It2,Compare> (range<It1> const& r, range<It2> const& space, Compare comp);
void range_sort_buffer<It1,It2,Compare> (range<It1> const& r, range<It2> const& space, Compare comp);
```

#### Merge sort supporting

```c++
// *r2.front() < *r1.back() (r1 not all < r2); stable: !(r1.back() < r2.front())
bool is_mergeable<It1,It2,Compare> (range<It1> const&, range<It2> const&, Compare comp);
bool is_mergeable_stable<It1,It2,Compare> (range<It1> const&, range<It2> const&, Compare comp);

// Merge sort. pre: [b1,e1) and [b2,e1) both sorted. post [out,out+n1+n2) sorted.
// optimize: if n1+n2>=1024, check whether [b1,e1) and [b2,e1) is not overlapped, then just range-move twice.
// keep move the smaller of first elements of the two ranges to out, until empty
It3 util::merge<It1,It2,It3,Compare> (It1 b1, It1 e1, It2 b2, It2 e2, It3 out, Compare comp);
T* util::merge_construct<It1,It2,T,Compare> (It1 f1, It1 l1, It2 f2, It2 l2, T* out, Compare comp); // similar
// Similar to `merge`, but `out` = `b2 - n1`. Omitted unnecessary movement for buffer 2
It2 util::merge_half<It1,It2,Compare> (It1 b1, It1 e1, It2 b2, It2 e2, It2 out, Compare comp);
It2 util::merge_half_backward<It1,It2,Compare> (It1 b1, It1 e1, It2 b2, It2 e2, It1 e_out, Compare comp); // similar, `e_out` = `e1 + n2`
// pre: [b1,e1) and [b2,e2) both sorted. post: [b1,e1) ++ [b2,e2) sorted. return true if no movement occurred.
bool util::merge_uncontiguous<It1,It2,It3,Compare> (It1 b1, It1 e1, It2 b2, It2 e2, It3 space, Compare comp);
// pre: [b1,b2) and [b2,e) both sorted. post: [b1,e) sorted.
bool util::merge_contiguous<It1,It2,Compare> (It1 b1, It1 b2, It2 e, It2 space, Compare comp);
// [b1,e1) and [b2,e2) are sorted ranges. Merge sort into `c`.
// o1 and o2 points to lefted elements after merge. (may left some elements not merged).
// One of the buffers fully merged, return true if buffer1 exhausted, false for buffer2.
bool util::merge_circular<It1,It2,Circular,Compare> (It1 b1, It1 e1, It2 b2, It2 e2, Circular& c, Compare comp, It1& o1, It2& o2);

range<It3> merge<It1,It2,It3,Compare> (range<It3>const& dest, range<I1>const& r1, range<I2> const& r2, Compare comp);
range<T*> merge_construct<It1,It2,It3,Compare> (range<T*>const& dest, range<I1>const& r1, range<I2> const& r2, Compare comp);
range<It2> merge_half<It1,It2,Compare> (range<It2>const& dest, range<I1>const& r1, range<I2> const& r2, Compare comp);
bool merge_uncontiguous<It1,It2,It3,Compare> (range<I1>const& r1, range<I2> const& r2, range<It3>const& space, Compare comp);
bool merge_contiguous<It1,It2,Compare> (range<I1>const& r1, range<I1> const& r2, range<It2>const& space, Compare comp);
// pre: data in rbuf++r2, size of r1, rbuf, r2 are the same; post: sorted data in a1++rbuf
// fill r1 with merge of rbuf and r2; then merge_half on remaining data of rbuf and r2
void merge_flow<It1,It2,Compare>(range<It1> r1, range<It2> rbuf, range<It1> r2, Compare comp);

// comparing stably
bool less_range<It,Compare=compare_iter<It>> (It i1, uint32_t pos1, It i2, uint32_t pos2, Compare comp = Compare());
// merge 0~4 sorted ranges.
range<It1> full_merge4<It1,It2,Compare> (range<It1> const& dest, range<It2> r[4], uint32_t n, Compare comp);
// similar, but use move_construct / merge_construct
range<T*> uninit_full_merge4<T,It,Compare> (range<T*> const& dest, range<It> r[4], uint32_t n, Compare comp);
void merge_level4<It1,It2,Compare> (range<It1> dest, std::vector<range<It2>>& v_in, std::vector<range<It1>>& v_out, Compare comp);
void uninit_level4<T,It,Compare> (range<T*> dest, std::vector<range<It>>& v_in, std::vector<range<T*>>& v_out, Compare comp);
// merge ranges stored in vectors. call `merge_level4` as core step
range<It2> merge_vector4<It1,It2,Compare> (range<It1> r_in, range<It2> r_out, std::vector<range<It1>>& v_in, std::vector<range<It2>>& v_out; Compare comp);

// class to merge contiguous blocks, which are each ordered.
struct merge_block<It,Compare,Power2=10> {
  using value_t = value_iter<It>;
  using range_pos = range<size_t>; using range_it = range<It>; using range_buf = range<value_t*>;
  using it_index = std::vector<size_t>::iterator;
  using circular_t = circular_buffer<value_t, Power2+1>; // twice size as block size

  const size_t BLOCK_SIZE = 1<<Power2, LOG_BLOCK=POWER2;

  const size_t nelem, nblock, ntail; // number of elements, blocks, and elements in last (tail) block
  range_it global_range, range_tail; // global range [f,l), and tail block [f+(nblock-1)*BLOCK_SIZE, l)
  std::vector<size_t> index; // block index map, arrangement of [0, nblock)
  Compare cmp;
  circular_t* ptr_circ; bool owned; // ownership flag for the circular buffer instance.

  ctor(It f, It l, Compare comp, circular_t* pbuf=nullptr); // initialize counts, ranges, index map, and buffer
  ~dtor();
  range_it get_range(size_t pos) const; // return range of pos-th block 
  range_it get_group_range(size_t pos, size_t nrange) const; // return range of blocks of indexes [pos,pos+n)
  bool is_tail(size_t pos) const; // is pos-th block the tail block?

  // merge blocks's indexes, call `util::merge_circular` as core step
  void merge_range_pos(it_index f, it_index m, it_index l);
  void move_range_pos_backward(it_index f, it_index l, size_t n); // used by `merge_range_pos`
  void rearrange_with_index(); // rearrange by blocks according to `index` array.
};
```

#### Indirect Sort via Indexing, Rearrangement

```c++
void create_index<It>(It f, It l, std::vector<It>& index); // place iterators over [f,l) into index
void sort_index<It>(It global_first, std::vector<It>& index); // move rearrange elements accordingly
void indirect_sort<Sorter,It,Compare=compare_iter<It>> (Sorter sort, It f, It l, Compare comp) {
  std::vector<It> index;
  create_index(f, l, index);
  auto index_comp = [comp](It i1, It i2){ return comp(*i1, *i2); };
  sort(index.begin(), index.end(), index_comp);
  sort_index(f, index);
}

using filter_iterator = [origin](It idx){return itx-origin;};
using filter_pos = identity<size_t>;
void rearrange<It,ItIdx,Mapper> (It data, ItIdx f, ItIdx l, Mapper mapper);
```

#### Concurrency Support

```c++
// Atomic operations
T util::atomic_read<T> (std::atomic<T>& a); // acquire
T util::atomic_add<T,T2> (std::atomic<T>& a, T2 v); //acq_rel
T util::atomic_sub<T,T2> (std::atomic<T>& a, T2 v); //acq_rel
void util::atomic_write<T,T2> (std::atomic<T>& a, T2 v); // release
struct util::counter_guard<T> { std::atomic<T>& c; ctor(std::atomic<T>&); ~dtor(){ atomic_sub(count,1); } }; // RAII guard

class spinlock_t {
  std::atomic_flag af;
public:
  ctor();
  void lock() noexcept { while af.test_and_set() std::this_thread::yield(); }
  bool try_lock() noexcept { return !af.test_and_set(); }
  void unlock() noexcept { af.clear(); }
};

struct stack_cnc<T,Alloc=std::allocator<T>> {
  using vector_t = std::vector<T,Alloc>; // wrapped container
  using size_type = vector_t::size_type; // also different_type, value_type, [const_]pointer, [const_]reference, allocator_type
  vector_t v_t; mutable spinlock_t spl; // data and locker, std::lock_guard on spl to protect
  ctor(); virtual ~dtor(); // no move-ctor
  // all member guarded by std::lock_guard(spl)
  void emplace_back<...Args>(Args&&...args); // push
  bool pop_move_back(T& v); // pop (move out), guarded
  stack_cnc& push_back<A2> (std::vector<T,A2> const& vec); // batched, guarded
};

struct deque_cnc<T,Alloc=std::allocator<T>> {
  using deque_t = std::deque<T,Alloc>;
  using size_type = deque_t::size_type; // also different_type, value_type, [const_]pointer, [const_]reference, allocator_type
  deque_t dq; mutable spinlock_t spl; // data and locker, std::lock_guard on spl to protect
  ctor(); explicit ctor(Alloc const&); virtual ~dtor(); // init & dtor
  // all member guarded by std::lock_guard(spl)
  void clear();
  void swap(deque_cnc&) noexcept;
  size_type size() const noexcept; size_type max_size() const noexcept;
  void shrink_to_fit(); bool empty() const noexcept;
  void push_{back|front}(T const&); void emplace_{back|front}<...Args>(Args&&...args);
  deque_cnc& push_{back|front}<A2> (std::deque<T,A2> {const|&}&);
  void pop_{back|front}(); bool pop_copy_{back|front}(T& v); bool pop_move_{back|front}(T& v);
};
```

#### Scheduler (not used)

```c++
struct scheduler<F,Alloc=std::allocator<F>> {
  using scoped_alloc = ;

  using deque_t = std::deque<F,std::scoped_allocator_adaptor<Alloc>>;
  using key_t = std::thread::id; using pair_t = std::pair<const key_t, deque_t>; 
  using map_t = std::unordered_map<key_t, deque_t, std::hash<key_t>, std::equal_to<key_t>, Alloc::rebind<pair_t>>
  using it_deque = deque_t::iterator; using it_map = map_t::iterator;
  using lock_t = std::unique_lock<spinlock_t>;

  map_t mp; // map of <thread id> -> <sequence of F>
  size_t nelem; // total items bookkeeping
  mutable spinlock_t spl; // lock with lock_t for each member function

  ctor(); virtual ~dtor(); // disable move/copy
  size_t size() const;
  void clear_all();

  void insert(F& f); // move f into the deque for current thread.
  void emplace<...Args>(Args&&...args); // emplace construct into the deque for current thread.
  void insert_range<it_F> (it_F f, it_F l); // insert all of [f,l) into the deque for current thread.

  bool extract(F& f); // move out one from deque for current thread
};
std::ostream& operator<< <...Args> (std::ostream& o, const std::deque<Args...>& dq);
std::ostream& operator<< <F,A> (std::ostream& o, const scheduler<F,A>& sch); // debug printing
```

#### Range

-----
### Algorithms

------
#### Spread Sort

Header `<boost/sort/sort.hpp>` or `<boost/sort/spreadsort/spreadsort.hpp>`

```c++
template<RandomAccessIterator RAIter>
  void spreadsort(RAIter first, RAIter last);
template<Range R>
  void spreadsort(Range auto& r);

template<RandomAccessIterator RAIter>
  void integer_sort(RAIter first, RAIter last[, RShift][, Comp]);
template<Range R>
  void integer_sort(R& r[, RShift][, Comp]);

template<RandomAccessIterator RAIter>
  void float_sort(RAIter first, RAIter last[, RShift][, Comp]);
template<Range R>
  void float_sort(R& r[, RShift][, Comp]);

Cast_type float_mem_cast<Cast_type>(const Data_Type&)
    requires sizeof(Cast_type) == sizeof(Data_Type) &&
              numeric_limits<Data_type>::is_iec559 && numeric_limits<Cast_type>::is_integer;

template<RandomAccessIterator RAIter>
  void [reverse_]string_sort<UCharT=unsigned char>(RAIter first, RAIter last);
template<Range R>
  void [reverse_]string_sort<UCharT=unsigned char>(R& r);

template<RandomAccessIterator RAIter>
  void [reverse_]string_sort(RAIter first, RAIter last, Get_char, Get_length[, Comp]);
template<Range R>
  void [reverse_]string_sort(R& r, Get_char, Get_length[, Comp]);
```

* `spreadsort` is an extremely fast hybrid radix sort algorithm.
  Spreadsort combines generic implementations of multiple high-speed sorting algorithms
  that outperform those in the C++ standard in both average and worst case performance
  when there are over 1000 elements in the list to sort.
* `spreadsort` will forward to `float_sort`, `integer_sort`, or `string_sort` based on `value_type` of `RAIter`.
* `wstring` is supported only when `wchar_t` is 16-bit.
* `float_mem_cast` is provided to help implement `RShift` functor for floating point.
* `RShift`, `Comp`, and `Get_char`, `Get_length` allow user project to a data field of the `RAIter`'s value type.
* Configuration constants are in `<boost/sort/spreadsort/detail/constants.hpp>`, but is fixed now.

------
#### PDQ Sort

Header `<boost/sort/sort.hpp>` or `<boost/sort/pdqsort/pdqsort.hpp>`

```c++
```

* `pdqsort` is a improvement of the quick sort algorithm. 

------
#### Spin Sort

Header `<boost/sort/sort.hpp>` or `<boost/sort/spinsort/spinsort.hpp>`

```c++
```

* `spinsort` is a stable sort that is fast with random or nearly sorted data.

------
#### Flat Stable Sort

Header `<boost/sort/sort.hpp>` or `<boost/sort/flat_stable_sort/flat_stable_sort.hpp>`

```c++
```

* `flat_stable_sort` is a stable sort that uses very little additional memory (around 1% of the size of the data), providing 80% - 90% of the speed of `spinsort`.

------
#### Block Indirect Sort

Header `<boost/sort/sort.hpp>` or `<boost/sort/block_indirect_sort/block_indirect_sort.hpp>`

```c++
```

* `block_indirect_sort` is a novel high-speed parallel sort algorithm with low additional memory consumption.

------
#### Sample Sort

Header `<boost/sort/sort.hpp>` or `<boost/sort/sample_sort/sample_sort.hpp>`

```c++
```

* `sample_sort` is a implementation of the Samplesort algorithm.

------
#### Spin Sort Algorithm

Header `<boost/sort/sort.hpp>` or `<boost/sort/parallel_stable_sort/parallel_stable_sort.hpp>`

```c++
```

* `parallel_stable_sort` is based on the Samplesort algorithm, but using a half of the memory used by `sample_sort`.

------
### Dependency

#### Boost.Config

* `<boost/cstdint.hpp>` - required by `spreadsort`

#### Boost.Core

* `<boost/utility/enable_if.hpp>` - required by `spreadsort`

#### Boost.Range

* `<boost/range/begin.hpp>`, `<boost/range/end.hpp>` - required by `spreadsort`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>` - required by `spreadsort`

#### Boost.TypeTraits

* `<boost/type_traits.hpp>` - required by `pdqsort`, `spreadsort`, `block_indirect_sort`

------
### Standard Facilities
