# Boost.CallableTraits

* lib: `boost/libs/callable_traits`
* repo: `boostorg/callable_traits`
* commit: `046d4dd`, 2025-06-26

------
### Traits for Function Types

* Header `<boost/static_string.hpp>`

```c++
struct detail::sfinae_error{};
struct detail::success<T> { static constexpr bool value=true; struct _{using type=T; }; };
struct detail::fail_if<b,T> : T { static constexpr bool value=B; };
using detail::sfinae_try<T,...FailIfs> = std::disjunction<FailIfs...,success<T>>::_::type;
struct detail::fail<FailMsg,ForceTwoPhaseLookup> { using type = std::conditional<std::is_same<ForceTwoPhaseLookup, std::false_type>::value, FailMsg, FailMsg>::type::_::type; };

struct error::parameters : sfinae_error { struct _{}; };
struct index_out_of_range_for_parameter_list : error::pararmeters<struct index_out_of_range_for_parameter_list_>{};
struct cannot_determine_parameters_for_this_type : error::pararmeters<struct cannot_determine_parameters_for_this_type_>{};
struct error::varargs : sfinae_error { struct _{}; };
struct varargs_are_illegal_for_this_type : error::varargs<struct varargs_are_illegal_for_this_type_>{};
struct error::member_qualifiers : sfinae_error { struct _{}; };
struct member_qualifiers_are_illegal_for_this_type : error::member_qualifiers<struct member_qualifiers_are_illegal_for_this_type_>{};
struct this_compiler_doesnt_support_abominable_function_types : error::member_qualifiers<struct this_compiler_doesnt_support_abominable_function_types_>{};
struct error::transaction_safe_ : sfinae_error { struct _{}; };
struct transaction_safe_is_not_supported_by_this_configuration : error::transaction_safe_<struct transaction_safe_is_not_supported_by_this_configuration_>{};
struct error::expand_args : sfinae_error { struct _{}; };
struct cannot_expand_the_parameter_list_of_first_template_argument : error::expand_args<struct cannot_expand_the_parameter_list_of_first_template_argument_>{};
struct error::member_pointer_required : sfinae_error { struct _{}; };
struct type_is_not_a_member_pointer : error::member_pointer_required<struct type_is_not_a_member_pointer_>{};
struct error::reference_error : sfinae_error { struct _{}; };
struct reference_type_not_supported_by_this_metafunction : error::reference_error<struct reference_type_not_supported_by_this_metafunction_>{};

enum detail::qualifier_flags : uint32_t { default_=0, const_=1, volatile_=2, cv_=3, lref_=4, rref_=8 }

using detail::remove_const_flag<flags> = std::integral_constant<qualifier_flags, flags&~const_>;
using detail::is_const<flags> = std::bool_constant<flags & const_ != 0>;
using detail::remove_volatile_flag<flags> = std::integral_constant<qualifier_flags, flags&~volatile_>;
using detail::cv_of<U,T=std::remove_reference_t<U>> = std::integral_constant<qualifier_flags, (is_const_v<T>?const_:default_) | (std::is_volatile_v<T>?volatile_:default_)>;
using detail::ref_of<T> = std::integral_constant<qualifier_flags, std::is_rvalue_reference_v<T> ? rref_ : std::is_lvalue_reference_v<T> ? lref_ : default_>;
using detail::collapse_flags<existing,other, hasRef=existing&(lref_|rref_)!=0, hasLRef=existing&lref_==lref_, addLRef=other&lref_==lref_>
    = std::integral_constant<qualifier_flags, !hasRef ? (existing|other) : hasLRef ? (existing|(other&~rref_)) : addLRef ? ((existing&~rref_)|other) : (existing|other)>;

struct detail::flag_map<T> { value = default_; };
struct detail::flag_map<T&> { value = lref_; };
struct detail::flag_map<T&&> { value = rref_; };
struct detail::flag_map<T const> { value = const_; };
struct detail::flag_map<T const&> { value = const_|lref_; };
struct detail::flag_map<T const&&> { value = const_|rref_; };
struct detail::flag_map<T volatile> { value = volatile_; };
struct detail::flag_map<T volatile&> { value = volatile_|lref_; };
struct detail::flag_map<T volatile&&> { value = volatile_|rref_; };
struct detail::flag_map<T const volatile> { value = const_|volatile_; };
struct detail::flag_map<T const volatile&> { value = const_|volatile_|lref_; };
struct detail::flag_map<T const volatile&&> { value = const_|volatile_|rref_; };

struct detail::cdecl_tag{};
struct detail::stdcall_tag{};
struct detail::fastcall_tag{};
struct detail::pascal_tag{};

struct detail::invalid_type { ctor()=delete; };
struct detail::reference_error { ctor()=delete; };
using detail::error_type<T> = std::conditional_t<std::is_reference_v<T>, reference_error, invalid_type>;
struct detail::dummy {};
struct detail::substitution_failure : std::false_type {};
using detail::bool_type<v> = std::bool_constant<v>;
using detail::at<i,Tup> = std::tuple_element_t<i,Tup>;
using detail::add_member_pointer<T,Cls> = T Cls::*;
using detail::fail_when_same<L,R,ErrorType> = fail_if<std::is_same_v<L,R>,ErrorType>;
using detail::try_but_fail_if_invalid<T,ErrorType,U=std::remove_reference_t<T>>
    = sfinae_try<T, fail_when_same<U,invalid_type,ErrorType>, fail_when_same<U,reference_error, reference_type_not_supported_by_this_metafunction>>;
using detail::fail_if_invalid<T,ErrorType,U=std::remove_reference_t<T>, is_reference_error=std::is_same_v<reference_error,U>>
    = fail_if<std::is_same_v<U,invalid_type> || is_reference_error, std::conditional_t<is_reference_error, reference_type_not_supported_by_this_metafunction, ErrorType>>;
using detail::fallback_if_invalid<T,Fallback> = std::conditional_t<std::is_same_v<T,invlaid_type>, Fallback, T>;
using detail::force_sfinae<T,<T>Alas,U=Alias<T>> { using type = U; };
using detail::shallow_decay<T> = std::remove_cv_t<std::remove_reference_t<T>>;
struct detail::is_reference_wrapper_t<T> { using type = std::false_type; };
struct detail::is_reference_wrapper_t<std::reference_wrapper<T>> { using type = std::true_type; };
using  detail::is_reference_wrapper<T> = is_reference_wrapper_t<shallow_decay<T>>::type;
struct detail::unwrap_reference_t<T,_=std::true_type> { using type = T; };
struct detail::unwrap_reference_t<T,is_reference_wrapper<T>> { using type = decltype(std::declval<T>().get()); };
using detail::unwrap_reference<T> = unwrap_reference_t<T>::type;

struct detail::has_normal_call_operator<T> { static constexpr bool value = /*.detect op().*/; };
struct detail::callable_dummy { void operator()() {} };
using detail::default_to_function_object<T> = std::conditional_t<has_normal_call_operator<T>::value, T, callable_dummy>;
using detail::function_object_base<F,T=std::remove_reference_t<F>>
    = std::conditional_t<has_normal_call_operator<T>::value, pmf<decltype(&default_to_function_object<T>::operator())>, default_callable_traits<T>>;

struct detail::default_callable_traits<T=void> {
    static constexpr bool value = false;
    using traits = default_callable_traits; using error_t = error_type<T>;
    using type = error_t; using has_varargs = std::false_type; using return_type = error_t;
    using arg_types = error_t; using non_invoke_arg_types = error_t;
    using function_type = error_t; using function_object_signature = error_t; using qualified_function_type = error_t;
    using remove_varargs = error_t; using add_varargs = error_t;
    using is_noexcept = std::false_type; using add_noexcept = error_t; using remove_noexcept = error_t;
    using is_transaction_safe = std::false_type; using add_transaction_safe = error_t; using remove_transaction_safe = error_t;
    using class_type = error_t; using invoke_type = error_t;
    using remove_reference = error_t; using add_member_lvalue_reference = error_t; using add_member_rvalue_reference = error_t;
    using add_member_const = error_t; using add_member_volatile = error_t; using add_member_cv = error_t;
    using remove_member_const = error_t; using remove_member_volatile = error_t; using remove_member_cv = error_t;
    using remove_member_pointer = error_t;

    using apply_member_pointer<C, U=T, K=std::remove_reference_t<U>,
            L=std::conditional_t<is_same_v<void,K>,error_t,K>, Class=std::conditional_t<is_class_v<C>,C,error_t>>
        = std::conditional_t<std::is_same_v<L,error_t> || std::is_same_v<Class,error_t>, error_t, L Class::*>;
    using apply_return<_> = error_t;
    using expand_args<Container<...>> = error_t;
    using expand_args_{left|right}<Container<...>,...Args> = error_t;
    using clear_args = error_t;
    using push_{front|back}<...NewArgs> = error_t;
    using pop_{front|back}<elemCount> = error_t;
    using insert_args<index,...NewArgs> = error_t;
    using remove_args<index,count> = error_t;
    using replace_args<index,...NewArgs> = error_t;

    static constexpr qualifier_flags cv_flags = cv_of<T>::value, ref_flags = ref_of<T>::value, q_flags = cv_flags|ref_flags;
    using has_member_qualifiers = std::bool_constant<q_flags!=default_>;
    using is_const_member = std::bool_constant<0 < (cv_flags & const_)>;
    using is_volatile_member = std::bool_constant<0 < (cv_flags & volatile_)>;
    using is_cv_member = std::bool_constant<cv_flags == const_|volatile_>;
    using is_reference_member = std::bool_constant<0 < ref_flags>;
    using is_lvalue_reference_member = std::bool_constant<ref_flags==lref_>;
    using is_rvalue_reference_member = std::bool_constant<ref_flags==rref_>;
};

using traits<T> = std::disjunction<function_object<unwrap_reference<T>>, function<T>, pmf<T>, pmd<T>, default_callable_traits<T>>::traits;

struct detail::set_function_qualifiers_t<applied, isTxSafe, isNoEx, Ret, ...Args> { using type = Ret(Args...); };
struct detail::set_varargs_function_qualifiers_t<applied, isTxSafe, isNoEx, Ret, ...Args> { using type = Ret(Args...,...); };
struct detail::set_function_qualifiers_t<flag_map<int const>::value,_,{false|true},Ret,Args...> { using type = Ret(Args...) <const><volatile><&|&&><noexcept>; };
struct detail::set_varargs_function_qualifiers_t<flag_map<int const>::value,_,{false|true},Ret,Args...> { using type = Ret(Args...,...) <const><volatile><&|&&><noexcept>; };

using detail::set_function_qualifiers<flags, isTxSafe, isNoEx, ...Ts> = set_function_qualifiers_t<flags, isTxSafe, isNoEx, Ts...>::type;
using detail::set_varargs_function_qualifiers<flags, isTxSafe, isNoEx, ...Ts> = set_varargs_function_qualifiers_t<flags, isTxSafe, isNoEx, Ts...>::type;

struct detail::set_member_function_qualifiers_t<applied,isTxSafe,isNoEx,CallConv,T,Ret,...Args>;
struct detail::set_varargs_member_function_qualifiers_t<applied,isTxSafe,isNoEx,CallConv,T,Ret,...Args>;
using detail::set_member_function_qualifiers<flags,isTxSafe,isNoEx,...Ts> = set_member_function_qualifiers_t<flags,isTxSafe,isNoEx,Ts...>::type;
using detail::set_varargs_member_function_qualifiers<flags,isTxSafe,isNoEx,...Ts> = set_varargs_member_function_qualifiers_t<flags,isTxSafe,isNoEx,Ts...>::type;

struct detail::pmf<T> : default_callable_traits<T> {}; // signatures specializations
struct detail::function_object<T,Base> : Base {
    using type = T; using error_t = error_type<T>; using traits = function_object;
    using class_type = error_t; using invoke_type = error_t;
    using remove_varargs = error_t; using add_varargs = error_t;
    using is_noexcept = Base::is_noexcept; using add_noexcept = error_t; using remove_noexcept = error_t;
    using is_transaction_safe = Base::is_transaction_safe; using add_transaction_safe = error_t; using remove_transaction_safe = error_t;
    using clear_args = error_t;
    using expand_args<Container<...>> = function<function_type>::expand_args<Container>;
    using expand_args_{left|right}<Container<...>,...Args> = function<function_type>::expand_args_{left|right}<Container,Args...>;
    using apply_member_pointer<C,U=T> = std::remove_reference_t<U> C::*;
    using apply_return<_> = error_t;
    using push_{front|back}<...> = error_t; using pop_{front|back}<count> = error_t;
    using pop_args_{front|back}<eleCount> = error_t;
    using insert_args<index,...Args> = error_t; using remove_args<index,count> = error_t; using replace_args<index,...Args> = error_t;
    using remove_member_reference = error_t;
    using add_member_{l|r}value_reference = error_t; using add_member_{const|volatile|cv} = error_t;
    using remove_member_{const|volatile|cv} = error_t;
};
struct detail::function_object<T U::*, Base> : default_callable_traits<> {};

struct detail::function<T> : default_callable_traits<T> {}; // signature specializations
struct detail::function<T&> : std::conditional<function<T>::value, function<T>, default_callable_traits<T&>>::type {
    static constexpr const bool value = !std::is_pointer_v<T>;
    using traits = function; using base = function<T>; using type = T&;
    using remove_varargs = base::remove_varargs&; using add_varargs = base::add_varargs&;
    using remove_member_reference = reference_error; using add_member_{l|r}value_reference = reference_error;
    using add_member_{const|volatile|cv} = reference_error; using remove_member_{const|volatile|cv} = reference_error;

    using apply_return<Ret> = base::apply_return<Ret>&;
    using clear_args = base::clear_args&;
    using push_{front|back}<...Args> = base::push_{front|back}<Args...>&;
    using pop_{front|back}<count> = base::pop_{front|back}<count>&;
    using insert_args<i,...Args> = base::insert_args<index,Args...>&;
    using remove_args<index,count> = base::remove_args<index,count>&;
    using replace_args<index,...Args> = base::replace_args<index,Args...>&;
};

struct detail::pmd<T> : default_callable_traits<T> {};
struct detail::pmd<D T::*> : default_callable_traits<> {
    static constexpr const bool value = true;
    using traits = pmd; using class_type = T; using invoke_type = T const&; using type = T&;
    using function_type = std::add_lvalue_reference_t<D> (invoke_type);
    using qualified_function_type = D(invoke_type);
    using arg_types = std::tuple<invoke_type>; using non_invoke_arg_types = std::tuple<>;
    using return_type = std::add_lvalue_reference_t<D>;

    using apply_member_pointer<C> = D C::*;
    using apply_return<Ret> = base::apply_return<Ret>&;
    using expand_args<Container<...>> = Container<invoke_type>;
    using is_member_pointer = std::true_type;
};

struct detail::can_dereference_t<T> { value = /* check `*v` */; };
using detail::can_dereference = std::bool_constant<can_dereference_t<T>::value>;
struct detail::generalize_t<T,_=std::true_type> { using type = T; };
struct detail::generalize_t<T,std::bool_constant<can_dereference<T>::value && !is_reference_wrapper<T>::value>> { using type = decltype(*std::declval<T>()); };
struct detail::generalize_t<T,is_reference_wrapper<T>>> { using type = decltype(std::declval<T>().get()); };
using detail::generalize<T> = generalize_t<T>::type;
using detail::generalize_if_dissimilar<Base,T,isBaseOf=std::is_base_of_v<Base,shallow_decay<T>>, isSame=std::is_same_v<Base,shallow_decay<T>>>
    = std::conditional_t<isBaseOf || isSame, T, generalize<T>>

struct detail::test_invoke<Tr,abominable=Tr::is_const_member || Tr::is_volatile_member || Tr::is_lvalue_reference_member || Tr::is_rvalue_reference_member> {
    auto operator() <...Rgs,U=Tr::type> (int,Rgs&&...rgs) const -> success<decltype(declval<U>()((Rgs&&)rgs...))>;
    auto operator()(long,...) const -> substitution_failure;
};
struct detail::test_invoke<F,true> {
    auto operator()(...) const -> substitution_failure;
};
struct detail::test_invoke<Pmf,_> {
    using class_t = pmf<Pmf>::class_type;
    auto operator()<U,...Rgs,Obj=generalize_if_dissimilar<class_t,U&&>>
        (int, U&& u, Rgs&&...rgs) const -> success<decltype((declval<Obj>().*declval<Pmf>())((Rgs&&)rgs...))>;
    auto operator()(long,...) const -> substitution_failure;
};
struct detail::test_invoke<Pmd,_> {
    using class_t = pmd<Pmd>::class_type;
    auto operator()<U,...Rgs,Obj=generalize_if_dissimilar<class_t,U&&>>
        (int, U&& u) const -> success<decltype(declval<Obj>().*declval<Pmf>())>;
    auto operator()(long,...) const -> substitution_failure;
};
struct detail::is_invocable_impl<T,...Args> {
    using traits = traits<T>; using test = test_invoke<traits>;
    using result = decltype(test{}(0, declval<Args()...>));
    using type = std::bool_constant<result::value>;
};
struct detail::is_invocable_impl<void,Args...> { using type = std::false_type; };
struct detail::is_invocable_r_impl<isInvocable,Ret,T,...Args> {
    using traits = traits<T>; using test = test_invoke<traits>;
    using result = decltype(test{}(0, declval<Args()...>));
    using type = std::bool_constant<is_convertible_v<result::_::type, Ret> || is_same_v<Ret, void>>;
};
struct detail::is_invocable_r_impl<false_type,Ret,T,Args...> { using type = std::false_type; };

struct detail::parameter_index_helper<i,T,ignThis,allowPlus=false,count=0> {
    using error_t = error_type<T>;
    using args_tuple = std::conditional_t<ignThis, traits<T>::non_invoke_arg_types, traits<T>::arg_types>;
    static constexpr bool has_parameter_list = !std::is_same_v<args_tuple, invalid_type> && !std::is_same_v<args_tuple, reference_error>;
    using temp_tuple = std::conditional_t<has_parameter_list, args_tuple, std::tuple<error_t>>;
    static constexpr size_t parameter_list_size = std::tuple_size_v<temp_tuple>;
    static constexpr bool is_out_of_range = has_parameter_list && i >= parameter_list_size + (size_t)allowPlus;
    static constexpr bool is_count_out_of_range = has_parameter_list && i+count >= parameter_list_size + (size_t)allowPlus;
    static constexpr size_t index = has_parameter_list && !is_out_of_range ? i : 0;
    static constexpr size_t count = has_parameter_list && !is_count_out_of_range ? i : 0;
    using permissive_tuple = std::conditional_t<has_parameter_list && !is_out_of_range, args_tuple, std::tuple<error_t>>;
    using permissive_function = std::conditional_t<has_parameter_list && !is_out_of_range, T, error_t(error_t)>;
};
```

------
### Dependency

------
### Standard Facilities
