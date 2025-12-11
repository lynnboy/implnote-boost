# Boost.Bimap

* lib: `boost/libs/bimap`
* repo: `boostorg/bimap`
* commit: `f64de6d`, 2025-01-28

------
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

#### Container Adaptors

```c++
namespace container_adaptor;

struct detail::iterator_to_base_identity<BaseIt,It,BaseCIt,CIt>
{ BaseIt operator()(It iter) const { return {iter}; } BaseCIt operator()(CIt iter) const { return {iter}; } };
struct detail::iterator_to_base_identity<BaseIt,It,BaseIt,It> { BaseIt operator()(It iter) const { return {iter}; } };
struct detail::iterator_from_base_identity<BaseIt,It,BaseCIt,CIt>
{ It operator()(BaseIt iter) const { return {iter}; } CIt operator()(BaseCIt iter) const { return {iter}; } };
struct detail::iterator_from_base_identity<BaseIt,It,BaseIt,CIt> { It operator()(BaseIt iter) const { return {iter}; } };
struct detail::value_to_base_identity<BaseV,V> { BaseV operator()(const V& val) const { return {val}; } };
struct detail::value_to_base_identity<V,V> { const V& operator()(const V& val) const { return val; } };
struct detail::value_from_base_identity<BaseV,V> { V operator()(const BaseV& val) const { return {val}; } };
struct detail::value_from_base_identity<V,V> { <const> V& operator()(<const> V& val) const { return val; } };
struct detail::key_to_base_identity<BaseK,K> { BaseK operator()(const K& k) const { return {k}; } };
struct detail::key_to_base_identity<K,const K> { const K2& operator()<K2>(const K2& k) const { return k; } };

struct detail::comparison_adaptor<Compare,NewType,Converter> {
  using first_argument_type = NewType; using second_argument_type = NewType; using result_type = bool;
  ctor(const Compare& comp, const Converter& conv): compare{comp}, converter{conv}{}
  bool operator()(call_traits<NewType>::param_type x, call_traits<NewType>::param_type y) const { return compare(converter(x), converter(y)); }
private: Compare compare; Converter converter;
};
struct detail::compatible_comparison_adaptor<Compare,NewType,Converter> {
  using first_argument_type = NewType; using second_argument_type = NewType; using result_type = bool;
  ctor(const Compare& comp, const Converter& conv): compare{comp}, converter{conv}{}
  bool operator()<T1,T2>(const T1& x, const T2& y) const { return compare(converter(x), converter(y)); }
private: Compare compare; Converter converter;
};
struct detail::unary_check_adaptor<Compare,NewType,Converter> {
  using argument_type = call_traits<NewType>::param_type; using result_type = bool;
  ctor(const Compare& comp, const Converter& conv): compare{comp}, converter{conv}{}
  bool operator()(call_traits<NewType>::param_type x) const { return compare(converter(x)); }
private: Compare compare; Converter converter;
};

struct detail::key_from_pair_extractor<T> { using argument_type = T; using result_type = T::first_type; result_type operator()(const T& p) { return p.first; } };

struct detail::data_with_functor_bag<Data,FunctorList> : public mpl::inherit_linearly<FunctorList,if_<is_base_of<_2,_1>,_1,inherit<_1,_2>>>::type {
  Data data;
  ctor(); ctor(add_reference_t<Data> d);
  <const> Functor& functor<Functor>() <const> { return *(Functor*)this; }
};

// support for iterator_facade
struct support::iterator_facade_to_base<It,CIt> { <C>It::base_type operator()(<C>It iter) const { return iter.base(); } };
struct support::iterator_facade_to_base<It,It> { It::base_type operator()(It iter) const { return iter.base(); } };

struct container_adaptor<Base,It,CIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>> {
  using iterator = It; using const_iterator = CIt;
  using value_type = iterator_value<It>::type; using pointer = iterator_pointer<It>::type; using <const>_reference = iterator_reference< <C>It>::type;
  using size_type = Base::size_type; using difference_type = Base::difference_type;
  using iterator_to_base = mpl::if_<is_na<ItToBaseCvt>,iterator_to_base_identity<Base::iterator,It,Base::const_iterator,CIt>,ItToBaseCvt>::type;
  using iterator_from_base = mpl::if_<is_na<ItFromBaseCvt>,iterator_from_base_identity<Base::iterator,It,Base::const_iterator,CIt>,ItFromBaseCvt>::type;
  using value_to_base = mpl::if_<is_na<VToBaseCvt>,value_to_base_identity<Base::value_type,value_type>,VToBaseCvt>::type;
  using value_from_base = mpl::if_<is_na<VFromBaseCvt>,value_from_base_identity<Base::value_type,value_type>,VFromBaseCvt>::type;
  explicit ctor(Base& c): dwfb{c}{}
  size_type size() const { return base().size(); }
  size_type max_size() const { return base().max_size(); }
  bool empty() const { return base().empty(); }
  <const>_iterator {begin|end}() <const> { return functor<iterator_from_base>()(base().{begin|end}()); }
  iterator erase(iterator p) { return functor<iterator_from_base>()(base().erase(functor<iterator_to_base>()(p))); }
  iterator erase(iterator f, itreator l) { return functor<iterator_from_base>()(base().erase(functor<iterator_to_base>()(f), functor<iterator_to_base>()(l))); }
  void clear() { base().clear(); }
  void insert<InIt>(InIt b, InIt e) { for(; b!=e; ++b) base().insert(functor<value_to_base>()(*b)); }
  std::pair<iterator,bool> insert(call_traits<value_type>::param_type x)
  { std::pair r = base().insert(functor<value_to_base>()(x)); return {functor<iterator_from_base>()(r.first), r.second}; }
  iterator insert(iterator p, call_traits<value_type>::param_type x)
  { return functor<iterator_from_base>()(base().insert(functor<iterator_to_base>()(p), functor<value_to_base>()(x))); }
  void swap(self& c) { base().swap(c.base()); }
protected: using base_type = Base;
  <const> Base& base() <const> { return dwfb.data; }
  <const> Functor& functor<Functor>() <const> { return dwfb.functor<Functor>(); }
private: data_with_functor_bag<Base&,mpl::copy<mpl::vector<iterator_to_base,iterator_from_base,value_to_base,value_from_base>,mpl::front_iterator<FunctorsFromDerivedClasses>>::type> dwfb;
};

struct sequence_container_adaptor_base<Base,It,CIt,RIt,CRIt,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,FunctorsFromDerivedClasses>
{ using type = container_adaptor<Base,It,CIt,ItToBaseCvt,ItFromBaseCvt,VToBaseCvt,VFromBaseCvt,
    mpl::push_front<FunctorsFromDerivedClasses,mpl::if_<mpl::is_na<RItFromBaseCvt>,iterator_from_base_identity<Base::reverse_iterator,RIt,Base::const_reverse_iterator,CRIt>,RItFromBaseCvt>::type>::type>; };
struct sequence_container_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public sequence_container_adaptor_base<Base,It,CIt,RIt,CRIt,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,FunctorsFromDerivedClasses>::type, totally_ordered<self>
{ using <const>_reverse_iterator = <C>RIt;
  explicit ctor(Base& c): base{c}{}
  <const>_reverse_iterator {rbegin|rend}() <const> { return functor<reverse_iterator_from_base>()(base().{rbegin|rend}()); }
  void resize(size_type n, call_traits<value_type>::param_type x={}) { base().resize(n, functor<value_to_base>()(x)); }
  <const>_reference {front|back}() <const> { return functor<value_from_base>()(base().{front|back}()); }
  void push_{front|back}(call_traits<value_type>::param_type x) { base().push_{front|back}(functor<value_to_base>()(x)); }
  void pop_{front|back}() { base().pop_{front|back}(); }
  std::pair<iterator,bool> insert(iterator p, call_traits<value_type>::param_type x)
  { std::pair r = base().insert(functor<iterator_to_base>()(p), functor<value_to_base>()(x)); return {functor<iterator_from_base>()(r.first), r.second}; }
  void insert(iterator p, size_type m, call_traits<value_type>::param_type x)
  { base().insert(functor<iterator_to_base>()(p), m, functor<value_to_base>()(x)); }
  void insert<InIt>(iterator p, InIt f, InIt l)
  { for (; f!=l; ++f) base().insert(functor<iterator_to_base>()(p), functor<value_to_base>()(*f)); }
  bool operator{==|<}(const self& c) const { return base() {==|<} c.base(); }
protected: using reverse_iterator_from_base = mpl::if_<is_na<RItFromBaseCvt>,iterator_from_base_identity<Base::reverse_iterator,RIt,Base::const_reverse_iterator,CRIt>,RItFromBaseCvt>::type;
};

struct associative_container_adaptor_base<Base,It,CIt,K,ItToBaseCvt,ItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ using type = container_adaptor<Base,It,CIt,ItToBaseCvt,ItFromBaseCvt,VToBaseCvt,VFromBaseCvt,
    mpl::push_front<FunctorsFromDerivedClasses,mpl::if_<mpl::is_na<KToBaseCvt>,key_to_base_identity<Base::key_type,K>,KToBaseCvt>::type>::type>; };
struct associative_container_adaptor<Base,It,CIt,K,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public associative_container_adaptor_base<Base,It,CIt,K,ItToBaseCvt,ItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>::type
{ using key_type = K;
  explicit ctor(Base& c): base{c}{}
  using base::erase;
  size_type erase<K2>(const K2& k) { return base().erase(functor<key_to_base>()(k)); }
  size_type count<K2>(const K2& k) const { return base().count(functor<key_to_base>()(k)); }
  <const>_iterator find<K2>(const K2& k) <const> { return functor<iterator_from_base>()(base().find(functor<key_to_base>()(k))); }
  std::pair<<const>_iterator,<const>_iterator> equal_range<K2>(const K2& k) <const>
  { std::pair r = base().equal_range(functor<key_to_base>()(k)); return {functor<iterator_from_base>()(r.first), functor<iterator_from_base>()(r.second)}; }
protected: using key_to_base = mpl::if_<is_na<KToBaseCvt>,key_to_base_identity<Base::key_type,K>,KToBaseCvt>::type;
};

struct ordered_associative_container_adaptor_base<Base,It,CIt,RIt,CRIt,K,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ using type = associative_container_adaptor<Base,It,CIt,K,ItToBaseCvt,ItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,
    mpl::push_front<FunctorsFromDerivedClasses,mpl::if_<mpl::is_na<RItFromBaseCvt>,iterator_from_base_identity<Base::reverse_iterator,RIt,Base::const_reverse_iterator,CRIt>,RItFromBaseCvt>::type>::type>; };
struct ordered_associative_container_adaptor<Base,It,CIt,RIt,CRIt,K,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public ordered_associative_container_adaptor_base<Base,It,CIt,RIt,CRIt,K,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>::type, totally_ordered<self>
{ using key_compare = compatible_comparison_adaptor<Base::key_compare,K,key_to_base>;
  using value_compare = comparison_adaptor<Base::value_compare,value_type,value_to_base>;
  using <const>_reverse_iterator = <C>RIt;
  explicit ctor(Base& c): base{c}{}
  <const>_reverse_iterator {rbegin|rend}() <const> { return functor<reverse_iterator_from_base>()(base().{rbegin|rend}()); }
  key_compare key_comp() const { return {base().key_comp(), functor<key_to_base>()}; }
  value_compare value_comp() const { return {base().value_comp(), functor<value_to_base>()}; }
  <const>_iterator {lower|upper}_bound<K2>(const K2& k) <const> { return functor<iterator_from_base>()(base().{lower|upper}_bound(functor<key_to_base>()(k))); }
  bool operator{==|<}(const self& c) const { return base() {==|<} c.base(); }
protected: using reverse_iterator_from_base = mpl::if_<is_na<RItFromBaseCvt>,iterator_from_base_identity<Base::reverse_iterator,RIt,Base::const_reverse_iterator,CRIt>,RItFromBaseCvt>::type;
};

struct ordered_associative_container_adaptor_base<Base,It,CIt,RIt,CRIt,K,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ using type = associative_container_adaptor<Base,It,CIt,K,ItToBaseCvt,ItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,
    mpl::push_front<FunctorsFromDerivedClasses,mpl::if_<mpl::is_na<RItFromBaseCvt>,iterator_from_base_identity<Base::reverse_iterator,RIt,Base::const_reverse_iterator,CRIt>,RItFromBaseCvt>::type>::type>; };
struct ordered_associative_container_adaptor<Base,It,CIt,RIt,CRIt,K,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public ordered_associative_container_adaptor_base<Base,It,CIt,RIt,CRIt,K,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>::type, totally_ordered<self>
{ using key_compare = compatible_comparison_adaptor<Base::key_compare,K,key_to_base>;
  using value_compare = comparison_adaptor<Base::value_compare,value_type,value_to_base>;
  using <const>_reverse_iterator = <C>RIt;
  explicit ctor(Base& c): base{c}{}
  <const>_reverse_iterator {rbegin|rend}() <const> { return functor<reverse_iterator_from_base>()(base().{rbegin|rend}()); }
  key_compare key_comp() const { return {base().key_comp(), functor<key_to_base>()}; }
  value_compare value_comp() const { return {base().value_comp(), functor<value_to_base>()}; }
  <const>_iterator {lower|upper}_bound<K2>(const K2& k) <const> { return functor<iterator_from_base>()(base().{lower|upper}_bound(functor<key_to_base>()(k))); }
  bool operator{==|<}(const self& c) const { return base() {==|<} c.base(); }
protected: using reverse_iterator_from_base = mpl::if_<is_na<RItFromBaseCvt>,iterator_from_base_identity<Base::reverse_iterator,RIt,Base::const_reverse_iterator,CRIt>,RItFromBaseCvt>::type;
};

struct unordered_associative_container_adaptor_base<Base,It,CIt,LIt,CLIt,K,ItToBaseCvt,ItFromBaseCvt,LItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ using type = associative_container_adaptor<Base,It,CIt,K,ItToBaseCvt,ItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,
    mpl::push_front<FunctorsFromDerivedClasses,mpl::if_<mpl::is_na<LItFromBaseCvt>,iterator_from_base_identity<Base::local_iterator,LIt,Base::const_local_iterator,CLIt>,LItFromBaseCvt>::type>::type>; };
struct unordered_associative_container_adaptor<Base,It,CIt,LIt,CLIt,K,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,LItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public unordered_associative_container_adaptor_base<Base,It,CIt,LIt,CLIt,K,ItToBaseCvt,ItFromBaseCvt,LItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>::type
{ using key_equal = Base::key_equal; using hasher = Base::hasher;
  using <const>_local_iterator = <C>LIt;
  explicit ctor(Base& c): base{c}{}
  size_type <max>_bucket_count() const { return base().<max>_bucket_count(); }
  size_type bucket_size(size_type n) const { return base().bucket_size(n); }
  size_type bucket<K2>(const K2& k) const { return base().bucket(functor<key_to_base>()(k)); }
  using base::{begin|end};
  <const>_local_iterator {begin|end}(size_type n) <const> { return functor<local_iterator_from_base>(base().{begin|end}(n)); }
  float <max>_load_factor() const { return base().<max>_load_factor(); }
  void max_load_factor(float z) { base().max_load_factor(z); }
  void rehash(size_type n) { base().rehash(n); }
protected: using local_iterator_from_base = mpl::if_<is_na<LItFromBaseCvt>,iterator_from_base_identity<Base::local_iterator,LIt,Base::const_local_iterator,CLIt>,LItFromBaseCvt>::type;
};

struct list_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public sequence_container_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,FunctorsFromDerivedClasses>::type
{ explicit ctor(Base& c): base{c}{}
  void splice(It p, self& x) { base().splice(functor<iterator_to_base>()(p), x.base()); }
  void splice(It p, self& x, It i) { base().splice(functor<iterator_to_base>()(p), x.base(), functor<iterator_to_base>()(i)); }
  void splice(It p, self& x, It f, It l) { base().splice(functor<iterator_to_base>()(p), x.base(), functor<iterator_to_base>()(f), functor<iterator_to_base>()(l)); }
  void remove(call_traits<value_type>::param_type v) { base().remove(functor<value_to_base>()(v)); }
  void remove_if<Pred>(Pred pred) { base().remove_if(unary_check_adaptor<Pred,value_type,value_from_base>{pred, functor<value_to_base>()}); }
  void unique() { base().unique(comparison_adaptor<std::equal_to<value_type>,value_type,value_from_base>{{}, functor<value_from_base>()}); }
  void unique<BPred>(BPred bpred) { base().unique(comparison_adaptor<BPred,value_type,value_from_base>{bpred, functor<value_from_base>()}); }
  void merge(self& x) { base().merge(x.base(), comparison_adaptor<std::less<value_type>,value_type,value_from_base>{{}, functor<value_from_base>()}); }
  void merge<Compare>(self& x, Compare comp) { base().merge(x.base(), comparison_adaptor<Compare,value_type,value_from_base>{comp, functor<value_from_base>()}); }
  void sort() { base().sort(comparison_adaptor<std::less<value_type>,value_type,value_from_base>{{}, functor<value_from_base()}); }
  void sort<Compare>(Compare comp) { base().sort(comparison_adaptor<Compare,value_type,value_from_base>{comp, functor<value_from_base>()}); }
  void reverse() { base.reverse(); }
};

struct list_map_adaptor_base<Base,It,CIt,RIt,CRIt,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KFromBaseVCvt,FunctorsFromDerivedClasses>
{ using type = list_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,
    mpl::push_front<FunctorsFromDerivedClasses,mpl::if_<mpl::is_na<KFromBaseVCvt>,key_from_pair_extractor<It::value_type>,KFromBaseVCvt>::type>::type>; };
struct list_map_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KFromBaseVCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
struct list_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public list_map_adaptor_base<Base,It,CIt,RIt,CRIt,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KFromBaseVCvt,FunctorsFromDerivedClasses>::type
{ using key_type = It::value_type::first_type; using data_type = It::value_type::second_type; using mapped_type = data_type;
  ctor(Base& c) :base{c}{}
  void remove_if<Pred>(Pred pred) { base().remove_if(unary_check_adaptor<Pred,value_type,key_from_base_value>{pred, functor<key_from_base_value>()}); }
  void unique() { base().unique(comparison_adaptor<std::equal_to<key_type>,value_type,key_from_base_value>{{}, functor<key_from_base_value>()}); }
  void unique<BPred>(BPred bpred) { base().unique(comparison_adaptor<BPred,value_type,key_from_base_value>{bpred, functor<key_from_base_value>()}); }
  void merge(self& x) { base().merge(x.base(), comparison_adaptor<std::less<key_type>,value_type,key_from_base_value>{{}, functor<key_from_base_value>()}); }
  void merge<Compare>(self& x, Compare comp) { base().merge(x.base(), comparison_adaptor<Compare,value_type,key_from_base_value>{comp, functor<key_from_base_value>()}); }
  void sort() { base().sort(comparison_adaptor<std::less<key_type>,value_type,key_from_base_value>{{}, functor<key_from_base_value>()}); }
  void sort<Compare>(Compare comp) { base().sort(comparison_adaptor<Compare,value_type,key_from_base_value>{comp, functor<key_from_base_value>()}); }
protected: using key_from_base_value = mpl::if_<mpl::is_na<KFromBaseVCvt>, key_from_pair_extractor<It::value_type>,KFromBaseVCvt>::type;
};

struct vector_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public sequence_container_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,FunctorsFromDerivedClasses>::type
{ ctor(){} explicit ctor(Base& c): base{c}{}
  size_type capacity() const { return base().capacity(); }
  void reserve(size_type m) { base().resize(m); }
  void resize(size_type n, call_traits<value_type>::param_type x={}) { base().resize(n, functor<value_to_base>()(x)); }
  <const>_reference operator[](size_type n) <const> { return functor<value_from_base>()(base().operator[](n)); }
  <const>_reference at(size_type n) <const> { return functor<value_from_base>()(base().at(n)); }
};

struct vector_map_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public vector_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,FunctorsFromDerivedClasses>
{ using key_type = It::value_type::first_type; using data_type = It::value_type::second_type; using mapped_type = data_type;
  ctor(Base& c) :base{c}{}
};

struct map_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public ordered_associative_container_adaptor<Base,It,CIt,RIt,CRIt,It::value_type::first_type,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ using data_type = It::value_type::second_type; using mapped_type = data_type;
  explicit ctor(Base& c): base{c}{}
  data_type& operator[]<K2>(const K2& k) { return base()[functor<key_to_base>()(k)]; }
  <const> data_type& at<K2>(const K2& k) <const> { return base().at(functor<key_to_base>()(k)); }
};

struct multimap_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public ordered_associative_container_adaptor<Base,It,CIt,RIt,CRIt,It::value_type::first_type,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ using data_type = It::value_type::second_type; using mapped_type = data_type;
  explicit ctor(Base& c): base{c}{}
  void insert<InIt>(InIt b, InIt e) { for (; b!=e; ++b) base().insert(functor<value_to_base>()(value_type{*b})); }
  iterator insert(call_traits<value_type>::param_type x) { return base().insert(functor<value_to_base>()(x)); }
  iterator insert(iterator p, call_traits<value_type>::param_type x) { return functor<iterator_from_base>(base().insert(functor<iterator_to_base>()(p), functor<value_to_base>()(x))); }
};

struct set_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public ordered_associative_container_adaptor<Base,It,CIt,RIt,CRIt,It::value_type,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ explicit ctor(Base& c): base{c}{}
};

struct multiset_adaptor<Base,It,CIt,RIt,CRIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,RItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public ordered_associative_container_adaptor<Base,It,CIt,RIt,CRIt,It::value_type,ItToBaseCvt,ItFromBaseCvt,RItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ explicit ctor(Base& c): base{c}{}
  void insert<InIt>(InIt b, InIt e) { for (; b!=e; ++b) base().insert(functor<value_to_base>()(value_type{*b})); }
  iterator insert(call_traits<value_type>::param_type x) { return base().insert(functor<value_to_base>()(x)); }
  iterator insert(iterator p, call_traits<value_type>::param_type x) { return functor<iterator_from_base>(base().insert(functor<iterator_to_base>()(p), functor<value_to_base>()(x))); }
};

struct unordered_map_adaptor<Base,It,CIt,LIt,CLIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,LItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public unordered_associative_container_adaptor<Base,It,CIt,LIt,CLIt,It::value_type::first_type,ItToBaseCvt,ItFromBaseCvt,LItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ using data_type = It::value_type::second_type; using mapped_type = data_type;
  explicit ctor(Base& c): base{c}{}
  data_type& operator[]<K2>(const K2& k) { return base()[functor<key_to_base>()(k)]; }
  <const> data_type& at<K2>(const K2& k) <const> { return base().at(functor<key_to_base>()(k)); }
};

struct unordered_multimap_adaptor<Base,It,CIt,LIt,CLIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,LItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public unordered_associative_container_adaptor<Base,It,CIt,LIt,CLIt,It::value_type::first_type,ItToBaseCvt,ItFromBaseCvt,LItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ using data_type = It::value_type::second_type; using mapped_type = data_type;
  explicit ctor(Base& c): base{c}{}
  void insert<InIt>(InIt b, InIt e) { for (; b!=e; ++b) base().insert(functor<value_to_base>()(value_type{*b})); }
  iterator insert(call_traits<value_type>::param_type x) { return base().insert(functor<value_to_base>()(x)); }
  iterator insert(iterator p, call_traits<value_type>::param_type x) { return functor<iterator_from_base>(base().insert(functor<iterator_to_base>()(p), functor<value_to_base>()(x))); }
};

struct unordered_set_adaptor<Base,It,CIt,LIt,CLIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,LItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public unordered_associative_container_adaptor<Base,It,CIt,LIt,CLIt,It::value_type,ItToBaseCvt,ItFromBaseCvt,LItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ explicit ctor(Base& c): base{c}{}
};

struct unordered_multiset_adaptor<Base,It,CIt,LIt,CLIt,ItToBaseCvt=mpl::na,ItFromBaseCvt=mpl::na,LItFromBaseCvt=mpl::na,VToBaseCvt=mpl::na,VFromBaseCvt=mpl::na,KToBaseCvt=mpl::na,FunctorsFromDerivedClasses=mpl::vector<>>
  : public unordered_associative_container_adaptor<Base,It,CIt,LIt,CLIt,It::value_type,ItToBaseCvt,ItFromBaseCvt,LItFromBaseCvt,VToBaseCvt,VFromBaseCvt,KToBaseCvt,FunctorsFromDerivedClasses>
{ explicit ctor(Base& c): base{c}{}
  void insert<InIt>(InIt b, InIt e) { for (; b!=e; ++b) base().insert(functor<value_to_base>()(value_type{*b})); }
  iterator insert(call_traits<value_type>::param_type x) { return base().insert(functor<value_to_base>()(x)); }
  iterator insert(iterator p, call_traits<value_type>::param_type x) { return functor<iterator_from_base>(base().insert(functor<iterator_to_base>()(p), functor<value_to_base>()(x))); }
};
```

#### Views

```c++
struct detail::relation_modifier_adaptor<M,NewArg,Ext1,Ext2> : M, Ext1, Ext2 {
  using argument_type = NewArg; using result_type = void;
  ctor(const M& m) : M{m}{} ctor(const M& m, const Ext1& e1, const Ext2& e2) : M{m}, Ext1{e1}, Ext2{e2}{}
  void operator()(NewArg& x) const { M::operator()(Ext1::operator()(x), Ext2::operator()(x)); }
};
struct detail::unary_modifier_adaptor<M,NewArg,Ext> : M, Ext {
  using argument_type = NewArg; using result_type = void;
  ctor(const M& m) : M{m}{} ctor(const M& m, const Ext& e) : M{m}, Ext{e}{}
  void operator()(NewArg& x) const { M::operator()(Ext::operator()(x)); }
};

struct detail::core_iterator_type_by<Tag,BimapCore>;
struct detail::reverse_core_iterator_type_by<Tag,BimapCore>;
struct detail::local_core_iterator_type_by<Tag,BimapCore>;

struct detail::<const>_map_view_iterator_adaptor<Tag,BimapCore>
{ using type = iterator_adaptor<<const>_map_view_iterator<Tag,BimapCore>, core_iterator_type_by<Tag,BimapCore>::type, value_type_by<Tag,BimapCore>::type>; };
struct detail::<const>_map_view_iterator<Tag,BimapCore> : public <const>_map_view_iterator_adaptor<Tag,BimapCore>::type {
  ctor(){}; ctor(base_type const& iter) :base{iter}{}
  ctor(map_view_iterator<Tag,BimapCore> i) :base{i.base()}{} // const version
  reference dereference() const { return pair_by<Tag>(*(value_type*)&(*base())); } // or pair_by<Tag>(*base())

  void serialize<Archive>(Archive& ar, unsigned v) { split_member(ar, *this, v); }
  void save<Archive>(Archive& ar, unsigned) const { ar << make_nvp("mi_iterator", base()); }
  void load<Archive>(Archive& ar, unsigned) { bae_type iter; ar >> make_nvp("mi_iterator", iter); base_reference() = iter; }
};

struct detail::<const>_reverse_map_view_iterator_adaptor<Tag,BimapCore>
{ using type = iterator_adaptor<<const>_reverse_map_view_iterator<Tag,BimapCore>, reverse_core_iterator_type_by<Tag,BimapCore>::type, value_type_by<Tag,BimapCore>::type>; };
struct detail::<const>_reverse_map_view_iterator<Tag,BimapCore> : public <const>_reverse_map_view_iterator_adaptor<Tag,BimapCore>::type {
  ctor(){}; ctor(base_type const& iter) :base{iter}{}
  ctor(reverse_map_view_iterator<Tag,BimapCore> i) :base{i.base()}{} // const version
  reference dereference() const { return pair_by<Tag>(*(value_type*)&(*base())); } // or pair_by<Tag>(*base())

  void serialize<Archive>(Archive& ar, unsigned v) { split_member(ar, *this, v); }
  void save<Archive>(Archive& ar, unsigned) const { ar << make_nvp("mi_iterator", base()); }
  void load<Archive>(Archive& ar, unsigned) { bae_type iter; ar >> make_nvp("mi_iterator", iter); base_reference() = iter; }
};

struct detail::<const>_local_map_view_iterator_adaptor<Tag,BimapCore>
{ using type = iterator_adaptor<<const>_local_map_view_iterator<Tag,BimapCore>, local_core_iterator_type_by<Tag,BimapCore>::type, value_type_by<Tag,BimapCore>::type>; };
struct detail::<const>_local_map_view_iterator<Tag,BimapCore> : public <const>_local_map_view_iterator_adaptor<Tag,BimapCore>::type {
  ctor(){}; ctor(base_type const& iter) :base{iter}{}
  ctor(local_map_view_iterator<Tag,BimapCore> i) :base{i.base()}{} // const version
  reference dereference() const { return pair_by<Tag>(*(value_type*)&(*base())); }

  void serialize<Archive>(Archive& ar, unsigned v) { split_member(ar, *this, v); }
  void save<Archive>(Archive& ar, unsigned) const { ar << make_nvp("mi_iterator", base()); }
  void load<Archive>(Archive& ar, unsigned) { bae_type iter; ar >> make_nvp("mi_iterator", iter); base_reference() = iter; }
};

class detail::map_view_base<Derived,Tag,Bimap> {
  using iterator_to_base_ = iterator_facade_to_base<map_view_iterator<Tag,Bimap>,const_map_view_iterator<Tag,Bimap>>;
  using value_to_base_ = pair_to_relation_functor<Bimap::relation>;
  using key_type_ = key_type_by<Tag,Bimap>::type; using data_type_ = data_type_by<Tag,Bimap>::type; using value_type_ = pair_type_by<Tag,Bimap::relation>::type;
  using iterator_ = map_view_iterator<Tag,Bimap>;
  Derived <const>& derived() <const> { return *(Derived <const>*)this; }
public:
  bool replace(iterator_ p, const value_type_& x) { return derived().base().replace(derived().functor<iterator_to_base_>()(p), derived().functor<value_to_base_>()(x)); }
  bool replace_key<K2>(iterator_ p, const K2& k) { return derived().base().replace(derived().functor<iterator_to_base_>()(p), derived().functor<value_to_base_>()(copy_with_first_replaced(*p,k))); }
  bool replace_data<D2>(iterator_ p, const D2& d) { return derived().base().replace(derived().functor<iterator_to_base_>()(p), derived().functor<value_to_base_>()(copy_with_second_replaced(*p,d))); }
  bool modify_key<M>(iterator_ p, M mod) { return derived().base().modify_key(derived().functor<iterator_to_base_>()(p), mod); }
  bool modify_data<M>(iterator_ p, M mod) {
    using data_extractor_ = data_extractor<opposite_tag<Tag,Bimap>::type, Bimap::relation>::type;
    return derived().base().modify(derived().functor<iterator_to_base_>()(p), unary_modifier_adaptor<M,Bimap::relation,data_extractor_>(mod)); }
};

class detail::mutable_data_unique_map_view_access<Derived,Tag,Bimap> {
  using data_type_ = data_type_by<Tag,Bimap>::type;
  Derived <const>& derived() <const> { return *(Derived <const>*)this; }
public:
  <const> data_type_& at<K2>(const K2& k) <const>;
  data_type_& operator[]<K2>(const K2& k);
};
class detail::non_mutable_data_unique_map_view_access<Derived,Tag,Bimap> {
  using data_type_ = data_type_by<Tag,Bimap>::type;
  Derived <const>& derived() <const> { return *(Derived <const>*)this; }
public:
  const data_type_& at<K2>(const K2& k) const;
  data_type_& operator[]<K2>(const K2& k) = delete;
};
struct detail::unique_map_view_access<Derived,Tag,Bimap>
{ using type = mpl::if_<is_const<value_type_by<Tag,Bimap>::type>, non_mutable_data_unique_map_view_access<Derived,Tag,Bimap>, mutable_data_unique_map_view_access<Derived,Tag,Bimap>>; };
struct detail::left_map_view_extra_typedefs<MapView>{};
struct detail::right_map_view_extra_typedefs<MapView>{};
const T& make_const<T>(const T& t) { return t; }


struct detail::<const>_set_view_iterator_base<CoreIt> { using type = iterator_adaptor<<const>_set_view_iterator<CoreIt>,CoreIt,CoreIt::value_type::above_view>; };
struct detail::<const>_set_view_iterator<CoreIt> : public <const>_set_view_iterator_base<CoreIt>::type {
  ctor(){} ctor(CoreIt const& iter) :base{iter}{}
  ctor(self const& i) :base{i.base()}{}
  ctor(set_view_iterator<CoreIt> i) :base{i.base()}{} // const version
  reference dereference() const { return ((value_type*)&*base())->get_view(); } // or base()->get_view()

  void serialize<Archive>(Archive& ar, unsigned v) { split_member(ar, *this, v); }
  void save<Archive>(Archive& ar, unsigned) const { ar << make_nvp("mi_iterator", base()); }
  void load<Archive>(Archive& ar, unsigned) { CoreIt iter; ar >> make_nvp("mi_iterator", iter); base_reference() = iter; }
};

struct detail::set_view_key_to_base<Key,Value,KeyToBase> { const Key operator()(const Value& v) const { return keyToBase(v); } private: KeyToBase keyToBase; };
struct detail::set_view_key_to_base<MRelStorage,MRelStorage,KeyToBase> {
  using non_mutable_storage = MRelStorage::non_mutable_storage;
  const MRelStorage& operator()(const non_mutable_storage& k) const { return mutate<MRelStorage>(k); }
  const MRelStorage& operator()(const MRelStorage& k) const { return k; }
};

class detail::set_view_base<Derived,Index> {
  using iterator_to_base_ = iterator_facade_to_base<set_view_iterator<Index::iterator>,const_set_view_iterator<Index::const_iterator>>;
  using value_type_ = Index::value_type; using left_type_ = value_type_::left_value_type; using right_type_ = value_type_::right_value_type;
  using iterator_ = set_view_iterator<Index::iterator>;

  Derived <const>& derived() <const> { return *(Derived <const>*)this; }
public:
  bool replace(iterator_ p, const value_type_& x) { return derived().base().replace(derived().functor<iterator_to_base_>()(p), x); }
  bool replace_left<LT>(iterator_ p, const LT& l) { return derived().base().replace(derived().functor<iterator_to_base_>()(p), copy_with_left_replaced(*p,l)); }
  bool replace_right<RT>(iterator_ p, const RT& r) { return derived().base().replace(derived().functor<iterator_to_base_>()(p), copy_with_right_replaced(*p,r)); }
};


namespace views;

struct list_map_view_base<Tag,Bimap>
{ using type = list_map_adaptor<Bimap::Index<Tag>::type, map_view_iterator<Tag,Bimap>, const_map_view_iterator<Tag,Bimap>, reverse_map_view_iterator<Tag,Bimap>, const_reverse_map_view_iterator<Tag,Bimap>,
    iterator_facade_to_base<map_view_iterator<Tag,Bimap>,const_map_view_iterator<Tag,Bimap>>, mpl::na, mpl::na,
    pair_to_relation_functor<Tag,Bimap::relation>, get_pair_functor<Tag,Bimap::relation>, data_extractor<Tag,Bimap::relation>::type>; };
struct list_map_view<Tag,Bimap> : public list_map_view_base<Tag,Bimap>::type, public map_view_base<self,Tag,Bimap>
{ using info_type = value_type::info_type;
  ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; }
  void assign<InIt>(InIt f, InIt l) { clear(); insert(end(), f, l); }
  void assign(size_type n, value_type& v) { clear(); for (size_type i=0; i<n; ++i) push_back(v); }
  <const>_reference {front|back}() <const> { return functor<value_from_base>()(<(value_type&)>base().{front|back}()); }
  void relocate(iterator p, iterator i) { base().relocate(functor<iterator_to_base>()(p), functor<iterator_to_base>()(i)); }
  void relocate(iterator p, iterator f, iterator l) { base().relocate(functor<iterator_to_base>()(p), functor<iterator_to_base>()(f), functor<iterator_to_base>()(l)); }
};
struct detail::left_map_view_extra_typedefs<list_map_view<Tag,Bimap>> { using left_<const>_reverse_iterator = list_map_view<Tag,Bimap>::<const>_reverse_iterator; };
struct detail::right_map_view_extra_typedefs<list_map_view<Tag,Bimap>> { using right_<const>_reverse_iterator = list_map_view<Tag,Bimap>::<const>_reverse_iterator; };

struct list_set_view<CoreIndex> : public list_adaptor<CoreIndex,
  set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>, set_view_iterator<CoreIndex::reverse_iterator>, const_set_view_iterator<CoreIndex::const_reverse_iterator>,
  iterator_facade_to_base<set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>>, mpl::na, mpl::na,
  get_mutable_relation_functor<CoreIndex::value_type>, get_above_view_functor<CoreIndex::value_type>>, public set_view_base<self, CoreIndex>
{ ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; }
  void assign<InIt>(InIt f, InIt l) { clear(); insert(end(), f, l); }
  void assign(size_type n, value_type& v) { clear(); for (size_type i=0; i<n; ++i) push_back(v); }
  <const>_reference {front|back}() <const> { return functor<value_from_base>()(<(value_type&)>base().{front|back}()); }
  void relocate(iterator p, iterator i) { base().relocate(functor<iterator_to_base>()(p), functor<iterator_to_base>()(i)); }
  void relocate(iterator p, iterator f, iterator l) { base().relocate(functor<iterator_to_base>()(p), functor<iterator_to_base>()(f), functor<iterator_to_base>()(l)); }
};

struct vector_map_view<Tag,Bimap> : public vector_map_adaptor<Bimap::core_type::index<Tag>::type,
  map_view_iterator<Tag,Bimap>, const_map_view_iterator<Tag,Bimap>, reverse_map_view_iterator<Tag,Bimap>, const_reverse_map_view_iterator<Tag,Bimap>,
  iterator_facade_to_base<map_view_iterator<Tag,Bimap>,const_map_view_iterator<Tag,Bimap>>, mpl::na,mpl::na, pair_to_relation_functor<Bimap::relation>, get_pair_functor<Tag,Bimap::relation>>,
  public map_view_base<self,Tag,Bimap>
{ using key_from_base_value = data_extractor<Tag,Bimap::relation>::type; using info_type = value_type::info_type;
  ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; }
  <const>_reference operator[](size_type n) <const> { return functor<value_from_base>()(base().operator[](n)); }
  <const>_reference at(size_type n) <const> { return functor<value_from_base>()(base().at(n)); }
  void assign<InIt>(InIt f, InIt l) { clear(); insert(end(), f, l); }
  void assign(size_type n, value_type& v) { clear(); for (size_type i=0; i<n; ++i) push_back(v); }
  <const>_reference {front|back}() <const> { return functor<value_from_base>()(<(value_type&)>base().{front|back}()); }
  void splice(iterator p, self& x) { base().splice(functor<iterator_to_base>()(p), x.base()); }
  void splice(iterator p, self& x, iterator i) { base().splice(functor<iterator_to_base>()(p), x.base(), functor<iterator_to_base>()(i)); }
  void splice(iterator p, self& x, iterator f, iterator l) { base().splice(functor<iterator_to_base>()(p), x.base(), functor<iterator_to_base>()(f), functor<iterator_to_base>()(l)); }
  void remove(call_traits<value_type>::param_type v) { base().remove(functor<value_to_base>()(v)); }
  void remove_if<Pred>(Pred pred) { base().remove_if(unary_check_adaptor<Pred,value_type,key_from_base_value>{pred, {}}); }
  void unique() { base().unique(comparison_adaptor<std::equal_to<key_type>,value_type,key_from_base_value>{{}, {}}); }
  void unique<BPred>(BPred bpred) { base().unique(comparison_adaptor<BPred,value_type,key_from_base_value>{bpred, {}}); }
  void merge(self& x) { base().merge(x.base(), comparison_adaptor<std::less<key_type>,value_type,key_from_base_value>{{}, {}}); }
  void merge<Compare>(self& x, Compare comp) { base().merge(x.base(), comparison_adaptor<Compare,value_type,key_from_base_value>{comp, {}}); }
  void sort() { base().sort(comparison_adaptor<std::less<key_type>,value_type,key_from_base_value>{{}, {}}); }
  void sort<Compare>(Compare comp) { base().sort(comparison_adaptor<Compare,value_type,key_from_base_value>{comp, {}}); }
  void reverse() { base.reverse(); }
  void relocate(iterator p, iterator i) { base().relocate(functor<iterator_to_base>()(p), functor<iterator_to_base>()(i)); }
  void relocate(iterator p, iterator f, iterator l) { base().relocate(functor<iterator_to_base>()(p), functor<iterator_to_base>()(f), functor<iterator_to_base>()(l)); }
};
struct detail::left_map_view_extra_typedefs<vector_map_view<Tag,Bimap>> { using left_<const>_reverse_iterator = list_map_view<Tag,Bimap>::<const>_reverse_iterator; };
struct detail::right_map_view_extra_typedefs<vector_map_view<Tag,Bimap>> { using right_<const>_reverse_iterator = list_map_view<Tag,Bimap>::<const>_reverse_iterator; };

struct vector_set_view<CoreIndex> : public vector_adaptor<CoreIndex,
  set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>, set_view_iterator<CoreIndex::reverse_iterator>, const_set_view_iterator<CoreIndex::const_reverse_iterator>,
  iterator_facade_to_base<set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>>, mpl::na, mpl::na,
  get_mutable_relation_functor<CoreIndex::value_type>, get_above_view_functor<CoreIndex::value_type>>, public set_view_base<self, CoreIndex>
{ ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; }
  <const>_reference operator[](size_type n) <const> { return functor<value_from_base>()(base().operator[](n)); }
  <const>_reference at(size_type n) <const> { return functor<value_from_base>()(base().at(n)); }
  void assign<InIt>(InIt f, InIt l) { clear(); insert(end(), f, l); }
  void assign(size_type n, value_type& v) { clear(); for (size_type i=0; i<n; ++i) push_back(v); }
  <const>_reference {front|back}() <const> { return functor<value_from_base>()(<(value_type&)>base().{front|back}()); }
  void splice(iterator p, self& x) { base().splice(functor<iterator_to_base>()(p), x.base()); }
  void splice(iterator p, self& x, iterator i) { base().splice(functor<iterator_to_base>()(p), x.base(), functor<iterator_to_base>()(i)); }
  void splice(iterator p, self& x, iterator f, iterator l) { base().splice(functor<iterator_to_base>()(p), x.base(), functor<iterator_to_base>()(f), functor<iterator_to_base>()(l)); }
  void remove(call_traits<value_type>::param_type v) { base().remove(functor<value_to_base>()(v)); }
  void remove_if<Pred>(Pred pred) { base().remove_if(unary_check_adaptor<Pred,value_type,value_from_base>{pred, functor<value_from_base>()}); }
  void unique() { base().unique(comparison_adaptor<std::equal_to<value_type>,value_type,value_from_base>{{}, functor<value_from_base>()}); }
  void unique<BPred>(BPred bpred) { base().unique(comparison_adaptor<BPred,value_type,value_from_base>{bpred, functor<value_from_base>()}); }
  void merge(self& x) { base().merge(x.base(), comparison_adaptor<std::less<value_type>,value_type,value_from_base>{{}, functor<value_from_base>()}); }
  void merge<Compare>(self& x, Compare comp) { base().merge(x.base(), comparison_adaptor<Compare,value_type,value_from_base>{comp, functor<value_from_base>()}); }
  void sort() { base().sort(comparison_adaptor<std::less<value_type>,value_type,value_from_base>{{}, functor<value_from_base>()}); }
  void sort<Compare>(Compare comp) { base().sort(comparison_adaptor<Compare,value_type,value_from_base>{comp, functor<value_from_base>()}); }
  void reverse() { base.reverse(); }
  void relocate(iterator p, iterator i) { base().relocate(functor<iterator_to_base>()(p), functor<iterator_to_base>()(i)); }
  void relocate(iterator p, iterator f, iterator l) { base().relocate(functor<iterator_to_base>()(p), functor<iterator_to_base>()(f), functor<iterator_to_base>()(l)); }
};

struct unconstrained_map_view<Tag,Bimap> {
  ctor<T>(const T&) {}
  using <const>_iterator = void; using <const>_reference = void; using info_type = void;
};

struct unconstrained_set_view<CoreIndex> {
  ctor<T>(const T&) {}
  using <const>_iterator = void;
};

struct map_view<Tag,Bimap> : public map_adaptor<Bimap::core_type::index<Tag>::type,
  map_view_iterator<Tag,Bimap>, const_map_view_iterator<Tag,Bimap>, reverse_map_view_iterator<Tag,Bimap>, const_reverse_map_view_iterator<Tag,Bimap>,
  iterator_facade_to_base<map_view_iterator<Tag,Bimap>,const_map_view_iterator<Tag,Bimap>>, mpl::na,mpl::na, pair_to_relation_functor<Bimap::relation>, get_pair_functor<Tag,Bimap::relation>>,
  public map_view_base<self,Tag,Bimap>, public unique_map_view_access<self,Tag,Bimap>::type
{ using info_type = value_type::info_type;
  ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; }
  using base::at; using base::operator[];
  <const> info_type& info_at<K2>(const K2& k) <const>;
};
struct detail::left_map_view_extra_typedefs<map_view<Tag,Bimap>>
{ using left_<const>_reverse_iterator = map_view<Tag,Bimap>::<const>_reverse_iterator;
  using left_<const>_range = map_view<Tag,Bimap>::<const>_range;
  using left_key_cmopare = map_view<Tag,Bimap>::key_compare; };
struct detail::right_map_view_extra_typedefs<map_view<Tag,Bimap>>
{ using right_<const>_reverse_iterator = map_view<Tag,Bimap>::<const>_reverse_iterator;
  using right_<const>_range = map_view<Tag,Bimap>::<const>_range;
  using right_key_cmopare = map_view<Tag,Bimap>::key_compare; };

struct multimap_view<Tag,Bimap> : public multimap_adaptor<Bimap::core_type::index<Tag>::type,
  map_view_iterator<Tag,Bimap>, const_map_view_iterator<Tag,Bimap>, reverse_map_view_iterator<Tag,Bimap>, const_reverse_map_view_iterator<Tag,Bimap>,
  iterator_facade_to_base<map_view_iterator<Tag,Bimap>,const_map_view_iterator<Tag,Bimap>>, mpl::na,mpl::na, pair_to_relation_functor<Bimap::relation>, get_pair_functor<Tag,Bimap::relation>>,
  public map_view_base<self,Tag,Bimap>
{ using info_type = value_type::info_type;
  ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; }
  using <const>_range_type = std::pair<<const>_iterator,<const>_iterator>;
  <const>_range_type range<LB,UB>(LB lower, UB upper) <const>
  { auto r = base().range(lower,upper); return <const>_range_type{functor<iterator_from_base>()(r.first), functor<iterator_from_base>()(r.second)}; }
  void insert<InIt>(InIt b, InIt e) { for(; b!=e; ++b) { base().insert(functor<value_to_base>()(value_type(*b))); } }
  std::pair<iterator,bool> insert(call_traits<value_type>::param_type x) { auto r = base().insert(functor<value_to_base>()(x)); return {functor<iterator_from_base>()(r.first), r.second}; }
  iterator insert(iterator p, call_traits<value_type>::param_type x) { return functor<iterator_from_base>()(base().insert(functor<iterator_to_base>()(p), functor<value_to_base>()(x)) ); }
};
struct detail::left_map_view_extra_typedefs<multimap_view<Tag,Bimap>>
{ using left_<const>_reverse_iterator = multimap_view<Tag,Bimap>::<const>_reverse_iterator;
  using left_<const>_range = multimap_view<Tag,Bimap>::<const>_range;
  using left_key_cmopare = multimap_view<Tag,Bimap>::key_compare; };
struct detail::right_map_view_extra_typedefs<multimap_view<Tag,Bimap>>
{ using right_<const>_reverse_iterator = multimap_view<Tag,Bimap>::<const>_reverse_iterator;
  using right_<const>_range = multimap_view<Tag,Bimap>::<const>_range;
  using right_key_cmopare = multimap_view<Tag,Bimap>::key_compare; };

struct set_view<CoreIndex> : public set_adaptor<CoreIndex,
  set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>, set_view_iterator<CoreIndex::reverse_iterator>, const_set_view_iterator<CoreIndex::const_reverse_iterator>,
  iterator_facade_to_base<set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>>, mpl::na, mpl::na,
  get_mutable_relation_functor<CoreIndex::value_type>, get_above_view_functor<CoreIndex::value_type>, set_view_key_to_base<CoreIndex::key_type, CoreIndex::value_type, CoreIndex::key_from_value>>,
  public set_view_base<self,CoreIndex>
{ ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; } };

struct multiset_view<CoreIndex> : public multiset_adaptor<CoreIndex,
  set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>, set_view_iterator<CoreIndex::reverse_iterator>, const_set_view_iterator<CoreIndex::const_reverse_iterator>,
  iterator_facade_to_base<set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>>, mpl::na, mpl::na,
  get_mutable_relation_functor<CoreIndex::value_type>, get_above_view_functor<CoreIndex::value_type>, set_view_key_to_base<CoreIndex::key_type, CoreIndex::value_type, CoreIndex::key_from_value>>,
  public set_view_base<self,CoreIndex>
{ ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; }
  void insert<InIt>(InIt b, InIt e) { for(; b!=e; ++b) { base().insert(functor<value_to_base>()(value_type(*b))); } }
  std::pair<iterator,bool> insert(call_traits<value_type>::param_type x) { auto r = base().insert(functor<value_to_base>()(x)); return {functor<iterator_from_base>()(r.first), r.second}; }
  iterator insert(iterator p, call_traits<value_type>::param_type x) { return functor<iterator_from_base>()(base().insert(functor<iterator_to_base>()(p), functor<value_to_base>()(x)) ); }
};


struct unordered_map_view<Tag,Bimap> : public map_adaptor<Bimap::core_type::index<Tag>::type,
  map_view_iterator<Tag,Bimap>, const_map_view_iterator<Tag,Bimap>, local_map_view_iterator<Tag,Bimap>, const_local_map_view_iterator<Tag,Bimap>,
  iterator_facade_to_base<map_view_iterator<Tag,Bimap>,const_map_view_iterator<Tag,Bimap>>, mpl::na,mpl::na, pair_to_relation_functor<Bimap::relation>, get_pair_functor<Tag,Bimap::relation>>,
  public map_view_base<self,Tag,Bimap>, public unique_map_view_access<self,Tag,Bimap>::type
{ using <const>_range_type = std::pair<<const>_iterator,<const>_iterator>; using info_type = value_type::info_type;
  ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; }
  using base::at; using base::operator[];
  <const> info_type& info_at<K2>(const K2& k) <const>;
};
struct detail::left_map_view_extra_typedefs<unordered_map_view<Tag,Bimap>>
{ using left_<const>_local_iterator = unordered_map_view<Tag,Bimap>::<const>_local_iterator;
  using left_<const>_range = unordered_map_view<Tag,Bimap>::<const>_range;
  using left_hasher = unordered_map_view<Tag,Bimap>::hasher;
  using left_key_equal = unordered_map_view<Tag,Bimap>::key_equal; };
struct detail::right_map_view_extra_typedefs<unordered_map_view<Tag,Bimap>>
{ using right_<const>_local_iterator = unordered_map_view<Tag,Bimap>::<const>_local_iterator;
  using right_<const>_range = unordered_map_view<Tag,Bimap>::<const>_range;
  using right_hasher = unordered_map_view<Tag,Bimap>::hasher;
  using right_key_cmopare = unordered_map_view<Tag,Bimap>::key_compare; };

struct unordered_multimap_view<Tag,Bimap> : public unordered_multimap_adaptor<Bimap::core_type::index<Tag>::type,
  map_view_iterator<Tag,Bimap>, const_map_view_iterator<Tag,Bimap>, local_map_view_iterator<Tag,Bimap>, const_local_map_view_iterator<Tag,Bimap>,
  iterator_facade_to_base<map_view_iterator<Tag,Bimap>,const_map_view_iterator<Tag,Bimap>>, mpl::na,mpl::na, pair_to_relation_functor<Bimap::relation>, get_pair_functor<Tag,Bimap::relation>>,
  public map_view_base<self,Tag,Bimap>
{ using <const>_range_type = std::pair<<const>_iterator,<const>_iterator>; using info_type = value_type::info_type;
  ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; }
  using <const>_range_type = std::pair<<const>_iterator,<const>_iterator>;
  void insert<InIt>(InIt b, InIt e) { for(; b!=e; ++b) { base().insert(functor<value_to_base>()(value_type(*b))); } }
  std::pair<iterator,bool> insert(call_traits<value_type>::param_type x) { auto r = base().insert(functor<value_to_base>()(x)); return {functor<iterator_from_base>()(r.first), r.second}; }
  iterator insert(iterator p, call_traits<value_type>::param_type x) { return functor<iterator_from_base>()(base().insert(functor<iterator_to_base>()(p), functor<value_to_base>()(x)) ); }
};
struct detail::left_map_view_extra_typedefs<unordered_map_view<Tag,Bimap>>
{ using left_<const>_reverse_iterator = unordered_map_view<Tag,Bimap>::<const>_reverse_iterator;
  using left_<const>_range = unordered_map_view<Tag,Bimap>::<const>_range;
  using left_hasher = unordered_map_view<Tag,Bimap>::hasher;
  using left_key_equal = unordered_map_view<Tag,Bimap>::key_equal; };
struct detail::right_map_view_extra_typedefs<unordered_map_view<Tag,Bimap>>
{ using right_<const>_reverse_iterator = unordered_map_view<Tag,Bimap>::<const>_reverse_iterator;
  using right_<const>_range = unordered_map_view<Tag,Bimap>::<const>_range;
  using right_hasher = unordered_map_view<Tag,Bimap>::hasher;
  using right_key_cmopare = unordered_map_view<Tag,Bimap>::key_compare; };

struct unordered_set_view<CoreIndex> : public unordered_set_adaptor<CoreIndex,
  set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>, set_view_iterator<CoreIndex::local_iterator>, const_set_view_iterator<CoreIndex::const_local_iterator>,
  iterator_facade_to_base<set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>>, mpl::na, mpl::na,
  get_mutable_relation_functor<CoreIndex::value_type>, get_above_view_functor<CoreIndex::value_type>, set_view_key_to_base<CoreIndex::key_type, CoreIndex::value_type, CoreIndex::key_from_value>>,
  public set_view_base<self,CoreIndex>
{ ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; } };

struct unordered_multiset_view<CoreIndex> : public unordered_multiset_adaptor<CoreIndex,
  set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>, set_view_iterator<CoreIndex::local_iterator>, const_set_view_iterator<CoreIndex::const_local_iterator>,
  iterator_facade_to_base<set_view_iterator<CoreIndex::iterator>, const_set_view_iterator<CoreIndex::const_iterator>>, mpl::na, mpl::na,
  get_mutable_relation_functor<CoreIndex::value_type>, get_above_view_functor<CoreIndex::value_type>, set_view_key_to_base<CoreIndex::key_type, CoreIndex::value_type, CoreIndex::key_from_value>>,
  public set_view_base<self,CoreIndex>
{ ctor(base& c) base{c}{} self& operator=(const self& v) { base()=v.base(); return *this; }
  void insert<InIt>(InIt b, InIt e) { for(; b!=e; ++b) { base().insert(functor<value_to_base>()(value_type(*b))); } }
  std::pair<iterator,bool> insert(call_traits<value_type>::param_type x) { auto r = base().insert(functor<value_to_base>()(x)); return {functor<iterator_from_base>()(r.first), r.second}; }
  iterator insert(iterator p, call_traits<value_type>::param_type x) { return functor<iterator_from_base>()(base().insert(functor<iterator_to_base>()(p), functor<value_to_base>()(x)) ); }
};
```

#### Bimap Core

```c++
struct detail::set_type_of_tag{}; struct detail::set_type_of_relation_tag{}; struct detail::side_based_tag : set_type_of_relation_tag{};
struct left_based : side_based_tag { struct bind_to<Relation>{using type=void;}; using left_mutable_key = bool_<true>; using right_mutable_key=bool_<true>; };
struct right_based : side_based_tag { struct bind_to<Relation>{using type=void;}; using left_mutable_key = bool_<true>; using right_mutable_key=bool_<true>; };
using _relation = mpl::_;
struct detail::is_set_type_of<Type> : is_base_of<set_type_of_tag,Type>{};
struct detail::is_set_type_of_relation<Type> : is_base_of<set_type_of_relation_tag,Type>{};

struct detail::manage_bimap_key<Type> { using set_type = mpl::eval_if<is_set_type_of<Type>::type,mpl::identity<Type>,mpl::identity<set_of<Type>>>::type; using type = set_type; };
struct with_info<Type> { using vaule_type = Type; };
struct detail::is_with_info<Type>;
struct detail::manage_additional_parameters<AP1,AP2,AP3> { struct type { using set_type_of_relation = left_based; using allocator = std::allocator<char>; using additional_info = mpl::na; }; };


struct list_of<Type> : public set_type_of_tag {
  using user_type = Type; using value_type = value_type_of<user_type>::type;
  struct lazy_concept_checked;
  struct index_bind<KeyExt,Tag> { using type = multi_index::sequenced<multi_index::tag<Tag>>; };
  struct map_view_bind<Tag,Bimap>{ using type = views::list_map_view<Tag,Bimap>; };
  struct set_view_bind<Index>{ using type = views::list_set_view<Index>; };
  using mutable_key = bool_<true>;
};
struct list_of_relation : public set_type_of_relation_tag {
  struct bind_to<Relation> { using type = list_of<Relation>; };
  using left_mutable_key = bool_<true>; using right_mutable_key = bool_<true>;
};

struct vector_of<Type> : public set_type_of_tag {
  using user_type = Type; using value_type = value_type_of<user_type>::type;
  struct lazy_concept_checked;
  struct index_bind<KeyExt,Tag> { using type = multi_index::random_access<multi_index::tag<Tag>>; };
  struct map_view_bind<Tag,Bimap>{ using type = views::vector_map_view<Tag,Bimap>; };
  struct set_view_bind<Index>{ using type = views::vector_map_view<Index>; };
  using mutable_key = bool_<true>;
};
struct vector_of_relation : public set_type_of_relation_tag {
  struct bind_to<Relation> { using type = vector_of<Relation>; };
  using left_mutable_key = bool_<true>; using right_mutable_key = bool_<true>;
};

struct set_of<Key,KeyCompare=std::less<value_type_of<Key>::type>> : public set_type_of_tag {
  using user_type = Key; using value_type = value_type_of<user_type>::type; using key_compare = KeyCompare;
  struct lazy_concept_checked;
  struct index_bind<KeyExt,Tag> { using type = multi_index::ordered_unique<multi_index::tag<Tag>,KeyExt,key_compare>; };
  struct map_view_bind<Tag,Bimap>{ using type = views::map_view<Tag,Bimap>; };
  struct set_view_bind<Index>{ using type = views::set_view<Index>; };
  using mutable_key = bool_<false>;
};
struct set_of_relation<KeyCompare=std::less<_relation>> : public set_type_of_relation_tag {
  using key_compare = KeyCompare;
  struct bind_to<Relation> { using type = set_of<Relation,mpl::apply<key_compare,Relation::storage_base>::type>; };
  using left_mutable_key = bool_<false>; using right_mutable_key = bool_<false>;
};

struct multiset_of<Key,KeyCompare=std::less<value_type_of<Key>::type>> : public set_type_of_tag {
  using user_type = Key; using value_type = value_type_of<user_type>::type; using key_compare = KeyCompare;
  struct lazy_concept_checked;
  struct index_bind<KeyExt,Tag> { using type = multi_index::ordered_non_unique<multi_index::tag<Tag>,KeyExt,key_compare>; };
  struct map_view_bind<Tag,Bimap>{ using type = views::multimap_view<Tag,Bimap>; };
  struct set_view_bind<Index>{ using type = views::multiset_view<Index>; };
  using mutable_key = bool_<false>;
};
struct multiset_of_relation<KeyCompare=std::less<_relation>> : public set_type_of_relation_tag {
  using key_compare = KeyCompare;
  struct bind_to<Relation> { using type = multiset_of<Relation,mpl::apply<key_compare,Relation::storage_base>::type>; };
  using left_mutable_key = bool_<false>; using right_mutable_key = bool_<false>;
};

struct unordered_set_of<Key,Hash=hash<value_type_of<Key>::type>,EqualKey=std::equal_to<value_type_of<Key>::type>> : public set_type_of_tag {
  using user_type = Key; using value_type = value_type_of<user_type>::type; using hasher = Hash; using key_equal = EqualKey;
  struct lazy_concept_checked;
  struct index_bind<KeyExt,Tag> { using type = multi_index::hashed_unique<multi_index::tag<Tag>,KeyExt,hasher,key_equal>; };
  struct map_view_bind<Tag,Bimap>{ using type = views::unordered_map_view<Tag,Bimap>; };
  struct set_view_bind<Index>{ using type = views::unordered_set_view<Index>; };
  using mutable_key = bool_<false>;
};
struct unordered_set_of_relation<Hash=hash<_relation>,EqualKey=std::equal_to<_relation>> : public set_type_of_relation_tag {
  using hasher = Hash; using key_equal = EqualKey;
  struct bind_to<Relation> { using type = unordered_set_of<Relation, mpl::apply<hasher,Relation::storage_base>::type, mpl::apply<key_equal,Relation::storage_base>::type>; };
  using left_mutable_key = bool_<false>; using right_mutable_key = bool_<false>;
};

struct unordered_multiset_of<Key,Hash=hash<value_type_of<Key>::type>,EqualKey=std::equal_to<value_type_of<Key>::type>> : public set_type_of_tag {
  using user_type = Key; using value_type = value_type_of<user_type>::type; using hasher = Hash; using key_equal = EqualKey;
  struct lazy_concept_checked;
  struct index_bind<KeyExt,Tag> { using type = multi_index::hashed_unique<multi_index::tag<Tag>,KeyExt,hasher,key_equal>; };
  struct map_view_bind<Tag,Bimap>{ using type = views::unordered_multimap_view<Tag,Bimap>; };
  struct set_view_bind<Index>{ using type = views::unordered_multiset_view<Index>; };
  using mutable_key = bool_<false>;
};
struct unordered_multiset_of_relation<Hash=hash<_relation>,EqualKey=std::equal_to<_relation>> : public set_type_of_relation_tag {
  using hasher = Hash; using key_equal = EqualKey;
  struct bind_to<Relation> { using type = unordered_multiset_of<Relation, mpl::apply<hasher,Relation::storage_base>::type, mpl::apply<key_equal,Relation::storage_base>::type>; };
  using left_mutable_key = bool_<false>; using right_mutable_key = bool_<false>;
};

struct unconstrained_set_of<Key> : public set_type_of_tag {
  using user_type = Key; using value_type = value_type_of<user_type>::type;
  struct lazy_concept_checked;
  struct index_bind<KeyExt,Tag> { using type = void; };
  struct map_view_bind<Tag,Bimap>{ using type = views::unconstrained_map_view<Tag,Bimap>; };
  struct set_view_bind<Index>{ using type = views::unconstrained_set_view<Index>; };
  using mutable_key = bool_<true>;
};
struct unconstrained_set_of_relation : public set_type_of_relation_tag {
  struct bind_to<Relation> { using type = unconstrained_set_of<Relation>; };
  using left_mutable_key = bool_<true>; using right_mutable_key = bool_<true>;
};

struct detail::get_value_type<Type> { using type = Type::value_type; };
struct detail::independent_index_tag{};
struct detail::bimap_core<LSet,RSet,AP1,AP2,AP3> {
	using {left|right}_set_type = manage_bimap_key<{L|R}Set>::type;
	using {left|right}_tagged_type = default_tagged<{left|right}_set_type::user_type, member_at::{left|right}>::type;
	using {left|right}_tag = {left|right}_tagged_type::tag;
	using {left|right}_key_type = {left|right}_set_type::value_type;
	using left_data_type = right_key_type; using right_data_type = left_key_type;
	using parameters = manage_additional_parameters<AP1,AP2,AP3>::type;
	using relation = mutant_relation<tagged<mpl::if_<and_<left_set_type::mutable_key,parameters::set_type_of_relation::left_mutable_key>, left_key_type, add_const_t<left_key_type>>::type, left_tag>,
									tagged<mpl::if_<and_<right_set_type::mutable_key,parameters::set_type_of_relation::right_mutable_key>, right_key_type, add_const_t<right_key_type>>::type, right_tag>,
									parameters::additional_info, true>;
	using {left|right}_value_type = relation::{left|right}_pair;
private:
	using {left|right}_member_extractor = multi_index::member< relation::storage_base,{left|right}_key_type,&relation::storage_base::{left|right}>
	using left_core_indices = mpl::if_<is_unconstrained_set_of<left_set_type>,mpl::vector<>,mpl::vector<left_set_type::index_bind<left_member_extractor,left_tag>::type>>::type;
	using basic_core_indices = mpl::if_<is_unconstrained_set_of<right_set_type>,left_core_indices,mpl::push_front<left_core_indices,right_set_type::index_bind<right_member_extractor,right_tag>::type>::type>::type;
	using tagged_set_of_relation_type = mpl::if_<is_same<parameters::set_type_of_relation,left_based>, tagged<left_set_type,left_tag>,
										mpl::if_<is_same<parameters::set_type_of_relation,right_based>, tagged<right_set_type,right_tag>,
											tagged<parameters::set_type_of_relation::bind_to<relation>::type,independent_index_tag>>::type>::type;
protected:
	using relation_set_tag = tagged_set_of_relation_type::tag;
	using relation_set_type_of = tagged_set_of_relation_type::value_type;
	using logic_left_tag = mpl::if_<is_unconstrained_set_of<left_set_type>, mpl::if_<is_unconstrained_set_of<right_set_type>, independent_index_tag, right_tag>::type, left_tag>::type;
	using logic_right_tag = mpl::if_<is_unconstrained_set_of<right_set_type>, mpl::if_<is_unconstrained_set_of<left_set_type>, independent_index_tag, left_tag>::type, right_tag>::type;
	using logic_relation_set_tag = mpl::if_<is_same<relation_set_tag,independent_index_tag>,
							mpl::if_<is_unconstrained_set_of<relation_set_type_of>, logic_left_tag,independent_index_tag>::type,
							mpl::if_<is_same<parameters::set_type_of_relation,left_based>, logic_left_tag, logic_right_tag>::type>::type;
private:
	using complete_core_indices = mpl::if_<and_<is_same<relation_set_tag,independent_index_tag>,not_<is_unconstrained_set_of<relation_set_type_of>>>,
					push_front<basic_core_indices, relation_set_type_of::index_bind<both_keys_extractor<relation>,independent_index_tag>::type>::type, basic_core_indices>::type;
	struct core_indices : public complete_core_indices{};
public:
	using core_type = multi_index::multi_index_container<relation,core_indices,allocator_rebind<parameters::allocator,relation>::type>;
	using {left|right}_index = multi_index::index<core_type, logic_{left|right}_tag>::type;
	using {left|right}_core_<const>_iterator = {left|right}_index::<const>_iterator;
	using relation_set_core_index = multi_index::index<core_type,logic_relation_set_tag>::type;
	using relation_set = relation_set_type_of::set_view_bind<relation_set_core_index>::type;
};

struct detail::left_map_view_type<BimapBase> {
	using left_set_type = BimapBase::left_set_type;
	using type = left_set_type::map_view_bind<BimapBase::left_tag,BimapBase>::type;
};
struct detail::right_map_view_type<BimapBase> {
	using right_set_type = BimapBase::right_set_type;
	using type = right_set_type::map_view_bind<BimapBase::right_tag,BimapBase>::type;
};
```

#### Supports

```c++
struct support::data_type_by<Tag,Bimap>; // {left|right}_data_type
struct support::key_type_by<Tag,Bimap>; // {left|right}_key_type
struct support::map_type_by<Tag,Bimap>; // {left|right}_map
struct support::value_type_by<Tag,Bimap>; // {left|right}_value_type
struct support::<const>_iterator_type_by<Tag,Bimap>; // {left|right}_<const>_iterator
struct support::<const>_reverse_iterator_type_by<Tag,Bimap>; // {left|right}_<const>_reverse_iterator
struct support::<const>_local_iterator_type_by<Tag,Bimap>; // {left|right}_<const>_local_iterator
struct support::result_of::map_by<Tag,Bimap>;
result_of::map_by<Tag,Bimap>::type support::map_by<Tag,Bimap>(Bimap& bimap);
auto& _key = lambda::_1; auto& _data = lambda::_2;

class bimap<KeyA,KeyB,AP1=mpl::na,AP2=mpl::na,AP3=mpl::na>
	: public bimap_core<KeyA,KeyB,AP1,AP2,AP3>, public bimap_core::relation_set,
	public left_map_view_extra_typedefs<left_map_view_type<bimap_core>::type>,
	public right_map_view_extra_typedefs<right_map_view_type<bimap_core>::type>
{	core_type core;
public:
	using {left|right}_map = {left|right}_map_view_type<bimap_core>::type;
	using {left|right}_<const>_iterator = {left|right}_map::<const>_iterator;
	using {left|right}_<const>_reference = {left|right}_map::<const>_reference;
	using info_type = relation::info_type; using allocator_type = core_type::allocator_type;
	left_map left; right_map right;
	using ctor_args_list = core_type::ctor_args_list;
	ctor(const allocator_type& al={}); ctor<InIt>(InIt f, InIt l, const allocator_type& al={});
	ctor(const self& x); self& operator=(const self& x);
	{left|right}_<const>_iterator project_{left|right}<It>(It iter) <const> { return core.project<logic_{left|right}_tag>(iter.base()); }
	relation_set::<const>_iterator project_up<It>(It iter) <const> { return core.project<logic_relation_set_tag>(iter.base()); }
	<const>_iterator_type_by<Tag,self>::type project<Tag,It>(It iter) <const> { return core.project<Tag>(iter.base()); }
	struct map_by<Tag> : public map_type_by<Tag,self>::type { using type = map_type_by<Tag,self>::type; };
	<const> map_type_by<Tag,self>::type& by() <const> { return map_by<Tag>(*this); }
	void serialize<Archive>(Archive& ar, unsigned) { ar & make_nvp("mi_core", core); }
};
```

#### Property Map Supporting

```c++
struct property_traits<views::map_view<Tag,Bimap>> {
	using value_type = data_type_by<Tag,Bimap>::type;
	using key_type = key_type_by<Tag,Bimap>::type;
	using category = readable_property_map_tag;
};
const data_type_by<Tag,Bimap>::type& get<Tag,Bimap>(const views::map_view<Tag,Bimap>& m, const key_type_by<Tag,Bimap>::type& key) { return m.at(key); }

struct property_traits<views::unordered_map_view<Tag,Bimap>> {
	using value_type = data_type_by<Tag,Bimap>::type;
	using key_type = key_type_by<Tag,Bimap>::type;
	using category = readable_property_map_tag;
};
const data_type_by<Tag,Bimap>::type& get<Tag,Bimap>(const views::unordered_map_view<Tag,Bimap>& m, const key_type_by<Tag,Bimap>::type& key) { return m.at(key); }
```

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
