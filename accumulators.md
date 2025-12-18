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
// PARAMETER_KEYWORD
struct tag::covariate1; struct tag::covariate2; struct tag::quantile_probability;

struct stats<...Stat> : mpl::vector<Stat...>{};
struct detail::error_of_tag<Feature> { using type=tag::error_of<Feature>; };
struct with_error<...Features> : mpl::transform_view<mpl::vector<Feature...>, error_of_tag<_1>>{};

using detail::times2_iterator = transform_iterator<decltype(std::bind(std::multiplies<size_t>{},2,_1)), counting_iterator<size_t>>;
times2_iterator detail::make_times2_iterator(size_t i) { return make_transform_iterator(make_counting_iterator(i), std::bind(multiplies<size_t>{},2,_1)); }
struct detail::lvalue_index_iterator<Base> : Base {ctor(){} ctor(Base b):base{b}{} Base::reference operator[](Base::difference_type n) const { return *(*this+n); } };

// modifiers
struct lazy{}; struct immediate{}; // for mean/variance
struct right{}; struct left{}; // for order, default is right
struct absolute{}; struct relative{}; // for tail_variate_means
struct with_density{}; struct with_p_square_cumulative_distribution{}; struct with_p_square_quantile{}; // for <weighted>_median
struct with_threshold_value{}; struct with_threshold_probability{}; // for peaks_over_threshold
struct weighted{}; struct unweighted{}; struct linear{}; struct quadratic{}; // for <weighted>_extended_p_square_quantile
struct regular{}; struct for_median{}; // for p_square_quantile
struct kahan{}; // for sum_..._kahan

struct tag::quantile : depends_on<>{ using impl=mpl::print<____MISSING_SPECIFIC_QUANTILE_FEATURE_IN_ACCUMULATOR_SET____>; };
extractor<tag::quantile> const extract::quantile={};
struct tag::tail_mean : depends_on<>{ using impl=mpl::print<____MISSING_SPECIFIC_TAIL_MEAN_FEATURE_IN_ACCUMULATOR_SET____>; };
extractor<tag::tail_mean> const extract::tail_mean={};

// count
struct impl::count_impl; // size_t cnt={0}; ++cnt;
struct tag::count : depends_on<>{ using impl=mpl::always<count_impl>; };
extractor<tag::count> const extract::count={};

// max
struct impl::max_impl<Sample>; // Sample max_={as_min(args[sample|Sample{}])}; max_assign(max_, args[sample])
struct tag::max : depends_on<>{ using impl=max_impl<_1>; };
extractor<tag::max> const extract::max = {};
// min
struct impl::min_impl<Sample>; // Sample min_={as_max(args[sample|Sample{}])}; min_assign(min_, args[sample])
struct tag::min : depends_on<>{ using impl=min_impl<_1>; };
extractor<tag::min> const extract::min = {};

// sum, sum_of_weights, sum_of_variates
struct impl::sum_impl<Sample,Tag=tag::sample>; // Sample sum={args[keyword<Tag>::get()|Sample{}]}; sum += args[keyword<Tag>::get()]
struct tag::sum : depends_on<> { using impl=sum_impl<_1,tag::sample>; };
struct tag::sum_of_weights : depends_on<> { using is_weight_accumulator=true_; using impl=sum_impl<_2,tag::weight>; };
struct tag::sum_of_variates<VType,VTag> : depends_on<> { using impl=mpl::always<sum_impl<VType,VTag>>; };
struct tag::abstract_sum_of_variates : depends_on<> {};
extractor<tag::sum> const extract::sum = {};
extractor<tag::sum_of_weights> const extract::sum_of_weights = {};
extractor<tag::abstract_sum_of_variates> const extract::sum_of_variates = {};
struct as_weighted_feature<tag::sum> { using type=tag::weighted_sum; };
struct feature_of<tag::sum_of_variates<VType,VTag>> : feature_of<tag::abstract_sum_of_variates>{};
// weighted_sum, weighted_sum_of_variates
struct impl::weighted_sum_impl<Sample,Weight,Tag>; // auto weighted_sum={args[keyword<Tag>::get()|Sample{}]*one<Weight>::value};
                                                   // weighted_sum += args[keyword<Tag>::get()] * args[weight]
struct tag::weighted_sum : depends_on<> { using impl=weighted_sum_impl<_1,_2,tag::sample>; };
struct tag::weighted_sum_of_variates<VType,VTag> : depneds_on<> { using impl=weighted_sum_impl<VType,_2,VTag>; };
struct tag::abstract_weighted_sum_of_variates : depends_on<>{};
extractor<tag::weighted_sum> const extract::weighted_sum = {};
extractor<tag::abstract_weighted_sum_of_variates> const extract::weighted_sum_of_variates = {};
struct featore_of<tag::weighted_sum> : feature_of<tag::sum>{};
struct feature_of<tag::weighted_sum_of_variates<VType,VTag>> : feature_of<tag::abstract_weighted_sum_of_variates>{};

// mean, immediate_mean, mean_of_weights, immediate_mean_of_weights, mean_of_variates, immediate_mean_of_variates
struct impl::mean_impl<Sample,SumFeature=tag::sum>; // fdiv(sum(args),count(args))
struct impl::immediate_mean_impl<Sample,Tag=tag::sample>; // auto mean={fdiv(args[sample|Sample{}],one<size_t>::value)};
                                                          // size_t cnt=count(args); mean=fdiv(mean*(cnt-1)+args[keyword<Tag>::get()], cnt)
struct tag::mean : depends_on<count,sum> { using impl=mean_impl<_1,sum>; };
struct tag::immediate_mean : depends_on<count> { using impl=immediate_mean_impl<_1,tag::sample>; };
struct tag::mean_of_weights : depends_on<count,sum_of_weights> { using is_weight_accumulator=true_; using impl=mean_impl<_2,sum_of_weights>; };
struct tag::immediate_mean_of_weights : depneds_on<count> { using is_weight_accumulator=true_; using impl=immediate_mean_impl<_2,tag::weight>; };
struct tag::mean_of_variates<VType,VTag> : depneds_on<count,sum_of_variates<VType,VTag>>
{ using impl=mpl::always<mean_impl<VType,sum_of_variates<VType,VTag>>>; };
struct tag::immediate_mean_of_variates<VType,VTag> : depneds_on<count> { using impl=mpl::always<immediate_mean_impl<VType,VTag>>; };
extractor<tag::mean> const extract::mean = {};
extractor<tag::mean_of_weights> const extract::mean_of_weights = {};
mpl::apply<AccSet,tag::mean_of_variates<VType,VTag>>::type::result_type extract::mean_of_variates<AccSet,VType,VTag>(AccSet const& acc)
{ return extract_result<tag::mean_of_variates<VType,VTag>>(acc); }
struct as_feature<tag::mean(lazy)> { using type=tag::mean; };
struct as_feature<tag::mean(immediate)> { using type=tag::immediate_mean; };
struct as_feature<tag::mean_of_weights(lazy)> { using type=tag::mean_of_weights; };
struct as_feature<tag::mean_of_weights(immediate)> { using type=tag::immediate_mean_of_weights; };
struct as_feature<tag::mean_of_variates<VType,VTag>(lazy)> { using type=tag::mean_of_variates<VType,VTag>; };
struct as_feature<tag::mean_of_variates<VType,VTag>(immediate)> { using type=tag::immediate_mean_of_variates<VType,VTag>; };
struct feature_of<tag::immediate_mean> : feature_of<tag::mean>{};
struct feature_of<tag::immediate_mean_of_weights> : feature_of<tag::mean_of_weights> {};
struct feature_of<tag::immediate_mean_of_variates<VType,VTag>> : feature_of<tag::mean_of_variates<VType,VTag>>{};
struct as_weighted_feature<tag::mean> { using type=tag::weighted_mean; };
struct feature_of<tag::weighted_mean> : feature_of<tag::mean>{};
struct as_weighted_feature<tag::immediate_mean> { using type=tag::immediate_weighted_mean; };
struct feature_of<tag::immediate_weighted_mean> : feature_of<tag::immediate_mean>{};
struct as_weighted_feature<tag::mean_of_variates<VType,VTag>> { using type=tag::weighted_mean_of_variates<VType,VTag>; };
struct feature_of<tag::weighted_mean_of_variates<VType,VTag>> : feature_of<tag::mean_of_variates<VType,VTag>> {};
struct as_weighted_feature<tag::immediate_mean_of_variates<VType,VTag>> { using type=tag::immediate_weighted_mean_of_variates<VType,VTag>; };
struct feature_of<tag::immediate_weighted_mean_of_variates<VType,VTag>> : feature_of<tag::immediate_mean_of_variates<VType,VTag>> {};
// weighted_mean, immediate_weighted_mean, weighted_mean_of_variates, immediate_weighted_mean_of_variates
struct impl::weighted_mean_impl<Sample,Weight,Tag>; // fdiv(some_weighted_sum(args),sum_of_weights(args))
struct impl::immediate_weighted_mean_impl<Sample,Weight,Tag>; // auto mean={fdiv(args[keyword<Tag>::get()|Sample{}]*one<Weight>, one<Weight>)};
                                                // Weight w_sum=sum_of_weights(args), w=args[weight]; mean = fdiv(mean*(w_sum-w) + args[keyword<Tag>::get()]*w, w_sum);
struct tag::weighted_mean : depends_on<sum_of_weights,weighted_sum> { using impl=weighted_mean_impl<_1,_2,tag::sample>; };
struct tag::immediate_weighted_mean : depneds_on<sum_of_weights> { using impl=immediate_weighted_mean_impl<_1,_2,tag::sample>; };
struct tag::weighted_mean_of_variates<VType,VTag> : depneds_on<sum_of_weights,weighted_sum_of_variates<VType,VTag>> { using impl=weighted_mean_impl<VType,_2,VTag>; };
struct tag::immediate_weighted_mean_of_variates<VType,VTag> : depneds_on<sum_of_weights> { using impl=immediate_weighted_mean_impl<VType,_2,VTag>; };
extractor<tag::mean> const extract::weighted_mean = {};
mpl::apply<AccSet,tag::weighted_mean_of_variates<VType,VTag>>::type::result_type extract::weighted_mean_of_variates<AccSet,VType,VTag>(AccSet const& acc)
{ return extract_result<tag::weighted_mean_of_variates<VType,VTag>>(acc); }
struct as_feature<tag::weighted_mean(lazy)> { using type=tag::weighted_mean; };
struct as_feature<tag::weighted_mean(immediate)> { using type=tag::immediate_weighted_mean; };
struct as_feature<tag::weighted_mean_of_variates<VType,VTag>(lazy)> { using type=tag::weighted_mean_of_variates<VType,VTag>; };
struct as_feature<tag::weighted_mean_of_variates<VType,VTag>(immediate)> { using type=tag::immediate_weighted_mean_of_variates<VType,VTag>; };

// covariance
struct impl::covariance_impl<Sample,VType,VTag>; // auto cov={outer_product(fdiv(args[sample|Sample{}],1), fdiv(args[keyword<VTag>::get()|VType{}],1))};
        // size_t cnt=count(args); if(cnt>1) cov = cov*(cnt-1)/cnt + outer_product(some_mean_of_variates(args)-args[keyword<VTag>::get()], mean(args)-args[sample])/(cnt-1)
struct tag::covariance<VType,VTag> : depneds_on<count,mean,mean_of_variates<VType,VTag>> { using impl=covariance_impl<_1,VType,VTag>; };
struct tag::abstract_covariance : depends_on<>{};
extractor<tag::abstract_covariance> const extract::covariance = {};
struct featore_of<tag::covariance<VType,VTag>> : featore_of<tag::abstract_covariance>{};
struct as_weighted_feature<tag::covariance<VType,VTag>> { using type=tag::weighted_covariance<VType,VTag>; };
struct feature_of<tag::weighted_covariance<VType,VTag>> : feature_of<tag::covariance<VType,VTag>>{};
// weighted_covariance
struct impl::weighted_covariance_impl<Sample,Weight,VType,VTag>; // auto cov={outer_product(fdiv(args[sample|Sample{}],1)*one<Weight>, fdiv(args[keyword<VTag>::get()|VType{}],1)*one<Weight>)}
        // size_t cnt=count(args); if(cnt>1) cov = cov*(sum_of_weights(args)-args[weight])/sum_of_weights(args) +
        //      outer_product(some_weighted_mean_of_variates(args)-args[keyword<VTag>::get()], weighted_mean(args)-args[sample]) * args[weight] / (sum_of_weights(args)-args[weight])
struct tag::weighted_covariance<VType,VTag> : depneds_on<count,sum_of_weights,weighted_mean,weighted_mean_of_variates<VType,VTag>> { using impl=weighted_covariance_impl<_1,_2,VType,VTag>; };
extractor<tag::abstract_covariance> const extract::weighted_covariance = {};

// density
struct tag::density_cache_size; struct tag::density_num_bins; // PARAMETER_NESTED_KEYWORD(num_bins)
struct impl::density_impl<Sample>; // INIT: size_t cache_size=args[density_cache_size], num_bins=args[density_num_bins];
        //  std::vector<f> cache{cache_size}, samples_in_bin{num_bins+2,0}, bin_positions{num_bins+2}; vector<pair<f,f>> histogram{num_bins+2,{fdiv(args[sample],1),fdiv(args[sample]),1}}; bool is_dirty{true};
        // ACC: is_dirty=true; size_t cnt=count(args); if (cnt <= cache_size>) cache[cnt-1]=args[sample];
        //  if (cnt==cache_size):
        //      f minimum=fdiv(min(args),1), maximum=fdiv(max(args),1), bin_size=fdiv(maximum-minimum,num_bins);
        //      for (i=0;i<num_bins+2;++i) bin_positions[i]=minimum+(i-1)*bin_size;
        //      for (auto e: cache) ++samples_in_bin[/*find pos in bin*/]
        //  else if (cnt > cache_size) ++samples_in_bin[/*find pos in bin*/]
        // RES: if (is_dirty) {is_dirty=false; for (i=0; i<num_bins+2;++i) histogram[i]={bin_positions[i],fdiv(samples_in_bin[i],cnt)}; } return make_iterator_range(histogram)
struct tag::density : depneds_on<count,min,max>, density_cache_size, density_num_bins{ using impl=density_impl<_1>; };
extractor<tag::density> const extract::density = {};
struct as_weighted_feature<tag::density> { using type=tag::weighted_density; };
struct feature_of<tag::weighted_density> : feature_of<tag::density> {};
// weighted_density
struct impl::weighted_density_impl<Sample,Weight>; // similar to density_impl, cache element is std::pair{args[sample],args[weight]}
    // ACC: `++` => `+= e.second`; RES: div by `sum_of_weights(args)`
struct tag::weighted_density : depneds_on<count,sum_of_weights,min,max>, density_cache_size, density_num_bins{ using impl=weighted_density_impl<_1,_2>; };
extractor<tag::density> const extract::weighted_density = {};

// error_of
struct impl::this_feature_has_no_error_calculation<Feature> : false_{};
struct impl::error_of_impl<Sample,Feature>; // just 0
struct tag::error_of<Feature> : depneds_on<Feature> { using impl=error_of_impl<_1,Feature>; };
mpl::apply<AccSet,tag::error_of<F>>::type::result_type extract::error_of<AccSet,F>(AccSet const& acc) { return extract_result<tag::error_of<F>>(acc); }
struct as_feature<tag::error_of<Feature>> { using type=tag::error_of<as_feature<Feature>::type>; };
struct as_weighted_feature<tag::error_of<Feature>> { using type=tag::error_of<as_weighted_feature<Feature>::type>; };
// error_of<mean>, error_of<immediate_mean>
struct impl::error_of_mean_impl<Sample,Variance>; // sqrt(fdiv(variance(args),count(args)-1))
struct tag::error_of<mean> : depends_on<lazy_variance,count> { using impl=error_of_mean_impl<_1,lazy_variance>; };
struct tag::error_of<immediate_mean> : depends_on<variance,count> { using impl=error_of_mean_impl<_1,variance>; };

// moment
struct impl::moment_impl<n,Sample>; // Sample sum=args[sample]; sum+=pow(args[sample],n); return fdiv(sum,count(args))
struct tag::moment<n> : depends_on<count> { using impl=moment_impl<mpl::int_<n>,_1>; };
mpl::apply<AccSet,tag::moment<n>>::type::result_type extract::moment<AccSet,n>(AccSet const& acc) { return extract_result<tag::moment<n>>(acc); }
struct as_weighted_feature<tag::moment<n>> { using type=tag::weighted_moment<n>; };
struct feature_of<tag::weighted_moment<n>> : feature_of<tag::moment<n>> {};
// weighted_moment
struct impl::weighted_moment_impl<n,Sample,Weight>; // auto sum={args[sample]*one<Weight>}; sum+=args[weight]*pow(args[sample],n); return fdiv(sum,sum_of_weights(args))
struct tag::weighted_moment<n> : depends_on<count,sum_of_weights> { using impl=weighted_moment_impl<mpl::int_<n>,_1,_2>; };
mpl::apply<AccSet,tag::weighted_moment<n>>::type::result_type extract::weighted_moment<AccSet,n>(AccSet const& acc) { return extract_result<tag::weighted_moment<n>>(acc); }

// p_square_cumulative_distribution
struct tag::p_square_cumulative_distribution_num_cells; // PARAMETER_NESTED_KEYWORD(num_cells)
struct impl::p_square_cumulative_distribution_impl<Sample>;
    // INIT: size_t num_cells=args[pscd_num_cells],b=num_cells; std::vector<f> h{b+1}, actual_p{b+1}, desired_p{b+1}, p_inc{b+1}; vector<pair<f,f>> histogram{b+1}; bool is_dirty{true};
    //  for (i=0;i<b+1;++i) { actual_p[i]=i+1, desired_p[i]=i+1, p_inc[i]=fdiv(i,b); }
    // ACC: is_dirty=true; size_t cnt=count(args), k=1;
    //  if (cnt < b+1) {h[cnt-1]=args[sample]; if(cnt==b+1) sort(h); }
    //  else: /* find cell k: h[k-1] <= args[sample] < h[k] */
    //      for (i : [k..b+1]) ++actual_p[i];  for (i : [1..b+1]) desired_p[i] += p_inc[i];
    //      /* adjust heights of [1..b] if necessary */
    // RES: if (is_dirty) {is_dirty=false; for (i=0; i<num_bins+2;++i) histogram[i]={h[i],fdiv(actual_p[i],cnt)}; } return make_iterator_range(histogram)
struct tag::p_square_cumulative_distribution : depneds_on<count>, p_square_cumulative_distribution_num_cells
{ using impl=p_square_cumulative_distribution_impl<_1>; };
extractor<tag::p_square_cumulative_distribution> const p_square_cumulative_distribution = {};
struct as_weighted_feature<tag::p_square_cumulative_distribution> { using type=tag::weighted_p_square_cumulative_distribution; };
struct feature_of<tag::weighted_p_square_cumulative_distribution> : feature_of<tag::p_square_cumulative_distribution> {};
// weighted_p_square_cumulative_distribution
struct impl::weighted_p_square_cumulative_distribution_impl<Sample,Weight>; // similar to p_square_cumulative_distribution_impl, no p_inc
    // ACC: `actual_p[i]+=args[weight]`; `desired_p[i] = actual_p[i] + fdiv((i-1)*(sum_of_weights(args)-actual_p[0]),b)`  RES: div by `sum_of_weights(args)`
struct tag::weighted_p_square_cumulative_distribution : depends_on<count,sum_of_weights>, p_square_cumulative_distribution_num_cells
{ using impl=weighted_p_square_cumulative_distribution_impl<_1,_2>; };
extractor<tag::weighted_p_square_cumulative_distribution> const weighted_p_square_cumulative_distribution = {};

// p_square_quantile, p_square_quantile_for_median
struct impl::p_square_quantile_impl<Sample,Impl>; // INIT: f p={is_same<Impl,for_median>?0.5:args[quantile_probaility|0.5]};
    //  array<f,5> h={}, actual_p={1,2,3,4,5}, desired_p{1,1+2*p,1+4*p,3+2*p,5}, p_inc{0,p/2,p,(1+p)/2,1};
    // ACC: cnt=count(args);
    //  if (cnt <= 5) { h[cnt-1]=args[sample]; if (cnt==5) sort(height); }
    //  else: k=1; /* find k => h[k-1]<=args[sample]<h[k]
    //      for (i : [k..5]) ++actual_p[i];     for (i : [0..5]) desired_p[i] += p_inc[i];
    //      for (i : [1..4]): adjust h[i]
    // RES: return h[2]
struct tag::p_square_quantile : depneds_on<count> { using impl=p_square_quantile_impl<_1,regular>; };
struct tag::p_square_quantile_for_median : depneds_on<count> { using impl=p_square_quantile_impl<_1,for_median>; };
extractor<tag::p_square_quantile> const p_square_quantile = {};
extractor<tag::p_square_quantile_for_median> const p_square_quantile_for_median = {};
struct as_weighted_feature<tag::p_square_quantile> { using type=tag::weighted_p_square_quantile; };
struct feature_of<tag::weighted_p_square_quantile> : feature_of<tag::p_square_quantile> {};
// weighted_p_square_quantile, weighted_p_square_quantile_for_median
struct impl::weighted_p_square_quantile_impl<Sample,Weight,Impl>; // similar to p_square_quantile_impl, no p_inc
    // ACC: `actual_p[i]+=args[weight]`, a0=actual_p[0]; sum=sum_of_weights(args); desired_p = {actual_p[0], (sum-a0)*p/2+a0, (sum-a0)*p+a0, (sum-a0)*(1+p)/2+a0, sum}
struct tag::weighted_p_square_quantile : depneds_on<count,sum_of_weights> { using impl=weighted_p_square_quantile_impl<_1,_2,regular>; };
struct tag::weighted_p_square_quantile_for_median : depneds_on<count,sum_of_weights> { using impl=weighted_p_square_quantile_impl<_1,_2,for_median>; };
extractor<tag::weighted_p_square_quantile> const weighted_p_square_quantile = {};
extractor<tag::weighted_p_square_quantile_for_median> const weighted_p_square_quantile_for_median = {};

// extended_p_square
struct tag::extended_p_square_probabilities; // PARAMETER_NESTED_KEYWORD(probabilitites)
struct impl::extended_p_square_impl<Sample>; // INIT: vector<f> p={args[extended_p_square_probabilities]}, h={2*p.size()+3}, actual_p={h.size()}, desired_p={h.size()}, p_inc={h.size()}
    //  num_q=p.size(), num_m=h.size(); actual_p={iota(1,num_m)}; p_inc=...; desired_p[i]=1+2*(num_q+1)*p_inc[i]
    // ACC: cnt=count(args);
    //  if (cnt <= num_m) { h[cnt-1]=args[sample]; if (cnt==num_m) sort(h); }
    //  else: k=1; /* find k => h[k-1]<=args[sample]<h[k]
    //      for (i : [k..num_m]) ++actual_p[i];     for (i : [0..5]) desired_p[i] += p_inc[i];
    //      for (i : [1..num_m-1]): adjust h[i]
    // RES: return {make_permutation_iterator(h.begin(),make_times2_iterator(1)), make_permutation_iterator(h.begin(),make_times2_iterator(num_p+1))}
struct tag::extended_p_square : depneds_on<count>, extended_p_square_probabilities
{ using impl=extended_p_square_impl<_1>; };
extractor<tag::extended_p_square> const extract::extended_p_square = {};
struct as_weighted_feature<tag::extended_p_square> { using type=tag::weighted_extended_p_square type; };
struct feature_of<tag::weighted_extended_p_square> : feature_of<tag::extended_p_square> {};
// weighted_extended_p_square
struct impl::weighted_extended_p_square_impl<Sample,Weight>; // similar to extended_p_square_impl, no p_inc
    // ACC: `actual_p[i]+=args[weight]`, update desired_p (interpolate)
struct tag::weighted_extended_p_square : depneds_on<count,sum_of_weights>, extended_p_square_probabilities
{ using impl=weighted_extended_p_square_impl<_1,_2>; };
extractor<tag::weighted_extended_p_square> const extract::weighted_extended_p_square = {};

// extended_p_square_quantile, extended_p_square_quantile_quadratic, weighted_extended_p_square_quantile, weighted_extended_p_square_quantile_quadratic
struct impl::extended_p_square_quantile_impl<Sample,Impl1,Impl2>; // INIT: vector<f> ps=args[extended_p_square_probabilitites], p=0
    // RES: h = some_extended_p_square(args); p = args[quantile_probability]
    //  if (p< ps[0]||p> ps[ps.size()-1]) return quiet_NaN || throw_exception()
    //  dist=distance(ps.begin(),lower_bound(ps, p)); return h[dist]; // or linear/quadratic interpolate
struct tag::extended_p_square_quantile : depends_on<extended_p_square> { using impl=extended_p_square_quantile_impl<_1,unweighted,linear>; };
struct tag::extended_p_square_quantile_quadratic : depends_on<extended_p_square> { using impl=extended_p_square_quantile_impl<_1,unweighted,quadratic>; };
struct tag::weighted_extended_p_square_quantile : depends_on<extended_p_square> { using impl=extended_p_square_quantile_impl<_1,weighted,linear>; };
struct tag::weighted_extended_p_square_quantile_quadratic : depends_on<extended_p_square> { using impl=extended_p_square_quantile_impl<_1,weighted,quadratic>; };
extractor<tag::extended_p_square_quantile> const extract::extended_p_square_quantile = {};
extractor<tag::extended_p_square_quantile_quadratic> const extract::extended_p_square_quantile_quadratic = {};
extractor<tag::weighted_extended_p_square_quantile> const extract::weighted_extended_p_square_quantile = {};
extractor<tag::weighted_extended_p_square_quantile_quadratic> const extract::weighted_extended_p_square_quantile_quadratic = {};
struct as_feature<tag::extended_p_square_quantile(linear)> { using type=tag::extended_p_square_quantile; };
struct as_feature<tag::extended_p_square_quantile(quadratic)> { using type=tag::extended_p_square_quantile_quadratic; };
struct as_feature<tag::weighted_extended_p_square_quantile(linear)> { using type=tag::weighted_extended_p_square_quantile; };
struct as_feature<tag::weighted_extended_p_square_quantile(quadratic)> { using type=tag::weighted_extended_p_square_quantile_quadratic; };
struct feature_of<tag::extended_p_square_quantile> : feature_of<tag::quantile>{};
struct feature_of<tag::extended_p_square_quantile_quadratic> : feature_of<tag::quantile>{};
struct as_weighted_feature<tag::extended_p_square_quantile> { using type=tag::weighted_extended_p_square_quantile; };
struct feature_of<tag::weighted_extended_p_square_quantile> : feature_of<tag::extended_p_square_quantile>{};
struct as_weighted_feature<tag::extended_p_square_quantile_quadratic> { using type=tag::weighted_extended_p_square_quantile_quadratic; };
struct feature_of<tag::weighted_extended_p_square_quantile_quadratic> : feature_of<tag::extended_p_square_quantile_quadratic>{};

// median
struct tag::median; struct impl::median_impl<Sample>;
struct tag::with_density_median; struct impl::with_density_median_impl<Sample>;
struct tag::with_p_square_cumulative_distribution_median;  struct impl::with_p_square_cumulative_distribution_median_impl<Sample>;
struct tag::weighted_median; struct impl::weighted_median_impl<Sample>;
struct tag::with_density_weighted_median; struct impl::with_density_weighted_median_impl<Sample>;
struct tag::with_p_square_cumulative_distribution_weighted_median; struct impl::with_p_square_cumulative_distribution_weighted_median_impl<Sample,Weight>;

struct tag::kurtosis; struct impl::kurtosis_impl<Sample>;
struct tag::min; struct impl::min_impl<Sample>;
struct tag::peaks_over_threadhold<LeftRight>;
struct tag::peaks_over_threadhold_prob<LeftRight>; struct impl::peaks_over_threadhold_prob_impl<Sample,LeftRight>;
struct tag::pot_tail_mean<LeftRight>; struct impl::pot_tail_mean_impl<Sample,Impl,LeftRight>;
struct tag::pot_tail_mean_prob<LeftRight>;
struct tag::pot_quantile<LeftRight>; struct impl::pot_quantile_impl<Sample,Impl,LeftRight>;
struct tag::pot_quantile_prob<LeftRight>;
struct tag::skewness; struct impl::skewness_impl<Sample>;
struct tag::sum_kahan; struct impl::sum_kahan_impl<Sample,Tag>;
struct tag::sum_of_weights_kahan;
struct tag::sum_of_variates_kahan<VType,VTag>;
struct tag::tail<LeftRight>; struct impl::tail_impl<Sample,LeftRight>;
struct tag::coherent_tail_mean<LeftRight>; struct impl::coherent_tail_mean_impl<Sample,LeftRight>;
struct tag::non_coherent_tail_mean<LeftRight>; struct impl::non_coherent_tail_mean_impl<Sample,LeftRight>;
struct tag::tail_quantile<LeftRight>; struct impl::tail_quantile_impl<Sample,LeftRight>;
struct tag::tail_variate<VType,VTag,LeftRight>; struct impl::tail_variate_impl<VType,VTag,LeftRight>;
struct tag::tail_weights<LeftRight>;
struct tag::right_tail_variate<VType,VTag,LeftRight>;
struct tag::left_tail_variate<VType,VTag,LeftRight>;
struct tag::tail_variate_means<LeftRight,VType,VTag>; struct impl::tail_variate_means_impl<Sample,Impl,LeftRight,VTag>;
struct tag::absolute_tail_variate_means<LeftRight,VType,VTag>;
struct tag::relative_tail_variate_means<LeftRight,VType,VTag>;
struct tag::lazy_variance; struct impl::lazy_variance_impl<Sample,MeanFeature>;
struct tag::variance; struct impl::variance_impl<Sample,MeanFeature,Tag>;
struct tag::weighted_kurtosis; struct impl::weighted_kurtosis_impl<Sample,Weight>;
struct tag::weighted_peaks_over_threshold<LeftRight>; struct impl::weighted_peaks_over_threshold_impl<Sample,Weight,LeftRight>;
struct tag::weighted_peaks_over_threshold_prob<LeftRight>; struct impl::weighted_peaks_over_threshold_probe_impl<Sample,Weight,LeftRight>;
struct tag::weighted_pot_tail_mean<LeftRight>;
struct tag::weighted_pot_tail_mean_prob<LeftRight>;
struct tag::weighted_pot_quantile<LeftRight>;
struct tag::weighted_pot_quantile_prob<LeftRight>;
struct tag::weighted_skewness; struct impl::weighted_skewness_impl<Sample,Weight>;
struct tag::weighted_tail_quantile<LeftRight>; struct impl::weighted_tail_quantile_impl<Sample,Weight,LeftRight>;
struct tag::non_coherent_weighted_tail_mean<LeftRight>; struct impl::non_coherent_weighted_tail_mean_impl<Sample,Weight,LeftRight>;
struct tag::weighted_tail_variate_means<LeftRight,VType,VTag>; struct impl::weighted_tail_variate_means_impl<Sample,Weight,Impl,LeftRight,VType>;
struct tag::absolute_weighted_tail_variate_means<LeftRight,VType,VTag>;
struct tag::relative_weighted_tail_variate_means<LeftRight,VType,VTag>;
struct tag::lazy_weighted_variance; struct impl::lazy_weighted_variance_impl<Sample,Weight,MeanFeature>;
struct tag::weighted_variance; struct impl::weighted_variance_impl<Sample,Weight,MeanFeature,Tag>;
    struct impl::weighted_sum_kahan_impl<Sample,Weight,Tag>;
struct tag::rolling_window_plus1; struct impl::rolling_window_plus1_impl<Sample>;
struct tag::rolling_window; struct impl::rolling_window_impl<Sample>;
struct tag::rolling_sum; struct impl::rolling_sum_impl<Sample>;
struct tag::rolling_count; struct impl::rolling_count_impl<Sample>;
struct tag::rolling_mean; struct impl::rolling_mean_impl<Sample>;
```

statistics/: kurtosis, median, moment, peaks_over_threadhold, pot_quantile, pot_tail_mean,
    rolling_{count,mean,moment,sum,variance,window},
	skewness, sum_kakan, tail_<mean,quantile>, tail_variate_<means>, variance,
    weighted_{kurtosis,median,moment}, weighted_peaks_over_threshold,
	weighted_{skewness,sum_kahan,variance}, weighted_tail_{mean,quantile,variant_means}

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
