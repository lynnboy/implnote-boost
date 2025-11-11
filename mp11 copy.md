# Boost.MP11

* lib: `boost/libs/mp11`
* repo: `boostorg/mp11`
* commit: `1f3d0bb`, 2025-09-09

------
### Integral Constants

* Header `<boost/mp11/integral.hpp>`

```c++
using mp_value<v> = std::integral_constant<decltype(v),v>;
using mp_bool<b> = std::integral_constant<bool,b>; // std::bool_constant<b>
using mp_true = mp_bool<true>; using mp_false = mp_bool<false>; // std::true_type; std::false_type
using mp_to_bool<T> = mp_bool<(bool)T::value>;
using mp_not<T> = mp_bool<!T::value>;
using mp_int<i> = std::integral_constant<int,i>;
using mp_size_t<n> = std::integral_constant<std::size_t,n>;
```

------
### List and Operations

* Header `<boost/mp11/list.hpp>`

```c++
struct mp_list<...T> {}; // just empty, encode types in params
struct mp_list_v<...v> {}; // just empty, encode values in params
using mp_list_c<T,T...i> = mp_list<std::integral_constant<T,i>...>; // list of integer types

struct detail::mp_is_list_impl<L> { using type = mp_false; }; //primary
struct detail::mp_is_list_impl<L<T...>> { using type = mp_true; };
using mp_is_list<L> = mp_is_list_impl<L>::type;
struct detail::mp_is_value_list_impl<L> { using type = mp_false; }; //primary
struct detail::mp_is_value_list_impl<L<v...>> { using type = mp_true; };
using mp_is_value_list<L> = mp_is_value_list_impl<L>::type;

struct detail::mp_size_impl<L> {};
struct detail::mp_size_impl<L<T...>> { using type = mp_size_t<sizeof...(T)>; };
struct detail::mp_size_impl<L<v...>> { using type = mp_size_t<sizeof...(v)>; }; // for value list
using mp_size<L> = mp_size_impl<L>::type;
using mp_empty<L> = mp_bool<mp_size<L>::value == 0>;

struct detail::mp_assign_impl<L1,L2> {}; // for non-lists
struct detail::mp_assign_impl<L1<T...>,L2<U...>> { using type = L1<U...>; }; // for type list => type list
struct detail::mp_assign_impl<L1<v...>,L2<U...>> { using type = L1<U::value...>; }; // for type list => value list
struct detail::mp_assign_impl<L1<T...>,L2<u...>> { using type = L1<mp_value<u>...>; }; // for value list => type list
struct detail::mp_assign_impl<L1<v...>,L2<u...>> { using type = L1<u...>; }; // for value list => value list
using mp_assign<L1,L2> = mp_assign_impl<L1,L2>::type;
using mp_clear<L> = mp_assign<L,mp_list<>>;

struct detail::mp_front_impl<L> {}; // for non-list or empty list
struct detail::mp_front_impl<L<T1,T...>> { using type = T1; };
struct detail::mp_front_impl<L<v1,v...>> { using type = mp_value<v1>; };
using mp_front<L> = mp_front_impl<L>::type;
using mp_first<L> = mp_front<L>;

struct detail::mp_pop_front_impl<L> {}; // for non-list or empty list
struct detail::mp_pop_front_impl<L<T1,T...>> { using type = L<T...>; };
struct detail::mp_pop_front_impl<L<v1,v...>> { using type = L<v...>; }; // for value list
using mp_pop_front<L> = mp_pop_front_impl<L>::type;
using mp_rest<L> = mp_pop_front<L>;

struct detail::mp_second_impl {}; // for non-list or list size < 2
struct detail::mp_second_impl<L<T1,T2,T...>> { using type = T2; };
struct detail::mp_second_impl<L<v1,v2,v...>> { using type = mp_value<v2>; };
using mp_second<L> = mp_second_impl<L>::type;

struct detail::mp_third_impl {}; // for non-list or list size < 3
struct detail::mp_third_impl<L<T1,T2,T3,T...>> { using type = T3; };
struct detail::mp_third_impl<L<v1,v2,v3,v...>> { using type = mp_value<v3>; };
using mp_third<L> = mp_third_impl<L>::type;

struct detail::mp_push_front_impl<L,...T> {}; // for non-list
struct detail::mp_push_front_impl<L<U...>,T...> { using type = L<T...,U...>; };
struct detail::mp_push_front_impl<L<v...>,T...> { using type = L<T::value...,v...>; }; // for value list
using mp_push_front<L,...T> = mp_push_front_impl<L,T...>::type;

struct detail::mp_push_back_impl<L,...T> {}; // for non-list
struct detail::mp_push_back_impl<L<U...>,T...> { using type = L<U...,T...>; };
struct detail::mp_push_back_impl<L<v...>,T...> { using type = L<v...,T::value...>; }; // for value list
using mp_push_back<L,...T> = mp_push_back_impl<L,T...>::type;

struct detail::mp_rename_impl<L,B<...>> {}; // for non-list
struct detail::mp_rename_impl<L<T...>,B> : mp_defer<B,T...> {}; // change L<T...> to B<T...>
struct detail::mp_rename_impl<L<v...>,B> : mp_defer<B,mp_value<v>...> {}; // change L<v...> to B<mp_value<v>...>
using mp_rename<L,B<...>> = mp_rename_impl<L,B>::type;
using mp_apply<F<...>,L> = mp_rename_impl<L,F>::type;
using mp_apply_q<Q,L> = mp_rename_impl<L,Q::fn>::type;
struct detail::mp_rename_v_impl<L,B<...>>{}; // for non-list
struct detail::mp_rename_v_impl<L<T...>,B> { using type = B<T::value>; }; // change L<T...> to B<v...>
struct detail::mp_rename_v_impl<L<v...>,B> { using type = B<v...>; }; // change L<v...> to B<v...>
using mp_rename_v<L,B<...>> = mp_rename_v_impl<L,B>::type;

struct detail::mp_replace_front_impl<L,T> {}; // for non-list or empty list
struct detail::mp_replace_front_impl<L<U1,U...>,T> { using type = L<T,U...>; };
struct detail::mp_replace_front_impl<L<v1,v...>,T> { using type = L<T::value,v...>; };
using mp_replace_front<L,T> = mp_replace_front_impl<L,T>::type;
using mp_replace_first<L,T> = mp_replace_front<L,T>;

struct detail::mp_replace_second_impl<L,T> {}; // for non-list or list size < 2
struct detail::mp_replace_second_impl<L<U1,U2,U...>,T> { using type = L<U1,T,U...>; };
struct detail::mp_replace_second_impl<L<v1,v2,v...>,T> { using type = L<v1,T::value,v...>; };
using mp_replace_second<L,T> = mp_replace_second_impl<L,T>::type;

struct detail::mp_replace_third_impl<L,T> {}; // for non-list or list size < 3
struct detail::mp_replace_third_impl<L<U1,U2,U3,U...>,T> { using type = L<U1,U2,T,U...>; };
struct detail::mp_replace_third_impl<L<v1,v2,v3,v...>,T> { using type = L<v1,v2,T::value,v...>; };
using mp_replace_third<L,T> = mp_replace_third_impl<L,T>::type;

struct detail::mp_transform_front_impl<L,F<...>> {}; // for non-list or empty list
struct detail::mp_transform_front_impl<L<U1,U...>,F> { using type = L<F<U1>,U...>; };
struct detail::mp_transform_front_impl<L<v1,v...>,F> { using type = L<F<mp_value<v1>::value,v...>; };
using mp_transform_front<L,F<...>> = mp_transform_front_impl<L,F>::type;
using mp_transform_front_q<L,Q> = mp_transform_front<L,Q::fn>;
using mp_transform_first<L,F<...>> = mp_transform_front<L,F>;
using mp_transform_first_q<L,Q> = mp_transform_front_q<L,Q>;

struct detail::mp_transform_second_impl<L,F<...>> {}; // for non-list or list size < 2
struct detail::mp_transform_second_impl<L<U1,U2,U...>,F> { using type = L<U1,F<U2>,U...>; };
struct detail::mp_transform_second_impl<L<v1,v2,v...>,F> { using type = L<v1,F<mp_value<v2>::value,v...>; };
using mp_transform_second<L,F<...>> = mp_transform_second_impl<L,F>::type;
using mp_transform_second_q<L,Q> = mp_transform_second<L,Q::fn>;

struct detail::mp_transform_third_impl<L,F<...>> {}; // for non-list or list size < 3
struct detail::mp_transform_third_impl<L<U1,U2,U3,U...>,F> { using type = L<U1,U2,F<U3>,U...>; };
struct detail::mp_transform_third_impl<L<v1,v2,v3,v...>,F> { using type = L<v1,v2,F<mp_value<v3>::value,v...>; };
using mp_transform_third<L,F<...>> = mp_transform_third_impl<L,F>::type;
using mp_transform_third_q<L,Q> = mp_transform_third<L,Q::fn>;

struct detail::append_11_impl<L1=mp_list<>,...,L11=mp_list<>> {}; // for non-list and empty list
struct detail::append_11_impl<L1<T1...>,...,L11<T11...>> { using type = L1<T1,...,...,T11...>; };
struct detail::append_111_impl<L00=mp_list<>,...,LA9=mp_list<>> { using type = append_11_impl<append_11_impl<L00,...,L0A>::type,...,append_11_impl<LA0,...,LA9>::type>::type; };
struct detail::append_inf_impl<L00,...,LA9,...Lr> { using type = append_111_impl<append_111_impl<L00,...,LA9>::type,Lr...>::type; };
struct detail::mp_append_impl<...L> : mp_cond<
        mp_bool(sizeof...(L) > 111)>, mp_quote<append_inf_impl>,
        mp_bool(sizeof...(L)>11)>, mp_quote<append_111impl>,
        mp_true, mp_quote<append_11_impl>
    >::fn<L...> {};
struct detail::append_type_lists{ using fn = mp_append_impl<L...>::type; } // quoted meta function
struct detail::append_value_11_impl<L1,...,L11>; // for value lists
struct detail::append_value_111_impl<L00,...,LA9>; // for value lists
struct detail::append_value_inf_impl<L00,...,LA9,...Lr>; // for value lists
struct detail::append_value_impl<...L>; // for value lists
struct detail::append_value_lists{ using fn = append_value_impl<L...>::type; } // quoted meta function
using mp_append<...L> = mp_if_c< // all are value lists
        (sizeof...(L) > 0 && sizeof...(L) == mp_count_if<mp_list<L...>, mp_is_value_list>::value),
        append_value_lists, append_type_lists
    >::fn<L...>;
```

List is class template with just type parameters: `mp_list<char[],void>`, `std::pair<int,float>`, `std::shared_ptr<X>`.

------
### Utilities

* Header `<boost/mp11/utility.hpp>`

```c++
struct mp_identity<T> { using type = T; };
using mp_identity_t<T> = mp_identity<T>::type;
struct mp_inherit<...T> : T... {};

struct mp_if_c_impl<cond,Then,...Else> {};
struct mp_if_c_impl<true,Then,Else...> { using type = Then; };
struct mp_if_c_impl<false,Then,Else> { using type = Else; };
using mp_if_c<cond,Then,...Else> = mp_if_c_impl<cond,Then,Else...>::type;
using mp_if<Cond,Then,...Else> = mp_if_c_impl<(bool)Cond::value,Then,Else...>::type;

struct detail::mp_valid_impl<F<...>, ...T> { using type = {SFINAE F<T...>}; };
using mp_valid<F<...>,...T> = mp_valid_impl<F,T...>::type;
using mp_valid_q<Q,...T> = mp_vlaid<Q::fn, T...>;

struct detail::mp_defer_impl<F<...>, ...T> { using type = F<T...>; };
struct detail::mp_no_type {};
using mp_defer<F<...>,...T> = mp_if<mp_valid<F,T...>, mp_defer_impl<F,T...>, mp_no_type>;

struct detail::mp_eval_if_c_impl<cond=true,Then,F<...>,...U> { using type = Then; }
struct detail::mp_eval_if_c_impl<false,Then,F,U...> : mp_defer<F,U...> {};
using mp_eval_if_c<cond,Then,F<...>,...U> = mp_eval_if_c_impl<cond,Then,F,U...>::type;
using mp_eval_if<Cond,Then,F<...>,...U> = mp_eval_if_c_impl<(bool)Cond::value,Then,F,U...>::type;
using mp_eval_if_q<Cond,Then,Q,...U> = mp_eval_if_c_impl<(bool)Cond::value,Then,Q::fn,U...>::type;
using mp_eval_if_not<Cond,Then,F<...>,...U> = mp_eval_if<mp_not<Cond>,Then,F,U...>;
using mp_eval_if_not_q<Cond,Then,Q,...U> = mp_eval_if<mp_not<Cond>,Then,Q::fn,U...>;
using mp_eval_or<T,F<...>,...U> = mp_eval_if_not<mp_valid<F,U...>,T,F,U...>;
using mp_eval_or_q<T,Q,...U> = mp_eval_or<T,Q::fn,U...>;

using mp_valid_and_true<F<...>,...T> = mp_eval_or<mp_false,F,T...>;
using mp_valid_and_true_q<Q,...T> = mp_valid_and_true<Q::fn,T...>;

using detail::mp_cond_<Cond,Then,...Else> = mp_eval_if<Cond,Then,mp_cond,Else...>; //recursion
struct detail::mp_cond_impl<Cond,Then,...Else> : mp_defer<mp_cond_, Cond,Then, Else...> {};
using mp_cond<Cond,Then,...Else> = mp_cond_impl<Cond,Then,Else...>;

struct mp_quote<F<...>> { using fn<...T> = mp_defer<F,T...>::type; };
struct mp_quote_trait<F<...>> { using fn<...T> = F<T...>::type; };

using mp_invoke_q<Q,...T> = Q::fn<T...>;

struct mp_not_fn<P<...>> { using fn<...T> = mp_not<mp_invoke_q<mp_quote<P>,T...>>; };
using mp_not_fn_q<Q> = mp_not_fn<Q::fn>;

using detail::mp_compose_helper<L,Q> = mp_list<mp_apply_q<Q,L>>;
struct mp_compose<...F<...>> { using fn<...T> = mp_front<mp_fold< mp_list<mp_quote<F>...>, mp_list<T...>, mp_compose_helper >>; };
struct mp_compose_q<...Q> { using fn<...T> = mp_front<mp_fold< mp_list<Q...>, mp_list<T...>, mp_compose_helper >>; };
```

------
### Algorithms

* Header `<boost/mp11/algorithm.hpp>`

```c++
struct detail::mp_transform_impl<F<...>,...L> {};
struct detail::mp_transform_impl<F,L<T...>> { using type = L<F<T>...>; }; // L<T> -> L<F<T>>
struct detail::mp_transform_impl<F,L1<T1...>,L2<T2...>> { using type = L1<F<T1,T2>...>; }; // L<T1>,L<T2> -> L<F<T1,T2>>
struct detail::mp_transform_impl<F,L1<T1...>,L2<T2...>,L3<T3...>> { using type = L1<F<T1,T2,T3>...>; }; // L<T1>,L<T2>,L<T3> -> L<F<T1,T2,T3>>
struct detail::mp_transform_impl<F,L<v...>> { using type = L<F<mp_value<v>>::value...>; }; // L<v> -> L<F<mp_value<v>>::value>
struct detail::mp_transform_impl<F,L1<v1...>,L2<v2...>> { using type = L1<F<mp_value<T1>,mp_value<T2>>::value...>; }; // L<T1>,L<T2> -> L<F<T1,T2>>
struct detail::mp_transform_impl<F,L1<v1...>,L2<v2...>,L3<v3...>> { using type = L1<F<mp_value<T1>,mp_value<T2>,mp_value<T3>>::value...>; }; // L<T1>,L<T2>,L<T3> -> L<F<T1,T2,T3>>
struct detail::mp_transform_impl<F,L1<v1...>,L2<v2...>,L3<v3...>,L4<v4...>,L...> {
    using A1 = L1<mp_list<T1,T2,T3,T4>...>; // zip-pack first 4 params into list
    using _f<V,T> = mp_transform<mp_push_back,V,T>; using A2 = mp_fold<mp_list<L...>, A1, _f>; // pack following params by folding
    using _g<T> = mp_apply<F,T>; using type = mp_transform<_g,A2>; // apply F on each packed tuple in list
};
using mp_transform<F<...>,...L> = mp_if<mp_same<mp_size<L>...>, mp_transform_impl<F,L...>, list_size_mismatch>::type; // all same size or report error
using mp_transform_q<Q,...L> = mp_transform<Q::fn,L...>;

struct detail::mp_transform_if_impl<P<...>,F<...>,...L> {
    using Qp = mp_quote<P>; using Qf = mp_quote<F>;
    using _f<...U> = mp_eval_if_q<mp_not<mp_invoke_q<Qp,U...>>, mp_first<mp_list<U...>, Qf, U...>>;
    using type = mp_transform<_f,L...>;
};
using mp_transform_if<P<...>,F<...>,...L> = mp_transform_if_impl<P,F,L...>::type;
using mp_transform_if_q<Qp,Qf,...L> = mp_transform_if_impl<Qp::fn,Qf::fn,L...>::type;

struct detail::mp_filter_impl {
    using _f<T1,...T> = mp_if< mp_invoke_q<mp_quote<P>,T1,T...>, mp_list<T1>, mp_list<> >;
    using type = mp_assign<L1, mp_apply< mp_append, mp_transform<_f,L1,L...> >>;
};
using mp_filter<P<...>,...L> = mp_filter_impl<P,L...>::type;
using mp_filter_q<Q,...L> = mp_filter_impl<Q::fn,L...>::type;

struct detail::mp_fill_impl<L,V> {}; // for non-list
struct detail::mp_fill_impl<L<T...>,V> { using type = L<V<T>...>; };
struct detail::mp_fill_impl<L<v...>,V> { using type = L<((void)v,V::value)...>; };
using mp_fill<L,V> = mp_fill_impl<L,V>::type;

constexpr size_t detail::cx_plus() { return 0; } // recursion end
constexpr size_t detail::cx_plus <T1,...T> (T1 t1, T...t) { return (size_t)t1 + cx_plus(t...); }
constexpr size_t detail::cx_count <V,...T> () {
    constexpr bool a[] = { false, std::is_same_v<T,V>... };
    for (std::size_t r=0, i=0; i<sizeof...(T); ++i) r+=a[i+1]; // count true (T==V)
    return r;
}
struct detail::mp_count_impl<L<T...>,V> { using type = mp_size_t<cx_count<V,T...>()>; }
using mp_count<L,V> = mp_count_impl<L,V>::type;
constexpr size_t detail::cx_count_if <P<...>,...T> () {
    constexpr bool a[] = { false, P<T>::value... };
    for (std::size_t r=0, i=0; i<sizeof...(T); ++i) r+=a[i+1]; // count true (P<T>::value is true)
    return r;
}
struct detail::mp_count_impl<L<T...>,P> { using type = mp_size_t<cx_count_if<P,T...>()>; }
using mp_count_if<L,P<...>> = mp_count_if_impl<L,P>::type;

using mp_contains<L,V> = mp_to_bool<mp_count<L,V>>;

struct detail::mp_repeat_c_impl<L,n> { using type = mp_append<mp_repeat_c_impl<L,n/2>::type, mp_repeat_c_impl<L,n/2>::type, mp_repeat_c_impl<L,n%2>::type>; };
struct detail::mp_repeat_c_impl<L,0> { using type = mp_clear<L>; };
struct detail::mp_repeat_c_impl<L,1> { using type = L; };
using mp_repeat_c<L,n> = mp_repeat_c_impl<L,n>::type;
using mp_repeat<L,N> = mp_repeat_c_impl<L,(size_t)N::value>::type;

// Cartesian product
struct detail::mp_product_impl<F<...>,...L> {};
struct detail::mp_product_impl<F> { using type = mp_list<F<>>; }; // 0 list
struct detail::mp_product_impl<F,L1,L...> { using type = mp_assign<L1, mp_product_impl_2<F,mp_list<>,L1,L...>::type>; };
using mp_product<F<...>,...L> = mp_product_impl<F,L...>::type;
using mp_product_q<Q,...L> = mp_product_impl<Q::fn,L...>::type;

struct detail::mp_drop_impl<L,L2,En>;
using mp_drop_c<L,n> = mp_assign<L,mp_drop_impl<mp_rename<L,mp_list>,mp_repeat_c<mp_list<void>,n>, mp_bool<n<= mp_size<L>::value>>::type;
using mp_drop<L,N> = mp_drop_c<L,(size_t)N::value>;

struct detail::mp_from_sequence_impl<S,From,z>;
using mp_from_sequence<S,From=mp_int<0>> = mp_from_sequence_impl<S,From,From::value==0>::type;
using mp_iota_c<n,from=0> = mp_from_sequence<make_index_sequence<n>, mp_size_t<from>>;
using mp_iota<N,From=mp_int<0>> = mp_from_sequence<make_index_sequence<decltype(N::value), N::value>, F>;

struct detail::mp_at_c_impl;
using mp_at_c<L,i> = mp_if_c<!(i > mp_size<L>::value), mp_at_c_impl<L,i>, void>::type;
using mp_at<L,I> = mp_at_c<L,(size_t)I::value>;

struct detail::mp_take_c_impl<n,L,E=void>;
using mp_take_c<L,n> = mp_assign<L,mp_take_c_impl<n,mp_rename<L,mp_list>>::type>;
using mp_take<L,N> = mp_take_c<L,(size_t)N::value>;

using mp_slice_c<L,i,j> = mp_drop_c<mp_take_c<L,j>,i>;
using mp_slice<L,I,J> = mp_drop<mp_take<L,J>,I>;

using mp_back<L> = mp_at_c<L,mp_size<L>::value-1>;
using mp_pop_back<L> = mp_take_c<L,mp_size<L>::value-1>;

struct detail::mp_replace_impl<L,V,W> { using _f<A> = mp_if<is_same<A,V>,W,A>; using type = L<_f<T>...>; };
using mp_replace<L,V,W> = mp_replace_impl<L,V,W>::type;
struct detail::mp_replace_if_impl<L,P<...>,W> { using _f<U> = mp_if<P<U>,W,U>; using type = L<_f<T>...>; };
using mp_replace_if<L,P<...>,W> = mp_replace_if_impl<L,P,W>::type;
using mp_replace_if_q<L,Q,W> = mp_replace_if<L,Q::fn,W>;

struct detail::mp_copy_if_impl<L,P<...>> {}; // for non-list
struct detail::mp_copy_if_impl<L<T...>,P> { using _f<U> = mp_if<P<U>,mp_list<U>,mp_list<>>; using type = mp_append<L<>,_f<T>...>; };
using mp_copy_if<L,P<...>> = mp_copy_if_impl<L,P>::type;
using mp_copy_if_q<L,Q> = mp_copy_if<L,Q::fn>;

struct detail::mp_remove_impl<L,V>;
struct detail::mp_remove_impl<L<T...>,V> { using _f<U> = mp_if<is_same<U,V>,mmp_list<>,mp_list<U>>; using type = mp_append<L<>, _f<T>...>; };
using mp_remove<L,V> = mp_remove_impl<L,V>::type;
struct detail::mp_remove_if_impl<L,P<...>> {};
struct detail::mp_remove_if_impl<L<T...>,P<...>> { using _f<U> = mp_if<P<U>,mmp_list<>,mp_list<U>>; using type = mp_append<L<>, _f<T>...>; };
using mp_remove_if<L,P<...>> = mp_remove_if_impl<L,P>::type;
using mp_remove_if_q<L,Q> = mp_remove_if<L,Q::fn>;

struct detail::mp_flatten_impl<L2> { using fn<T> = mp_if<mp_similar<L2,T>,T,mp_list<T>>; }
using mp_flatten<L,L2=mp_clear<L>> = mp_apply<mp_append, mp_push_front<mp_transform_q<mp_flatten_impl<L2>,L>, mp_clear<L>>>;

struct detail::mp_partition_impl<L,P<...>>;
struct detail::mp_partition_impl<L<T...>,P> { using type = L<mp_copy_if<L<T...>, P>, mp_remove_if<L<T...>, P>>; }
using mp_partition<L,P<...>> = mp_partition_impl<L,P>::type;
using mp_partition_q<L,Q> = mp_partition<L,Q::fn>;

struct detail::mp_sort_impl<L,P<...>>; // non-list
struct detail::mp_sort_impl<L<>,P> { using type = L<>; }; // empty list
struct detail::mp_sort_impl<L<T1>,P> { using type = L<T1>; }; // size 1 list
struct detail::mp_sort_impl<L<T1,T...>,P> { // quick sort
    using F<U> = P<U,T1>; using part = mp_partition<L<T...>,F>;
    using type = mp_append<mp_push_back< mp_sort_impl<mp_first<part>,P>::type, T1>, mp_sort_impl<mp_second<part>,P>::type>;
};
using mp_sort<L,P<...>> = mp_sort_impl<L,P>::type;
using mp_sort_q<L,Q> = mp_sort<L,Q::fn>;

struct detail::mp_nth_element_impl<L,i,P<...>>;
using mp_nth_element_c<L,i,P<...>> = mp_nth_element_impl<L,i,P>::type;
using mp_nth_element<L,I,P<...>> = mp_nth_element<L,(size_t)I::value,P>;
using mp_nth_element_q<L,I,Q> = mp_nth_element<L,I,Q::fn>;

struct detail::mp_find_impl<L,V>;
struct detail::mp_find_impl<L<>,V> { using type = mp_size_t<0>; };
constexpr size_t detail::cx_find_index(bool const* first, bool const* last)
{ size_t m=0 for (; first != last && !*first; ++m, ++first); return m; }
struct detail::mp_find_impl<L<T...>,V> {
    static constexpr bool _v[] = { is_same<T,V>::value... }; // true for T==V
    using type = mp_size_t< cx_find_index(_v, _v+sizeof...(T))>;
};
using mp_find<L,V> = mp_find_impl<L,V>::type;
struct detail::mp_find_if_impl<L,P<...>>;
struct detail::mp_find_impl<L<>,P> { using type = mp_size_t<0>; };
struct detail::mp_find_impl<L<T...>,P> {
    static constexpr bool _v[] = { P<T>::value... }; // true for P<T> passed
    using type = mp_size_t< cx_find_index(_v, _v+sizeof...(T))>;
};
using mp_find_if<L,P<...>> = mp_find_if_impl<L,P>::type;
using mp_find_if_q<L,Q> = mp_find_if<L,Q::fn>;

struct detail::mp_reverse_impl<L>;
struct detail::mp_reverse_impl<L<>> { using type = L<>; }
struct detail::mp_reverse_impl<L<T1,...,Tn>> { using type = L<Tn,...,T1>; } // for 1 ~ 9
struct detail::mp_reverse_impl<L<T1,...,T10,T...>> { using type = mp_push_back<mp_reverse_impl<L<T...>>::type, T10,...,T1>; }
using mp_reverse<L> = mp_remove_impl<L>::type;

struct detail::mp_fold_impl<L,V,F<...>> {}; // for non-list
struct detail::mp_fold_impl<L<>,V,F> { using type = V; };
struct detail::mp_fold_Qn<V,F<...>> { using fn<T1,...Tn> = F<... F<F<V,T1>,T2> ...Tn>; }; // n = 1~9
struct detail::mp_fold_impl<L<T1,...,Tn>,V,F> : mp_defer<mp_fold_Qn<V,F>::fn, T1,...,Tn> {};
struct detail::mp_fold_impl<L<T1,...,T10,T...>,V,F> { using type = mp_fold_impl<L<T...>, F<...F<V,T1>,...T10>,F>::type; };
using mp_fold<L,V,F<...>> = mp_fold_impl<L,V,F>::type;
using mp_fold_q<L,V,Q> = mp_fold<L,V,Q::fn>;

struct detail::mp_reverse_fold_impl<L,V.F<...>>;
struct detail::mp_reverse_fold_impl<L<>,V,F> { using type = V; };
struct detail::mp_reverse_fold_impl<L<T1,T...>,V,F> { using type = F<T1, mp_reverse_fold_impl<L<T...>,V,F>::type>; };
struct detail::mp_reverse_fold_impl<L<T1,...,T10,T...>,V,F> { using type = F<T1,...,F<10, mp_reverse_fold_impl<L<T...>,V,F>::type>...>; };
using mp_reverse_fold<L,V,F<...>> = mp_reverse_fold_impl<L,V,F>::type;
using mp_reverse_fold_q<L,V,Q> = mp_reverse_fold<L,V,Q::fn>;

struct detail::mp_unique_impl<L>;
struct detail::mp_unique_impl<L<T...>> { using type = mp_set_push_back<L<>,T...>; };
using mp_unique<L> = mp_unique_impl<L>::type;
struct detail::mp_unique_if_push_back<P<...>>;
using mp_unique_if<L,P<...>> = mp_fold_q<L,mp_clear<L>, mp_unique_if_push_back<P>>;
using mp_unique_if_q<L,Q> = mp_unique_if<L,Q::fn>;

using mp_all_of<L,P<...>> = mp_bool<mp_count_if<L,P>::value == mp_size<L>::value>;
using mp_all_of_q<L,Q> = mp_all_of<L,Q::fn>;
using mp_none_of<L,P<...>> = mp_bool<mp_count_if<L,P>::value == 0>;
using mp_none_of_q<L,Q> == mp_none_of<L,Q::fn>;
using mp_any_of<L,P<...>> = mp_bool<mp_count_if<L,P>::value != 0>;
using mp_any_of_q<L,Q> == mp_any_of<L,Q::fn>;

struct detail::mp_replace_at_impl<L,I,W> {
    using _p<T1,T2> = is_same<T2,mp_size_t<I::value>>; using _f<T1,T2> = W;
    using type = mp_transform_if< _p, _f, L, mp_iota<mp_size<L>> >;
};
using mp_replace_at<L,I,W> = mp_replace_at_impl<L,I,W>::type;
using mp_replace_at_c<L,i,W> = mp_replace_at<L,mp_size_t<i>,W>;

constexpr F detail::mp_for_each_impl <...T,F> (mp_list<T...>, F&& f)
{ using A = int[sizeof...(T)]; return (void)A{ ((void)f(T{}), 0)... }, (F&&)f; }
constexpr F detail::mp_for_each_impl <F> (mp_list<>, F&& f) { return (F&&)f; }
constexpr F mp_for_each <L,F> (F&& f) { return mp_for_each_impl(mp_rename<L,mp_list>{}, (F&&)f); }

struct detail::mp_with_index_impl_<n> { // optimize, n/2
    static constexpr auto call <k,F> (size_t i, F&& f) -> decltype(...)
    { if (i<n/2) return mp_with_index_impl_<n/2>::call<k>(i, (F&&)f);
        else return mp_with_index_impl_<n-n/2>::call<k+n/2>(i-n/2, (F&&)f); };
};
struct detail::mp_with_index_impl_<0> {}; // optimize for 0-16
struct detail::mp_with_index_impl_<n> // for n of 1 ~ 16, i < n
{ static constexpr auto call <k,F> (size_t i, F&& f) -> decltype(...) { return ((F&&)f)(mp_size_t<k+i>()); } };
constexpr auto mp_with_index <n,F> (size_t i, F&& f) {return mp_with_index_impl_<n>::call<0>(i, (F&&)f); }
constexpr auto mp_with_index <N,F> (size_t i, F&& f) {return mp_with_index<(size_t)N::value>(i, (F&&)f); }

using mp_insert<L,I,...T> = mp_append<mp_take<L,I>, mp_push_front<mp_drop<L,I>, T...>>;
using mp_insert_c<L,i,...T> = mp_insert<L,mp_size_t<i>,T...>;
using mp_erase<L,I,J> = mp_append<mp_take<L,I>, mp_drop<L,J>>;
using mp_erase_c<L,i,j> = mp_erase<L,mp_size_t<i>,mp_size_t<j>>;

struct detail::mp_starts_with_impl<L1,L2> {};
using mp_starts_with<L1,L2> = mp_starts_with_impl<L1,L2>::type;

using detail::canonical_left_rotation<ln,n> = mp_size_t<n%(ln==0?1:ln)>;
using detail::canonical_right_rotation<ln,n> = mp_size_t<ln - n%(ln==0:1:ln)>;
using detail::mp_rotate_impl<L,N,L2=mp_rename<L,mp_list>> = mp_assign<L, mp_append<mp_drop<L2,N>, mp_take<L2,N>>>;
using mp_rotate_left_c<L,n> = mp_rotate_impl<L,canonical_left_rotation<mp_size<L>::value, n>>;
using mp_rotate_left<L,N> = mp_rotate_left_c<L, (size_t)N::value>;
using mp_rotate_right_c<L,n> = mp_rotate_left<L,canonical_right_rotation<mp_size<L>::value, n>>;
using mp_rotate_right<L,N> = mp_rotate_right_c<L, (size_t)N::value>;

struct detail::select_min<P<...>> { using fn<T1,T2> = mp_if<P<T1,T2>, T1,T2>; };
using mp_min_element<L,P<...>> = mp_fold_q<mp_rest<L>, mp_first<L>, select_min<P>>;
using mp_min_element_q<L,Q> = mp_min_element<L,Q::fn>;
struct detail::select_max<P<...>> { using fn<T1,T2> = mp_if<P<T2,T1>, T1,T2>; };
using mp_max_element<L,P<...>> = mp_fold_q<mp_rest<L>, mp_first<L>, select_max<P>>;
using mp_max_element_q<L,Q> = mp_max_element<L,Q::fn>;

struct detail::mp_power_set_impl<L>;
struct detail::mp_power_set_impl<L<>> { using type = L<L<>>; };
struct detail::mp_power_set_impl<L<T1,T...>> {
    using S1 = mp_power_set<L<T...>>;
    using _f<L2> = mp_push_front<L2,T1>; using S2 = mp_transform<_f,S1>;
    using type = mp_append<S1,S2>;
};
using mp_power_set<L> = mp_power_set_impl<L>::type;

struct detail::mp_partial_sum_impl_f<F<...>> { using fn<V,T,N=F<mp_first<V>,T>> = mp_list<N,mp_push_back<mp_second<V>,N>>; };
using mp_partial_sum<L,V,F<...>> = mp_second<mp_fold_q<L,mp_list<V,mp_clear<L>>, mp_partial_sum_impl_f<F>>>;
using mp_partial_sum_q<L,V,Q> = mp_partial_sum<L,V,Q::fn>;

struct detail::mp_iterate_impl<V,F<...>,R<...>,N=mp_true> { using type = mp_push_front<mp_iterate<R<V>,F,R>,F<V>>; };
struct detail::mp_iterate_impl<V,F,R,mp_false> { using _f<X> = mp_list<F<x>>; using type = mp_eval_or<mp_list<>,_f,V>; };
using mp_iterate<V,F<...>,R<...>> = mp_iterate_impl<V,F,R,mp_valid<R,V>>::type;
using mp_iterate_q<V,Qf,Qr> = mp_iterate<V,Qf::fn,Qr::fn>;

using detail::mp_pairwise_fold_impl<L,Q> = mp_transform_q<Q,mp_pop_back<L>,mp_pop_front<L>>;
using mp_pairwise_fold_q<L,Q> = mp_eval_if<mp_empty<L>, mp_clear<L>, mp_pairwise_fold_impl,L,Q>;
using mp_pairwise_fold<L,F<...>> = mp_pairwise_fold_q<L,mp_quote<F>>;

struct detail::mp_sliding_fold_impl<C=mp_false,L,Q,S> { using type = mp_clear<L>; };
struct detail::mp_sliding_fold_impl<mp_true,L,Q,S> {
    static const size_t m = mp_size<L>::value - N::value + 1;
    using F<I> = mp_slice_c<L,I::value,I::value+m>; using J = mp_transform<F, mp_iota<N>>;
    using type = mp_apply<mp_transform_q, mp_push_front<J,Q>>;
};
using mp_sliding_fold_q<L,N,Q> = mp_sliding_fold_impl<mp_bool<(mp_size<L>::value >= N::value)>, L,N,Q>::type;
using mp_sliding_fold<L,N,F<...>> = mp_sliding_fold_q<L,N,mp_quote<F>>;

struct detail::mp_intersperse_impl<L,S> {};
struct detail::mp_intersperse_impl<L<>,S> { using type = L<>; };
struct detail::mp_intersperse_impl<L<T1,T...>,S> { using type = mp_append<L<T1>, L<S,T>...>; };
using mp_intersperse<L,S> = mp_intersperse_impl<L,S>::type;

using detail::mp_split_impl_<L,S,J> = mp_push_front<mp_split<mp_drop_c<L,J::value+1>,S>, mp_take<L,J>>;
struct detail::mp_split_impl<L,S,J>
{ using type = mp_eval_if_c<mp_size<L>::value == J::value, mp_push_back<mp_clear<L>,L>, mp_split_impl_,L,S,J>; }
using mp_split<L,S> = mp_split_impl<L,S,mp_find<L,S>>::type;

using mp_join<L,S> = mp_apply<mp_append, mp_intersperse<L,mp_list<S>>>;
```

------
### Set Operations

* Header `<boost/mp11/set.hpp>`

```c++
struct detail::mp_set_contains_impl<S,V> {};
struct detail::mp_set_contains_impl<L<T...>,V> { using type = mp_to_bool<is_base_of<mp_identity<V>,mp_inherit<mp_identity<T>...>>>; };
using mp_set_contains<S,V> = mp_set_contains_impl<S,V>::type;

struct detail::mp_set_push_back_impl<S,...T> {};
struct detail::mp_set_push_back_impl<L<U...>> { using type = L<U...>; };
struct detail::mp_set_push_back_impl<L<U...>,T1,T...>
{ using type = mp_set_push_back_impl<mp_if<mp_set_contains<L<U...>,T1>,L<U...>,L<U...,T1>>, T...>::type; };
using mp_set_push_back<S,...T> = mp_set_push_back_impl<S,T...>::type;

struct detail::mp_set_push_front_impl<S,...T> {};
struct detail::mp_set_push_front_impl<L<U...>> { using type = L<U...>; };
struct detail::mp_set_push_front_impl<L<U...>,T1> { using type = mp_if<mp_set_contains<L<U...>,T1>,L<U...>,L<T1,U...>>; };
struct detail::mp_set_push_front_impl<L<U...>,T1,T...>
{ using type = mp_set_push_front_impl<mp_set_push_front_impl<L<U...>,T...>::type, T1>::type; };
using mp_set_push_front<S,...T> = mp_set_push_front_impl<S,T...>::type;

struct detail::mp_is_set_helper_start { static constexpr bool value = true; static mp_false contains<T>(T); };
struct detail::mp_is_set_helper<Base,T> : Base { // recursion
    static constexpr bool value = Base::value && !Base::contains(mp_identity<T>{}).value;
    using Base::contains; static mp_true contains(mp_identity<T>);
};
struct detail::mp_is_set_impl<S> { using type = mp_false; };
struct detail::mp_is_set_impl<L<T...>>
{ using type = mp_bool<mp_fold<mp_list<T...>, mp_is_set_helper_start, mp_is_set_helper>::value>; };
using mp_is_set<S> = mp_is_set_impl<S>::type;

struct detail::mp_set_union_impl<...L> {};
struct detail::mp_set_union_impl<> { using type = mp_list<>; };
struct detail::mp_set_union_impl<L<T...>> { using type = L<T...>; };
struct detail::mp_set_union_impl<L1<T1...>,L2<T2...>> { using type = mp_set_push_back<L1<T1...>,T2...>; };
using detail::mp_set_union_<L1,...L> = mp_set_union_impl<L1,mp_append<mp_list<>,L...>>::type;
struct detail::mp_set_union_impl<L1,L2,L3,L...> : mp_defer<mp_set_union_, L1,L2,L3,L...> {};
using mp_set_union<...L> = mp_set_union_impl<L...>::type;

struct detail::in_all_sets<...S> { using fn<T> = mp_all<mp_set_contains<S,T>...>; };
using detail::mp_set_intersection_<L,...S> = mp_if<mp_all<mp_is_list<S>...>, mp_copy_if_q<L,in_all_sets<S...>>>;
struct detail::mp_set_intersection_impl<...S> {};
struct detail::mp_set_intersection_impl<> { using type = mp_list<>; };
struct detail::mp_set_intersection_impl<L,S...> : mp_defer<mp_set_intersection_,L,S...> {};
using mp_set_intersection<...S> = mp_set_intersection_impl<S...>::type;

struct detail::in_any_set<...S> { using fn<T> = mp_any<mp_set_contains<S,T>...>; };
using mp_set_difference<L,...S> = mp_if<mp_all<mp_is_list<S>...>, mp_remove_if_q<L,in_any_set<S...>>>;
```

------
### Map Operations

* Header `<boost/mp11/map.hpp>`

```c++
struct detail::mp_map_find_impl<M,K>;
struct detail::mp_map_find_impl<M<T...>,K>;
using mp_map_find<M,K> = mp_map_find_impl<M,K>::type;

using mp_map_contains<M,K> = mp_not<is_same<mp_map_find<M,K>,void>>;

using mp_map_insert<M,T> = mp_if<mp_map_contains<M,mp_first<T>>, M,mp_push_back<M,T>>;

struct detail::mp_map_replace_impl<M,T>;
struct detail::mp_map_replace_impl<M<U...>,T> { using K = mp_first<T>;
    struct f_<V>{ using type = mp_if<is_same<mp_first<V>,K>, T,V>; };
    using type = mp_if< mp_map_contains<M<U...>,K>, M<_f<U>::type..., M<U...,T> >;
};
using mp_map_replace<M,T> = mp_map_replace_impl<M,T>::type;

struct detail::mp_map_update_impl<M,T,F<...>> {
    using _f<U> = is_same<mp_first<T>,mp_first<U>>; // compare by key
    using _f3<L> = mp_assign<L, mp_list<mp_first<L>,mp_rename<L,F>>>; // L<X,Y...> => L<X,F<X,Y...>>
    using type = mp_if< mp_map_contains<M,mp_first<T>>, mp_transform_if<_f,_f3,M>, mp_push_back<M,T> >;
};
using mp_map_update<M,T,F<...>> = mp_map_update_impl<M,T,F>::type;
using mp_map_update_q<M,T,Q> = mp_map_update<M,T,Q::fn>;

struct detail::mp_map_erase_impl<M,K> { using _f<T> = is_same<mp_first<T>,K>; using type = mp_remove_if<M,_f>; };
using mp_map_erase<M,K> = mp_map_erase_impl<M,K>::type;

using mp_map_keys<M> = mp_transform<mp_first,M>;
using mp_keys_are_set<M> = mp_is_set<mp_map_keys<M>>;

struct detail::mp_is_map_element<L>: mp_false {};
struct detail::mp_is_map_element<L<T1,T...>>: mp_true {}; // list size >= 1
struct detail::mp_is_map_impl<M> { using type = mp_false; };
struct detail::mp_is_map_impl<M<T...>> { using type = mp_eval_if<mp_not<mp_all<mp_is_map_element<T>...>>, mp_false, mp_keys_are_set, M<T...>>; };
using mp_is_map<M> = mp_is_map_impl<M>::type;
```

------
### Meta Functions

* Header `<boost/mp11/function.hpp>`

```c++
struct detail::mp_void_impl<...T> { using type = void; };
using mp_void<...T> = mp_void_impl<T...>::type;

struct detail::mp_and_impl<L,E=void> { using type = mp_false; };
struct detail::mp_and_impl<mp_list<T...>,mp_void<mp_if<T,vid>...>> { using type = mp_true; };
using mp_and = mp_and_impl<mp_list<T...>>::type;

using mp_all<...T> = mp_bool<mp_count<mp_list<mp_to_bool<T>...>, mp_false>::value == 0>;

struct detail::mp_or_impl<...T>;
struct detail::mp_or_impl<> { using type = mp_false; };
struct detail::mp_or_impl<T> { using type = T; };
struct detail::mp_or_impl<T1,T...> { using type = mp_eval_if<T1, T1, mp_or,T...>; };
using mp_or<...T> = mp_to_bool<mp_or_impl<T...>::type>;

using mp_any<...T> = mp_bool<mp_count<mp_list<mp_to_bool<T>...>, mp_true>::value != 0>;

struct detail::mp_same_impl<...T>;
struct detail::mp_same_impl<> { using type = mp_true; };
struct detail::mp_same_impl<T1,T...> { using type = mp_bool<mp_count<mp_list<T...>,T1>::value == sizeof...(T)>; };
using mp_same<...T> = mp_same_impl<T...>::type;

struct detail::mp_similar_impl<...T>;
struct detail::mp_similar_impl<> { using type = mp_true; };
struct detail::mp_similar_impl<T> { using type = mp_true; };
struct detail::mp_similar_impl<T,T> { using type = mp_true; };
struct detail::mp_similar_impl<T1,T2> { using type = mp_false; };
struct detail::mp_similar_impl<L<T1...>,L<T2...>> { using type = mp_true; };
struct detail::mp_similar_impl<L<T...>,L<T...>> { using type = mp_true; };
struct detail::mp_similar_impl<T1,T2,T3,T...> { using type = mp_all<mp_similar_impl<T1,T2>::type, mp_similar_impl<T1,T3>::type, mp_similar_impl<T1,T>::type...>; };
using mp_similar<...T> = mp_similar_impl<T...>::type;


using mp_less<T1,T2> = mp_bool<(T1::value < 0 && T2::value >= 0) || ((T1::value<T2::value) && !(T1::value>=0 && T2::value<0))>;

using mp_min<T1,...T> = mp_min_element<mp_list<T1,T...>,mp_less>;
using mp_max<T1,...T> = mp_max_element<mp_list<T1,T...>,mp_less>;
```

------
### Meta Function Parameter Binding

* Header `<boost/mp11/bind.hpp>`

```c++
struct mp_bind_front<F<...>,...T> { using fn<...U> = mp_defer<F,T...,U...>::type; };
using mp_bind_front_q<Q,...T> = mp_bind_front<Q::fn,T...>;
struct mp_bind_back<F<...>,...T> { using fn<...U> = mp_defer<F,U...,T...>::type; };
using mp_bind_back_q<Q,...T> = mp_bind_back<Q::fn,T...>;

struct mp_arg<i> { using fn<...T> = mp_at_c<mp_list<T...>, i>; };
using _1 = mp_arg<0>; // for _1 ~ _9

struct detail::eval_bound_arg<V,...T> { using type = V; };
struct detail::eval_bound_arg<mp_arg<i>,T...> { using type = mp_arg<i>::fn<T...>; };
struct detail::eval_bound_arg<mp_bind<F,U...>,T...> { using type = mp_bind<F,U...>::fn<T...>; };
struct detail::eval_bound_arg<mp_bind_front<F,U...>,T...> { using type = mp_bind_front<F,U...>::fn<T...>; };
struct detail::eval_bound_arg<mp_bind_back<F,U...>,T...> { using type = mp_bind_back<F,U...>::fn<T...>; };
struct mp_bind<F<...>,...T> { using fn<...U> = F<eval_bound_arg<T,U...>::type...>; };
using mp_bind_q<Q,...T> = mp_bind<Q::fn,T...>;
```

------
### Meta Lambda Functions

* Header `<boost/mp11/lambda.hpp>`

```c++
struct detail::lambda_impl<T> { using make<...U> = T; using type = mp_bind<make>; };
struct detail::lambda_impl<mp_arg<i>> { using type = mp_arg<i>; };
struct detail::lambda_impl<[const][volatile]T> { using make<U> = <const><volatile>U; using type = mp_bind<make, mp_lambda<T>>; };
struct detail::lambda_impl<T{*|&|&&|[]|[n]}> { using make<U> =U{*|&|&&|[]|[n]}; using type = mp_bind<make,mp_lambda<T>>; };
struct detail::lambda_impl<R(T...<,...>)<const><volatile><&|&&><noexcept>> // normal and var-arg function types with qualifiers
{ using make_fct<R,...T> = R(T...<,...>)[const][volatile][&|&&][noexcept];
    using type = mp_bind<make_fct,mp_lambda<R>,mp_lambda<T>...>; };
struct detail::lambda_impl<R(C::*)(T...<,...>)<const><volatile><&|&&><noexcept>> // normal and var-arg mfptr types with qualifiers
{ using make_mfptr<R,C,...T> = R(C::*)(T...<,...>)[const][volatile][&|&&][noexcept];
    using type = mp_bind<make_mfptr,mp_lambda<R>,mp_lambda<C>,mp_lambda<T>...>; };
struct detail::lambda_impl<T C::*> { using make<U,D> = U D::*; using type = mp_bind<make,mp_lambda<T>,mp_lambda<C>>; }; // data member ptr
struct detail::lambda_impl<C<Ts...>> { using type = mp_bind<C,mp_lambda<Ts>...>; }; // temp-spec
using mp_lambda<T> = lambda_impl<T>::type;
```

------
### Integer Sequence

* Header `<boost/mp11/integer_sequence.hpp>`

```c++
struct integer_sequence<T,T...i> {};

using detail::iseq_if_c<cond,Then,Else> = mp_if_c<cond,Then,Else>::type;
struct detail::iseq_identity<T> = mp_identity<T>;
struct detail::append_integer_sequence<S1,S2>;
struct detail::append_integer_sequence<integer_sequence<T,i...>,integer_sequence<T,j...>> { using type = integer_sequence<T,i...,(j+sizeof...(i))...>; };
struct detail::make_integer_sequence_impl_<T,T n> { using type = append_integer_sequence<S2,S3>; }; // n/2 twice, and n%2, optimize
struct detail::make_integer_sequence_impl<T,T n>
    : mp_if_c<n==0, mp_identity<integer_sequence<T>>,
            mp_if_c<n==1, mp_identity<integer_sequence<T,0>>, make_integer_sequence_impl_<T,n>>> {};
using make_integer_sequence<T,T n> = make_integer_sequence_impl<T,n>::type;

using index_sequence<...i> = integer_sequence<size_t,i...>;
using make_index_sequence<n> = make_integer_sequence<size_t,n>;
using index_sequence_for<...T> = make_index_sequence<sizeof...(T)>;
```

------
### Tuple

* Header `<boost/mp11/tuple.hpp>`

```c++
constexpr auto detail::tuple_apply_impl<F,Tp,...j> (F&&f, Tp&& tp, index_sequence<j...>) ->decltype(...) { return ((F&&)f)( get<j>((Tp&&)tp)... ); }
constexpr auto tuple_apply <F,Tp> (F&& f, Tp&& tp) -> decltype(...) // std::apply
{ return tuple_apply_impl((F&&)f, (Tp&&)(tp), make_index_sequence<tuple_size_v<remove_reference_t<Tp>>>{} ); }

constexpr T detail::construct_from_tuple_impl<T,Tp,...j> (Tp&& tp, index_sequence<j...>) { return T( get<j>((Tp&&)tp)... ); }
constexpr T construct_from_tuple <T,Tp> (Tp&& tp) // std::make_from_tuple
{ return construct_from_tuple_impl<T>( (Tp&&)tp, make_index_sequence<tuple_size_v<remove_reference_t<Tp>>>{} ); }

constexpr F detail::tuple_for_each_impl<Tp,...j,F> (Tp&& tp, index_sequence<j...>, F&& f)
{ using A=int[sizeof...(j)]; return (void)A{ ((void)f(get<j>((Tp&&)tp)), 0)... }, (F&&)f; } // return f
constexpr F detail::tuple_for_each_impl<Tp,F> (Tp&& tp, index_sequence<>, F&& f) { return (F&&)f; }
constexpr F tuple_for_each <Tp,F> (Tp&& tp, F&& f)
{ return tuple_for_each_impl((Tp&&)tp, make_index_sequence<tuple_size_v<remove_reference_t<Tp>>>{}, (F&&)f); }

constexpr auto detail::tp_forward_r<...T> (T&&...t) -> std::tuple<T&&...> { return { (T&&)t... }; } // std::forward_as_tuple
constexpr auto detail::tp_forward_v<...T> (T&&...t) -> std::tuple<T...> { return { (T&&)t... }; } // move to tuple
constexpr auto detail::tp_extract <j,...Tp> (Tp&&...tp)-> decltype(...) { return tp_forward_r( get<j>((Tp&&)tp)... ); }
constexpr auto detail::tuple_transform_impl<F,...Tp,...j> (index_sequence<j...>, F const& f, Tp&&...tp) -> decltype(...)
{ return tp_forward_v( tuple_apply(f,tp_extract<j>((Tp&&)tp...))... ); }
constexpr auto tuple_transform<F,...Tp> tuple_transform (F const& f, Tp&&...tp) -> decltype(...)
{   using Z = mp_list<mp_size_t<tuple_size_v<remove_reference_t<Tp>>>...>;
    using Seq = make_index_sequence<mp_if<mp_apply<mp_same,Z>, mp_front<Z>>>;
    return tuple_transform_impl(Seq{}, f, (Tp&&)tp...);
}
```

------
### Interoperations with MPL

* Header `<boost/mp11/mpl.hpp>`

```c++
namespace boost::mpl;
struct aux::mp11_tag {};
struct aux::mp11_iterator<L> // iterator support
{ using category = forward_iterator_tag; using type = mp_first<L>; using next = mp11_iterator<mp_rest<L>>; };
struct at_impl<mp11_tag> { struct apply<L,I> { using type = mp_at<L,I>; }; }; // at
struct back_impl<mp11_tag> { struct apply<L> { using type = mp_at_c<L, mp_size<L>::value-1>; }; }; // back
struct begin_impl<mp11_tag> { struct apply<L> { using type = mp11_iterator<L>; }; }; // begin
struct clear_impl<mp11_tag> { struct apply<L> { using type = mp_clear<L>; }; }; // clear
struct end_impl<mp11_tag> { struct apply<L> { using type = mp_iterator<mp_clear<L>>; }; }; // end
struct front_impl<mp11_tag> { struct apply<L> { using type = mp_front<L>; }; };
struct pop_front_impl<mp11_tag> { struct apply<L> { using type = mp_pop_front<L>; }; };
struct push_back_impl<mp11_tag> { struct apply<L,T> { using type = mp_push_back<L,T>; }; };
struct push_front_impl<mp11_tag> { struct apply<L,T> { using type = mp_push_front<L,T>; }; };
struct size_impl<mp11_tag> { struct apply<L> { using type = mp_size<L>; }; };

struct sequence_tag<mp_list<T...>> { using type = mp11_tag; };
struct sequence_tag<std::tuple<T...>> { using type = mp11_tag; };
```

------
#### Dependency

------
### Standard Facilities

Library:
* `bool_constant`, `void_t` (C++17)
* `integer_sequence`, `index_sequence` (C++14)
* `apply`, `make_from_tuple` (C++17)
