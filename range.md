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
struct detail::indirect_forwarder {};
const auto adaptors::indirected = indirect_forwarder{};
indirected_range<[const]SinglePassR> adaptors::indirect<SinglePassR>(<const>SinglePassR& r);
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
