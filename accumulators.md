# Boost.Accumulators

* lib: `boost/libs/accumulators`
* repo: `boostorg/accumulators`
* commit: `eb7ff25`, 2025-06-26

------
### Accumulators Framework

```c++
concept Accumulator<State>;

void detail::ignore_variable(void const*){}
#define IGNORE_GLOBAL(X)
// for each: PARAMETER_NAME, IGNORE_GLOBAL
struct tag::sample; struct tag::weight; struct tag::accumulator; struct tag::weights;

struct feature_of<Feature> { using type=Feature; };
struct as_feature<Feature> { using type=Feature; };
struct as_weighted_feature<Feature> { using type=Feature; };

struct features<...Features> : mpl::vector<Features...>{};

// depends_on
struct detail::feature_tag<Acc> { using type=Acc::feature_tag; };
struct detail::undroppable<Feature> { using type=Feature; };
struct detail::undroppable<tag::droppable<Feature>> { using type=Feature; };
struct detail::is_dependent_on<A,B> : is_base_and_derived<feature_of<undroppable<B>::type>::type, undroppable<A>::type>{};
struct detail::dependencies_of<Feature> { using type=Feature::dependencies; };
struct detail::set_insert_range<Set,Range> : mpl::fold<Range,Set,mpl::insert<_1,_2>>{};
struct detail::collect_abstract_features<Features> : mpl::fold<Features,mpl::set0<>,set_insert_range<mpl::insert<_1,feature_of<_2>>, collect_abstract_features<dependencies_of<_2>>>>{};
struct detail::depends_on_base<Features> : mpl::inherit_linearly<mpl::sort<mpl::copy<collect_abstract_features<Features>::type,mpl::back_inserter<mpl::vector0<>>>::type, is_dependent_on<_1,_2>>::type,
    mpl::if_<is_dependent_on<_1,_2>,_1,mpl::inherit<_1,_2>>>::type {};
struct depends_on<...Features> : depends_on_base<mpl::transform<mpl::vector<Features...>,as_feature<_1>>::type> {
    using is_weight_accumulator = false_;
    using dependencies = mpl::transform<mpl::vector<Feature...>, as_feature<_1>>::type;
};

struct detail::matches_feature<Feature> { struct apply<Acc> : is_same<feature_of<as_feature<Feature>::type>::type, feature_of<as_feature<feature_tag<Acc>::type>::type>::type>{}; };
struct detail::contains_feature_of<Features,Acc> {
    using features_list = mpl::transform_view<Features,featore_of<as_feature<mpl::_>>>;
    using the_feature = feature_of<feature_tag<Acc>::type>::type;
    using type = mpl::contains<features_list,the_feature>::type;
};
struct detail::build_acc_list<First,Last,is_empty=fusion::result_of::equal_to<First,Last>::value>;
struct detail::build_acc_list<First,Last,true> { using type=fusion::nil_; static type call<Args>(Args const&, First const&, Last const&) { return{}; } };
struct detail::build_acc_list<First,Last,false> {
    using next_build_acc_list = build_acc_list<fusion::result_of::next<First>::type, Last>;
    using type=fusion::cons<fusion::result_of::value_of<First>::type,next_build_acc_list::type>;
    static type call<Args>(Args const& args, First const& f, Last const& l) { return{args, next_build_acc_list::call(args,fusion::next(f),l)}; }
};
struct detail::meta::make_acc_list<Seq> : build_acc_list<fusion::result_of::begin<Seq>::type, fusion::result_of::end<Seq>::type>{};
meta::make_acc_list<[const] Seq>::type detail::make_acc_list<Seq,Args>(Seq const& seq, Args const& args)
{ return meta::make_acc_list::call(args,fusion::begin(seq), fusion::end(seq)); }

struct detail::checked_as_weighted_feature<Feature> { using feature_type = as_feature<Feature>::type; using type=as_weighted_feature<feature_type>::type; };
struct detail::as_feature_list<Features,Weight> : mpl::transform_view<Features,checked_as_weighted_feature<_1>>{};
struct detail::as_feature_list<Features,void> : mpl::transform_view<Features,as_feature<_1>>{};
struct detail::accumulator_wrapper<Acc,Feature> : Acc {
    using feature_tag=Feature;
    ctor(self const& o) :base{*(Acc const*)&o}  ctor<Args>(Args const& args) :base{args}{}
    self& operator(self const& o) { *(Acc*)this = *(Acc const*)&o; return *this; }
};
struct detail::to_accumulator<Feature,Sample,Weight> { using type=accumulator_wrapper<mpl::apply2<Feature::impl,Sample,Weight>::type,Feature>; };
struct detail::to_accumulator<Feature,Sample,tag::external<Weight,Tag,AccSet>> {
    using accumulator_type = accumulator_wrapper<mpl::apply2<Feature::impl,Sample,Weight>::type,Feature>;
    using type = mpl::if_<Feature::is_weight_accumulator, accumulator_wrapper<impl::external_impl<accumulator_type,tag::weights>, Feature>, accumulator_type>::type;
};
struct detail::insert_feature<FeatureMap,Feature> : mpl::eval_if<mpl::has_key<FeatureMap,feature_of<Feature>::type>,
    mpl::identity<FeatureMap>, mpl::insert<FeatureMap,mpl::pair<feature_of<Feature>::type,Feature>>>{};
struct detail::insert_dependencies<FeatureMap,Feature,Weight>
    : mpl::fold<as_feature_list<Feature::dependencies,Weight>, FeatureMap, insert_dependencies<insert_feature<_1,_2>,_2,Weight>>{};
struct detail::insert_sequence<FeatureMap,Features,Weight> : mpl::fold<as_feature_list<Features,Weight>, FeatureMap, insert_feature<_1,_2>>{};
struct detail::make_accumulator_tuple<Features,Sample,Weight> {
    using feature_map = mpl::fold<as_feature_list<Features,Weight>,mpl::map0<>,mpl::if_<mpl::is_sequence<_2>, insert_sequence<_1,_2,Weight>, insert_feature<_1,_2>>>::type;
    using feature_map_with_dependencies = mpl::fold<feature_map, feature_map, insert_dependencies<_1,mpl::secnd<_2>,Weight>>::type;
    using feature_vector_with_dependencies = mpl::insert_range<mpl::vector<>, mpl::end<mpl::vector<>>::type, mpl::transform_view<feature_map_with_dependencies,mpl::second<_1>>>::type;
    using sorted_feature_vector = mpl::sort<feature_vector_with_dependencies, is_dependent_on<_2,_1>>::type;
    using type = mpl::transform<sorted_feature_vector, to_accumulator<_1,Sample,Weight>>::type;
};

struct detail::_enabler{};

// extractor
struct detail::accumulator_set_result<AccSet,Feature> {
    using feature_type=as_feature<Feature>::type;
    using type = mpl::apply<remove_cv_ref_t<AccSet>, feature_type>::type::result_type;
};
struct detail::accumulator_pack_result<Args,Feature> : accumulator_set_result<remove_reference_t<parameter::binding<remove_cv_ref_t<Args>, tag::accumulator>::type>, Feature>{};
struct detail::extractor_result<A,Feature> : mpl::eval_if<is_accumulator_set<A>, accumulator_set_result<A,Feature>, argument_pack_result<A,Feature>>{};
extractor_result<AccSet,Feature>::type detail::do_extract<Feature,AccSet>(AccSet const& acc, true_)
{ return extract_result<as_feature<Feature>::type>(acc); }
extractor_result<Args,Feature>::type detail::do_extract<Args,AccSet>(Args const& args, false_)
{ return find_accumulator<as_feature<Feature>::type>(args[accumulator]).result(args); }
struct extractor<Feature> {
    struct result<F>;
    struct result<self(A...)> : extractor_result<A...[0],Feature>{};
    extractor_result<Arg,Feature>::type operator()(Arg const& arg) const { return do_extract<Feature>(arg, is_accumulator_set<Arg>()); }
    extractor_result<AccSet,Feature>::type operator()<AccSet,...A>(AccSet const& acc, A const&... a) const { return extract_result<as_feature<Feature>::type>(acc,a...); }
};

struct detail::void_ = void;
struct dont_care {ctor<A>(A const&){}};
struct accumulator_base {
    void operator()(dont_care){}
    using is_droppable = false_;
    void add_ref(dont_care){} void drop(dont_care){} void on_drop(dont_care){}
};

// accumulator_set
struct detail::accumulator_visitor<Args> {
    explicit ctor(Args const& a) : args{a}{} ctor(self const&)=default; self& operator=(self const&)=delete;
    void operator()<Acc>(Acc& accumulator) const { accumulator(args); }
private: Args const& args;
};
const accumulator_visitor<Args> detail::make_accumulator_visitor<Args>(Args const& args) { return {args}; }
struct detail::accumulator_set_base{};
struct detail::is_accumulator_set<T> : mpl::if_<is_base_of<accumulator_set_base, remove_cv_ref_t<T>>, true_,false_>::type{};
struct detail::serialize_accumulator<Archive>{
    ctor(Archive& ar_, unsigned ver) : ar{ar_}, file_version{ver}{}
    void operator()<Acc>(Acc& accumulator) { accumulator.serialize(ar, file_version); }
private: Archive& ar; const unsigned int file_version;
};
struct accumulator_set<Sample,Features,Weight=void> :accumulator_set_base {
    using sample_type=Sample; using features_type=Features; using weight_type=Weight;
    using accumulators_mpl_vector=make_accumulator_tuple<Features,Sample,Weight>::type;
    using accumulators_type = meta::make_acc_list<accumulators_mpl_vector>::type;
    ctor() :accumulators{make_acc_list(accumulators_mpl_vector{}, (accumulator=*this))}
    { visit_if<contains_feature_of<Features>>(make_add_ref_visitor(accumulator=*this)); }
    explicit ctor<...A>(const A&...a) requires parameter::is_argument_pack<A...[0]>
        : accumulators{ make_acc_list(accumulators_mpl_vector{}, (accumulator=*this, a...))}
    { visit_if<contains_feature_of<Features>>(make_add_ref_visitor(accumulator=*this)); }
    explicit ctor<A0,...A>(const A0& a0, const A&...a) requires !parameter::is_argument_pack<A0>
        : accumulators{ make_acc_list(accumulators_mpl_vector{}, (accumulator=*this, sample=a0, a...))}
    { visit_if<contains_feature_of<Features>>(make_add_ref_visitor(accumulator=*this)); }

    void visit<UFunc>(UFunc const& func) { fusion::for_each(accumulators, func); }
    void visit_if<FilterPred,UFunc>(UFunc const& func) { fusion::for_each(fusion::filter_view<accumulators_type,FilterPred>{accumulators}, func); }
    using result_type = void;
    void operator()<...A>(const A&...a) requires is_argument_pack<A...[0]>
    { visit(make_accumulator_visitor(accumulator=*this, a...)); }
    void operator()<A0,...A>(const A0& a0, const A&...a) requires !is_argument_pack<A0>
    { visit(make_accumulator_visitor(accumulator=*this, sample=a0, a...)); }
    struct apply<Feature> : fusion::result_of::value_of<fusion::result_of::find_if<accumulators_type, matches_feature<Feature>>::type>{};
    apply<Feature>::type <const>& extract<Feature>() <const> { return *fusion::find_if<matches_feature<Feature>>(accumulators); }
    void drop<Feature>() {
        using the_accumulator = apply<Feature>::type; using the_feature = feature_of<as_feature<Feature>::type>::type;
        (*fusion::find_if<matches_feature<Feature>>(accumulators)).drop(accumulator=*this);
        visit_if<contains_feature_of<the_feature::dependencies>>(make_drop_visitor(accumulator=*this));
    }
    void serialize<Archive>(Archive& ar, unsigned ver) { serialize_accumulator<Archive> ser{ar,ver}; fusion::for_each(accumulators, ser); }
private: accumulators_type accumulators;
};
mpl::apply<AccSet,Feature>::type <const>& find_accumulator<Feature,AccSet>(AccSet <const>& acc) { return acc.extract<Feature>(); }
mpl::apply<AccSet,Feature>::type::result_type extract_result<Feature,AccSet,...A>(AccSet const& acc, const A&...a)
    requires is_argument_pack<A...[0]>
{ return find_accumulator<Feature>(acc).result(accumulator=acc, a...); }
mpl::apply<AccSet,Feature>::type::result_type extract_result<Feature,AccSet,A0,...A>(AccSet const& acc, const A0& a0, const A&...a)
    requires !is_argument_pack<A0>
{ return find_accumulator<Feature>(acc).result(accumulator=acc, sample=a0, a...); }

// accumulators
struct impl::value_accumulator_impl<ValueType,Tag> : accumulator_base {
    using result_type=ValueType;
    ctor<Args>(Args const& args) : val{args[parameter::keyword<Tag>::instance]}{}
    result_type result(dont_care) const { return val; }
private: Value_type val;
};
struct tag::value_tag<Tag>{};
struct tag::value<ValueType,Tag> : depneds_on<> { using impl=mpl::always<value_accumulator_impl<Value_type,Tag>>; };
mpl::apply<AccSet,tag::value<ValueType,Tag>>::type::result_type extract::value<ValueType,Tag>(AccSet const& acc)
{ return extract_result<tag::value<ValueType,Tag>>(acc); }
mpl::apply<AccSet,tag::value_tag<Tag>>::type::result_type extract::value_tag<Tag>(AccSet const& acc)
{ return extract_result<tag::value_tag<Tag>>(acc); }
struct featore_of<tag::value<ValueType,Tag>> : featore_of<tag::value_tag<Tag>>{};

struct impl::reference_accumulator_impl<Referent,Tag> : accumulator_base {
    using result_type = Referent&;
    ctor<Args>(Args const& args) : ref{args[parameter::keyword<Tag>::instance]}{}
    result_type result(dont_care) const { return ref; }
private: reference_wrapper<Referent> ref;
};
struct tag::reference_tag<Tag>{};
struct tag::reference<Referent,Tag> : depends_on<> { using impl=mpl::always<reference_accumulator_impl<Referent,Tag>>; };
mpl::apply<AccSet,tag::reference<Referent,Tag>>::type::result_type extract::reference<Referent,Tag>(AccSet const& acc)
{ return extract_result<tag::reference<Referent,Tag>>(acc); }
mpl::apply<AccSet,tag::reference_tag<Tag>>::type::result_type extract::reference_tag<Tag>(AccSet const& acc)
{ return extract_result<tag::reference_tag<Tag>>(acc); }
struct featore_of<tag::reference<ValueType,Tag>> : featore_of<tag::reference_tag<Tag>>{};

struct impl::external_impl<Acc,Tag> : accumulator_base {
    using result_type = Acc::result_type; using feature_tag=feature_tag<Acc>::type;
    ctor(dont_care){}
    result_type result<Args>(Args const& args) const { return extract_(args, args[paraketer::keyword<Tag>::instance|0]); }
private: static result_type extract_<Args>(Args const& args, int) { return extractor<feature_tag>{}(reference_tag<Tag>(args)); }
    static result_type extract_<Args,AccSet>(Args const&, AccSet const& acc) { return extractor<feature_tag>{}(acc); }
};
struct tag::external<Feature,Tag=void,AccSet=void> : depends_on<reference<AccSet,Tag>>
{ using impl=external_impl<to_accumulator<Feature,_1,_2>, Tag>; };
struct tag::external<Feature,Tag,void> : depends_on<>
{ using impl=external_impl<to_accumulator<Feature,_1,_2>, Tag>; };
struct featore_of<tag::external<ValueType,Tag,AccSet>> : featore_of<Feature>{};

struct detail::add_ref_visitor<Args> {
    explicit ctor(Args const& args):args_{args}{} ctor(self const& o)=default; self& operator=(self const&)=delete;
    void operator()<Acc>(Acc& acc) const
    { acc.add_ref(args_); args_[accumulator].visit_if<contains_feature_of<Acc::feature_tag::dependencies>>(*this); }
private: Args const& args_;
}
add_ref_visitor<Args> detail::make_add_ref_visitor<Args>(Args const& args) { return {args}; }
struct detail::drop_visitor<Args> {
    explicit ctor(Args const& args):args_{args}{} self& operator=(self const&)=delete;
    void operator()<Acc>(Acc& acc) const
    { if (Acc::is_droppable()) { acc.drop(args_); args_[accumulator].visit_if<contains_feature_of<Acc::feature_tag::dependencies>>(*this); } }
private: Args const& args_;
};
drop_visitor<Args> detail::make_drop_visitor<Args>(Args const& args) { return {args}; }
struct droppable_accumulator_base<Acc> : Acc {
    using is_droppable=true_;
    ctor<Args>(Args const& args) : base{args}{} ctor(self const& o)=default;
    void operator()<Args>(Args const& args) { if (!is_dropped()) base::operator()(args); }
    void add_ref<Args>(Args const&) { ++ref_count_; }
    void drop<Args>(Args const& args) { if (ref_count_==1) ((droppable_accumulator<Acc>*)this)->on_drop(args); --ref_count_; }
    bool is_dropped() const { return ref_count_==0; }
private: int ref_count_{0};
};
struct droppable_accumulator<Acc> : droppable_accumulator_base<Acc> {};
struct with_cached_result<Acc> : Acc {
    ctor<Args>(Args const& args) : base{args}{} self& operator=(self const&)=delete;
    ctor(self const& o) :base{*(Acc const*)&o} { if (o.has_result()) set(o.get()); }
    ~dtor() { if (has_result()) get().~dtor(); }
    void on_drop<Args>(Args const& args) { set(base::result(args)); }
    result_type result<Args>(Args const& args) const { return has_result() ? get() : base::result(args); }
private: aligned_storage<sizeof(result_type)> cache{};
    void set(result_type const& r) { ::new(cache.address()) result_type{r}; }
    result_type const& get() const { return *(result_type const*)cache.address(); }
    bool has_result() const { return ((droppable_accumulator_base<self> const*)this)->is_dropped(); }
};
struct tag::as_droppable<Feature> { using type=droppable<Feature>; };
struct tag::as_droppable<droppable<Feature>> { using type=droppable<Feature>; };
struct tag::droppable<Feature> : as_feature<Feature>::type {
    using feature_type=as_feature<Feature>::type;
    using dependencies = mpl::transform<feature_type::dependencies,as_droppable<_1>>::type;
    struct impl { struct apply<Sample,Weight> { using type=droppable_accumulator<mpl::apply<feature_type::impl,Sample,Weight>::type>; }; };
};
struct as_feature<tag::droppable<Feature>> { using type=tag::droppable<as_feature<Feature>::type>; };
struct as_weighted_feature<tag::droppable<Feature>> { using type=tag::droppable<as_weighted_feature<Feature>::type>; };
struct featore_of<tag::droppable<Feature>> : featore_of<Feature>{};
```

------
### Statistics Accumulators

```c++
```

statistics_<fwd>
statistics/parameters/quantile_probability
statistics/variates/covariate
statistics/: count, covariance, density, error_of_<mean>, extended_p_square_<quantile>,
	kurtosis, max, mean, median, min, moment, p_square_{cumul_dist,cumulative_distribution,quantile},
	peaks_over_threadhold, pot_quantile, pot_tail_mean, rolling_{count,mean,moment,sum,variance,window},
	skewness, stats, sum_<kakan>, tail_<mean,quantile>, tail_variate_<means>, timers2_iterator,
	variance, weighted_{covariance,density,extended_p_square,kurtosis,mean,median,moment},
	weighted_p_source_{cumul_dist,cumulative_distribution,quantile}, weighted_peaks_over_threshold,
	weighted_{skewness,sum_kahan,sum,variance}, weighted_tail_{mean,quantile,variant_means}, with_error

numeric/: functional_<fwd>
numeric/functional/: complex, valarray, vector
numeric/detail/: function{1,2,3,4,_n}, pod_singleton

------
### Configuration

* `MAX_FEATURES`: default/max: `MPL_LIMIT_VECTOR_SIZE` (20 by default)
* `MAX_ARGS`: default 15

------
### Dependency

#### Boost.Array

* `<boost/array.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.CircularBuffer

* `<boost/circular_buffer.hpp>`

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/config/no_tr1/cmath.hpp>`
* `<boost/version.hpp>`

#### Boost.Core

* `<boost/ref.hpp>`
* `<boost/core/enable_if.hpp>`, `<boost/utility/enable_if.hpp>`

#### Boost.Fusion

* `<boost/fusion/include/**.hpp>`

#### Boost.Iterator

* `<boost/iterator/counting_iterator.hpp>`
* `<boost/iterator/permutation_iterator.hpp>`
* `<boost/iterator/reverse_iterator.hpp>`
* `<boost/iterator/transform_iterator.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.Numeric/Conversion

* `<boost/numeric/conversion/cast.hpp>`

#### Boost.Numeric/uBLAS

* `<boost/numeric/ublas/io.hpp>`
* `<boost/numeric/ublas/matrix.hpp>`

#### Boost.Parameter

* `<boost/parameter/binding.hpp>`
* `<boost/parameter/is_argument_pack.hpp>`
* `<boost/parameter/name.hpp>`
* `<boost/parameter/nested_keyword.hpp>`
* `<boost/parameter/keyword.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.Range

* `<boost/range.hpp>`
* `<boost/range/begin.hpp>`, `<boost/range/end.hpp>`
* `<boost/range/iterator_range.hpp>`

#### Boost.Serialization

* `<boost/serialization/boost_array.hpp>`
* `<boost/serialization/split_free.hpp>`
* `<boost/serialization/utility.hpp>`
* `<boost/serialization/vector.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Tuple

* `<boost/tuple/tuple.hpp>`

#### Boost.TypeTraits

* `<boost/aligned_storage.hpp>`
* `<boost/type_traits/**.hpp>`

#### Boost.TypeOf

* `<boost/typeof/typeof.hpp>`
* `<boost/typeof/std/complex.hpp>`
* `<boost/typeof/std/valarray.hpp>`
* `<boost/typeof/std/vector.hpp>`

------
### Standard Facilities
