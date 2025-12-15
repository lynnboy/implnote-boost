# Boost.TypeErasure

* lib: `boost/libs/type_erasure`
* repo: `boostorg/type_erasure`
* commit: `5281f14`, 2025-05-05

------
### Concept wrappers: Any, Param

```c++
struct any_base<Derived> {
    using _b_te_is_any=void; using _b_te_derived_type=Derived;
    void* _b_te_deduce_constructor(...) const volatile {return 0;}
    void* _b_te_deduce_assign(...) {return 0;}
};

struct detail::placeholder_conversion<From,To> : false_{};
struct detail::placeholder_conversion<T,T>: true_{}; // T -> T|T&|const T&, const T -> T|const T&|, T& -> T|T&|const T&, const T& -> T|const T&, T&& -> T|const T&|T&&

struct param<C,T>{
  using _b_te_is_any = void; using _b_te_derived_type = self;
  ctor<U>(any<C,U> {const&|&|&&} a) requires (placeholder_conversion<{U|const U|U&&},T>::value) : _impl{<std::move>(a)}{}
  ctor(const storage& data, const binding<C>& table) :_impl{data,table}{}
  any<C,T> get()const { return _impl; }
private: any<C,T> _impl;
};
struct param<C,const T&>{
  using _b_te_is_any = void; using _b_te_derived_type = self;
  ctor(const storage& data, const binding<C>& table) :_impl{data,table}{}
  ctor<U>(U& u) requires (is_same_v<U,const any<C,T>>) :_impl{u}{}
  any<C,const T&> get()const { return _impl; }
protected:
  struct _impl_t{
    ctor(const storage& d, const binding<C>& t) : table{t}, data{d}{}
    ctor(const any<C,T>& u) : table{access::table(u)}, data{access::data(u)}{}
    const binding<C>& table; storage data;
  } _impl;
};
struct param<C,T&>: public param<C,const T&> {
  using _b_te_is_any = void; using _b_te_derived_type = self;
  ctor(const storage& data, const binding<C>& table) :base{data,table}{}
  any<C,T&> get()const { return {access::data{_impl}, access::table{_impl}}; }
};
struct param<C,T&&>: public param<C,const T&> {
  using _b_te_is_any = void; using _b_te_derived_type = self;
  ctor(const storage& data, const binding<C>& table) :base{data,table}{}
  any<C,T&&> get()const { return {access::data{_impl}, access::table{_impl}}; }
};

struct rebind_any<Any,T> { using type = mpl::if_<is_placeholder<remove_cv_ref_t<T>>,T>::type; };
using rebind_any_t<Any,T> = rebind_any<Any,T>::type;

struct as_param<Any,T> { using type = mpl::if_<is_placeholder<remove_cv_ref_t<T>>, param<concept_of<Any>::type,T>,T>::type };
using as_param_t<Any,T> = as_param<Any,T>::type;

struct static_binding<Map> { using map_type = Map; };
static_binding<Map> make_binding() { return {}; }

struct detail::can_optimize_conversion<Source,Dest,Map> : and_<is_same<Source,Dest>, is_same<find_if<Map,not_<is_same<first<_1>,second<_1>>>>::type, end<Map>::type>>::type {};
class binding<C> {
    using actual_concept = transform<normalize_concept<C>::type, maybe_adapt_to_vtable<_1>>::type;
    using table_type = make_vtable<actual_concept>::type;
    using placeholder_subs = get_placeholder_normalization_map<C>::type;
    struct impl_type {
        ctor() { table = &make_vtable_init_impl<transform<actual_concept,get_null_vtable_entry<_1>>::type, table_type>::type::value; }
        ctor<Map>(const static_binding<Map>&) { table = &make_vtable_init_impl<transform<actual_concept,
            rebind_placeholders<_1,add_deductions<Map,placeholder_subs>::type>>::type, table_type>::type::value; }
        ctor<C2,Map>(const binding<C2>& other, const static_binding<Map>&, false_) : manager(new table_type)
        { manager->convert_from<convert_deductions<Map,placeholder_subs,binding<C2>::placeholder_subs>::type>(*other.impl.table); table = manager.get(); }
        ctor<Placeholders,Map>(const dynamic_binding<Placeholders>& other, const static_binding<Map>&) : manager(new table_type)
        { manager->convert_from<convert_deductions<Map,placeholder_subs>::type>(*other.impl); table = manager.get(); }
        ctor<C2,Map>(const binding<C2>& other, const static_binding<Map>&, false_) : table{other.impl.table}, manager(new table_type) {}
    } impl;
public: ctor(){}
    explicit ctor<Map>(const {Map|static_binding<Map>}&):impl{(make_instantiate_concept<C,Map>::apply<instantiate_concept_impl>*)0, static_binding<Map>{}}{}
    ctor<C2,Map>(const self<C2>& other, const {Map|static_binding<Map>}&) requires (check_map<C,Map> && is_subconcept<C,C2,Map>)
        :impl{other, static_binding<Map>{}, can_optimize_conversion<C2,C,Map>()}{}
    ctor<Placeholders,Map>(const dynamic_binding<Placeholders>& other, const static_binding<Map>&) :impl{other, static_binding<Map>{}}{}
    friend bool operator{==|!=}(const self& lhs, const self& rhs) { return *lhs.impl.table {==|!=} *rhs.impl.table; }
    T::type find<T>() const { return impl.table->lookup((T*)0); }
};

class dynamic_binding<PlaceholderList> {
  make_dynamic_vtable<PlaceholderList>::type impl;
public: ctor<Map>(const static_binding<Map&>) { impl.init<Map>(); }
  ctor<C,Map>(const binding<C>& other, const static_binding<Map>&) { impl.convert_from(*other.impl.table); }
};

const binding<C>& binding_of<C,T>(const any<C,T>& arg) { return access::table(arg); }

struct detail::default_concept_interface { using apply<C,Base,ID> = concept_interface<C,Base,ID>; };
default_concept_interface detail::b_te_find_interface(...);
struct detail::choose_concept_interface<C,Base,ID> { using type = b_te_find_interface(declval<C>()).apply<C,Base,ID>; };
struct detail::compute_bases_f<ID> { using apply<C,Base> = choose_concept_interface<C,Base,ID>::type; };
using detail::compute_bases_t<Derived,C,T> = mp_reverse_fold<collect_concepts_t<C>, any_base<Derived>, computebases_f<T>::apply>;
using detail::compute_bases<Derived,C,T> = identity<compute_bases_t<Derived,C,T>>;

struct detail::is_any<T> : false_{};
struct detail::is_any<any<C,T>>: true_{};
struct detail::has_constructor<Any,...U> : ...{};
using detail::has_copy_constructor<Any> = is_subconcept<constructible<placeholder_of<Any>::type(placeholder_of<Any>::type const&)>, concept_of<Any>::type>;
using detail::has_move_constructor<Any> = is_subconcept<constructible<placeholder_of<Any>::type(placeholder_of<Any>::type &&)>, concept_of<Any>::type>;
using detail::has_mutable_copy_constructor<Any> = is_subconcept<constructible<placeholder_of<Any>::type(placeholder_of<Any>::type &)>, concept_of<Any>::type>;

struct detail::is_binding_arg<T> : false_{};
struct detail::is_binding_arg<binding<T> [const&|&|&&]> : true_{};
struct detail::is_static_binding_arg<T> : false_{};
struct detail::is_static_binding_arg<static_binding<T> [const&|&|&&]> : true_{};
struct detail::is_any_arg<T> : false_{};
struct detail::is_any_arg<any<C,T> [const&|&|&&]> : true_{};
struct detail::safe_concept_of<any<C,T> [const&|&|&&]> { using type = C; };
struct detail::safe_placeholder_of<any<C,T> [const&|&|&&]> { using type = T; };
using detail::safe_placeholder_t<T> = remove_cv_ref_t<safe_placeholder_of<T>::type>;

struct any_constructor_control<Base,Enable=void> : Base { using base::ctor; };
struct any_constructor_control<Base,enable_if_c<!has_copy_constructor<Base> && has_move_constructor<Base> && has_mutable_copy_constructor<Base>>::type> : Base
{ using base::ctor; /* all special ctor/assign =default */ };
struct any_constructor_control<Base,enable_if_c<!has_copy_constructor<Base> && !has_move_constructor<Base> && has_mutable_copy_constructor<Base>>::type> : Base
{ using base::ctor; /* move-ctor =delete */ };
struct any_constructor_control<Base,enable_if_c<!has_copy_constructor<Base> && has_move_constructor<Base> && !has_mutable_copy_constructor<Base>>::type> : Base
{ using base::ctor; /* copy-ctor =delete */ };
struct any_constructor_control<Base,enable_if_c<!has_copy_constructor<Base> && !has_move_constructor<Base> && !has_mutable_copy_constructor<Base>>::type> : Base
{ using base::ctor; /* copy/move-ctor =delete */ };
struct any_constructor_impl<C,T> : compute_bases<any<C,T>, C,T>::type {
  using _b_te_base = base; using _b_te_table_type = binding<C>;
  ctor(storage {const&|&&} data_arg, const _b_te_table_type& table_arg) : _b_te_table{table_arg}, _b_te_data[data_arg]{}
  ctor() { _b_te_data.data=0; }
  ctor<U>(U&& data_arg) requires(!is_any_arg<U> && !is_binding_arg<U> && !is_static_binding_arg<U>)
    : _b_te_table{make_binding<mpl::map<pair<T,decay_t<U>>>>()}, _b_te_data{std::forward<U>(data_arg)}{}
  ctor<U>(U&& data_arg, const static_binding<Map>& b) requires(!is_any_arg<U> && !is_binding_arg<U> && !is_static_binding_arg<U>)
    : _b_te_table{b}, _b_te_data{std::forward<U>(data_arg)}{}
  ctor<U>(U&& other) requires(is_subconcept<C, safe_concept_of<U>::type, if_c<is_same<T,safe_placeholder_t<U>>::value, void, mpl::map<pair<T,safe_placeholder_t<U>>>>>)
    : _b_te_table{access::table(other), if_c<is_same<T,safe_placeholder_t<U>>,make_identity_placeholder_map<C>,map<pair<T,safe_placeholder_t<U>>>>::type{}}, _b_te_data{std::forward<U>(other)}{}
  ctor<U>(U&& other, const binding<C>& binding_arg) requires(is_any_arg<U>)
    : _b_te_table{binding_arg}, _b_te_data{std::forward<U>(other)}{}
  ctor<U,Map>(U&& other, const static_binding<Map>& binding_arg) requires(is_subconcept<C,safe_concept_of<U>::type, M>>)
    : _b_te_table{access::table(other), binding_arg}, _b_te_data{std::forward<U>(other)}{}
  ctor(self {const&|&|&&} other) : _b_te_table{access::table(other)}, _b_te_data{<std::move>(other)}{}
  const _b_te_table_type& _b_te_extract_table<R,...A,...U>(constructible<R(A...)>*, U&&...u) { return *extract_table((void(*)(A...))0, u...); } // called by each ctor
  explicit ctor<...U>(U&&...u) requires has_constructor<self,U...> : _b_te_table{std::forward<U>(u)...}, _b_te_data{std::forward<U>(u)...}{}
  explicit ctor<...U>(const binding<c>& binding_arg, U&&...u) requires has_constructor<self,U...> : _b_te_table{binding_arg}, _b_te_data{std::forward<U>(u)...}{}
  self& operator=(self {const&|&|&&} other) { _b_te_resolve_assign(other); return *this; }
  ~dtor() { _b_te_table.find<destructible<T>>()(_b_te_data); }
protected: _b_te_table_type _b_te_table; storage _b_te_data;
};

struct detail::is_rvalue_for_any<T> : not_<is_lvalue_reference<T>>{};
struct detail::is_rvalue_for_any<any<C,P>> : not_<is_lvalue_reference<T>>{};

class any<C,T=_self> : public any_constructor_control<any_constructor_impl<C,T>> {
  using table_type = binding<C>;
public: using _b_te_concept_type = C; using _b_te_base = base;
  using base::ctor;
  self& operator=<U>(U&& other) { _b_te_resolve_assign(std::forward<U>(other)); return *this; }
  operator param<C,T{&|&&}>() {&|&&} { return {access::data(*this); access::table(*this)}; }
private: void _b_te_swap(self& other) { std::swap(_b_te_data, other._b_te_data); std::swap(b_te_table, other._b_te_table); }
  void _b_te_resolve_assign<Other>(Other&& other){ call(assignable<T,U>{}, *this, std::forward<Other>(other)); }
  void _b_te_resolve_assign<C2,T2>(any<C2,T2> {const&|&|&&} other) { _b_te_resolve_assign_any(<std::move>(other)); }
  void _b_te_resolve_assign_any<Other>(Other&& other);
};
class any<C,T&> : public compute_bases<self,C,T>::type {
  using table_type = binding<C>;
public: using _b_te_concept_type = C;
  ctor(const storage& data_arg, const table_type& table_arg): data{data_arg}, table{table_arg}{}
  ctor<U>(U& arg) requires !(is_const_v<U>||is_any<U>) : table{make_binding<map<pair<T,U>>>{}} { data.data = addressof(arg); }
  ctor<U,Map>(U& arg, const static_binding<Map>& binding_arg) : table{binding_arg} { data.data = addressof(arg); }
  ctor(self {const&|&} other) : data{other.data}, table{other.table}{}
  ctor(any<C,T>& other) : data{access::data(other)}, table{access::table(other)}{}
  ctor<C2,T2>(any<C2,T2&>& other) requires !(is_same_v<C,C2>||is_const_v<T2>) : data{access::data(other), access::table(other), map<pair<T,T2>>{}}{}
  ctor<C2,T2>(any<C2,T2>& other) requires !(is_same_v<C,C2>||is_const_v<remove_reference_t<T2>>) : data{access::data(other)}, table{access::table(other), map<pair<T,remove_reference_t<T2>>>{}}{}
  ctor<C2,T2,Map>(const any<C2,T2&>& other, const static_binding<Map>& binding_arg) requires !is_const_v<T2> : data{access::data(other)}, table{access::table(other), binding_arg}{}
  ctor<C2,T2,Map>(any<C2,T2>& other, const static_binding<Map>& binding_arg) requires !is_const_v<remove_reference_t<T2>> : data{access::data(other)}, table{access::table(other), binding_arg}{}
  ctor<C2,T2>(const any<C2,T2&>& other, const binding<C>& binding_arg) requires !is_const_v<T2> : data{access::data(other)}, table{binding_arg}{}
  ctor<C2,T2>(any<C2,T2>& other, const binding<C>& binding_arg) requires !is_const_v<remove_reference_t<T2>> : data{access::data(other)}, table{binding_arg}{}
  self& operator=(self {const&|&|&&} other) { _b_te_resolve_assign(<std::move>(other)); return *this; }
  self& operator=<U>(U&& other) { _b_te_resolve_assign(std::forward<U>(other)); return *this; }
  operator param<C,T&>() const { return {data, table}; }
private: void _b_te_swap(self& other) { std::swap(_b_te_data, other._b_te_data); std::swap(b_te_table, other._b_te_table); }
  void _b_te_resolve_assign<Other>(Other&& other){ call(assignable<T,U>{}, *this, std::forward<Other>(other)); }
  storage data; table_type table;
};
class any<C,T const&> : public compute_bases<self,C,T>::type {
  using table_type = binding<C>;
public: using _b_te_concept_type = C;
  ctor(const storage& data_arg, const table_type& table_arg): data{data_arg}, table{table_arg}{}
  ctor<U>(U const& arg) requires !(is_const_v<U>||is_any<U>) : table{make_binding<map<pair<T,U>>>{}} { data.data = addressof(arg); }
  ctor<U,Map>(U const& arg, const static_binding<Map>& binding_arg) : table{binding_arg} { data.data = addressof(arg); }
  ctor(self const& other) : data{other.data}, table{other.table}{}
  ctor(any<C,T>const& other) : data{access::data(other)}, table{access::table(other)}{}
  ctor<C2,T2>(any<C2,T2&>const& other) requires !(is_same_v<C,C2>||is_const_v<T2>) : data{access::data(other), access::table(other), map<pair<T,T2>>{}}{}
  ctor<C2,T2>(any<C2,T2>const& other) requires !(is_same_v<C,C2>||is_const_v<remove_reference_t<T2>>) : data{access::data(other)}, table{access::table(other), map<pair<T,remove_reference_t<T2>>>{}}{}
  ctor<C2,T2,Map>(any<C2,T2&>const& other, const static_binding<Map>& binding_arg) requires !is_const_v<T2> : data{access::data(other)}, table{access::table(other), binding_arg}{}
  ctor<C2,T2,Map>(any<C2,T2>const& other, const static_binding<Map>& binding_arg) requires !is_const_v<remove_reference_t<T2>> : data{access::data(other)}, table{access::table(other), binding_arg}{}
  ctor<C2,T2>(any<C2,T2&>const& other, const binding<C>& binding_arg) requires !is_const_v<T2> : data{access::data(other)}, table{binding_arg}{}
  ctor<C2,T2>(any<C2,T2>const& other, const binding<C>& binding_arg) requires !is_const_v<remove_reference_t<T2>> : data{access::data(other)}, table{binding_arg}{}
  self& operator=(self const& other) { _b_te_resolve_assign(<std::move>(other)); return *this; }
  self& operator=<U>(U const& other) { _b_te_resolve_assign(std::forward<U>(other)); return *this; }
  operator param<C,T const&>() const { return {data, table}; }
private: void _b_te_swap(self& other) { std::swap(_b_te_data, other._b_te_data); std::swap(b_te_table, other._b_te_table); }
  storage data; table_type table;
};
class any<C,T&&> : public compute_bases<self,C,T>::type {
  using table_type = binding<C>;
public: using _b_te_concept_type = C;
  ctor(const storage& data_arg, const table_type& table_arg): data{data_arg}, table{table_arg}{}
  ctor<U>(U&& arg) requires !(is_const_v<U>||is_any<U>) : table{make_binding<map<pair<T,U>>>{}} { data.data = addressof(arg); }
  ctor<U,Map>(U&& arg, const static_binding<Map>& binding_arg) : table{binding_arg} { data.data = addressof(arg); }
  ctor(self {const&|&&} other) : data{other.data}, table{<std::move>(other.table)}{}
  ctor(any<C,T>&& other) : data{access::data(other)}, table{std::move(access::table(other))}{}
  ctor<C2,T2>(any<C2,T2&&>&& other) requires !(is_reference_v<T2>||is_same_v<C,C2>||is_const_v<T2>) : data{access::data(other), access::table(std::move(other)), map<pair<T,T2>>{}}{}
  ctor<C2,T2>(any<C2,T2>&& other) requires !(is_same_v<C,C2>||is_const_v<remove_reference_t<T2>>) : data{access::data(other)}, table{std::move(access::table(other)), map<pair<T,remove_reference_t<T2>>>{}}{}
  ctor<C2,T2,Map>(const any<C2,T2&&>& other, const static_binding<Map>& binding_arg) requires !is_const_v<T2> : data{access::data(other)}, table{std::move(access::table(other)), binding_arg}{}
  ctor<C2,T2,Map>(any<C2,T2>&& other, const static_binding<Map>& binding_arg) requires !is_const_v<remove_reference_t<T2>> : data{access::data(other)}, table{access::table(other), binding_arg}{}
  ctor<C2,T2>(const any<C2,T2&&>& other, const binding<C>& binding_arg) requires !is_const_v<T2> : data{access::data(other)}, table{binding_arg}{}
  ctor<C2,T2>(any<C2,T2>&& other, const binding<C>& binding_arg) requires !is_const_v<remove_reference_t<T2>> : data{access::data(other)}, table{binding_arg}{}
  self& operator=(self const& other) { _b_te_resolve_assign(<std::move>(other)); return *this; }
  self& operator=<U>(U&& other) { _b_te_resolve_assign(std::forward<U>(other)); return *this; }
  operator param<C,T&&>() const { return {data, table}; }
private: void _b_te_swap(self& other) { std::swap(_b_te_data, other._b_te_data); std::swap(b_te_table, other._b_te_table); }
  void _b_te_resolve_assign<Other>(Other&& other){ call(assignable<T,U>{}, *this, std::forward<Other>(other)); }
  storage data; table_type table;
};

using any_ref<C,T> = any<C,T&>;
using any_cref<C,T> = any<C,const T&>;
using any_rvref<C,T> = any<C,T&&>;

<const> void* detail::get_pointer<C,T>(any<C,T[&|const&]> <const>& arg) { return access::data(arg).data; }
bool detail::check_any_cast<T,C,Tag>(const any<C,T>& arg)
{ if constexpr (is_void_v<remove_reference_t<T>>) {return true;}
  else { return access::table(arg).find<typeid_<remove_cv_ref_t<Tag>>>()() == typeid(T); } }
T any_cast<T,C,Tag>(any<C,T <const>&> arg) { if (check_any_cast<T>(arg) return *(remove_reference_t<const T>*)get_pointer(arg)); else THROW_EXCEPTION(bad_any_cast{}); }
T any_cast<T,C,Tag>(any<C,T <const>*> arg) { if (check_any_cast<T>(*arg) return *(remove_reference_t<const T>*)get_pointer(*arg)); else return 0; }

struct detail::make_ref_placeholder<P,P2<&&>,Any> { using type = P&&; }; // (P2,<const>Any&)-><const>P&, (P2&,<const>Any<&>)->P&, (const P2&,<const>Any<&>)->const P&, (P2&&,<const>Any&)-><const> P&
struct detail::make_ref_placeholder<P,P2<&|const&>,Any{const&|&}> { using type = <const>P&; };
struct detail::make_result_placeholder_map<R,Tag> { using type = map<pair<remove_cv_ref_t<placeholder_of<R>::type>, remove_cv_ref_t<Tag>>>; };
R detail::dynamic_any_cast_impl<R,Any,Map>(Any&& arg, const static_binding<Map>& map);
R dynamic_any_cast<R,C,Tag>(any<C,Tag>{const&|&|&&} arg) { return dynamic_any_cast_impl<R>(<std::move>(arg), make_binding<make_result_placeholder_map<R,Tag>::type>()); }
R dynamic_any_cast<R,C,Tag,Map>(any<C,Tag>{const&|&|&&} arg, const static_binding<Map>& map) { return dynamic_any_cast_impl<R>(<std::move>(arg), map); }

using detail::first_placeholder_index_t<...T> = first_placeholder_index<remove_cv_ref_t<T>...>::type;
using detail::free_param_t<Base,Tn,i,...T> = eval_if_c<first_placeholder_index_t<T...>::value==i, maybe_const_this_param<Tn,Base>,as_param<Base,Tn>>::type;
struct detail::free_interface_chooser<Sig,ID> { using apply<Base,C<_>,F<...>> = Base; };
struct detail::free_interface_chooser<R(A...),first_placeholder<remove_cv_ref_t<A>...>::type>
{ using apply<Base,C<_>,F<...>> = F<R(A...),Base,make_index_list_t<sizeof...(A)>>; };
struct detail::free_choose_interface<Sig,C<_>,F<...>> { using apply<C,Base,ID> = free_interface_chooser<Sig,ID>::apply<Base,C,F>; };

bool is_empty<T>(const T& arg) { return access::data(arg).data == 0; }
struct detail::mp_set_has_key<S,K> : mp_set_contains<S,K>{};
struct detail::is_subconcept_f<Super,Bindings> { using apply<T> = mp_set_contains<Super,rebind_placeholders_t<T,Bindings>>; };
struct detail::is_subconcept_f<Super,void> { using apply<T> = mp_set_contains<Super,T>; };
struct detail::is_subconcept_impl<Sub,Super,PhMap> {
  using super_set = normalize_concept_t<Super>; using ph_subs_super = get_placeholder_normalization_map_t<Super>;
  using normalized_sub = normalize_concept_t<Sub>; using ph_subs_sub = get_placeholder_normalization_map_t<Sub>;
  using bindings = mp_eval_if<is_same<PhMap,void>, void, convert_deductions_t, PhMap, ph_subs_sub, ph_subs_super>;
  using type = mp_all_of<normalized_sub, is_subconcept_f<super_set,bindings>::apply>;
};
struct is_subconcept<Sub,Super,PhMap=void> : and_<check_map<Sub,PhMap>, is_subconcept_impl<Sub,Super,PhMap>>::type {};
struct is_subconcept<Sub,Super,void> : is_subconcept_impl<Sub,Super,void>::type {};
struct is_subconcept<Sub,Super,static_binding<PhMap>> : and_<check_map<Sub,PhMap>, is_subconcept_impl<Sub,Super,PhMap>>::type {};

const std::type_info& typeid_of<C,T>(const any<C,T>& arg) { return access::table(arg).find<type_id<remove_cv_ref_t<T>>>()(); }
const std::type_info& typeid_of<C,T>(const param<C,T>& arg) { return access::table(arg).find<type_id<remove_cv_ref_t<T>>>()(); }
const std::type_info& typeid_of<C,T>(const binding<C>& binding_arg) { return binding_arg.find<typeid_<T>>()(); }
```

### Placeholder

```c++
struct placeholder { using _b_te_is_placeholder = void; };
struct _a : placeholder{}; // up to _g, and _self

struct is_placeholder<T>; // detect T::_b_te_is_placeholder

struct placeholder_of<T> { using type = placeholder_of<T::_b_te_derived_type>::type; };
struct placeholder_of<any<C,T>> { using type = T; };
struct placeholder_of<param<C,T>> { using type = T; };
using placeholder_of_t<T> = placeholder_of<T>::type;

struct deduced<MF> : placeholder { using type = mpl::eval_if<empty<get_placeholders<MF,mp_list<>>::type>, MF, identity<self>>::type; };

struct same_type<T,U>{};

struct cons<C,...T>;
struct cons<C> { ctor<Bindings>(const Bindings&){} };
struct cons<C,T0,T...> { using value_type = any<C,T0>; using rest_type = cons<Concept,T...>;
  ctor<Binding,U0,...U>(const Binding& b, U0&& u0, U&&...u) : value{std::forward<U0>(u0), b}, rest(b, std::forward<U>(u)...){}
  any<C,T0> value; cons<C,T...> rest; }
struct detail::cons_advance<n,Cons> { using type = self<n-1,Cons>::type::rest_type;
  static const type& call(const Cons& c) { return self<n-1,Cons>::call(c).rest; } };
struct detail::cons_advance<0,Cons> { using type = Cons; static const type& call(const Cons& c) { return c; } };
struct detail::make_map<...T>;
struct detail::make_map<T0,T...> { using type = mpl::insert<make_map<T...>::type,T0>::type; };
struct detail::make_map<> { using type = map<>; };
struct tuple_iterator<Tuple,n> : fusion::iterator_facade<self, random_access_iterator_tag> {
  using index = int_<n>; explicit ctor(Tuple& t): t{&t}{}
  struct value_of<It> { using type = Tuple::value_at<Tuple,index>::type; };
  struct deref<It> { using type = Tuple::at<Tuple,index>::type; static type call(It it) { return tuple::call(*it.t); } };
  struct advance<It,M> { using type = tuple_iterator<Tuple,It::index::value+M::value>; static type call(It it) { return {*it.t}; } };
  struct next<It> : advance<It,int_<1>>{}; struct prior : advance<It,int_<-1>>{};
  struct distance<It1,It2> { using type = mpl::minus<It2::index,It1::index>::type; static type call(It1,It2) { return {}; } };
private: Tuple* t;
};
struct tuple<C,T...> : fusion::sequence_facade<self,forward_traversal_tag> {
  explicit ctor<...U>(U&&...args) : impl{make_binding<make_map<pair<remove_cv_ref_t<T>,remove_cv_ref_t<U>>...>::type>(), std::forward<U>(args)...}{}
  struct begin<Seq> { using type = tuple_iterator<Seq,0>; static type call(Seq& seq) { return {seq}; } };
  struct end<Seq> { using type = tuple_iterator<Seq,sizeof...(T)>; static type call(Seq& seq) { return {seq}; } };
  struct size<Seq> { using type = int_<sizeof...(T)>; static type call(Seq& seq) { return {}; } };
  struct empty<Seq> { using type = bool_<sizeof...(T)==0>; static type call(Seq& seq) { return {}; } };
  struct at<Seq,N> { using value_type = cons_advance<N::value,cons<C,T...>>::type::value_type;
    using type = if_<is_const<Seq>, const value_type&, value_type&>::type;
    static type call(Seq& seq) { return {cons_advance<N::value, cons<C,T...>>::call(seq.impl).value; } };
  };
  struct value_at<Seq,N> { using value_type = cons_advance<N::value, cons<C,T...>>::type::value_type; };
  cons<C,T...> impl;
};
<const> cons_advance<n,cons<C,T...>>::type::value_type& get<n,C,...T>(<const> tuple<C,T...>& t) { return R::call(t.impl).value; }

using detail::key_type = std::vector<const std::type_info*>;
using detail::value_type = void(*)();
void detail::register_function_impl(const key_type& key, value_type fn);
value_type detail::lookup_function_impl(const key_type& key);
struct detail::append_to_key_static<Map> { void operator()<P>(P) { key->push_back(&typeid(mp_second<mp_map_find<Map,P>>)); } key_type* key; };
struct detail::_<n> : placeholder{};
struct detail::counting_map_appender { struct apply<State,Key> { using type = insert<State,pair<Key,_<size<State>::value>>>::type; } };
struct detail::register_function<Map> { void operator()<F>(F); };
void register_binding<C,Map>(const static_binding<Map>&);
void register_binding<C,T>();
```

------
### Concepts

```c++
struct concept_of<T> { type = concept_of<T::_b_te_derived_type>::type; };
struct concept_of<any<C,T>> { using type = C; };
struct concept_of<param<C,T>> { using type = C; };
using concept_of_t<T> = concept_of<T>::type;

struct concept_interface<C,Base,ID,Enable=void> : Base{};

struct relaxed : vector<>{};
struct is_relaxed<C>;

struct constructible<Sig>;
struct constructible<R(T...)> { static storage apply(T...arg) { return {.data=new R(std::forward<T>(arg)...)}; } };
struct concept_interface<constructible<Tag(T...)>, Base,Tag> :Base
{ using Base::_b_te_deduce_constructor; constructible<Tag(T...)>* _b_te_deduce_constructor(as_param<Base,T>::type...) const { return 0; } };

struct detail::null_construct<Sig>;
struct detail::null_construct<void(T...)>{ static storage value(T...) { return {.data=0}; } };
struct detail::get_null_vtable_entry<C>;
struct detail::get_null_vtable_entry<vtable_adapter<constructible<T(const T&)>, R(U...)>> { using type = null_construct<void(U...)>; };

struct destructible<T=_self> { using type = void(*)(storage&); static void {value|apply}(storage& arg) { delete (T*)arg.data; } };

struct copy_constructible<T=_self> : mpl::vector<constructible<T(const T&)>, destructible<T>>{};

struct assignable<T=_self, U=const T&> : mpl::vector<assignable<T, const U&>>{};
struct assignable<T, U&&> { static void apply(T& dst, U&& src){ dst = std::forward<U>(src); } };
struct assignable<T, U&> { static void apply(T& dst, U& src){ dst = src; } };
struct concept_interface<assignable<T,U>, Base,T> requires (is_reference_v<U>) :Base
{ using Base::_b_te_deduce_assign; assignable<T,U>* _b_te_deduce_assign(as_param<Base,T>::type) { return 0; } };

struct typeid_<T=_self> { using type = const std::type_info& (*)(); static const std::type_info& {value|apply}() { return typeid(T); } };
struct detail::get_null_vtable_entry<typeid_<T>> { using type = typeid_<void>; };
struct detail::null_destroy { static void value(storage&){} };
struct detail::get_null_vtable_entry<destructible<T>> { using type = null_destroy; };

struct detail::result_of_callable<Sig>;
struct callable<R(T...),F> { static R apply(F&f, T...arg) { return f(std::forward<T>(arg)...); } };
struct callable<void(T...),F> { static void apply(F&f, T...arg) { f(std::forward<T>(arg)...); } };
struct concept_interface<callable<R(T...),<const>F>, Base,F,Enable> : Base {
  struct result<Sig> : result_of_callable<Sig>{};
  using _b_te_is_callable=void; using _b_te_callable_results = vector<R>; using _b_te_callable_size = char(&)[1];
  _b_te_callable_size _b_te_deduce_callable(as_param<Base,T>::type...) <const>;
  rebind_any<Base,R>::type operator()(as_param<Base,T>::type...arg) { return call(callable<R(T...),<const>F>(),*this,std::forward<as_param<Base,T>::type>(arg)...); }
};
struct concept_interface<callable<R(T...),<const>F>, Base,F,Base::_b_te_is_callable> : Base {
  using _b_te_callable_results = push_back<Base::_b_te_callable_results,R>; using _b_te_callable_size = char(&)[mpl::size<_b_te_callable_results>::value];
  _b_te_callable_size _b_te_deduce_callable(as_param<Base,T>::type...) <const>; using base::_b_te_deduce_callable;
  rebind_any<Base,R>::type operator()(as_param<Base,T>::type...arg) { return call(callable<R(T...),<const>F>(),*this,std::forward<as_param<Base,T>::type>(arg)...); }
};
struct detail::result_of_callable<This(T...)> { using type = at_c<This::_b_te_callable_results, sizeof(declval<This>()._b_te_deduce_callable(declval<T>()...))-1>:type;  };

struct derived<T> { using type = T::_b_te_derived_type; };
using derived_t<T> = T::_b_te_derived_type;

// operators
struct incrementable<T=_self> { static void apply(T& arg){ ++arg; } };
struct concept_interface<incrementable<T>,Base,T> requires(should_be_<non>_const<T,Base>) : Base
{ using _derived = derived<Base>::type;
  <const> _derived& operator++() <const> { call(incrementable<T>{}, *this); return {*this}; }
  rebind_any<Base,T>::type operator++(int) <const> { rebind_any::type result={(<const>_derived&)*this}; call(name<T>{},*this); return result; }
};
struct decrementable<T=_self> { static void apply(T& arg){ --arg; } };
struct concept_interface<decrementable<T>,Base,T> requires(should_be_<non>_const<T,Base>) : Base
{ using _derived = derived<Base>::type;
  <const> _derived& operator--() <const> { call(decrementable<T>{}, *this); return {*this}; }
  rebind_any<Base,T>::type operator--(int) <const> { rebind_any::type result={(<const>_derived&)*this}; call(name<T>{},*this); return result; }
};
struct complementable<T=_self,R=T> { static R apply(const T& arg) { return ~arg;} };
struct concept_interface<complementable<T,R>,Base,T> : Base
{ rebind_any<Base,R>::type operator~() const { return call(complementable<T,R>{}, *this); } };
struct negatable<T=_self,R=T> { static R apply(const T& arg) { return -arg;} };
struct concept_interface<negatable<T,R>,Base,T> : Base
{ rebind_any<Base,R>::type operator-() const { return call(negatable<T,R>{}, *this); } };
struct dereferenceable<T=_self,R=T> { static R apply(const T& arg) { return *arg;} };
struct concept_interface<dereferenceable<R,T>,Base,T> : Base
{ rebind_any<Base,R>::type operator*() const { return call(dereferenceable<R,T>{}, *this); } };

// (NAME,OP): (addable,+), (subtractable,-), (multipliable,*), (dividable,/), (modable,%), (left_shiftable,<<), (right_shiftable,>>), (bitandable,&), (bitorable,|), (bitxorable,^)
struct NAME<T=_self,U=T,R=T> { static R apply(const T& lhs, const U& rhs){ return lhs OP rhs; } };
struct concept_interface<NAME<T,U,R>,Base,T> : Base
{ friend rebind_any<Base,R>::type operator OP(const derived<Base>::type& lhs, as_param<Base,const U&>::type rhs) { return call(NAME<T,U,R>{}, lhs, rhs); } };
struct concept_interface<NAME<T,U,R>,Base,U> requires (!is_placeholder<T>) : Base
{ friend rebind_any<Base,R>::type operator OP(const T& lhs, const derived<Base>::type& rhs) { return call(NAME<T,U,R>{}, lhs, rhs); } };
// (NAME,OP): (add_assignable,+=), (subtract_assignable,-=), (multiply_assignable,*=), (divide_assignable,/=), (mod_assignable,%=), (left_shift_assignable,<<=), (right_shift_assignable,>>=), (bitand_assignable,&=), (bitor_assignable,|=), (bitxor_assignable,^=)
struct NAME<T=_self,U=T> { static void apply(T& lhs, const U& rhs){ lhs OP rhs; } };
struct concept_interface<NAME<T,U>,Base,T> requires (!is_same<placeholder_of<Base>::type, const T&>) : Base
{ friend non_const_this_param<Base>::type& operator OP(non_const_this_param<Base>::type& lhs, as_param<Base,const U&>::type rhs) { call(NAME<T,U>{}, lhs, rhs); return lhs; } };
struct concept_interface<NAME<T,U>,Base,U> requires (!is_placeholder<T>) : Base
{ friend T& operator OP(T& lhs, const derived<Base>::type& rhs) { call(NAME<T,U>{}, lhs, rhs); return lhs; } };

struct equality_comparable<T=_self,U=T> { static bool apply(const T& lhs, const U& rhs) { return lhs == rhs; } };
struct concept_interface<equality_comparable<T,U>,Base,T> : Base {
  friend bool operator==(const derived<Base>::type& lhs, as_param<Base,const U&>::type rhs)
  { if (check_match(equality_comparable<T,U>{}, lhs,rhs)) return unchecked_call(equality_comparable<T,U>{}, lhs,rhs); return false; }
  friend bool operator!=(const derived<Base>::type& lhs, as_param<Base,const U&>::type rhs) { return !(lhs==rhs); }
};
struct concept_interface<equality_comparable<T,U>,Base,U> requires(!is_placeholder<T>) : Base {
  friend bool operator==(const T& lhs, const derived<Base>::type& rhs) { return call(equality_comparable<T,U>{}, lhs, rhs); }
  friend bool operator!=(const T& lhs, const derived<Base>::type& rhs) { return !(lhs==rhs); }
};
struct less_than_comparable<T=_self,U=T> { static bool apply(const T& lhs, const U& rhs) { return lhs < rhs; } };
struct concept_interface<less_than_comparable<T,T>,Base,T> : Base {
  friend bool operator<(const derived<Base>::type& lhs, as_param<Base,const T&>::type rhs)
  { if constexpr (is_relaxed<concept_of<Base>::type>()) {
      if (check_match(f,lhs,rhs)) return unchecked_call(f,lhs,rhs);
      else return typeid_of((const derived<T>::type&)lhs).before(typeid_of((const derived<U>::type&)rhs));
    } else return call(f,lhs,rhs); }
  friend bool operator>=(const derived<Base>::type& lhs, as_param<Base,const T&>::type rhs);
  friend bool operator>(as_param<Base,const T&>::type lhs, const derived<Base>::type& rhs);
  friend bool operator<=(as_param<Base,const T&>::type lhs, const derived<Base>::type& rhs);
};
struct concept_interface<less_than_comparable<T,U>,Base,T> : Base {
  friend bool operator<(const derived<Base>::type& lhs, as_param<Base,const U&>::type rhs)
  { return call(less_than_comparable<T,U>{}, lhs, rhs); }
  friend bool operator>=(const derived<Base>::type& lhs, as_param<Base,const U&>::type rhs);
  friend bool operator>(as_param<Base,const U&>::type lhs, const derived<Base>::type& rhs);
  friend bool operator<=(as_param<Base,const U&>::type lhs, const derived<Base>::type& rhs);
};
struct concept_interface<less_than_comparable<T,U>,Base,U> requires(is_placeholder<T>) : Base {
  friend bool operator<(const T& lhs, const derived<Base>::type& rhs)
  { return call(less_than_comparable<T,U>{}, lhs, rhs); }
  friend bool operator>=(const T& lhs, const derived<Base>::type& rhs);
  friend bool operator>(const derived<Base>::type& lhs, const T& rhs);
  friend bool operator<=(const derived<Base>::type& lhs, const T& rhs);
};
struct subscriptable<R,T=_self,N=ptrdiff_t> { static R apply(T& arg, const N& index) { return arg[index]; } };
struct concept_interface<subscriptable<R,T,N>,Base,remove_const_t<T>> requires(should_be_<non>_const<T,Base>) : Base
{ rebind_any<Base,R>::type operator[](as_param<Base,const N&>::type index) <const> { return call(subscriptable<R,<const> T,N>{}, *this, index); } };

struct ostreamable<Os=std::ostream,T=_self> { static void apply(Os& out, const T& arg) { out << arg; } };
struct concept_interface<ostreamable<Os,T>,Base,Os> : Base
{ friend non_const_this_param<Base>::type& operator<<(non_const_this_param<Base>::type& lhs, as_param<Base,const T&>::type rhs)
  { call(ostreamable<Os,T>{}, lhs, rhs); return lhs; } };
struct concept_interface<ostreamable<Os,T>,Base,T> requires(!is_placeholder<Os>) : Base
{ friend Os& operator<<(Os& lhs, const derived<Base>::type& rhs) { call(ostreamable<Os,T>{}, lhs, rhs); return lhs; } };
struct istreamable<Is=std::istream,T=_self> { static void apply(Is& in, T& arg) { in >> arg; } };
struct concept_interface<istreamable<Is,T>,Base,Is> : Base
{ friend non_const_this_param<Base>::type& operator>>(non_const_this_param<Base>::type& lhs, as_param<Base,const T&>::type rhs)
  { call(istreamable<Is,T>{}, lhs, rhs); return lhs; } };
struct concept_interface<istreamable<Is,T>,Base,T> requires(!is_placeholder<Is>) : Base
{ friend Is& operator>>(Is& lhs, const derived<Base>::type& rhs) { call(istreamable<Is,T>{}, lhs, rhs); return lhs; } };

// iterator
struct is_placeholder<use_default> : false_{};
struct detail::iterator_value_type_impl<T> { using type = std::iterator_traits<T>::value_type; };
struct iterator_value_type<T> { using type = eval_if<is_placeholder<T>, identity<void>, iterator_value_type_impl<T>>::type; };
struct iterator<Traversal,T=_self,Ref=use_default,Diff=ptrdiff_t,V=deduced<iterator_value_type<T>>::type>;
struct iterator_reference<Ref,V> { using type = Ref; };
struct iterator_reference<use_default,V> { using type = V&; };
struct iterator<no_traversal_tag,T,Ref,Diff,V>
  : mpl::vector<copy_constructible<T>,constructible<T()>,equality_comparable<T>, dereferenceable<iterator_reference<Ref,V>::type,T>,assignable<T>>
{ using value_type = V; using reference = iterator_reference<Ref,V>::type; using difference_type = Diff; };
struct iterator<incrementable_traversal_tag, T,Ref,Diff,V> : mpl::vector<iterator<no_traversal_tag, T,Ref,Diff>, incrementable<T>>
{ using value_type = V; using reference = iterator_reference<Ref,V>::type; using difference_type = Diff; };
struct iterator<single_pass_traversal_tag, T,Ref,Diff,V> : mpl::vector<iterator<incrementable_traversal_tag, T,Ref,Diff,V>> {};
struct iterator<forward_traversal_tag, T,Ref,Diff,V> : mpl::vector<iterator<incrementable_traversal_tag, T,Ref,Diff,V>> {};
struct iterator<bidirectional_traversal_tag, T,Ref,Diff,V> : mpl::vector<iterator<incrementable_traversal_tag, T,Ref,Diff,V>, decrementable<T>>
{ using value_type = V; using reference = iterator_reference<Ref,V>::type; using difference_type = Diff; };
struct iterator<random_access_traversal_tag, T,Ref,Diff,V>
  : mpl::vector<iterator<bidirectional_traversal_tag, T,Ref,Diff,V>,
      addable<T,Diff,T>, addable<Diff,T,T>, subtractable<T,Diff,T>, subtractable<T,T,Diff>, subscriptable<iterator_reference<Ref,V>::type,T,Diff>>
{ using value_type = V; using reference = iterator_reference<Ref,V>::type; using difference_type = Diff; };
struct forward_iterator<T=_self,Ref=use_default,Diff=ptrdiff_t,V=deduced<iterator_value_type<T>>::type> : iterator<forward_traversal_tag, T,Ref,Diff,V> {};
struct bidirectional_iterator<T=_self,Ref=use_default,Diff=ptrdiff_t,V=deduced<iterator_value_type<T>>::type> : iterator<bidirectional_traversal_tag, T,Ref,Diff,V> {};
struct random_access_iterator<T=_self,Ref=use_default,Diff=ptrdiff_t,V=deduced<iterator_value_type<T>>::type> : iterator<random_access_traversal_tag, T,Ref,Diff,V> {};
struct concept_interface<iterator<no_traversal_tag, T,Ref,Diff,V>,Base,T> : Base {
  using value_type = rebind_any<Base,V>::type; using reference = rebind_any<Base,iterator_reference<Ref,V>::type>::type;
  using difference_type = Diff; using pointer = if_<is_reference<reference>, remove_reference_t<reference>*, value_type*>::type;
};
struct concept_interface<iterator<forward_traversal_tag, T,Ref,Diff,V>, Base,T> : Base { using iterator_category = std::forward_iterator_tag; };
struct concept_interface<forward_iterator<T,Ref,Diff,V>, Base,T> : Base { using iterator_category = std::forward_iterator_tag; };
struct concept_interface<iterator<bidirectional_traversal_tag, T,Ref,Diff,V>, Base,T> : Base { using iterator_category = std::bidirectional_iterator_tag; };
struct concept_interface<bidirectional_iterator<T,Ref,Diff,V>, Base,T> : Base { using iterator_category = std::bidirectional_iterator_tag; };
struct concept_interface<iterator<random_access_traversal_tag, T,Ref,Diff,V>, Base,T> : Base { using iterator_category = std::random_access_iterator_tag; };
struct concept_interface<random_access_iterator<T,Ref,Diff,V>, Base,T> : Base { using iterator_category = std::random_access_iterator_tag; };
```
------
### Concept Constraining and Calling

```c++
struct bad_function_call : public std::invalid_argument {};
struct bad_any_cast : public std::bad_cast {};

bool detail::check_table<C,R>(const binding<C>*, R(*)()) { return true; }
bool detail::check_table<C,R,T0,...T,U0,...U>(const binding<C>* t, R(*)(T0,T...), const U0& arg0, const U&...arg)
{ using t0 = remove_cv_ref_t<T0>;
  if (!maybe_check_table<t0>(arg0,t,should_check<C,t0>())) return false;
  return check_table(t, (void(*)(T...))0, arg...);
}
bool check_match<C,Op,...U>(const binding<C>& table, const Op&, U&&...arg)
{ return check_table(&table, (get_signature<Op>::type*)0, arg...); }
bool check_match<Op,...U>(const Op&, U&&...arg)
{ const binding<extract_concept<get_signature<Op>::type,U...>::type>* p =0; return check_table(p, (get_signature<Op>::type*)0, arg...); }


void require_match<C,Op,...U>(const binding<C>& table, const Op& op, U&&...arg)
{ if constexpr (is_relaxed<C>::value)
  if (!check_match(table, op, std::forward<U>(arg)...)) THROW_EXCEPTION(bad_function_call{}); }
void require_match<Op,...U>(const Op& op, U&&...arg)
{ if constexpr (is_relaxed<extract_concept<get_signature<Op>::type, U...>::type>::value)
  if (!check_match(op, std::forward<U>(arg)...)) THROW_EXCEPTION(bad_function_call{}); }

struct detail::is_placeholder_arg<T> : is_placeholder<remove_cv_ref_t<T>> {};
storage&       detail::convert_arg<T>  (any_base<T>                    & arg, true_) { return access::data(arg); }
storage const& detail::convert_arg<T>  (any_base<T>               const& arg, true_);
storage&       detail::convert_arg<C,T>(any_base<any<C,T      &>>      & arg, true_);
storage const& detail::convert_arg<C,T>(any_base<any<C,T const&>>      & arg, true_);
storage const& detail::convert_arg<C,T>(any_base<any<C,T const&>> const& arg, true_);
storage const& detail::convert_arg<C,T>(any_base<any<C,T const&>>     && arg, true_);
storage&       detail::convert_arg<C,T>(any_base<any<C,T&>>           && arg, true_);
storage&&      detail::convert_arg<C,T>(any_base<any<C,T>>            && arg, true_);
storage&&      detail::convert_arg<C,T>(any_base<any<C,T&&>>           & arg, true_);
storage&&      detail::convert_arg<C,T>(any_base<any<C,T&&>>      const& arg, true_);

storage&       detail::convert_arg<C,T>(param<C,T>        & arg, true_);
storage const& detail::convert_arg<C,T>(param<C,T const&> & arg, true_);
storage const& detail::convert_arg<C,T>(param<C,T>        & arg, true_);
storage const& detail::convert_arg<C,T>(param<C,T const&>&& arg, true_);
storage&       detail::convert_arg<C,T>(param<C,T&>      && arg, true_);
storage&&      detail::convert_arg<C,T>(param<C,T>       && arg, true_);
storage&&      detail::convert_arg<C,T>(param<C,T&&>      & arg, true_);
storage&&      detail::convert_arg<C,T>(param<C,T&&> const& arg, true_);

T&& detail::convert_arg<T>(T&& arg, false_) { return std::forward<T>(arg); }

struct detail::call_impl<Sig,Args,C=void,check=checkk_call<Sig,Args>::value>{};
struct detail::call_result<Op,Args,C=void> : call_impl<get_signature<Op>::type,Args,C>{};
struct detail::call_result<binding<C1>,Args,C>{};
void detail::ignore<...T>(const T&...){}
int detail::maybe_get_table<T,Table>(const T& arg, const Table*& table, ) { if (table==0) table=&access::table(arg); return 0; }
const binding<extract_concept_t<mp_list<T...>, mp_list<U...>>>*
  detail::extract_table<R,...T,...U>(R(*)(T...), const U&...arg)
{ const binding<...>* result =0; ignore(maybe_get_table(arg,result,is_placeholder_arg<T>())...); return result; }

struct detail::call_impl_dispatch<Sig,Args,C,returnsAny>;
struct detail::call_impl_dispatch<R(T...),void(U...),C,false> { using type = R;
  static type apply<F>(const binding<C>* table, U...arg)
  { return table->find<F>()(convert_arg(std::forward<U>(arg), is_placeholder_arg<T>())...); } };
struct detail::call_impl_dispatch<R(T...),void(U...),C,true> { using type = any<C,R>;
  static type apply<F>(const binding<C>* table, U...arg)
  { return type{table->find<F>()(convert_arg(std::forward<U>(arg), is_placeholder_arg<T>())...), *table}; } };
struct detail::call_impl<R(T...),void(U...),C,true> : call_impl_dispatch<R(T...),void(U...),C,is_placeholder_arg<R>::value>{};
struct detail::call_impl<R(T...),void(U...),void,true>
  : call_impl_dispatch<R(T...),void(U...), extract_concept_t<mp_list<T...>,mp_list<remove_reference_t<U>...>>, is_placeholder_arg<R>::value>{};

call_result<Op,void(U&&...),C>::type unchecked_call<C,Op,...U>(const binding<C>& table, const Op&, U&&...arg)
{ return call_impl<get_signature<Op>::type,void(U&&...),C>::apply<adapt_to_vtable<Op>::type>(&table, std::forward<U>(arg)...); }
call_result<Op,void(U&&...),C>::type call<C,Op,...U>(const binding<C>& table, const Op& f, U&&...arg)
{ require_match(table,f,std::forward<U>(arg)...); return unchecked_call(table,f,std::forward<U>(arg)...); }
call_result<Op,void(U&&...)>::type unchecked_call<Op,...U>(const Op&, U&&...arg)
{ return call_impl<get_signature<Op>::type,void(U&&...)>::apply<adapt_to_vtable<Op>::type>(
    extract_table((get_signature<Op>::type*)0, arg...), std::forward<U>(arg)...); }
call_result<Op,void(U&&...)>::type call<Op,...U>(const Op& f, U&&...arg)
{ require_match(f,std::forward<U>(arg)...); return unchecked_call(f,std::forward<U>(arg)...); }
```

------
### Common Details

```c++
using detail::make_mp_list<T> = ...;
using detail::eval_if<b, Temp<...>, F<...>, ...A> = ...;
using detail::first<T0,...> = T0;

struct detail::get_signature<C> { using type = remove_pointer_t<remove_cv_ref_t<decltype(&C::apply)>>; };

using detail::get_placeholders_f<Out,T> = ...;
struct detail::get_placeholders<F<T...>,Out> { using type = ...; };
using detail::get_placeholders_t<T,Out> = get_placeholders<T,Out>::type;
struct detail::get_placeholders<vtable_adaptor<PrimC,Sig>,Out> { using type = get_placeholders<PrimC,Out>::type; };

struct detail::replace_param_for_vtable<T> { using type = if_<is_placeholder<remove_cv_t<T>>, const storage&, T>; }; // T& -> storage&, const T& -> const storage&, T&& -> storage&&
struct detail::replace_result_for_vtable<T> { using type = if_<is_placeholder<remove_cv_t<T>>, storage, T>; }; // T& -> storage&, const T& -> storage&, T&& -> storage&&
struct detail::has_type<T>;
struct detail::is_internal_concept<T> : has_type<T>{};
struct detail::adapt_to_vtable<PrimC> { using type = vtable_adaptor<PrimC, get_vtable_signature<get_signature<PrimC>::type>::type>; };
struct detail::maybe_adapt_to_vtable<C> { using type = mpl::eval_if<is_internal_concept<C>, identity<C>, adapt_to_vtable<C>>::type; };
struct detail::vtable_adapter_impl<PrimC,R(T...),R2(U...)> { using type = R(*)(T...); static R value(T...arg); };
struct detail::vtable_adapter_impl<PrimC,storage[&|&&](T...),R2(U...)> { using type = storage(*)(T...); static storage value(T...arg); };
struct detail::vtable_adapter<PrimC,Sig> : vtable_adapter_impl<PrimC,Sig,get_signature<PrimC>::type> {};
struct detail::get_vtable_signature<R(T...)> { using type = replace_result_for_vtable<R>::type (replace_param_for_vtable<T>::type...); };

struct detail::null_throw<Sig>;
struct detail::null_throw<R(T...)> { static R value(T...){ THROW_EXCEPTION(bad_function_call{}); } };
struct detail::get_null_vtable_entry{ using type = null_throw<remove_pointer_t<C::type>>; };

struct detail::stored_arg_pack<...T>;
struct detail::make_arg_pack_impl<It,End,...T> { using type = make_arg_pack_impl<mpl::next<It>::type, End, T..., mpl::deref<It>::type>::type; };
struct detail::make_arg_pack_impl<End,End,T...> { using type = stored_arg_pack<T...>; };
struct detail::make_arg_pack<Seq> { using type = make_arg_pack_impl<mpl::begin<Seq>::type, mpl::end<Seq>::type>::type; };
struct detail::make_vtable<Args> { using type = make_vtable_impl<make_arg_pack<Seq>::type>::type; };
struct detail::make_vtalbe_init<Seq,Table> { using type = make_vtable_init_impl<Table,make_arg_pack<Seq>::type>::type; };
struct detail::vtable_entry<T> { T::type value; ctor()=default; constexpr ctor(T::type arg): value{arg}{} };
struct detail::compare_vtable<> { static bool apply<S>(const S&, const S&) { return true; } };
struct detail::compare_vtable<T0,T...> { static bool apply<S>(const S& s1, const S& s2) { return ((const vtable_entry<T0>&)s1).value == ((const vtable_entry<T0>&)s1).value && compare_vtable<T...>::apply(s1, s2); } };
struct detail::vtable_storage<...T> : vtable_entry<T>... {
    ctor()=default; explicit ctor(T::type...arg) : base(arg)... {}
    void convert_from<Bindings,Src>(const Src& src) { *this = {src.lookup((rebind_placeholders<T,Bindings>::type*)0)...}; }
    bool operator==(const self& other) const { return compare_vtable<T...>::apply(*this, other); }
    U::type lookup<U>(U*) const { return ((const vtable_entry<U>*)this)->value; }
};
struct detail::vtable_storage<> { ctor()=default; void convert_from<Bindings,Src>(const Src&){} bool operator==(const self&) const { return true; } };
struct detail::make_vtable_impl<stored_arg_pack<T...>> { using type = vtable_storage<T...>; };
struct detail::vtable_init<Table,...T> { static constexpr Table value = Table{T::value...}; };
struct detail::make_vtable_init_impl<Table,stored_arg_pack<T...>> { using type = vtable_init<Table,T...>; };

struct detail::dynamic_binding_impl<P> { const std::type_info* type; ctor()=default; constexpr ctor(const std::type_info* t):type{t}{} };
struct detail::dynamic_binding_element<T> { using type = const std::type_info*; };
struct detail::append_to_key<Table> {
  void operator()<P>(P) { key->push_back(((const dynamic_binding_impl<P>*)table)->type)};
  const Table* table; key_type* key; }
struct detail::dynamic_vtable<...P> : dynamic_binding_impl<P>... {
  ctor()=default; constexpr ctor(dynamic_binding_element<P>::type...t): base(t)...{}
  F::type lookup<F>(F*) const;
  void init() { *this = dynamic_vtable{&typeid(at<Bindings,P>::type)...}; }
  void convert_from<Bindings,Src>(const Src& src) { *this = dynamic_vtable(&src.lookup((typeid_<at<Bindings,P>::type>*0)())...); }
};
struct detail::make_dynamic_vtable_impl<stored_arg_pack<P...>> { using type = dynamic_vtable<P...>; };
struct detail::make_dynamic_vtable<PlaceholderList> : make_dynamic_vtable_impl<make_arg_pack<PlaceholderList>::type> {};

struct detail::identity<T> { using type = T; };
struct detail::rebind_placeholders<T,Bindings> { using type = void; };
using detail::rebind_placeholders_t<T,Bindings> = rebind_placeholders<T,Bindings>::type;
using detail::rebind_one_placeholder<T,Bindings> = mp_second<mp_map_find<Bindings,T>>;
using detail::rebind_placeholders_in_argument_t<T,Bindings> = ...;
using detail::rebind_placeholders_in_argument<T,Bindings> = identity<rebind_placeholders_in_argument_t<T,Bindings>>;
using detail::rebind_placeholders_in_deduced<F,Bindings> = deduced<rebind_placeholders_t<F,Bindings>>::type;
using detail::rebind_deduced_placeholder<F,Bindings> = mp_second<mp_map_find<Bindings,deduced<F>>>;
struct detail::rebind_placeholders<T<U...>,Bindings> { using type = T<rebind_placeholders_in_argument_t<U,make_mp_list<Bindings>>...>; };
struct detail::rebind_placeholders<vtable_adaptor<PrimC,Sig>,Bindings>
{ using type = vtable_adaptor<rebind_placeholders_t<PrimC,make_mp_list<Bindings>>, rebind_placeholders_in_argument_t<Sig,make_mp_list<Bindings>>...>; };

struct detail::combine_concepts<T,U>; // <T,T> => T, <T,void> => T, <void,T> => T, <void,void> => void
using detail::combine_concepts_t<T,U> = combine_concepts<T,U>::type;
using detail::extract_concept_or_void<T,U> = mp_eval_if_c<is_placeholder<remove_cv_ref_t<T>::value>,void,concept_of_t,U>;
using detail::extract_concept_t<L1,L2> = mp_fold<mp_transform<extract_concept_or_void,L1,L2>,void,combine_concepts_t>;
struct detail::maybe_extract_concept<T,U> { using type = mpl::eval_if<is_placeholder<remove_cv_ref_t<T>>, concept_of<remove_reference_t<U>>, identity<void>>::type; };
struct detail::extract_concept<R(T0,T...), U0,U...> { using type = combine_concepts<maybe_extract_concept<T0,U0>::type, extract_concept<void(T...),U...>::type>::type; };
struct detail::extract_concept<void()> { using type = void; };

struct detail::normalize_deduced<M,T<U...>> { using type = deduced<T<normalize_placeholder<M,U>...>>; };

struct detail::substitution_map_tag{};
struct detail::substitution_map<M> { using tag = substitution_map_tag; using map_type = M; };
struct detail::select_pair<T,U>; // mpl::pair<T,U> if any of T, U is deduced
using detail::resolve_same_type_t<M,T> = if_<mp_map_contains<M,T>, resolve_same_type_t<M,mp_second<mp_map_find<M,T>>>, T>;
using detail::resolve_same_type<M,T> = identity<resolve_same_type_t<M,T>>;
struct detail::normalize_deduced_impl<T> { using apply<M> = T; };
struct detail::normalize_deduced_impl<deduced<F<T...>>> { using apply<M> = deduced<F<normalize_placeholder_t<M,T>...>>::type; };
using detail::normalize_placeholder_t<M,T> = if_<mp_map_contains<M,T>, normalize_placeholder_t<M,mp_second<mp_map_find<M,T>>>,
    normalize_deduced_impl<T>::apply<M>>;
using detail::normalize_placeholder<M,T> = identity<normalize_placeholder_t<M,T>>;

struct detail::create_placeholder_map<M> {
    using transform_one<P> = mpl::pair<first<P>::type, normalize_placeholder_t<M,second<P>::type>>;
    using type = mp_transform<transform_one,M>;
};
using detail::create_placeholder_map_t<M> = create_placeholder_map<M>::type;
struct detail::convert_deduced<Binding,P,Out,Sub> {
    using result = mp_second<mp_map_find<Sub,rebind_placeholders_in_argument_t<P::first,Bindings>>>;
    using type = mp_map_insert<Out, pair<P::second,result>>;
};
struct convert_deduced_f<Bindings,Sub> { apply<Out,P> = convert_deduced<Bindings,P,Out,Sub>::type; };
using detail::convert_deductions_t<Bindings,M,Sub> = mp_fold<M, make_mp_list<Bindings>, convert_deduced_f<Bindings,Sub>::apply>;
using detail::convert_deductions<Bindings,M,Sub> = identity<convert_deductions_t<Bindings,M,Sub>>;
struct detail::add_deduced<Bindings,P,Out> {
    using result = rebind_placeholders_in_argument<P::first,Bindings>;
    using type = mp_map_insert<Out, pair<P::second,result>>;
};
struct detail::add_deduced_f<Bindings> { using apply<Out,P> = add_deduced<Bindings,P,Out>::type; };
using detail::add_deductions_t<Bindings,M> = mp_fold<M,make_mp_list<Bindings>, add_deduced_f<make_mp_list<Bindings>>::apply>;
using detail::add_deductions<Bindings,M> = identity<add_deductions_t<Bindings,M>>;
struct detail::insert_concept_impl<T> { using apply<Out> = pair<mp_set_push_back<Out::first,T>,Out::second>; };
using detail::insert_concept_same_type<T1,T2,Out> = pair<Out::first, eval_if<is_same_v<T1,T2>, first,mp_map_insert, Out::second,select_pair<T1,T2>::type>>;
struct detail::insert_concept_impl<same_type<T,U>> { using apply<Out> = insert_concept_same_type<resolve_same_type_t<Out::second,T>, resolve_same_type_t<Out::second,U>, Out>; };
struct detail::normalize_concept_impl_test<true> { using apply<Out,C> = mp_fold<make_mp_list<C>,Out,normalize_concept_impl_f>; };
struct detail::normalize_concept_impl_test<false> { using apply<Out,C> = insert_concept_impl<C>::apply<Out>; };
using detail::normalize_concept_impl_f<Out,C> = normalize_concept_impl_test<mpl::is_sequence<C>::value>::apply<Out,C>;
using detail::normalize_concept_impl_t<C> = normalize_concept_impl_f<pair<mp_list<>,mp_list<>>,C>;
using detail::normalize_concept_impl<C = identity<normalize_concept_impl_t<C>>>;
using detail::get_all_placeholders_impl<S,T> = get_placeholders<T,S>::type;
using detail::get_all_placeholders<Seq> = mp_fold<Seq,mp_list<>,get_all_placeholders_impl>;
using detail::make_identity_pair<T> = mpl::pair<T,T>;
using detail::make_identity_placeholder_map<C> = mp_transform<make_identity_pair,get_all_placeholders<normalize_concept_impl_t<C>::first>>;
using detail::append_type_info<S,T> = mp_set_push_back<S,typeid_<T>>;
using detail::add_typeinfo_t<Seq> = mp_fold<get_all_placeholders<Seq>, Seq, append_type_info>;
using detail::add_typeinfo<Seq> = identity<add_typeinfo_t<Seq>>;
struct detail::normalize_concept_substitute_f<Subs> { using apply<Set,C> = mp_set_push_back<Set,rebind_placeholders<C,Subs>::type>; };
using detail::normalize_concept_adjustments<C,Pair> = eval_if<is_relaxed<C>::value, add_typeinfo_t, first, mp_fold<Pair::first,mp_list<>,normalize_concept_substitute_f<create_placeholder_map_t<Pair::second>>::apply>>;
using detail::get_placeholder_normalization_map_t<C> = create_placeholder_map_t<normalize_concept_impl_t<C>::second>;
using detail::get_placeholder_normalization_map<C> = create_placeholder_map<normalize_concept_impl_t<C>::second>;
using detail::normalize_concept_t<C> = normalize_concept_adjustments<C,normalize_concept_impl_t<C>>;
using detail::normalize_concept<C> = identity<normalize_concept_t<C>>;
using detail::collect_concepts_recursive<Out,C,M> = mp_fold<make_mp_list<C>,Out,collect_concepts_f<M>::apply>;
using detail::collect_concepts_impl<C,M,Out,Transformed> = eval_if<is_same_v<Transformed,void>, first, mp_set_push_front, eval_if<is_sequence<C>::value, collect_concepts_recursive, first, Out,C,M>, Transformed>;
using detail::collect_concepts_t<C,M=create_placeholder_map_t<normalize_concept_impl_t<C>::second>,Out=mp_list<>> = collect_concepts_impl<C,M,Out,rebind_placeholders<C,M>::type>;
struct detail::collect_concepts_f<M> { using apply<Out,C> = collect_concepts_f<C,M,Out>; };
using detail::collect_concepts<C> = identity<collect_concepts_t<C>>;

struct detail::is_deduced<T> : false_{};
struct detail::is_deduced<deduced<T>> : true_{};
struct detail::check_map<C,Map> {
    using placeholders = get_all_placeholders<normalize_concept_t<C>>;
    using placeholder_subs = get_placeholder_normalization_map<C>::type;
    using okay_placeholders = mp_unique<mp_append< mp_transform<mp_first,make_mp_list<Map>>, mp_transform<mp_second,make_mp_list<placeholder_subs>> >>;
    using check_placeholder<P> = or_<is_deduced<P>,mp_set_contains<okay_placeholders,P>>;
    using type = mp_all_of<placeholders, check_placeholder>;
};
struct detail::check_map<C,static_binding<Map>> : check_map<C,Map>{};

struct detail::storage {
    ctor(){} ctor(self {<const>&|&&} o); self& operator=(const self& o);
    explicit ctor(T&& arg) : data{new decay_t<T>{std::forward<T>(arg)}}{}
    void* data;
};
T detail::extract<T>(T arg) { return std::forward<T>(arg); }
T detail::extract(storage {<const>&|&&} arg) { return <std::move>(*(remove_reference_t<T>*)arg.data); }

struct detail::access {
    static const D       ::table_type& table<D>  (any_base<D       >& arg) { return ((const D&)arg); }
    static const any<C,T>::table_type& table<C,T>(any_base<any<C,T>>& arg) requires !is_reference_v<T> { return ((const any<C,T>&)arg)._b_te_table; }
    static const binding<C>& table<C,T>(param<C,T>& arg) { return table(arg._impl); }
    static const binding<C>& table<C,T>(param<C,T{&|&&}>& arg) { return arg._impl.table; }

    static storage      &  data<D>  (any_base<D>      &  arg) { return static_cast<D&>(arg).data; }
    static storage const&  data<D>  (any_base<D> const&  arg) { return static_cast<const D&>(arg).data; }
    static storage      && data<D>  (any_base<D>      && arg) { return std::move(static_cast<D&>(arg).data); }
    static storage      &  data<C,T>(any_base<any<C,T        >>      &  arg) { return static_cast<any<C,T>&>(arg)._boost_type_erasure_data; }
    static storage const&  data<C,T>(any_base<any<C,T        >> const&  arg) { return static_cast<const any<C,T>&>(arg)._boost_type_erasure_data; }
    static storage      && data<C,T>(any_base<any<C,T        >>      && arg) { return std::move(static_cast<any<C,T>&&>(arg)._boost_type_erasure_data); }
    static storage      &  data<C,T>(any_base<any<C,T      & >>      &  arg) { return const_cast<storage&>(static_cast<any<C,T&>&>(arg).data); }
    static storage      &  data<C,T>(any_base<any<C,T      & >> const&  arg) { return const_cast<storage&>(static_cast<const any<C,T&>&>(arg).data); }
    static storage      &  data<C,T>(any_base<any<C,T      & >>      && arg) { return std::move(static_cast<any<C,T&>&>(arg).data); }
    static storage const&  data<C,T>(any_base<any<C,T const& >>      &  arg) { return static_cast<any<C,const T&>&>(arg).data; }
    static storage const&  data<C,T>(any_base<any<C,T const& >> const&  arg) { return static_cast<const any<C,const T&>&>(arg).data; }
    static storage      && data<C,T>(any_base<any<C,T      &&>>      &  arg) { return std::move(static_cast<any<C,T&&>&>(arg).data); }
    static storage      && data<C,T>(any_base<any<C,T      &&>> const&  arg) { return std::move(const_cast<storage&>(static_cast<const any<C,T&&>&>(arg).data)); }
    static storage      && data<C,T>(any_base<any<C,T      &&>>      && arg) { return std::move(static_cast<any<C,T&&>&>(arg).data); }
    static storage      &  data<C,T>(param<C,T        >      &  arg) { return data(arg._impl); }
    static storage const&  data<C,T>(param<C,T        > const&  arg) { return data(arg._impl); }
    static storage      && data<C,T>(param<C,T        >      && arg) { return std::move(data(arg._impl)); }
    static storage      &  data<C,T>(param<C,T      & >      &  arg) { return arg._impl.data; }
    static storage const&  data<C,T>(param<C,T      & > const&  arg) { return arg._impl.data; }
    static storage      &  data<C,T>(param<C,T      & >      && arg) { return arg._impl.data; }
    static storage const&  data<C,T>(param<C,T const& >      &  arg) { return arg._impl.data; }
    static storage const&  data<C,T>(param<C,T const& >      && arg) { return arg._impl.data; }
    static storage      && data<C,T>(param<C,T      &&>      &  arg) { return std::move(arg._impl.data); }
    static storage      && data<C,T>(param<C,T      &&> const&  arg) { return std::move(const_cast<storage&>(arg._impl.data)); }
    static storage      && data<C,T>(param<C,T      &&>      && arg) { return std::move(arg._impl.data); }
};

struct detail::check_call<Sig,Args>:false_{};
struct detail::qualified_placeholder<T>;
struct detail::check_placeholder_arg<P,Arg>;
using detail::check_placeholder_arg_t<P,Arg> = ...;
using detail::check_nonplaceholder_arg_t<P,Arg> = ...;
using detail::check_arg_t<P,Arg> = ...;
struct detail::check_arg<P,Arg> { using type = check_arg_t<P,Arg>; };
struct detail::check_call<R(T...), void(U...)> { using type = mp_all<check_arg<T,U>::type...>; };

struct detail::instantiate_concept<T,T t>;
using detail::instantiate_concept_impl<T> = instantiate_concept<decltype(&T::apply), &T::apply>;
struct detail::make_instantiate_concept_impl<mp_list<T...>> { using apply<F<_>> = void(F<T>...); };
struct detail::instantiate_concept_rebind_f<Map> { using apply<T> = rebind_placeholders<T,Map>::type; };
using detail::make_instantiate_concept<C,Map> = make_instantiate_concept_impl<mp_transform<
    instantiate_concept_rebind_f<add_deductions< make_mp_list<Map>, get_placeholder_normalization_map<C>::type >::type>::apply, normalize_concept_t<C>>;

using detail::choose_member_interface<P,Interface<...>,Sig,C,Base,ID>
  = if_c<is_reference_v<P>, if_c<is_non_const_ref<P>::value,Interface<Sig,C,Base,const ID>,Bae>::type, Interface<Sig,C,Base,ID>>::type;
struct detail::choose_member_impl<Sig>;
struct detail::choose_member_impl<R(C::*)(T...)const> { using apply<P,M<_>,S<_,_>> = vector<S<R(T...),const P>>; };
struct detail::choose_member_impl<R(C::*)(T...)const> { using apply<P,M<_>,S<_,_>> = M<R(P&,T...)>; };
using detail::choose_member_impl_t<Sig,P,M<_>,Self<_,_>> = choose_member_impl<Sig(dummy::*)>::apply<P,M,Self>;
struct detail::member_interface_choose<Sig,T,ID> { using apply<Base,C<_,_>,M<...>> = Base; };
struct detail::member_interface_choose<R(A...),T,T> { using apply<Base,C<_,_>,M<...>> = choose_member_interface<placeholder_of_t<Base>,M,R(A...),C<R(A...),T>,Base,T>; };
struct detail::member_interface_choose<R(A...),const T,T> { using apply<Base,C<_,_>,M<...>> = M<R(A...),C<R(A...),const T>,Base,const T>; };
struct detail::member_choose_interface<Sig,T,C<_,_>,M<...>> { using apply<C,Base,ID> = member_interface_choose<Sig,T,ID>::apply<Base,C,M>; };

struct detail::is_non_const_ref<T> : false_{};
struct detail::is_non_const_ref<T&> : true_{};
struct detail::is_non_const_ref<const T&> : false_{};
struct detail::should_be_const<Ph,Base> : or_<is_const<Ph>,is_non_const_ref<placeholder_of<Base>::type>>{};
struct detail::should_be_non_const<Ph,Base> : and_<not_<is_const<Ph>>,not_<is_reference<placeholder_of<Base>::type>>>{};
struct detail::non_const_this_param<Base> { using ph = placeholder_of<Base>::type; using t = derived<Base>::type;
  using type = if_<is_same<ph, remove_cv_ref_t<ph>&>,const t,t>::type; };
struct detail::uncallable<T>{};
struct detail::maybe_const_this_param<Ph,Base> { using ph = remove_reference_t<Ph>; using t = derived<Derived>::type;
  using type = if_<is_reference<Ph>, if_<should_be_non_const<ph,Base>, t&, if_<should_be_const<ph,Base>, const t&, uncallable<t>>::type>::type, t>::type; };
```

------
### Configuration

* `MAX_FUNCTIONS`: 50
* `MAX_ARITY`: 5
* `MAX_TUPLE_SIZE`: 5
* `USE_MP11`

------
### Dependencies

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`, `<boost/config/workaround.hpp>`
* `<boost/config/auto_link.hpp>`

#### Boost.Core

* `<boost/utility/addressof.hpp>`
* `<boost/utility/enable_if.hpp>`

#### Boost.Fusion

* `<boost/fusion/include/**.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_adaptor.hpp>`
* `<boost/iterator/iterator_categories.hpp>`

#### Boost.MP11

* `<boost/mp11/*.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.SmartPtr

* `<boost/make_shared.hpp>`
* `<boost/shared_ptr.hpp>`

#### Boost.Thread

* `<boost/thread/shared_mutex.hpp>`
* `<boost/thread/lock_types.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`
* `<boost/utility/declval.hpp>`

#### Boost.TypeOf

* `<boost/typeof/typeof.hpp>`

#### Boost.VMD

* `<boost/vmd/is_empty.hpp>`

------
### Standard Facilities
