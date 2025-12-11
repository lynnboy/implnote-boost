# Boost.TypeErasure

* lib: `boost/libs/type_erasure`
* repo: `boostorg/type_erasure`
* commit: `5281f14`, 2025-05-05

------
### Any

```c++
struct any_base<Derived> {
    using _b_te_is_any=void; using _b_te_derived_type=Derived;
    void* _b_te_deduce_constructor(...) const volatile {return 0;}
    void* _b_te_deduce_assign(...) {return 0;}
};
```

### Placeholder

```c++
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

struct relaxed : vector<>{};
struct is_relaxed<C>;
```

any_cast, any, binding_of, binding, builtin, call, callable, check_match, concept_interface,
constructible, derived, dynamic_any_cast, dynamic_binding, exception, free,
is_empty, is_subconcept, iterator, member, operators, param, placeholder,
rebind_any, register_binding, require_match, same_type, static_binding, tuple, typeid_of

detail/: auto_link, check_map, const, construct, dynamic_vtable,
instantiate, macro, member11, normalize_deduced, normalize, null, vtable

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
