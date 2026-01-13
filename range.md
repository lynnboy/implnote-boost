# Boost.Range

* lib: `boost/libs/range`
* repo: `boostorg/range`
* commit: `e2b50ef`, 2025-09-19

------
### Range concept

```c++
concept IncrementableIterator<It> = CopyConstructible<It> && requires (It i) {
    requires Convertible<It::traversal_category,incrementable_traversal_tag>; ++i; i++;
};
concept SinglePassIterator<It> = IncrementableIterator<It> && EqualityComparable<It> && requires (It i) {
    requires Convertible<It::traversal_category,single_pass_traversal_tag>;
    {++i}->convertible_to<It>;
    {*i}->convertible_to<std::iterator_traits<It>::reference>;
    {*(++i)}->convertible_to<std::iterator_traits<It>::reference>;
};
concept ForwardIterator<It> = SinglePassIterator<It> && DefaultConstructible<It> && requires (It i) {
    using difference_type = std::iterator_traits<It>::difference_type;
    requires is_integral_v<difference_type> && std::numeric_limits<difference_type>::is_signed &&
        Convertible<It::traversal_category,forward_traversal_tag>;
    {i++}->convertible_to<It>;
    {*(i++)}->convertible_to<std::iterator_traits<It>::reference>;
};
concept BidirectionalIterator<It> = ForwardIterator<It> && requires (It i) {
    requires Convertible<It::traversal_category,bidirectional_traversal_tag>; --i; i--;
};
concept RandomAccessIterator<It> = BidirectionalIterator<It> && requires (It i, It j, difference_type n) {
    requires Convertible<It::traversal_category,random_access_traversal_tag>;
    i += n; i = i + n; i = n + i;
    i -= n; i = i - n; n = i - j;
};
concept SinglePassRange<T,Rng=remove_reference<T>::type> = requires (Rng* r, Rng const& cr) {
    using <const>_iterator = range_iterator<Rng <const>>::type;
    requires SinglePassIterator<iterator> && SinglePassIterator<const_iterator>;
    {begin(*r)}->convertible_to<iterator>; {end(*r)}->convertible_to<iterator>;
    {begin(cr)}->convertible_to<const_iterator>; {end(cr)}->convertible_to<const_iterator>;
};
concept ForwardRange<T> = SinglePassRange<T> && ForwardIterator<<const>_iterator>;
concept WriteableRange<T> = requires (range_iterator<T>::type i, range_value<T>::type v) { *i=v; };
concept WriteableForwardRange<T> = ForwardRange<T> && WriteableRange<T>;
concept BidirectionalRange<T> = ForwardRange<T> && BidirectionalIterator<<const>_iterator>;
concept WriteableBidirectionalRange<T> = BidirectionalRange<T> && WriteableRange<T>;
concept RandomAccessRange<T> = BidirectionalRange<T> && RandomAccessIterator<<const>_iterator>;
concept WriteableRandomAccessRange<T> = RandomAccessRange<T> && WriteableRange<T>;

// Single Pass Range metafunctions
struct range_iterator<T,_=void>
    : if_c<is_const_v<remove_reference_t<T>>, range_const_iterator<remove_const_t<remove_reference_t<T>>>,
                                              range_mutable_iterator<remove_reference_t<T>>>{};
struct range_iterator<[const]R> {using type=R::<const>_iterator;};
struct range_iterator<[const]Pair>{using type=Pair::first_type;};
struct range_iterator<[const]V*>{using type=<const> V*;};
struct range_mutable_iterator<T,_=void>;
struct range_const_iterator<T,_=void>;
struct range_value<R> {using type=iterator_value<range_iterator<R>::type>::type;};
struct range_reference<R> {using type=iterator_reference<range_iterator<R>::type>::type;};
struct range_pointer<R> {using type=iterator_pointer<range_iterator<R>::type>::type;};
struct range_category<R> {using type=iterator_category<range_iterator<R>::type>::type;};

// Forward Range metafunctions
struct range_difference<R> {using type=iterator_difference<range_iterator<R>::type>::type;};

// Bidirectional Range metafunctions
struct range_reverse_iterator<R> {using type=reverse_iterator<range_iterator<R>::type>::type;};
struct range_const_reverse_iterator<R> : range_reverse_iterator<const remove_reference_t<R>> {};
struct range_size<R>;

struct has_range_iterator<X> : range_mutable_iterator<X> {}; // sfinae
struct has_range_const_iterator<X> : range_const_iterator<X> {}; // sfinae

// Other
struct range_traversal<SPR> : iterator_traversal<range_iterator<SPR>::type> {};
struct range_result_iterator<C> : range_iterator<C>{};
struct range_reverse_result_iterator<C> : range_reverse_iterator<C>{};

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

#### Commons

```c++
struct collection_traits<T> {
    using function_type = ...;
    using value_type = function_type::value_type; // and size_type, <const>_iterator, result_iterator, difference_type
};
struct value_type_of<C>;
struct difference_type_of<C>;
struct iterator_of<C>;
struct const_iterator_of<C>;
struct result_iterator_of<C>;
collection_traits<C>::size_type size<C>(const C& c);
bool empty<C>(const C& c);
collection_traits<C>::<const>_iterator {begin|end}<C>(<const> C& c);

enum range_return_value
{
    // (*) indicates the most common values
    return_found,       // only the found resulting iterator (*)
    return_next,        // next(found) iterator
    return_prior,       // prior(found) iterator
    return_begin_found, // [begin, found) range (*)
    return_begin_next,  // [begin, next(found)) range
    return_begin_prior, // [begin, prior(found)) range
    return_found_end,   // [found, end) range (*)
    return_next_end,    // [next(found), end) range
    return_prior_end,   // [prior(found), end) range
    return_begin_end    // [begin, end) range
};

struct range_return<SPR,range_return_value> { // spec for each range_return_value for different impl of
    using type = iterator_range<range_iterator<SPR>::type>;
    static type pack(range_iterator<SPR>::type found, SPR& r);
};
```

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
### Provided Ranges

#### `any_range`

```c++
struct any_iterator_buffer<size> : noncopyable {
    ctor(); ~dtor(){delete[]m_ptr;}
    void* allocate(size_t bytes){if (bytes<=size) return m_buffer.data(); m_ptr=new char[bytes]; return m_ptr;}
    void deallocate(){delete[]m_ptr; m_ptr=nullptr;}
private: char* m_ptr{}; array<char,size> m_buffer;
};
struct any_iterator_heap_only_buffer : noncopyable {
    ctor(); ~dtor(){delete[]m_ptr;}
    void* allocate(size_t bytes){m_ptr=new char[bytes]; return m_ptr;}
    void deallocate(){delete[]m_ptr; m_ptr=nullptr;}
private: char* m_ptr{};
};
struct any_iterator_stack_only_buffer<size> {
    void* allocate(size_t bytes) { return m_buffer.data(); }
    void deallocate(){}
private: array<char,size> m_buffer;
};
using any_iterator_default_buffer = any_iterator_buffer<64>;

struct detail::const_reference_type_generator<T>; // is_ref<T> ? const rem_ref<T>& : T
struct detail::reference_as_value_type_generator<T>; // VT=rem_cv<T>; is_convertible<const VT&,T> ? VT : T
struct detail::any_incrementable_iterator_interface<Ref,Buf> {
    using reference=Ref; using const_reference=const_reference_type_generator<T>::type;
    using reference_as_value_type=reference_as_value_type_generator<Ref>::type; using buffer_type=Buf;
    virtual ~dtor(){}
    virtual self* clone(Buf& buf) const=0;
    virtual self<const_reference,Buf>* clone_const_ref(Buf& buf) const=0;
    virtual self<reference_as_value_type,Buf>* clone_reference_as_value(Buf& buf) const=0;
    virtual void increment() =0;
};
struct detail::any_single_pass_iterator_interface<Ref,Buf> : any_incrementable_iterator_interface<Ref,Buf> {
    // all base members, covariance clone_xx return type
    virtual reference dereference() const=0;
    virtual bool equal(const self& other) const=0;
};
struct detail::any_forward_iterator_interface<Ref,Buf> : any_single_pass_iterator_interface<Ref,Buf> {
    // all base members, covariance clone_xx return type
};
struct detail::any_bidirectional_iterator_interface<Ref,Buf> : any_forward_iterator_interface<Ref,Buf> {
    // all base members, covariance clone_xx return type
    virtual void decrement() =0;
};
struct detail::any_random_access_iterator_interface<Ref,Diff,Buf> : any_bidirectional_iterator_interface<Ref,Buf> {
    // all base members, covariance clone_xx return type
    virtual void advance(Diff offset) =0;
    virtual Diff distance_to(const self& other) const =0;
};
struct detail::any_iterator_interface_type_generator<Trav,Ref,Diff,Buf>; // select interfaces by Trav

Tgt& detail::polymorphic_ref_downcast<Tgt,Src>(Src& source);
Ref detail::dereference_cast<Ref,T>(<const> T& x);

struct detail::any_incrementable_iterator_wrapper<It,Ref,Buf> : any_incrementable_iterator_interface<Ref,Buf>;
struct detail::any_single_pass_iterator_wrapper<It,Ref,Buf> : any_single_pass_iterator_interface<Ref,Buf>;
struct detail::any_forward_iterator_wrapper<It,Ref,Buf> : any_forward_iterator_interface<Ref,Buf>;
struct detail::any_bidirectional_iterator_wrapper<It,Ref,Buf> : any_bidirectional_iterator_interface<Ref,Buf>;
struct detail::any_random_access_iterator_wrapper<It,Ref,Diff,Buf> : any_random_access_iterator_interface<Ref,Diff,Buf>;
struct detail::any_iterator_wrapper_type_generator<It,Trav,Ref,Diff,Buf>; // select wrapper by Trav

struct any_iterator<V,Trav,Ref,Diff,Buf=any_iterator_default_buffer> {
private:
    using abstract_base_type = any_iterator_interface_type_generator<Trav,Ref,Diff,Buf>::type;
    using buffer_type = Buf;
    buffer_type m_buffer; abstract_base_type* m_impl;
};
struct detail::is_any_iterator<It>;

struct any_range<V,Traversal,Ref=V&,Diff=ptrdiff_t,Buf=use_default>
    : iterator_range<any_iterator<V,Traversal,Ref,Diff,[Buf or any_iterator_default_buffer]>>{
};
struct any_range_type_generator<R,V=use_default,Trav=use_default,Ref=use_default,Diff=use_default,Buf=use_default>;
```

#### `counting_range`

```c++
iterator_range<counting_iterator<V>> counting_range<V>(V first, V last);
iterator_range<counting_iterator<range_iterator<[const] R>::type>> counting_range<R>(<const> R& r);
```

#### `istream_range`

```c++
iterator_range<std::istream_iterator<T,Ch,Tr>> istream_range<T,Ch,Tr>(std::basic_istream<Ch,Tr>& in);
```

#### `irange`

```c++
struct detail::integer_iterator<Int> : public iterator_facade<self, Int, random_access_traversal_tag, Int, ptrdiff_t> {
    ctor(); explicit ctor(value_type x);
private: void increment(); void decrement();
    void advance(difference_type offset); difference_type distance_to(const integer_iterator& other) const;
    bool equal(const self& other) const; reference dereference() const;
    value_type m_value{};
};
struct detail::integer_iterator_with_step<Int> : public iterator_facade<self, Int, random_access_traversal_tag, Int, ptrdiff_t> {
    ctor(value_type first, difference_type step, value_type step_size);
private: void increment(); void decrement();
    void advance(difference_type offset); difference_type distance_to(const integer_iterator& other) const;
    bool equal(const self& other) const; reference dereference() const;
    value_type m_first; difference_type m_step, m_step_size;
};
struct integer_range<Int> : public iterator_range<integer_iterator<Int>> { ctor(Int first, Int last); };
struct strided_integer_range<Int> : public integer_range<integer_iterator_with_step<Int>> { ctor<It>(It first, It last); };
integer_range<Int> irange<Int>(Int first, Int last);
strided_integer_range<Int> irange<Int,Step>(Int first, Int last, Step step_size);
integer_range<Int> irange<Int>(Int last);
```

------
### Utilities

#### `iterator_range`

```c++
struct detail::range_tag{}; struct detail::const_range_tag{}; struct detail::iterator_range_tag{};
struct detail::pure_iterator_traversal<It>;
struct detail::iterator_range_base<It,Tr> : public iterator_range_tag {
    using value_type = iterator_value<It>::type;
    using difference_type = iterator_difference<It>::type; using size_type = size_t;
    using reference = iterator_reference<It>::type;
    using const_iterator = It; using iterator = It;
protected: ctor(); ctor<It>(It begin, It end);
    void assign<It>(It first, It last); void assign<SPR>(<const> SPR& r);
    It m_begin, m_end;
public: It begin() const; It end() const;
    bool empty() const; explicit operator bool() const; bool operator!() const;
    bool equal(const self& r) const;
    reference front() const; void drop_front(<difference_type n>); void pop_front();
};
struct detail::iterator_range_base<It,bidirectional_traversal_tag>: public self<It,incrementable_traversal_tag> {
    reference back() const; void drop_back(<difference_type n>); void pop_back();
};
struct detail::iterator_range_base<It,random_access_traversal_tag> : public self<It, bidirectional_traversal_tag> {
    using abstract_value_type = if_< or_<is_abstract<value_type>||is_array<value_type>||is_function<value_type>>, reference, value_type>::type;
    reference operator[](difference_type at) const;
    abstract_value_type operator()(difference_type at) const;
    size_type size() const;
};
struct iterator_range<It> : public iterator_range_base<It,pure_iterator_traversal<It>::type> {
    struct is_compatible_range<Src>;
    using type = self;
    ctor(); ctor<It>(It first, It last);
    ctor<SPR>(<const> SPR& r) requires is_compatible_range<SPR>; ctor<SPR>(<const> SPR& r, <const>_range_tag);
    self& operator=<It>(<const> self<It>& o); self& operator=<SPR>(<const> SPR& r);
    self& advance_{begin|end}(difference_type n);
};
bool operator{==|!=|<|<=|>|>=} <It,FwdR> (const FwdR& l, const iterator_range<It>& r);
bool operator{==|!=|<|<=|>|>=} <It,FwdR> (const iterator_range<It>& l, const FwdR& r);
bool operator{==|!=|<|<=|>|>=} <It1,It2> (const iterator_range<It1>& l, const iterator_range<It2>& r);
iterator_range<It> make_iterator_range<It>(It begin, It end);
iterator_range<It> make_iterator_range_n<It,Int>(It first, Int n);
iterator_range<range_iterator<<const> FwdR>::type> make_iterator_range<FwdR>(<const> FwdR& r);
iterator_range<range_iterator<<const> R>::type> make_iterator_range<R>(<const> R& r, range_difference<R>::type begin, range_difference<R>::type end);
Seq copy_range<Seq,R>(const R& r);

size_t hash_value<T>(const iterator_range<T>& r);
std::basic_ostream<Ch,Tr>& operator<< <It,Ch,Tr> (std::basic_ostream<Ch,Tr>& os, const iterator_range<It>& r);
```

#### `sub_range`

```c++
struct detail::sub_range_base<FwdR,TraversalTag> : public iterator_range<range_iterator<FwdR>::type> {
    using value_type = range_value<FwdR>::type;
    using <const>_iterator = range_iterator<[const] FwdR>::type;
    using difference_type = range_difference<FwdR>::type; using size_type = range_size<FwdR>::type;
    using <const>_reference = range_reference<[const] FwdR>::type;
    ctor(); ctor<It>(It first, It last);
    <const>_reference first() <const>;
};
struct detail::sub_range_base<FwdR,bidirectional_traversal_tag> : public sub_range_base<FwdR,forward_traversal_tag> {
    <const>_reference back() <const>;
};
struct detail::sub_range_base<FwdR,random_access_traversal_tag> : public sub_range_base<FwdR,bidirectional_traversal_tag> {
    <const>_reference operator[](difference_type n) <const>;
};
struct sub_range<FwdR> : public sub_range_base<FwdR,iterator_traversal<range_iterator<FwdR>::type>::type> {
    struct is_compatible_range<Src>;
    ctor(); ctor<FwdR2>(<const> FwdR2& r) requires is_compatible_range<<const> FwdR2>;
    ctor<It>(It first, It last);
    self& operator=<FwdR2> (<const> FwdR2& r); self& operator=( const self& r);
    <const>_iterator {begin|end}() <const>;
    self& advance_{begin|end}(difference_type n);
};
```

#### `combine`

```c++
struct combined_range<ItTup> : public iterator_range<zip_iterator<ItTup>> {
    ctor(ItTup first, ItTup last);
};
auto combine<...Ranges>(Ranges&&...rngs) -> combined_range<decltype(make_tuple(begin(rngs)...))>;
```

#### `join`

```c++
struct detail::join_iterator_link<It1,It2>{ It1 last1; It2 first2; };
struct join_iterator_begin_tag{}; struct join_iterator_end_tag{};

struct join_iterator_union<It1,It2,Ref> {
    using iterator1_t = It1; using iterator2_t = It2;
    ctor(); ctor(unsigned, const It1& it1, const It2& it2);
    <const> It1& it1() <const>; <const> It2& it2() <const>;
    Ref dereference(unsigned selected) const { return selected ? *m_it2 : *m_it1; }
    bool equal(const self& other, unsigned selected) const;
private: It1 m_it1; It2 m_it2;
};
struct join_iterator_union<It,It,Ref> {
    using iterator1_t = It; using iterator2_t = It;
    ctor(); ctor(unsigned selected, const It& it1, const It& it2) : m_it{selected? it2:it1}{}
    <const> It& it1() <const>; <const> It& it2() <const>;
    Ref dereference(unsigned selected) const { return *m_it; }
    bool equal(const self& other, unsigned) const;
private: It m_it;
};

struct join_iterator<It1,It2,V=iterator_value<It1>::type,Ref=...,Traversal=...> : iterator_facade<self,V,Traversal,Ref> {
    using link_t = join_iterator_link<It1,It2>; using iterator_union = join_iterator_union<It1,It2,Ref>;
    using iterator1_t = It1; using iterator2_t = It2;
    ctor();
    ctor(unsigned section, It1, cur1, It1 last1, It2 first2, It2 cur2)
        : m_section{section}, m_it{section,cur1,cur2}, m_link{link_t{last1,first2}}{}
    ctor<R1,R2>(<const> R1& r1, <const> R2& r2, join_iterator_begin_tag)
        : m_section{empty(r1)?1:0}, m_it{empty(r1)?1:0, <const>_begin(r1), <const>_begin(r2)}, m_link{link_t{<const>_end(r1),<const>_begin(r2)}}{}
    ctor<R1,R2>(<const> R1& r1, <const> R2& r2, join_iterator_end_tag)
        : m_section{empty(r1)?1:0}, m_it{empty(r1)?1:0, <const>_end(r1), <const>_end(r2)}, m_link{link_t{<const>_end(r1),<const>_begin(r2)}}{}
private: void increment(); void decrement();
    reference dereference() const;
    bool equal(const self& other) const;
    void advance(difference_type offset); difference_type distance_to(const self& other) const;
    unsigned m_section; iterator_union m_it; link_t m_link;
};

struct detail::joined_type<SPR1,SPR2> { using type=iterator_range<join_iterator<range_iterator<SPR1>::type, range_iterator<SPR2>::type, range_value<SPR1>::type>>; };

struct joined_range<SPR1,SPR2> : public joined_type<SPR1,SPR2>::type {
    ctor(SPR1& r1, SPR2& r2);
};

joined_range<<const> SPR1, <const> SPR2> join<SPR1,SPR2>(<const> SPR1& r1, <const> SPR2& r2);
```

#### `as_array`, `as_literal`

```c++
iterator_range<range_iterator<[const] R>::type> as_array<R>(<const> R& r);

iterator_range<range_iterator<[const] R>::type> as_literal<R>  (<const> R& r);
iterator_range<[const] Ch*> as_literal<Ch,sz>(<const> Ch (&arr)[sz]);
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
