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

any_cast, any, binding_of, binding, callable,
derived, dynamic_any_cast, dynamic_binding, free,
is_empty, is_subconcept, iterator, member, operators,
register_binding, same_type, static_binding, tuple, typeid_of

detail/: auto_link, check_map, const, construct, dynamic_vtable,
instantiate, macro, member11, normalize, null, vtable

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

struct detail::normalize_deduced<M,T>;
struct detail::normalize_placeholder<M,T>;

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
