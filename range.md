# Boost.Range

* lib: `boost/libs/range`
* repo: `boostorg/range`
* commit: `e2b50ef`, 2025-09-19

------
### Range concept

```c++
// Single Pass Range metafunctions
struct range_iterator<T,_=void>;
struct range_iterator<[const]R> {using type=R::<const>_iterator;};
struct range_iterator<[const]Pair>{using type=Pair::first_type;};
struct range_iterator<[const]V*>{using type=<const> V*;};
struct range_value<R> {using type=iterator_value<range_iterator<R>::type>::type;};
struct range_reference<R> {using type=iterator_reference<range_iterator<R>::type>::type;};
struct range_pointer<R> {using type=iterator_pointer<range_iterator<R>::type>::type;};
struct range_category<R> {using type=iterator_category<range_iterator<R>::type>::type;};

// Forward Range metafunctions
struct range_difference<R> {using type=iterator_difference<range_iterator<R>::type>::type;};

// Bidirectional Range metafunctions
struct range_reverse_iterator<R> {using type=reverse_iterator<range_iterator<R>::type>::type;};

struct has_range_iterator<X> : range_mutable_iterator<X> {}; // sfinae
struct has_range_const_iterator<X> : range_const_iterator<X> {}; // sfinae

// Single Pass Range functions
range_iterator<[const] R>::type begin<R>( <const> R& r );
range_iterator<[const] R>::type end<R>( <const> R& r );
bool empty<R>( const R& r );

// Forward Range functions
range_difference<R>::type distance<R>( const R& r );
range_size<R>::type size<R>( const R& r );

// Bidirectional Range functions
range_reverse_iterator<[const]R>::type rbegin<R>( <const> R& r );
range_reverse_iterator<[const]R>::type rend<R>( <const> R& r );

// Special const Range functions
range_iterator<const R>::type const_begin<R>( const R& r );
range_iterator<const R>::type const_end<R>( const R& r );
range_reverse_iterator<const R>::type const_rbegin<R>( const R& r );
range_reverse_iterator<const R>::type const_rend<R>( const R& r );

// String utilities
iterator_range< U/*..*/ > as_literal<R>( <const> R& r ); // U is `Ch*` if R is str or char-array, otherwise `iterator_range<R>`
iterator_range< range_iterator<[const] R>::type > as_array<R>( <const> R& r );
```

------
### Range Adaptors

```c++
struct detail::holder<T>;
struct detail::forwarder<Holder<T>>;

struct detail::skip_iterator<It,Pred,def_pass>;
struct adjacent_filtered_range<P,R,def_pass>;
const auto adaptors::adjacent_filtered = forwarder<adjacent_holder>{};
const auto adaptors::adjacent_filtered_excl = forwarder<adjacent_excl_holder>{};
adjacent_filtered_range<BPred,[const]FwdR,true> adaptors::adjacent_filter<FwdR,BPred>(<const> FwdR& r, BPred filter_pred);

struct adaptors::copied;
CopyableRndR adaptors::copy<CopyableRndR>(const CopyableRndR& r, size_t t, size_t u);

struct filtered_range<P,R>;
const auto adaptors::filtered = forwarder<filter_holder>{};
filtered_range<Pred,[const]SinglePassR,true> adaptors::filter<SinglePassR,Pred>(<const> SinglePassR& r, Pred filter_pred);

struct detail::formatted_holder<Sep,Prefix,Postfix>;
struct formatted_range<It,Sep,Prefix,Postfix>;
formatted_holder<Sep,Prefix,Postfix> formatted<Sep=char,Prefix=char,Postfix=char>(const Sep& sep=',', const Prefix& prefix='{', const Postfix& postfix='}');
formatted_range<range_iterator<const SinglePassR>::type, Sep,Prefix,Postfix>
    adaptors::format<SinglePassR,Sep=char,Prefix=char,Postfix=char>(const SinglePassR& r, const Sep& sep=',', const Prefix& prefix='{', const Postfix& postfix='}');

struct adaptors::indexed;
struct index_value<T,Indexable=ptrdiff_t>;
struct detail::indexed_iterator<It>;
struct indexed_range<SinglePassR>;
indexed_range<[const]SinglePassR> adaptors::index<SinglePassR>(<const> SinglePassR& r, range_difference<SinglePassR>::type index_value=0);

struct indirected_range<R>;
const auto adaptors::indirected = struct detail::indirect_forwarder{};
indirected_range<[const]SinglePassR> adaptors::indirect<SinglePassR>(<const>SinglePassR& r);

struct detail::select_first<M>; struct detail::select_second_{mutable|const}<M>;
struct select_first_range<M>; struct select_second_{mutable|const}_range<M>;
const auto adaptors::map_keys = struct detail::map_keys_forwarder{};
const auto adaptors::map_values = struct detail::map_values_forwarder{};
select_first_range<StdPairR> adaptors::keys<StdPairR>(const StdPairR& r);
select_second_const_range<StdPairR> adaptors::values<StdPairR>(const StdPairR& r);
select_second_mutable_range<StdPairR> adaptors::values<StdPairR>(StdPairR& r);

struct detail::unwrap_ref<SinglePassR>;
struct unwrap_ref_range<SinglePassR>;
const auto adaptors::ref_unwrapped = struct detail::ref_unwrapped_forwarder{};
unwrap_ref_range<SinglePassR> adaptors::ref_unwrap<SinglePassR>(const SinglePassR& r);

struct detail::replace_value<V>;
struct replaced_range<R>;
struct detail::replace_holder<T>;
const auto replaced = detail::forwarder<replace_holder>{};
replaced_range<[const] SinglePassR> adaptors::replace<SinglePassR,V>(<const> SinglePassR& r, V from, V to);

struct detail::replace_value_if<Pred,V>;
struct replaced_if_range<Pred,R>;
struct detail::replace_if_holder<Pred,T>;
const auto replaced_if = detail::forwarder<replace_if_holder>{};
replaced_if_range<Pred,[const] SinglePassR> adaptors::replace_if<Pred,SinglePassR,V>(<const> SinglePassR& r, Pred pred, V to);

struct reversed_range<R>;
const auto reversed = struct detail::reverse_forwarder{};
reversed_range<[const] BiR> adaptors::reverse<BiR>(<const> BiR& r);

struct adaptors::sliced{ size_t t, u; };
struct sliced_range<RndR>;
struct sliced_range<RndR> adaptors::slice<RndR>(RndR& r, size_t t, size_t u);
struct iterator_range<range_iterator<const RndR>::type> adaptors::slice<RndR>(const RndR& r, size_t t, size_t u);

struct detail::strided_iterator<BaseIt,Cat>; // spec for bidi, ra
struct strided_iterator<range_iterator<[const]R>::type, forward_traversal_tag> detail::make_{begin|end}_strided_iterator<R,Diff>(<const> R& r, Diff stride, forward_traversal_tag);
struct strided_iterator<range_iterator<[const]R>::type, bidirectional_traversal_tag> detail::make_{begin|end}_strided_iterator<R,Diff>(<const> R& r, Diff stride, bidirectional_traversal_tag);
struct strided_iterator<range_iterator<[const]R>::type, random_access_traversal_tag> detail::make_{begin|end}_strided_iterator<R,Diff>(<const> R& r, Diff stride, random_access_traversal_tag);
struct strided_range<R,Cat=pure_iterator_traversal<range_iterator<R>::type>::type>;
struct detail::strided_holder<Diff>;
const auto strided = detail::forwarder<strided_holder>{};
strided_range<[const] R> adaptors::stride<R,Diff>(<const> R& r, Diff step);

struct tokenized_range<R>;
struct detail::regex_holder<T,U,V>;
struct detail::regex_forwarder;
const auto tokenized = struct detail::regex_forwarder{};
tokenized_range<[const] BiR> adaptors::tokenize<BiR,Regex,Submatch,Flag>(<const> BiR& r, const Regex& re, const Submatch& sub, Flag f);

struct detail::transform_iterator_gen<P,It>;
struct transformed_range<F,R>;
struct detail::transform_holder<T>;
const auto transformed = detail::forwarder<transform_holder>{};
transformed_range<UFunc,[const] SinglePassR> adaptors::transform<UFunc,SinglePassR>(<const> SinglePassR& r, UFunc fn);

struct adaptors::type_erased<V=use_default,Traversal=use_default,Ref=use_default,Diff=use_default,Buf=use_default>{};
any_range_type_generator<[const]SinglePassR,V,T,R,D,B> adaptors::type_erase<SinglePassR,V,T,R,D,B>(<const>SinglePassR& r, type_erased<V,T,R,D,B> ={});

struct detail::unique_not_equal_to;
struct uniqued_range<FwdR>;
const auto uniqued = struct detail::unique_forwarder{};
uniqued_range<[const] FwdR> adaptors::unique<FwdR>(<const> FwdR& r);
```

------
### Range Algorithms

#### Mutating Algorithms

```c++
OutIt copy<SPR,OutIt>(const SPR& r, OutIt out);
BidiWriteIt copy_backward<BidiR,BidiWriteIt>(const BidiR& r, BidiWriteIt out);
<const> FwdR& fill<FwdR,V>(<const> FwdR& r, const V& v);
<const> FwdR& fill_n<FwdR,Size,V>(<const> FwdR& r, Size n, const V& v);
<const> FwdR& generate<FwdR,Gen>(<const> FwdR& r, Gen gen);
<const> BidiR& inplace_merge<BidiR,[BPred]>(<const> BidiR& r, range_iterator<<const> BidiR>::type middle, <BPred pred>);
OutIt merge<SPR1,SPR2,OutIt,[BPred]>(const SPR1& r1, const SPR2& r2, OutIt out, <BPred pred>);
<const> RndR& nth_element<RndR,[BPred]>(<const> RndR& r, range_iterator<RndR>::type nth, <BPred sort_pred>);
<const> RndR& partial_sort<RndR,[BPred]>(<const> RndR& r, range_iterator<RndR>::type middle, <BPred sort_pred>);
range_iterator<<const> RndR>::type partial_sort_copy<SPR,RndR,[BPred]>(const SPR& r1, <const> RndR& r2, <BPred sort_pred>);
range_iterator<FwdR>::type partition<FwdR,UPred>(<const> FwdR& r, UPred pred);
range_return<FwdR,re>::type partition<range_return_value re,FwdR,UPred>(<const> FwdR& r, UPred pred);
<const> RndR& random_shuffle<RndR,[Gen]>(<const> RndR& r, <Gen& gen>);
range_iterator<<const> FwdR>::type remove<FwdR,V>(<const> FwdR& r, const V& v);
range_return<FwdR,re>::type remove<range_return_value re,FwdR,V>(<const> FwdR& r, const V& v);
range_iterator<<const> FwdR>::type remove_if<FwdR,UPred>(<const> FwdR& r, UPred pred);
range_return<FwdR,re>::type remove_if<range_return_value re,FwdR,UPred>(<const> FwdR& r, UPred pred);
OutIt remove_copy<SPR,OutIt,V>(const SPR& r, OutIt out, const V& v);
OutIt remove_copy_if<SPR,OutIt,Pred>(const SPR& r, OutIt out, Pred pred);
<const> FwdR& replace<FwdR,V>(<const> FwdR& r, const V& what, const V& with_what);
<const> FwdR& replace_if<FwdR,UPred,V>(<const> FwdR& r, UPred pred, const V& v);
OutIt replace_copy<FwdR,OutIt,V>(const FwdR& r, OutIt out, const V& what, const V& with_what);
OutIt replace_copy_if<FwdR,OutIt,Pred,V>(const FwdR& r, OutIt out, Pred pred, const V& v);
<const> BidiR& reverse<BidiR>(<const> BidiR& r);
OutIt reverse_copy<BidiR,OutIt>(const BidiR& r, OutIt out);
<const> FwdR& rotate<FwdR>(<const> FwdR& r, range_iterator<<const> FwdR>::type middle);
OutIt rotate_copy<FwdR,OutIt>(const FwdR& r, range_iterator<const FwdR>::type middle, OutIt target);
<const> RndR& sort<RndR,[BPred]>(<const> RndR& r, <BPred pred>);
range_iterator<BidiR>::type stable_partition<BidiR,UPred>(<const> BidiR& r, UPred pred);
range_return<BidiR,re>::type stable_partition<range_return_value re,BidiR,UPred>(<const> BidiR& r, UPred pred);
<const> RndR& stable_sort<RndR,[BPred]>(<const> RndR& r, <BPred pred>);
<const> SPR2& swap_ranges<SPR1,SPR2>(<const> SPR1& r1, <const> SPR2& r2); // 2x2
OutIt transform<SPR1,[SPR2],OutIt,UOp>(const SPR1& r1, <const SPR2& r2>, OutIt out, UOp fun);
range_return<FwdR,re>::type unique<range_return_value re=return_begin_found,FwdR,[BPred]>(<const> FwdR& r, <BPred pred>);
OutIt unique_copy<SPR,OutIt,[BPred]>(const SPR& r, OutIt out, <BPred pred>);
```

#### Non-mutating Algorithms

```c++
range_iterator<<const>FwdR>::type adjacent_find<FwdR,[BPred]>(<const> FwdR& r, <BPred pred>);
range_return<<const>FwdR,re>::type adjacent_find<range_return_value re,FwdR,[BPred]>(<const> FwdR& r, <BPred pred>);
bool binary_search<FwdR,V,[BPred]>(const FwdR& r, const V& v, <BPred pred>);
range_difference<<const> SPR>::type count<SPR,V>(<const> SPR& r, const V& v);
range_difference<<const> SPR>::type count_if<SPR,UPred>(<const> SPR& r, UPred pred);
bool equal<SPR1,SPR2,[BPred]>(const SPR1& r1, const SPR2& r2, <BPred pred>);
std::pair<<const>range_iterator<FwdR>::type,<const>range_iterator<FwdR>::type>
    equal_range<FwdR,V,[Pred]>(<const> FwdR& r, const V& v, <Pred pred>);
range_iterator<<const>SPR>::type find<SPR,V>(<const> SPR& r, const V& v);
range_return<<const>SPR,re>::type find<range_return_value re,SPR,V>(<const> SPR& r, const V& v);
range_iterator<<const> FwdR1>::type find_end<FwdR1,FwdR2,[BPred]>(<const> FwdR1& r1, const FwdR2& r2, <BPred pred>);
range_return<<const> FwdR1,re>::type find_end<range_return_value re,FwdR1,FwdR2,[BPred]>(<const> FwdR1& r1, const FwdR2& r2, <BPred pred>);
range_iterator<<const>SPR1>::type find_first_of<SPR1,FwdR2,[BPred]>(<const> SPR1& r1, const FwdR2& r2, <BPred pred>);
range_return<<const>SPR1,re>::type find_first_of<range_return_value re,SPR1,FwdR2,[BPred]>(<const> SPR1& r1, const FwdR2& r2, <BPred pred>);
range_iterator<<const>SPR>::type find_if<SPR,UPred>(<const> SPR& r, UPred pred);
range_return<<const>SPR,re>::type find_if<range_return_value re,SPR,UPred>(<const> SPR& r, UPred pred);
UFunc for_each<SPR,UFunc>(<const> SPR& r, UFunc fun);
bool lexicographical_compare<SPR1,SPR2,[BPred]>(const SPR1& r1, const SPR2& r2, <BPred pred>);
range_iterator<<const>FwdR>::type lower_bound<FwdR,V,[Pred]>(<const> FwdR& r, const V& v, <Pred pred>);
range_return<<const>FwdR,re>::type lower_bound<range_return_value re,FwdR,V,[Pred]>(<const> FwdR& r, const V& v, <Pred pred>);
range_iterator<<const>FwdR>::type max_element<FwdR,[BPred]>(<const> FwdR& r, <BPred pred>);
range_return<<const>FwdR,re>::type max_element<range_return_value re,FwdR,[BPred]>(<const> FwdR& r, <BPred pred>);
range_iterator<<const>FwdR>::type min_element<FwdR,[BPred]>(<const> FwdR& r, <BPred pred>);
range_return<<const>FwdR,re>::type min_element<range_return_value re,FwdR,[BPred]>(<const> FwdR& r, <BPred pred>);
std::pair<range_iterator<<const>SPR1>::type,range_iterator<<const>SPR2>::type> // 2x2
    mismatch<SPR1,SPR2,[BPred]>(<const> SPR1& r1, <const> SPR2& r2, <BPred pred>);
range_iterator<<const>FwdR1>::type search<FwdR1,FwdR2,[BPred]>(<const> FwdR1& r1, const FwdR2& r2, <BPred pred>);
range_return<<const> FwdR1,re>::type search<range_return_value re,FwdR1,FwdR2,[BPred]>(<const> FwdR1& r1, const FwdR2& r2, <BPred pred>);
range_iterator<<const>FwdR>::type search_n<FwdR,Int,V,[BPred]>(<const> FwdR& r, Int count, const V& v, <BPred pred>);
range_return<<const>FwdR,re>::type search_n<range_return_value re,FwdR,Int,V,[BPred]>(<const> FwdR& r, Int count, const V& v, <BPred pred>);
range_iterator<<const>FwdR>::type upper_bound<FwdR,V,[Pred]>(<const> FwdR& r, const V& v, <Pred pred>);
range_return<<const>FwdR,re>::type upper_bound<range_return_value re,FwdR,V,[Pred]>(<const> FwdR& r, const V& v, <Pred pred>);
```

#### Set Algorithms

```c++
bool includes<SPR1,SPR2,[BPred]>(const SPR1& r1, const SPR2& r2, <BPred pred>);
OutIt set_union<SPR1,SPR2,OutIt,[BPred]>(const SPR1& r1, const SPR2& r2, OutIt out, <BPred pred>);
OutIt set_intersection<SPR1,SPR2,OutIt,[BPred]>(const SPR1& r1, const SPR2& r2, OutIt out, <BPred pred>);
OutIt set_difference<SPR1,SPR2,OutIt,[BPred]>(const SPR1& r1, const SPR2& r2, OutIt out, <BPred pred>);
OutIt set_symmetric_difference<SPR1,SPR2,OutIt,[BPred]>(const SPR1& r1, const SPR2& r2, OutIt out, <BPred pred>);
```

#### Heap Algorithms

```c++
<const> RndR& push_heap<RndR,[Comp]>(<const> RndR& r, <Comp comp_pred>);
<const> RndR& pop_heap<RndR,[Comp]>(<const> RndR& r, <Comp comp_pred>);
<const> RndR& make_heap<RndR,[Comp]>(<const> RndR& r, <Comp comp_pred>);
<const> RndR& sort_heap<RndR,[Comp]>(<const> RndR& r, <Comp comp_pred>);
```

#### Permutation Algorithms

```c++
bool next_permutation<BidiR,[Comp]>(<const> BidiR& r, <Comp comp_pred>);
bool prev_permutation<BidiR,[Comp]>(<const> BidiR& r, <Comp comp_pred>);
```

#### New Algorithms

```c++
OutIt copy_n<SPR,Size,OutIt>(const SPR& r, Size n, OutIt out);
Container& erase<Container>(Contaner& on, iterator_range<Container::iterator> to_erase);
Container& remove_erase<Container,T>(Contaner& on, const T& v);
Container& remove_erase_if<Container,T>(Contaner& on, Pred pred);
Container& unique_erase<Container,[Pred]>(Container& on, <Pred pred>);
Fn2 for_each<SPR1,SPR2,Fn2>(<const> SPR1& r1, <const> SPR2& r2, Fn2 fn); // 2x2
Container& insert<Container,Range>(Container& on, <Container::iterator before>, const Range& from);
<const> FwdR& iota<FwdR,V>(<const> FwdR& r, V x);
bool is_sorted<SPR,[BPred]>(const SPR& r, <BPred pred>);
void overwrite<SPR1,SPR2>(const SPR1& from, <const> SPR2& to);
Container& push_back<Container,Range>(Container& on, const Range& from);
Container& push_front<Container,Range>(Container& on, const Range& from);
```

#### Numeric Algorithms

```c++
V accumulate<SPR,V,[BOp]>(const SPR& r, V init, [BOp op]);
V inner_product<SPR1,SPR2,V,[BOp1,BOp2]>(const SPR1& r1, const SPR2& r2, V init, <BOp1 op1, BOp2 op2>);
OutIt partial_sum<SPR,OutIt,[BOp]>(const SPR& r, OutIt result, <BOp op>);
OutIt adjacent_difference<SPR,OutIt,[BOp]>(const SPR& r, OutIt result, <BOp op>);
```

------
### Dependency

#### Boost.Array

* `<boost/array.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`, `<boost/config/workaround.hpp>`
* `<boost/config/header_deprecated.hpp>`
* `<boost/cstdint.hpp>`

#### Boost.ContainerHash

* `<boost/functional/hash.hpp>`

#### Boost.Conversion

* `<boost/polymorphic_cast.hpp>`

#### Boost.Core

* `<boost/core/addressof.hpp>`
* `<boost/core/enable_if.hpp>`
* `<boost/core/noncopyable.hpp>`
* `<boost/core/ref.hpp>`

#### Boost.Detail

* `<boost/detail/is_sorted.hpp>`

#### Boost.Iterator

* `<boost/iterator/counting_iterator.hpp>`
* `<boost/iterator/distance.hpp>`
* `<boost/iterator/enable_if_convertible.hpp>`
* `<boost/iterator/filter_iterator.hpp>`
* `<boost/iterator/indirect_iterator.hpp>`
* `<boost/iterator/iterator_adaptor.hpp>`
* `<boost/iterator/iterator_categories.hpp>`
* `<boost/iterator/iterator_concepts.hpp>`
* `<boost/iterator/iterator_facade.hpp>`
* `<boost/iterator/iterator_traits.hpp>`
* `<boost/iterator/reverse_iterator.hpp>`
* `<boost/iterator/transform_iterator.hpp>`
* `<boost/iterator/zip_iterator.hpp>`
* `<boost/next_prior.hpp>`
* `<boost/pointee.hpp>`

#### Boost.MPL

* `<boost/mpl/*.hpp>`

#### Boost.Optional

* `<boost/optional/optional.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.Regex

* `<boost/regex.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Tuple

* `<boost/tuple/tuple.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits.hpp>`
* `<boost/type_traits/*.hpp>`

#### Boost.Utility

* `<boost/utility/result_of.hpp>`
* `<boost/utility.hpp>`

------
### Standard Facilities
