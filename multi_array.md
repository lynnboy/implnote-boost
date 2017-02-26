# Boost.MultiArray

* lib: `boost/libs/multi_array`
* repo: `boostorg/multi_array`
* commit: `83c37385`, 2016-10-07

------
### Multi-dimensional Array

Header `<boost/multi_array.hpp>`

------
#### Common Types

```c++
using size_type = std::size_t; using difference_type = std::ptr_diff_t; using index = std::ptr_diff_t;

struct index_range<Index,SizeType> {
  index start_, finish_, stride_; // default to std::numeric_limits<index> min, max, 1
  bool degenerate_; // default false

  index_range(); // default, [min,max)/1,false
  explicit index_range(index pos); // [pos,pos+1)/1,true
  index_range(index start, index finish, index stride=1); // [start,finish)/stride,false
  // chaining:
  index_range& start|finish|stride (index);
  // accessors for start, finish, stride, and is_degenerate

  void set_index_range(index start, index finish, index stride=1);
  static index_range all(); // [min,max)/1
  
  // shifting operator: '- offset' and '+ offset'
  // stride calculating: '[i]' and '(i)'
  // range splicing: `s <= r`, `s < r`, `r < f`, `r <= f`
};
struct extent_range<Extent,SizeType> : std::pair<Extent,Extent> {
  extent_range(index start, index finish); extent_range(index finish); extent_range();
  index start() const; index finish() const; size_type size() const;
};

using index_range = index_range<index,size_type>;
using extent_range = extent_range<index,size_type>;

struct index_gen<NumRanges,NumDims> {
  array<index_range, max(NumRanges,1)> ranges_; // accumulated list by '[]'
  index_gen();

  index_gen<ND>(const index_gen<NumRanges-1,ND>&, const index_range&); // don't care dimension, append
  index_gen<NumRanges+1,NumDims+1> operator[](const index_range&) const; // add one sub-dimension's range
  index_gen<NumRanges+1,NumDims>   operator[](index) const; // add one 'degenerated' range, index fixed
};
struct extent_gen<NumRanges> {
  array<extent_range, max(NumRanges,1)> ranges_; // accumulated list by '[]'
  extent_gen();

  extent_gen(const extent_gen<NumRanges-1>&, const extent_range&); // append a range
  extent_gen<NumRanges+1> operator[](const extent_range&) const; // add one sub-dimension's range
  extent_gen<NumRanges+1> operator[](index) const; // add one range, start from 0
};

index_gen<0,0> indices;  // global
extent_gen<0> extents; // global

class general_storage_order<NumDims> {
  array<size_type, NumDims> ordering_;
  array<bool, NumDims> ascending_;
};
class c_storage_order; // ordering = [ N-1, ..., 0 ], all_dims_ascending = true
class fortran_storage_order; // ordering = [ 0, ..., N-1 ], all_dims_ascending = true

struct multi_array_base<T, size_t NumDims>;
```

* `index_range` use `min` and `max` for infinit mark (all available value) values
* Making a `index_range` can use syntax `index_range().start(s).finish(f).stride(s)`, either start or finish required
* Use `get_start` and `get_finish` on `index_range` to handle infinit markers, `all` gives all index range
* `index_gen` is used as `indices[r0][r1]...`, and similar with `extends` for `extent_gen`

------
#### Indexing Structure

```c++
class value_accessor_one<T> {
  typedef T element; typedef T value_type; typedef T& reference; typedef T const& const_reference;
  Ref access<Ref,TPtr>(type<Ref>, index idx, TPtr base,
    const size_type* extents, const index* strides, const index* index_bases) const {
      return *(base+idx*strides[0]);  // base already shifted
  }
};
class value_accessor_n<T, NumDims> {
  typedef T element;
  typedef multi_array<T,NumDims-1> value_type; // sub-dimension
  typedef [const_]sub_array<T,NumDims-1> [const_]reference;

  Ref access<Ref,TPtr>(type<Ref>, index idx, TPtr base,
    const size_type* extents, const index* strides, const index* index_bases) const {
      TPtr newbase = base+idx*strides[0];  // base already shifted
      return Ref(newbase, extents+1, strides+1, index_bases+1); // sub_array into next dimension
  }
};

template <typename T, std::size_t NumDims, typename TPtr = const T*>
class [const_]multi_array_ref {
  using size_list = array<size_type, NumDims>;
  using index_list = array<index, NumDims>;

  TPtr        base_;                        // base address, contiguously store num_elements
  general_storage_order<NumDims> storage_;  // array indexing order
  size_list   extent_list_;                 // size of each dimension
  index_list  index_base_list_;             // each dimension can indexed by [base, base+extent)

  index_list  stride_list_;                 // stride of each dimension
  index origin_offset_;                     // total offset upon base_ when striding
  index directional_offset_;                // offset caused by descending index
  size_type num_elements;                   // size, product of each extends

  void init_multi_array_ref(InputIterator extents_iter) {
    // copy extents_iter into extent_list
    // calcuate num_elements_ = accumulate of extent_list by multiply
    // compute stride_list_, start from outermost dimension, obeying storage_
    directional_offset_ = calculate_descending_dimension_offset(stride_list, extent_list, storage_);
    origin_offset_ = directional_offset + calculate_indexing_offset(stride_list_, index_base_list_);
  }
  void init_from_extent_gen(const extent_gen<NumDims>& ranges) {
    // fill index_base_list_ from each extent_range.start()
    extent_list extents;
    // fill extents from each extent_range.size()
    init_multi_array_ref(extents.begin());
  }
  index calculate_descending_dimension_offset(const& stride_list, const& extent_list, const& storage) {
    index offset = 0;
    for (size_type n = 0; n != NumDims; ++n)
      if (!storage.ascending(n)) offset -= (extent_list[n] - 1) * stride_list[n];
    return offset;
  }
  index calculate_indexing_offset(const& stride_list, const& index_base_list) {
    index offset = 0;
    for (size_type n = 0; n != NumDims; ++n)
        offset -= stride_list[n] * index_base_list[n];  // offset caused by basing, thus '-='
    return offset;
  }
  
  Ref access_element<Ref,IndexList,TPtr>(type<Ref>, const IndexList& indices, TPtr base,
    const size_type* extents, const index* strides, const index* index_bases) const {
      index offset = inner_product(indices, strides, 0);
      return base[offset];  // access the element specified by 'indices'
  }
  Ref access<Ref,TPtr>(type<Ref>, index idx, TPtr base,
    const size_type* extents, const index* strides, const index* index_bases) const {
      using accessor = NumDims > 1 ? choose_value_accessor_n<T,NumDims> : choose_value_accessor_one<T>;
      return accessor::access(type<Ref>(), idx, base, extents, strides, index_bases);
  }
  ArrayRef generate_array_view<ArrayRef,NDims,TPtr>(type<ArrayRef>, // [const_]multi_array_view
    index_gen<NumDims,NDims> const & indices,         // slicing index ranges for each dimension
    const size_type* extents, const index* strides, const index* index_bases) const {
      index offset = 0; size_type dim = 0;
      array<index,NDims> new_strides, new_extents;    // degenerated dimensions are skipped
      for (size_type n = 0; n != NumDims; ++n) {      // for each dimension
        auto range = indices.ranges_[n];
        // start, finish, stride from range, limited at boundary of [index_bases[n],+extents[n])
        index len = ...; // length of dimension, (finish - start)/stride
        offset += start * strides[n];     // offset caused by slicing, thus '+='
        if (!range.is_degenerate()) {     // degenerated dimensions are skipped
          new_strides[dim] = stride * strides[n]; new_extents[dim] = len; ++dim;
        }
      }
      return ArrayRef(base+offset, new_extents, new_strides);
  }
};

template <typename T, std::size_t NumDims, typename TPtr>
class [const_]multi_array_view {
  TPtr        base_;                        // base address, contiguously store num_elements
  size_list   extent_list_;                 // size of each dimension
  index_list  index_base_list_;             // each dimension can indexed by [base, base+extent)
  index_list  stride_list_;                 // stride of each dimension

  index       origin_offset_;               // total offset upon base_ when striding
  size_type   num_elements;                 // size, product of each extends
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
class multi_array : public multi_array_ref<T,NumDims> {
  Allocator allocator_;
  T* base_;   size_type allocated_elements;             // allocated buffer

  allocate_space();   // called at every ctor, allocate, then uninitialized_fill_n with T()
  deallocate_space(); // called at dtor, destroy each element, deallocate
public:
  multi_array();
  multi_array<ExtentList>(ExtentList const&,[general_storage_order<NumDims> const&],[Allocator const&]);
  multi_array(extent_gen<NumDims> const&,   [general_storage_order<NumDims> const&],[Allocator const&]);
  multi_array([const_]multi_array_ref|sub_array|multi_array_view,
      [general_storage_order<NumDims> const& = c_storage_order()]);     // create copy from other array

  multi_array& resize<ExtentList>(const ExtentList&);
  multi_array& resize(const extent_gen<NumDims>&);    // make copy via assigning same-shaped views, then swap
};

template <typename T, std::size_t NumDims, typename TPtr>
class [const_]sub_array {
  TPtr              base_;        // base address, contiguously store num_elements
  const size_type*  extents_;     // size of each dimension
  const index*      index_base_;  // each dimension can indexed by [base, base+extent)
  const index*      strides_;     // stride of each dimension
};

template <typename T, std::size_t NumDims, typename TPtr, typename Ref, typename IteratorCategory>
struct array_iterator : iterator_facade<...> {
  index             idx_;         // current index of array
  TPtr              base_;        // base address, contiguously store num_elements
  const size_type*  extents_;     // size of each dimension
  const index*      index_base_;  // each dimension can indexed by [base, base+extent)
  const index*      strides_;     // stride of each dimension

  // ctor, copy-ctor
  bool equal<Iter>(Iter& rhs) const; // idx_, base_, extents_, index_base_, strides_ contents all equal
  Ref dereference() const; // access()
  void increment() { ++idx_; } void decrement() { --idx_; }
  void advance<DiffType>(DiffType n) { idx_ += n; } difference_type distance_to<Iter>(Iter&) const;
};

template <typename T, std::size_t NumDims, typename TPtr>
class <[array_type]> {    // multi_array, multi_array_ref, multi_array_view, sub_array
public:
  // typedefs
  // ctor, copy-ctor, op= (member-wise, same shape)
  [const_]reference operator[](index idx) [const];    // call 'access' to get 'sub_array' or 'T&'
  [const] element& operator()(const IndexList& indices) [const] // call 'access_element' to get 'T&'
  [const_]multi_array_view<NDims> operator[]
        (index_gen<NumDims,NDims>& indices) [const];  // slicing, call generate_array_view()
  bool operator< (const &) const;     // lexicographical_compare
  bool operator==(const &) const;     // element-wise equal
  // '!=', '>', '<=', '>='
  // begin, end, rbegin, rend
  
  TPtr              origin() const;         // offsetted base, directly index-able
  size_type         size() const;           // outermost dimension's extent
  size_type         max_size() const;       // max possible size after reshape, = num_elements
  size_type         num_elements() const;
  bool              empty() const;
  size_type         num_dimensions() const;
  const size_type*  shape() const;          // extents_
  const index*      strides() const;
  const index*      index_bases() const;
  
  void reindex<BaseList>(const BaseList&);  // reset each dimension's indexing base
  void reindex(index value);                // reset each dimension's indexing base to value
  void reshape<SizeList>(const SizeList& extents);  // change array shape, must hold num_elements invariant
  void assign<InputIter>(InputIter, InputIter);     // element-wise assignment
  const element* data() const;              // data base address, not offseted.
};
```

* `multi_aray_view`s can only be constructed by slicing.
* `sub_array`s can be constructed by subscripting and iterator dereferencing.
* `multi_array`, `multi_array_ref`, and `multi_array_view` can `reindex()`
* `multi_array` and `multi_array_ref` provides `reshape`, `data` and `assign`
* `multi_array` can `resize` to different `num_elements`

------
#### Algorithm for `multi_array`

```c++
template <typename Array1, typename Array2>
  void copy_array(Array1 & source, Arra2 & dest);
```

Recursively copy each dimension from `source` to `dest`.

------
#### Concepts

* `ConstMultiArrayConcept<Array,NumDims>`
* `MutableMultiArrayConcept<Array,NumDims>`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Core

* `<boost/type.hpp>`
* `<boost/utility/enable_if.hpp>`
* `<boost/iterator.hpp>` - deprecated

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_concepts.hpp>`
* `<boost/iterator/iterator_facade.hpp>`
* `<boost/iterator/reverse_iterator.hpp>`

#### Boost.Array

* `<boost/array.hpp>`

#### Boost.MPL

* `<boost/mpl/if.hpp>`, `<boost/mpl/eval_if.hpp>`
* `<boost/mpl/size_t.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_integral.hpp>`
* `<boost/type_traits.hpp>`

#### Boost.Functional

* `<boost/functional.hpp>`

------
### Standard Facilities

* Proposals:
  * N4355 - Shared Multidimensional Arrays with Polymorphic Layout
  * N4512 - Multidimensional bounds, offset and array_view, revision 7, Fundamentals TS V2
  * P0009r00 - Polymorphic Multidimensional Array View
  * P0122R0 - `array_view` : bounds-safe views for sequences of objects
