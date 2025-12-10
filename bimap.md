# Boost.Bimap

* lib: `boost/libs/bimap`
* repo: `boostorg/bimap`
* commit: `f64de6d`, 2025-01-28

------
### Commons

#### Tags

```c++
namespace tags;
struct tagged<Type,Tag> { using value_type = Type; using tag = Tag; };
struct support::apply_to_value_type<F,TaggedType>;
struct support::default_tagged<Type,DefaultTag>;
struct support::is_tagged<Type>;
struct support::overwrite_tagged<Type,NewTag>;
struct support::tag_of<Type>;
struct support::value_type_of<Type>;
```

#### Relations

```c++
namespace relation;
namespace member_at { struct left{}; struct right{}; struct info{}; }
auto detail::mutate<View,Type>(Type& m) -> conditional_t<is_const_v<Type>, const View&, View&>;

struct normal_layout{}; struct mirror_layout{};
struct inverse_layout<Layout>; // normal <-> mirror

struct symmetrical_base<TA,TB,force_mutable=false> {
	using tl = default_tagged<TA,member_at::left>::type; using tr = default_tagged<TB,member_at::right>::type;
	using left_value_type = mpl::if_c<force_mutable,remove_const_t<tl::value_type>,tl::value_type>;
	using right_value_type = mpl::if_c<force_mutable,remove_const_t<tr::value_type>,tr::value_type>;
	using left_tag = tl::tag; using right_tag = tr::tag;
};

struct support::member_with_tag<Tag,Relation>;
struct support::is_tag_of_member_at_left<Tag,Relation>;
struct support::is_tag_of_member_at_right<Tag,Relation>;
struct support::is_tag_of_member_at_info<Tag,Relation>;
struct support::pair_type_by<Tag,Relation>;
struct support::value_type_of<Tag,Relation>;
struct support::result_of::get<Tag,Relation>;
result_of::get<Tag,Relation>::type support::get<Tag,Relation>(Relation& rel);
struct support::data_extractor_implementation<member_at::left,Relation> {
	using argument_type = Relation; using result_type = Relation::left_value_type;
	Relation::left_value_type <const> & operator()(Relation <const>& rel) const { return rel.left; }
};
struct support::data_extractor_implementation<member_at::right,Relation> {
	using argument_type = Relation; using result_type = Relation::right_value_type;
	Relation::right_value_type <const> & operator()(Relation <const>& rel) const { return rel.right; }
};
struct support::data_extractor<Tag,Relation> { using type = data_extractor_implementation<member_with_tag<Tag,Relation>::type,Relation>; };
struct support::both_keys_extractor<Relation> {
	using result_type = Relation::storage_base;
	<const> result_type& operator()(<const> Relation& rel) const { return rel; }
};
struct support::get_pair_functor<Tag,Relation>
{ result_of::pair_by<Tag,<const>Relation>::type operator()(<const>Relation& r) const { return pair_by<Tag>(r); } };
struct support::get_above_view_functor<Relation>
{ <const> Relation::above_view& operator()(<const>Relation& r) const { return r.get_view(); } };
struct support::opposite_tag<Tag,Relation>;
struct support::result_of::pair_by<Tag,Relation>;
result_of::pair_by<Tag,Relation>::type support::pair_by<Tag,Relation>(Relation& rel);

struct detail::pair_to_relation_functor<Tag,Relation>;
struct detail::get_mutable_relation_functor<Relation>;

struct detail::normal_storage<FirstType,SecondType> : public symmetrical_base<FirstType,SecondType> {
	using first_type = left_value_type; using second_type = right_value_type;
	first_type first; second_type second;
	ctor(); ctor(call_traits<first_type>::param_type f, call_traits<second_type>::param_type s);
	<const> left_value_type& get_left() const; <const> right_value_type& get_right() const;
};
struct detail::mirror_storage<FirstType,SecondType> : public symmetrical_base<SecondType,FirstType> {
	using second_type = left_value_type; using first_type = right_value_type;
	first_type first; second_type second;
	ctor(); ctor(call_traits<first_type>::param_type f, call_traits<second_type>::param_type s);
	<const> left_value_type& get_left() const; <const> right_value_type& get_right() const;
};

struct detail::storage_finder<FirstType,SecondType,Layout>;
struct detail::pair_info_hook<TA,TB,Info,Layout> : public storage_finder<TA,TB,Layout>::type {
	using ti = default_tagged<Info,member_at::info>::type;
	using info_type = ti::value_type; using info_tag = ti::tag;
	info_type info;
protected: ctor(); ctor<Pair>(const Pair& p);
	ctor(call_traits<first_type>::param_type f, call_traits<second_type>::param_type s, call_traits<info_type>::param_type i={});
	void change_to<Pair>(const Pair& p);
	void clear_info();
};
struct detail::pair_info_hook<TA,TB,mpl::na,Layout> : public storage_finder<TA,TB,Layout>::type {
	using info_type = mpl::na; using info_tag = member_at::info;
protected: ctor(); ctor<Pair>(const Pair& p);
	ctor(call_traits<first_type>::param_type f, call_traits<second_type>::param_type s);
	void change_to<Pair>(const Pair& p);
	void clear_info();
};
struct structured_pair<FT,ST,Info,L=normal_layout> : public pair_info_hook<FT,ST,Info,L> {
	using mutant_views = mpl::vector3<self<FT,ST,Info,normal_layout>,self<FT,ST,Info,mirror_layout>,
		if_<is_same<L,normal_layout>,mutant_relation<FT,ST,Info,true>,mutant_relation<ST,FT,Info,true>>::type>;
	ctor();
	ctor(call_traits<first_type>::param_type f, call_traits<second_type>::param_type s, <call_traits<info_type>::param_type i>);
	ctor<OtherLayout>(const self<FT,ST,Info,OtherLayout>& p); self& operator=<OtherLayout>(const self<FT,ST,OtherLayout>& p);
	ctor<F,S>(const std::pair<F,S>& p); self& operator=<F,S>(const std::pair<F,S>& p);
	<const> result_of::get<Tag, <const> self>::type get<Tag>() <const>;
};
bool operator{==|!=|<|<=|>|>=}<FT,ST,Info,L1,L2>(const structured_pair<FT,ST,Info,L1>& a, const structured_pair<FT,ST,Info,L2>& b);
bool operator{==|!=|<|<=|>|>=}<FT,ST,Info,L,F,S>(const structured_pair<FT,ST,Info,L>& a, const std::pair<F,S>& b);
bool operator{==|!=|<|<=|>|>=}<FT,ST,Info,L,F,S>(const std::pair<F,S>& a, const structured_pair<FT,ST,Info,L>& b);
structured_pair<FT,ST,Info,L> detail::copy_with_first_replaced<FT,ST,Info,L>(structured_pair<FT,ST,Info,L> const& p, call_traits<structured_pair<FT,ST,Info,L>::first_type>::param_type f);
structured_pair<FT,ST,mpl::na,L> detail::copy_with_first_replaced<FT,ST,L>(structured_pair<FT,ST,Info,L> const& p, call_traits<structured_pair<FT,ST,mpl::na,L>::first_type>::param_type f);
structured_pair<FT,ST,Info,L> detail::copy_with_second_replaced<FT,ST,Info,L>(structured_pair<FT,ST,Info,L> const& p, call_traits<structured_pair<FT,ST,Info,L>::second_type>::param_type f);
structured_pair<FT,ST,mpl::na,L> detail::copy_with_second_replaced<FT,ST,L>(structured_pair<FT,ST,Info,L> const& p, call_traits<structured_pair<FT,ST,mpl::na,L>::second_type>::param_type f);

struct detail::relation_storage<LT,RT,force_mutable> : public symmetrical_base<LT,RT,force_mutable> {
	using non_mutable_storage = self<LT,RT,false>;
	using mutant_views = mpl::vector2<self<LT,RT,true>,self<LT,RT,false>>;
	left_value_type left; right_value_type right;
	ctor(); ctor(call_traits<left_value_type>::param_type l, call_traits<right_value_type>::param_type r);
	<const> left_value_type& get_left() const; <const> right_value_type& get_right() const;
};
struct detail::relation_info_hook<TA,TB,Info,force_mutable> : public relation_storage<TA,TB,force_mutable> {
	using ti = default_tagged<Info,member_at::info>::type;
	using info_type = ti::value_type; using info_tag = ti::tag;
	info_type info;
protected: ctor(); ctor<Relation>(const Relation& rel);
	ctor(call_traits<left_value_type>::param_type l, call_traits<right_value_type>::param_type r, call_traits<info_type>::param_type i={});
	void change_to<Relation>(const Relation& rel);
	void serialize<Archive>(Archive& ar, unsigned);
};
struct detail::relation_info_hook<TA,TB,mpl::na,force_mutable> : public relation_storage<TA,TB,force_mutable> {
	using info_type = mpl::na; using info_tag = member_at::info;
protected: ctor(); ctor<Relation>(const Relation& rel);
	ctor(call_traits<left_value_type>::param_type l, call_traits<right_value_type>::param_type r);
	void change_to<Relation>(const Relation& rel);
	void serialize<Archive>(Archive& ar, unsigned);
};
struct mutant_relation<TA,TB,Info=mpl::na,force_mutable=false> : public relation_info_hook<TA,TB,Info,force_mutable> {
	using storage_base = relation_storage<TA,TB,force_mutable>;
	using above_view = self<TA,TB,Info,false>;
	using left_pair = structured_pair<TA,TB,Info,normal_layout>;
	using right_pair = structured_pair<TA,TB,Info,mirror_layout>;
	using mutant_views = mpl::vector4<left_pair,right_pair,self<TA,TB,Info,true>,self<TA,TB,Info,false>>;
	ctor();
	ctor(call_traits<left_value_type>::param_type l, call_traits<right_value_type>::param_type r, <call_traits<info_tag>::param_type i>);
	ctor(const self<TA,TB,Info,{false|true}>& rel); self& operator=(const self<TA,TB,Info,{false|true}>& rel);
	<const> left_pair& get_left_pair() <const>;
	<const> right_pair& get_right_pair() <const>;
	<const> above_view& get_view() <const>;
	<const> result_of::get<Tag,self>::type get<Tag>() <const>
	void serialize<Archive>(Archive& ar, unsigned);
};
size_t hash_value<FT,ST,fm>(const relation_storage<FT,ST,fm>& r);
bool operator{==|!=|<|<=|>|>=}<FT,ST,fm1,fm2>(const relation_storage<FT,ST,fm1>& a, const relation_storage<FT,ST,fm2>& b);
mutant_relation<TA,TB,Info,fm> detail::copy_with_left_replaced<TA,TB,Info,fm>(mutant_relation<TA,TB,Info,fm> const& p, call_traits<mutant_relation<TA,TB,Info,fm>::left_value_type>::param_type l);
mutant_relation<TA,TB,mpl::na,fm> detail::copy_with_left_replaced<TA,TB,fm>(mutant_relation<TA,TB,mpl::na,fm> const& p, call_traits<mutant_relation<TA,TB,mpl::na,fm>::left_value_type>::param_type l);
mutant_relation<TA,TB,Info,fm> detail::copy_with_right_replaced<TA,TB,Info,fm>(mutant_relation<TA,TB,Info,fm> const& p, call_traits<mutant_relation<TA,TB,Info,fm>::right_value_type>::param_type r);
mutant_relation<TA,TB,mpl::na,fm> detail::copy_with_right_replaced<TA,TB,fm>(mutant_relation<TA,TB,mpl::na,fm> const& p, call_traits<mutant_relation<TB,TA,mpl::na,fm>::right_value_type>::param_type r);
```

bimap, list_of, multiset_of, set_of, unconstrained_set_of, unordered_multiset_of, unordered_set_of, vector_of
views/: {list|unconstrained|vector}_{map|set}_view, <unordered>_<multi>{map|set}_view
support/: {data|iterator|key|map|value}_type_by, lambda, map_by
property_map/: <unordered>_set_support
detail/: bimap_core, concept_tags, generate_{index|relation|view}_binder, is_set_type, manage_additional_parameters, manage_bimap_key,
	{map|set}_view_{base|iterator}, modifier_adaptor, non_unique_views_helper
detail/test/check_metadata
container_adaptor/: <<ordered|unordered>_associative|sequence>_container_adaptor, {list_<map>|vector_<map>|<unordered>_<multi>{map|set}}_adaptor
container_adaptor/detail/: comparison_adaptor, functor_bag, identity_converters, key_extractor, non_unique_container_helper
container_adaptor/support/: iterator_facade_converters

-----
### Configuration

* `DISABLE_SERIALIZATION`

-----
### Dependencies

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.ContainerHash

* `<boost/functional/hash.hpp>`
* `<boost/functional/hash/hash.hpp>`

#### Boost.Core

* `<boost/core/allocator_access.hpp>`
* `<boost/core/enable_if.hpp>`
* `<boost/core/serialization.hpp>`
* `<boost/utility/addressof.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_adaptor.hpp>`
* `<boost/iterator/iterator_traits.hpp>`

#### Boost.Lambda

* `<boost/lambda/lambda.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.MultiIndex

* `<boost/multi_index/*.hpp>`
* `<boost/multi_index_container.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/cat.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

#### Boost.Utility

* `<boost/call_traits.hpp>`
* `<boost/operators.hpp>`

------
### Standard Facilities
