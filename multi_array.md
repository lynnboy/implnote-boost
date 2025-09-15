# Boost.MultiArray

* lib: `boost/libs/multi_array`
* repo: `boostorg/multi_array`
* commit: `333c304`, 2025-06-26

------
### Multi-dimensional Array

Header `<boost/multi_array.hpp>`

------
#### Concepts & Concept Check

```c++
template<class Array, size_t NumDims>
concept ConstMultiArrayConcept =
  ForwardTraversalConcept<Array::iterator> && ReadableIteratorConcept<Array::iterator> && // and `const_iterator`
  requires(Array a, Array::index id, Array::index_gen idgen, Array::index_range range)
  { { a[id] }; {a[idgen[id]]}; {a[idgen[range]]}; } &&
  requires(Array a) { // following: convertible_to<>
    typename Array::value_type; // also [const_]reference, size_type, difference_type, [const_][reverse_]iterator
        // element, index, index_gen, index_range, extent_gen, extent_range
    { a.size(), a.num_dimensions(), Array::index_range, a.num_elemnts() } -> size_t;
    { a.shape() } -> const size_t*;
    { a.strides(), a.index_bases() } -> const Array::index*;
    { a.begin(), a.end() } -> Array::const_iterator;
    { a.rbegin(), a.rend() } -> Array::const_reverse_iterator;
    { a.origin() } -> const Array::element*;
  };

template<class Array, size_t NumDims>
concept MutableMultiArrayConcept =
  ForwardTraversalConcept<Array::iterator> && ReadableIteratorConcept<Array::iterator> && // and `const_iterator`
  WritableIteratorConcept<Array::iterator> && OutputIterator<Array::iterator, Array::value_type> &&
  requires(Array a, const Array& ca, Array::index id, Array::index_gen idgen, Array::index_range range)
  { { a[id] } -> Array::value_type; {ca[idgen[range][id]]}; {ca[idgen[range][range]]}; } &&
  requires(const Array& a) { // ->: convertible_to<>
    typename Array::value_type; // also [const_]reference, size_type, difference_type, [const_][reverse_]iterator
        // element, index, index_gen, index_range, extent_gen, extent_range
    { a.size(), a.num_dimensions(), Array::index_range, a.num_elemnts() } -> size_t;
    { a.shape() } -> const size_t*;
    { a.strides(), a.index_bases() } -> const Array::index*;
    { a.begin(), a.end() } -> Array::const_iterator;
    { a.rbegin(), a.rend() } -> Array::const_reverse_iterator;
    { a.origin() } -> const Array::element*;
  } &&
  requires (Array a) {
    { a.begin(), a.end() } -> Array::iterator;
    { a.rbegin(), a.rend() } -> Array::reverse_iterator;
  };
```

------
#### Common Types

```c++
namespace multi_array_types {
  using size_type = std::size_t;
  using difference_type = std::ptr_diff_t;
  using index = std::ptr_diff_t;

  using index_range = detail::index_range<index,size_type>;
  using extent_range = detail::extent_range<index,size_type>;

  using index_gen = detail::index_gen<0,0>;
  using extent_gen = detail::extent_gen<0>;
}

struct detail::index_range<Index,SizeType> {
  index start_, finish_, stride_; // default to std::numeric_limits<index> min, max, 1
  bool degenerate_; // default false, means the range is just one index

  index_range(); // default, [min,max)/1,false
  explicit index_range(index pos); // [pos,pos+1)/1,true
  index_range(index start, index finish, index stride=1); // [start,finish)/stride,false

  index_range& start|finish|stride (index);  // chaining setters:
  index start|finish|stride() const; bool is_degenerate() const; // getters
  index get_start(index low_index_range=min) const; index get_finish(index high_index_range=max) const;

  size_t size(index def) const; // (finish - start)/stride or default
  void set_index_range(index start, index finish, index stride=1);
  static index_range all(); // [min,max)/1
  
  index_range operator+(index shift) const; // also `-`, shift range
  index operator[](uint i) const {return start + i*stride; } // also `()`
  // range splicing: `s <= r`, `s < r`, `r < f`, `r <= f`
};
struct detail::extent_range<Extent,SizeType> : std::pair<Extent,Extent> { // first/second
  extent_range(index start, index finish); extent_range(index finish); extent_range();
  index start() const; index finish() const; size_type size() const;
};

struct detail::index_gen<NumRanges,NumDims> {
  array<index_range, max(NumRanges,1)> ranges_; // accumulated list by '[]'
  index_gen();

  index_gen<ND>(const index_gen<NumRanges-1,ND>&, const index_range&); // don't care dimension, append
  index_gen<NumRanges+1,NumDims+1> operator[](const index_range&) const; // add one sub-dimension's range
  index_gen<NumRanges+1,NumDims>   operator[](index) const; // add one 'degenerated' range, index fixed
};
struct detail::extent_gen<NumRanges> {
  array<extent_range, max(NumRanges,1)> ranges_; // accumulated list by '[]'
  extent_gen();

  extent_gen(const extent_gen<NumRanges-1>&, const extent_range&); // append a range
  extent_gen<NumRanges+1> operator[](const extent_range&) const; // add one sub-dimension's range
  extent_gen<NumRanges+1> operator[](index) const; // add one range, start from 0
};

namespace {
multi_array_types::extent_gen extents; // global, defined when no `BOOST_MULTI_ARRAY_NO_GENERATORS`
multi_array_types::index_gen indices; // global, defined when no `BOOST_MULTI_ARRAY_NO_GENERATORS`
}

struct general_storage_order<NumDims> {
  array<size_type, NumDims> ordering_;
  array<bool, NumDims> ascending_;
  size_t ordering(size_t dim) const {return ordering_[dim];}
  bool ascending(size_t dim) const {return ascending_[dim];}
};
class c_storage_order; // ordering = [ N-1, ..., 0 ], all_dims_ascending = true
class fortran_storage_order; // ordering = [ 0, ..., N-1 ], all_dims_ascending = true

struct detail::value_accessor_one<T> { // 1-dimension only
  // with the 7 types in multi_array_types: size_type, index, ...
  using element = T; using value_type = T; using reference = T&; using const_reference = T const&;
  Ref access<Ref,TPtr>(type<Ref>, index idx, TPtr base,
    const size_type* extents, const index* strides, const index* index_bases) const {
      return *(base+idx*strides[0]);  // base already shifted
  }
};
struct detail::value_accessor_n<T, NumDims> { // go down one dimension
  // with the 7 types in multi_array_types: size_type, index, ...
  using element = T; using value_type = multi_array<T, NumDims-1>;
  using reference = sub_array<T, NumDims-1>; using const_reference = const_sub_array<T, NumDims-1>;
  Ref access<Ref,TPtr>(type<Ref>, index idx, TPtr base,
    const size_type* extents, const index* strides, const index* index_bases) const {
      TPtr newbase = base+idx*strides[0];  // base already shifted
      return Ref(newbase, extents+1, strides+1, index_bases+1); // sub_array into next dimension
  }
};

using detail::associated_types<T, NumDimsT> =
  NumDimsT::value == 1 ? value_accessor_one<T> : value_accessor_n<T, NumDimsT::value>;

struct detail::multi_array_impl_base<T, size_t NumDims> : associated_types<T, NumDimsT> {
  // types from associated_types: index, size_type, element, value_type, reference, ...
  using iterator = detail::array_iterator<T, T*, mpl::size_t<NumDims>, reference, random_access_traversal_tag>; // and `const_iterator`
  using reverse_iterator = boost::reverse_iterator<iterator>; // and `const_reverse_iterator`
  static constexpr size_t dimensionality = NumDims;
  // convenient templates
  using [const_]subarray<size_t NDims>::type = detail::[cosnt_]sub_array<T,NDims>;
  using [const_]array_view<size_t NDims>::type = detail::[cosnt_]multi_array_view<T,NDims>;
protected:
  Ref access_element<Ref,IndexList,TPtr>(type<Ref>, IndexList const& indices, TPtr base,
    const size_type* extents, const index* strides, const index* index_bases) const
    requires CollectionConcept<IndexList>
  { index offset{0}; for (auto [i, n] : enumerate(indices)) offset += i*strides[n]; return base[offset]; }
  void compute_strides<StrideList, ExtentList>
    (StrideList& stride_list, ExtentList& extent_list, general_storage_order<NumDims> const& storage) {
    index stride = 1;
    for (size_t n : range(NumDims)) {
      index stride_sign = storage.ascending(storage.ordering(n)) ? +1 : -1;
      stride_list[storage.ordering(n)] = stride * stride_sign;
      stride *= extent_list[storage.ordering(n)];
    }
  }
  index calculate_origin_offset<StrideList, ExtentList, BaseList>
    (StrideList const& stride_list, ExtentList const& extent_list,
     general_storage_order<NumDims> const& storage, BaseList const& index_base_list)
  { return calculate_descending_dimension_offset(stride_list, extent_list, storage) + calculate_indexing_offset(stride_list, index_base); }
  index calculate_descending_dimension_offset<StrideList, ExtentList>
    (StrideList const& stride_list, ExtentList const& extent_list, general_storage_order<NumDims> const& storage)
  { for (size_t n : range(NumDims))
      if (!storage.ascending(n)) offset -= stride_list[n]*(extent_list[n]-1); return offset; }
  index calculate_indexing_offset<StrideList, BaseList>(StrideList const& stride_list, BaseList const& index_base_list)
  { for (size_t n : range(NumDims)) offset -= stride_list[n]*index_base_list[n]; return offset; }
  ArrayRef generate_array_view<ArrayRef, NDims, TPtr>(type<ArrayRef>, detail::index_gen<NumDims,NDims> const& indices,
    const size_type* extents, const index* strides, const index* index_bases, TPtr base) const {
    array<index,NDims> new_strides, new_extents;
    index offset=0; size_t dim=0;
    for (size_t n: range(NumDims)) { // map index_bases and extents into indices.ranges_ for every dimension
      index_range const& current_range = indices.ranges_[n];
      index start = current_range.get_start(index_bases[n]);
      index finish = current_range.get_finish(index_bases[n] + extents[n]);
      index stride = current_range.stride();
      index len = (finish-start)/stride < 0 ? 0 : (finish-start + stride - (stride>0?1:-1))/stride;
      offset += start * strides[n];
      if (!current_range.is_degenerate()) {
        new_strides[dim] = stride * strides[n]; new_extents[dim] = len;
      }
    }
    return ArrayRef(base+offset, new_extents, new_strides);
  }
}
```

* `index_range` use `min` and `max` for infinit mark (all available value) values
* Making a `index_range` can use syntax `index_range().start(s).finish(f).stride(s)`, either start or finish required
* Use `get_start` and `get_finish` on `index_range` to handle infinit markers, `all` gives all index range
* `index_gen` is used as `indices[r0][r1]...`, and similar with `extends` for `extent_gen`

------
#### Multidimensional Array Adaptors (`array_ref`)

```c++
template <typename T, std::size_t NumDims, typename TPtr = const T*>
class const_multi_array_ref : public multi_array_impl_base<T,NumDims> {
  // disable copy-assign
protected:
  using size_list = array<size_type, NumDims>;
  using index_list = array<index, NumDims>;
  TPtr        base_;                        // base address, contiguously store num_elements
  storage_order_type storage_;              // array indexing order
  size_list   extent_list_;                 // size of each dimension
  index_list  stride_list_;                 // stride of each dimension
  index_list  index_base_list_;             // each dimension can indexed by [base, base+extent)
  index origin_offset_;                     // total offset upon base_ when striding
  index directional_offset_;                // offset caused by descending index
  size_type num_elements_;                  // size, product of each extends

  void init_multi_array_ref<Iter>(Iter extents_iter) { // called by ctor with extentlist arg
    extent_list_ = [[*extents_iter]]; // copy extents_iter into extent_list_
    num_elements_ = fold(extent_list_, *);// calcuate num_elements_ = accumulate of extent_list by multiply
    compute_strides(stride_list, extent_list_, storage_);
    directional_offset_ = calculate_descending_dimension_offset(stride_list, extent_list, storage_);
    origin_offset_ = directional_offset + calculate_indexing_offset(stride_list_, index_base_list_);
  }
  void init_from_extent_gen(const extent_gen<NumDims>& ranges) { // called by ctor with extent_gen arg
    // fill index_base_list_ with each extent_range.start()
    index_list extents;
    // fill extents from each extent_range.size()
    init_multi_array_ref(extents.begin());
  }

  ctor(TPtr base, const storage_order_type& so, const index* index_bases, const size_type* extents);

public:
  using storage_order_type = general_storage_order<NumDims>;
  // types & members from base class: value_type, element, size_type, index, ...
  ctor<OPtr>(const const_multi_array_ref<T,NumDims,OPtr>& other); // copy-ctor
  ctor<ExtentList>(TPtr base, ExtentList const& extents, storage_order_type const& so = c_storage_order());
  ctor(TPtr base, extent_gen<NumDims> const& ranges, storage_order_type const& so = c_storage_order());
  void assign<Iter>(Iter b, Iter e) { for (T* p = base_; b!=e && p-base_ < num_elements_; b++) *p = *b; } // just copy each element
  void reindex<BaseList>(BaseList const& values); // copy values to index_base_list_; then calculate_origin_offset()
  void reindex(index value); // copy value to all index_base_list_; then calculate_origin_offset()
  void reshape<SizeList>(SizeList const& extents); // copy extents to extent_list_; then compute_strides() and calculate_origin_offset()

  // accessors
  size_type num_dimensions() const { return NDims; }
  size_type size() const { return extent_list_.front(); } // first dimension's size
  size_type max_size() const { return NDims; } // max possible size to reshape
  bool empty() const { return size() == 0; }
  const size_type* shape() const { return extent_list_.data(); }
  const index* strides() const { return stride_list_.data(); }
  const element* origin() const { return base_ + origin_offset_; }
  const element* data() const { return base_; }
  size_type num_elements() const { return num_elements_; }
  const index* index_bases() const { return index_base_list.data(); }
  const storage_order_type& storage_order() const { return storage_; }

  // indexing
  const element& operator()<IndexList> (IndexList indices) const { return access_element(...); }
  const_reference operator[] (index idx) const { return access(...); }
  const_array_view<NDims>::type operator[] <NDims> (index_gen<NumDims,NDims> const& indices) const { return generate_array_view(...); }

  const_iterator begin() const; const_iterator end() const;
  const_reverse_iterator rbegin() const; const_reverse_iterator rend() const;
  // operator ==, !=, <, >, <=, >= based on `std::equal`, `std::lexicographical_compare`
};

template <typename T, std::size_t NumDims>
class multi_array_ref : public const_multi_array_ref<T,NumDims, T*> {
public:
  // types & members from base class: value_type, element, size_type, index, ...
  // forward ctors to base class ctors
  multi_array_ref& operator=(const multi_array_ref& other); // copy-assign, `std::copy`, assert same shape
  // non-const API:
  element* origin(); element* data();
  element& operator()<IndexList> (IndexList indices) { return access_element(...); }
  reference operator[] (index idx) { return access(...); }
  array_view<NDims>::type operator[] <NDims> (index_gen<NumDims,NDims> const& indices) { return generate_array_view(...); }
  iterator begin(); iterator end();
  reverse_iterator rbegin(); reverse_iterator rend();
};
```

* Each multi-array remember the base address and storage model, extent and indexing base for each dimension.
* The indexing is calculated by inner product of indices and strides, thus multi-array should pre-calculate
  the stride of each dimension, and adjust offset of the base address according to the storage indexing
* Accessing one element with a list of indices for each dimension
* Subscripting one level will get `sub_array` of next dimension, but last dimension gets normal reference to element
* Slicing with `index_gen` will make an `array_view`, which provides the same indexing calculation, but
  the base address and extent/stride/index_base are calculated according to the list of `index_range`
* `multi_array`, `multi_array_ref` and `multi_array_view` remembers array shape data

------
#### API of `multi_array`

```c++
template <typename T, std::size_t NumDims, typename Allocator>
class multi_array : public multi_array_ref<T,NumDims>, private Allocator { // priv impl for EBO
  allocate_space();   // called at every ctor; allocate `num_elements` elements, then uninitialized_fill_n with T()
  deallocate_space(); // called at dtor; destroy each element, deallocate
  T* base_;
  size_type allocated_elements_;
public:
  // types & members from base class
  ctor(Allocator const& = Allocator());
  ctor<ExtentList>(ExtentList const&, [general_storage_order<NumDims> const&], [Allocator const&]);
  ctor(extent_gen<NumDims> const&, [general_storage_order<NumDims> const&],[Allocator const&]);
  ctor(multi_array const&); // copy-ctor
  ctor<OPtr>([const_]multi_array_ref<T,NumDims,OPtr> const&,     // create copy from other array
      [general_storage_order<NumDims> const& = c_storage_order()],[Allocator const& = Allocator()]);
  ctor<OPtr>(detail::[const_]sub_array<T,NumDims,OPtr> const&,     // create copy from other array
      [general_storage_order<NumDims> const& = c_storage_order()],[Allocator const& = Allocator()]);
  ctor<OPtr>(detail::[const_]multi_array_view<T,NumDims,OPtr> const&,     // create copy from other array
      [general_storage_order<NumDims> const& = c_storage_order()],[Allocator const& = Allocator()]);
  multi_array& operator=<ConstMultiArray>(ConstMultiArray const&); // assign, also copy-assign

  multi_array& resize<ExtentList>(const ExtentList&);
  multi_array& resize(const extent_gen<NumDims>&);    // make copy via assigning same-shaped views, then swap
  // friend operator ==, !=
};
```

* `multi_aray_view`s can only be constructed by slicing.
* `sub_array`s can be constructed by subscripting and iterator dereferencing.
* `multi_array`, `multi_array_ref`, and `multi_array_view` can `reindex()`
* `multi_array` and `multi_array_ref` provides `reshape`, `data` and `assign`
* `multi_array` can `resize` to different `num_elements`

-----
#### Iterator, Sub Arrays (Reference), Views

```c++
template <typename T, typename TPtr, typename NumDimsT, typename Ref, typename Cat>
struct detail::array_iterator : iterator_facade<...>, private detail::associated_types<T, NumDimsT> { // priv impl for access() and types
  // iterator API from iterator_facade: value_type, reference, *, ->, ...
  index             idx_;         // current index of array
  TPtr              base_;        // base address, contiguously store num_elements
  const size_type*  extents_;     // size of each dimension
  const index*      index_base_;  // each dimension can indexed by [base, base+extent)
  const index*      strides_;     // stride of each dimension

  // ctor, copy-ctor
  ctor<OPtr, ORef, OCat>(array_iterator<T,OPtr,NumDimsT,ORef,OCat> const&); // copy-ctor for another type

  bool equal<Iter>(Iter& rhs) const; // idx_, base_, extents_, index_base_, strides_ contents all equal
  Ref dereference() const; // access()
  void increment() { ++idx_; } void decrement() { --idx_; }
  void advance<DiffType>(DiffType n) { idx_ += n; }
  difference_type distance_to<Iter>(Iter& r) const { return r.idx_ - idx_; }
};

template <typename T, std::size_t NumDims, typename TPtr>
class detail::const_sub_array : public multi_array_impl_base<T,NumDims> { // similar to array_ref
  // disable copy-assign
protected: // only store data&shape pointers, just reference, like array_iterator.
  TPtr              base_;        // base address, contiguously store num_elements
  const size_type*  extents_;     // size of each dimension
  const index*      index_base_;  // each dimension can indexed by [base, base+extent)
  const index*      strides_;     // stride of each dimension

  ctor(TPtr base, const size_type* extents, const index* strides, const index* index_bases);

public:
  // types & members from base class: value_type, element, size_type, index, ...
  ctor<OPtr>(const const_multi_array_ref<T,NumDims,OPtr>& other); // copy-ctor

  // accessors
  size_type num_dimensions() const { return NumDims; }
  size_type size() const { return extents_[0]; } // first dimension's size
  size_type max_size() const { return num_elements(); } // max possible size to reshape
  bool empty() const { return size() == 0; }
  const size_type* shape() const { return extents_; }
  const index* strides() const { return strides_; }
  TPtr origin() const { return base_; }
  const index* index_bases() const { return index_base_; }
  size_type num_elements() const { return fold(extents_, *); }

  // indexing
  const element& operator()<IndexList> (IndexList indices) const { return access_element(...); }
  const_reference operator[] (index idx) const { return access(...); }
  const_array_view<NDims>::type operator[] <NDims> (index_gen<NumDims,NDims> const& indices) const { return generate_array_view(...); }

  const_iterator begin() const; const_iterator end() const;
  const_reverse_iterator rbegin() const; const_reverse_iterator rend() const;
  // operator ==, !=, <, >, <=, >= based on `std::equal`, `std::lexicographical_compare`
};

template <typename T, std::size_t NumDims>
class detail::sub_array : public const_sub_array<T,NumDims, T*> {
public:
  // types & members from base class: value_type, element, size_type, index, ...
  // forward ctors to base class ctors
  sub_array& operator=<ConstMultiArray>(const ConstMultiArray& other); // copy-assign, `std::copy`, assert same shape
  // non-const API:
  element* origin();
  element& operator()<IndexList> (IndexList indices) { return access_element(...); }
  reference operator[] (index idx) { return access(...); }
  array_view<NDims>::type operator[] <NDims> (index_gen<NumDims,NDims> const& indices) { return generate_array_view(...); }
  iterator begin(); iterator end();
  reverse_iterator rbegin(); reverse_iterator rend();
};

template <typename T, std::size_t NumDims, typename TPtr = const T*>
class detail::const_multi_array_view : public multi_array_impl_base<T,NumDims> {
  // disable copy-assign
protected:
  using size_list = array<size_type, NumDims>;
  using index_list = array<index, NumDims>;
  TPtr        base_;                        // base address, contiguously store num_elements
  size_list   extent_list_;                 // size of each dimension
  index_list  stride_list_;                 // stride of each dimension
  index_list  index_base_list_;             // each dimension can indexed by [base, base+extent)
  index origin_offset_;                     // total offset upon base_ when striding
  size_type num_elements_;                  // size, product of each extends

  ctor<ExtentList,Index>(TPtr base, ExtentList const extents, array<Index,NumDims> const& strides) {
    base_ = base; origin_offset_ = 0;
    index_base_list_ = 0; extent_list_ = extents; stride_list_ = strides; // copy arrays
    num_elements_ = fold(extents, *);
  }

public:
  // types & members from base class: value_type, element, size_type, index, ...
  ctor<OPtr>(const const_multi_array_view<T,NumDims,OPtr>& other); // copy-ctor

  void reindex<BaseList>(BaseList const& values); // copy values to index_base_list_; then calculate_origin_offset()
  void reindex(index value); // copy value to all index_base_list_; then calculate_origin_offset()
  void reshape<SizeList>(SizeList const& extents); // copy extents to extent_list_; then compute_strides() and calculate_origin_offset()

  // accessors
  size_type num_dimensions() const { return NDims; }
  size_type size() const { return extent_list_.front(); } // first dimension's size
  size_type max_size() const { return NDims; } // max possible size to reshape
  bool empty() const { return size() == 0; }
  const size_type* shape() const { return extent_list_.data(); }
  const index* strides() const { return stride_list_.data(); }
  const element* origin() const { return base_ + origin_offset_; }
  size_type num_elements() const { return num_elements_; }
  const index* index_bases() const { return index_base_list.data(); }

  // indexing
  const element& operator()<IndexList> (IndexList indices) const { return access_element(...); }
  const_reference operator[] (index idx) const { return access(...); }
  const_array_view<NDims>::type operator[] <NDims> (index_gen<NumDims,NDims> const& indices) const { return generate_array_view(...); }

  const_iterator begin() const; const_iterator end() const;
  const_reverse_iterator rbegin() const; const_reverse_iterator rend() const;
  // operator ==, !=, <, >, <=, >= based on `std::equal`, `std::lexicographical_compare`
};

template <typename T, std::size_t NumDims>
class detail::multi_array_view : public const_multi_array_view<T,NumDims, T*> {
public:
  // types & members from base class: value_type, element, size_type, index, ...
  // forward ctors to base class ctors
  multi_array_view& operator=<ConstMultiArray>(const ConstMultiArray& other); // copy-assign, `std::copy`, assert same shape
  // non-const API:
  element* origin(); element* data();
  element& operator()<IndexList> (IndexList indices) { return access_element(...); }
  reference operator[] (index idx) { return access(...); }
  array_view<NDims>::type operator[] <NDims> (index_gen<NumDims,NDims> const& indices) { return generate_array_view(...); }
  iterator begin(); iterator end();
  reverse_iterator rbegin(); reverse_iterator rend();
};

using array_view_gen<Array,N>::type = detail::multi_array_view<Array::element,N>;
using const_array_view_gen<Array,N>::type = detail::const_multi_array_view<Array::element,N>;
```

* `sub_array` types serve as reference returned by `[]`. They wraps pointers to shape & indexing data.
* `array_view` types keep a map into a sub-space of the indexing space of some `array` or `array_ref`.

------
#### Algorithm for `multi_array`

```c++
template <typename Array1, typename Array2>
  void copy_array(Array1 & source, Arra2 & dest);
template <class InIter, class Size, class OutIter>
  OutIter copy_n(InIter first, Size count, OutIter result);
```

Recursively copy each dimension from `source` to `dest`.

------
### Dependency

#### Boost.Array

* `<boost/array.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/limits.hpp>`

#### Boost.Core

* `<boost/type.hpp>`
* `<boost/utility/enable_if.hpp>`
* `<boost/core/alloc_construct.hpp>`
* `<boost/core/empty_value.hpp>`

#### Boost.Functional

* `<boost/functional.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_concepts.hpp>`
* `<boost/iterator/iterator_facade.hpp>`
* `<boost/iterator/reverse_iterator.hpp>`

#### Boost.MPL

* `<boost/mpl/if.hpp>`, `<boost/mpl/eval_if.hpp>`
* `<boost/mpl/size_t.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_integral.hpp>`
* `<boost/type_traits.hpp>`

------
### Standard Facilities

Standard library: `<mdspan>` (C++23)
