# Boost.Lambda

* lib: `boost/libs/lambda`
* repo: `boostorg/lambda`
* commit: `dddfec6`, 2025-06-28

Deprecated by **Boost.Lambda2**

------
#### Main API

* `<boost/lambda/lambda.hpp>`

```c++
struct detail::generate_error<T>;
class action<i,Act>;
class lambda_functor<Base>;
class lambda_functor_base<Act,Args>;

enum { NONE=0, FIRST=1, SECOND=2, TTHIRD=4, EXCEPTION=8, RETHROW=16 };
struct get_arity<T>;
struct get_tuple_arity<T>;
struct has_placeholder<T,i>; // get_arity<T>::value & i !=0
struct includes_placeholder<i,j>; // j&i != 0
struct lacks_placeholder<i,j>; // j&i == 0

// actions
class action<arity,Act>;
class assignment_action{};
class subscript_action{};
class other_action<Action>;
class explicit_return_type_action<Ret>{};
struct protect_action{};
struct comma_action{};
struct is_protectable<Action>; // false
struct is_protectable<other_action<comma_action>>; // true
class detail::unspecified{};

class function_action<i,Result_type=unspecified>{};
class function_action<1~10,T>{ static RET apply<RET,A1...A10>(A1& a1,...,A10& a10); };

struct is_lambda_functor<T>;
struct const_copy_argument<T>;
struct const_copy_argument<<volatile>T[n]>;
struct const_copy_argument<T&>{};
struct const_copy_argument<void<const>>;
struct bound_argument_conversion<T>;
struct bound_argument_conversion<T&>;
struct reference_argument<T>;
struct reference_argument<T&>;
struct reference_argument<<const><volatile>lambda_functor<Arg>>;
struct reference_argument<void>;

struct detail::bind_traits<T>;
struct detail::bind_traits<T&>;
struct detail::bind_traits<<const> null_type>;
struct detail::bind_traits<<const><volatile> T[n]>;
struct detail::bind_traits<R(Args...args)>;
struct detail::bind_traits<<const> reference_wrapper<T>>;
struct detail::bind_traits<void>;
struct detail::bind_tuple_mapper<...Ts>;
struct detail::remove_const_reference<T>;
struct detail::remove_const_reference<const T&>;
struct detail::bind_type_generator<...Ts>;
const T& make_const<T>(const T& t);

struct function_adaptor<Func> {
  using plainF = remove_cv_ref_t<Func>;
  struct result_converter<Tuple,len,hasSig>;
  struct sig<Args>;
  static Ret apply<Ret,...Args>(Args&...args);
};
struct function_adaptor<const F>; // no impl
struct function_adaptor<T Obj::*>{
  class sig<Args>;
  static Ret apply<Ret>(T Obj::*data, <const><volatile> Obj& o) { return o.*data; }
  static Ret apply<Ret>(T Obj::*data, <const><volatile> Obj* o) { return o->*data; }
};
struct function_adaptor<Res (Args...)> {
  struct sig<T> { using type = Res; };
  static Res apply<Ret>(Res(*func)()) { return func(args...); }
};
struct function_adaptor<Res (*)(Args...)> {
  struct sig<T> { using type = Res; };
  static Res apply<Ret>(Res(*func)()) { return func(args...); }
};
struct function_adaptor<Res (Obj::*)(Args...)<const>> {
  struct sig<T> { using type = Res; };
  static Res apply<Ret>(Res(Obj::*func)()<const>, const Obj* o) { return (o->*func)(); }
  static Res apply<Ret>(Res(Obj::*func)()<const>, const Obj& o) { return (o.*func)(); }
};

struct return_type_1<Act,A1>;
struct return_type_2<Act,A1,A2>;
struct return_type_N<Act,Args>;

class identity<T> {
  T elem;
public: using element_t = T: using par_t = add_reference_t<add_const_t<T>>;
  explicit ctor(par_t t) : elem{t}{}
  struct sig<SigArgs> { using type = remove_const_t<T>; };
  Ret call<Ret,A,B,C,Env>(A& a, B& b, C& c, Env& env) const { return elem; }
};
lambda_functor<identity<T&>> var<T>(T& t) { return identity<T&>{t}; }
lambda_functor<T> var<T>(const lambda_functor<T>& t) { return t; }
struct var_type<T> { using type = lambda_functor<identity<T&>>; };
lambda_functor<identity<bound_argument_conversion<const T>::type>> constant<T>(const T& t);
lambda_function<T> constant<T>(const lambda_functor<T>& t);
struct constant_type<T> { using type = lambda_functor<identity<bound_argument_conversion<const T>::type>>; };
lambda_functor<identity<const T&>> constant_ref<T>(const T& t);
lambda_functor<T> constant_ref<T>(const lambda_functor<T>& t);
struct constant_ref_type<T> { using type = lambda_functor<identity<const T&>>; };
struct as_lambda_functor<T>;
lambda_functor<idnetity<bound_argument_conversion<const T>::type>> to_lambda_functor<T>(const T& t);
lambda_functor<T> to_lambda_functor<T>(const lambda_functor<T>& t);

struct lambda_functor_base<explicit_return_type_action<Ret>,Args>;
struct lambda_functor_base<protect_action, Args>;
class do_nothing_action{};
struct lambda_functor_base<do_nothing_action, Args>;
struct lambda_functor_base<action<0,Act>, Args>;
struct lambda_functor_base<action<n,Act>, Args>; // 1~10

struct placeholder<i> {
  struct sig<SigArgs>;
  Ret call<R,A,B,C,Env>(A& a, B& b, C& c, Env& env) const { return i == FIRST ? a : i == SECOND ? b : THIRD ? c : env; }
};
using placeholder1_type = const lambda_functor<placeholder<FIRST>>;
using placeholder2_type = const lambda_functor<placeholder<SECOND>>;
using placeholder3_type = const lambda_functor<placeholder<THIRD>>;

class lambda_function<T> : public T {
  constexpr static int arity_bits = get_arity<T>::value;
public: ctor(); ctor(const self&); ctor(const T& t);
  struct sig<SigArgs>;
  using nullary_return_type = base::sig<null_type>::type;
  struct result<F()> { using type = nullary_return_type; };
  struct result<F(A...)> { using type = sig<tuple<F,A...>>:type; };
  nullary_return_type operator()() const;
  sig<tuple<A<const>&...>> operator()<A...>(A <const>&... a) const;
  base::sig<tule<A&...>>::type internal_call<...A>(A&...a) const;
  const lambda_functor<lambda_functor_base<other_action<assignment_action>, tuple<lambda_functor, const_copy_argument<const A>::type>>>
    operator=<A>(const A& a) const;
  const lambda_functor<lambda_functor_base<other_action<subscript_action>, tuple<lambda_functor, const_copy_argument<const A>::type>>>
    operator[]<A>(const A& a) const;
};
struct is_placeholder<lambda_functor<placeholder<FIRST>>> { enum{value=1}; };
struct is_placeholder<lambda_functor<placeholder<SECOND>>> { enum{value=2}; };
struct is_placeholder<lambda_functor<placeholder<THIRD>>> { enum{value=3}; };

lambda_functor<lambda_functor_base<explicit_return_type_action<Ret>, tuple<lambda_functor<Arg>>>>
  ret<Ret,Arg>(const lambda_functor<Arg>& a);
const T& protect<T>(const T& t);
lambda_functor<lambda_functor_base<protect_action, tuple<lambda_functor<Arg>>>> protect<Arg>(const lambda_functor<Arg>& a);
class non_lambda_functor<LambdaFunctor> {
  LambdaFunctor lf;
public: explicit ctor(const LambdaFunctor& a);
  struct sig<SigArg>;
  LambdaFunctor::nullary_return_type operator()() const;
  sig<tuple<const self, A&...>>::type operator()<...A>(A&... a) const;
};
const Arg& unlambda<Arg>(const Arg& a);
non_lambda_functor<lambda_functor<Arg>> unlambda<Arg>(const lambda_functor<Arg>& a);

class const_incorrect_lambda_functor<LambdaFunctor> {
  LambdaFunctor lf;
public: explicit ctor(const LambdaFunctor& a);
  struct sig<SigArg>;
  sig<tuple<const self, A&...>>::type operator()<...A>(A&... a) const;
};
class const_parameter_lambda_functor<LambdaFunctor> {
  LambdaFunctor lf;
public: explicit ctor(const LambdaFunctor& a);
  struct sig<SigArg>;
  sig<tuple<const self, A&...>>::type operator()<...A>(A&... a) const;
};
const const_incorrect_lambda_functor<lambda_functor<Arg>> break_const<Arg>(const lambda_functor<Arg>& lf);
const const_parameter_lambda_functor<lambda_functor<Arg>> const_parameters<Arg>(const lambda_functor<Arg>& lf);

struct voidifier_action{ static void apply<Ret,A>(A&){} };
struct return_type_N<voidifier_action,Args>{ using type=void; };
const lambda_functor<lambda_functor_base<action<1,voidifier_action>, tuple<lambda_functor<Arg>>>>
  make_void<Arg>(const lambda_functor<Arg1>& a1);
const lambda_functor<lambda_functor_base<do_nothing_action, null_type>> make_void<Arg>(const Arg&);
struct result_type_to_sig<T> : public T { struct sig<Args> { using type=T::result_type; }; };
result_type_to_sig<F> std_functor<F>(const F& f) {return f;}

placeholder1_type& _1 = {};
placeholder2_type& _2 = {};
placeholder3_type& _3 = {};
```

------
#### Operators

```c++
// operators
class plus_action{}; class minus_action{}; class multiply_action{}; class divide_action{}; class remainder_action{};
class leftshift_action{}; class rightshift_action{}; class xor_action{};
class and_action{}; class or_action{}; class not_action{};
class less_action{}; class greater_action{}; class lessorequal_action{}; class greaterorequal_action{}; class equal_action{}; class notequal_action{};
class increment_action{}; class decrement_action{};
class addressof_action{}; class contentsof_action{};

class arithmetic_action<Action>; class bitwise_action<Action>; class logical_action<Action>;
class relational_action<Action>; class arithmetic_assignment_action<Action>; class bitwise_assignment_action<Action>;
class unary_arithmetic_action<Action>; class pre_increment_decrement_action<Action>; class post_increment_decrement_action<Action>;
struct is_protectable<arithmetic_action<Act>> { constexpr static bool value=true; }; // all above 9
struct is_protectable<other_action<addressof_action>> { constexpr static bool value=true; }; // and contentsof_action, subscript_action, assignment_action

struct lambda_functor_base<other_action<comma_action>,Args>; // subscript, assignment, addressof, contentsof
struct lambda_functor_base<logical_action<and_action>,Args>; // or, not
struct lambda_functor_base<arithmetic_action<plus_action>,Args>; // minus, multiply, divide, remainder
struct lambda_functor_base<bitwise_action<leftshift_action>,Args>; // rightshift, and, or, xor, not
struct lambda_functor_base<relational_action<less_action>,Args>; // greater, lessorequal, greaterorequal, equal, notequal
struct lambda_functor_base<arithmetic_assignment_action<plus_action>,Args>; // minus, multiply, divide, remainder
struct lambda_functor_base<bitwise_assignment_action<leftshift_action>,Args>; // rightshift, and, or, xor
struct lambda_functor_base<unary_arithmetic_action<plus_action>,Args>; // minus
struct lambda_functor_base<pre_increment_decrement_action<increment_action>,Args>; // decrement
struct lambda_functor_base<post_increment_decrement_action<increment_action>,Args>; // decrement

struct is_instance_of_i<From,To<T...>>;

struct plain_return_type_1<Act,A> { using type = unspecified; };
struct plain_return_type_1<unary_arithmetic_action<Act>,A> { using type = A; };
struct return_type_1<unary_arithmetic_action<Act>,A> { using type = plain_return_type_1<unary_arithmetic_action<Act>, remove_cv_ref_t<A>>::type; };
struct return_type_1<bitwise_action<not_action>,A>;
struct return_type_1<pre_increment_decrement_action<Act>,A>;
struct return_type_1<post_increment_decrement_action<Act>,A>;
struct return_type_1<logical_action<not_action>,A>;
struct return_type_1<other_action<addressof_action>,A>; // contentsof
struct return_type_2<arithmetic_action<Act>,A,B>;
struct return_type_2<bitwise_action<Act>,A,B>;
struct return_type_2<bitwise_action<leftshift_action>,A,B>; // rightshift_action, these are I/O streaming
struct return_type_2<logical_action<Act>,A,B>;
struct return_type_2<relational_action<Act>,A,B>;
struct return_type_2<arithmetic_assignment_action<Act>,A,B>;
struct return_type_2<bitwise_assignment_action<Act>,A,B>;
struct return_type_2<other_action<assignment_action>,A,B>; // comma, subscript

// following (OP, ACTION):
//   (+,arithmetic_action<plus_action>), (-,minus), (*,multiply), (/,divide), (%,remainder)
//   (<<,bitwise_action<leftshift_action>), (>>,rightshift), (&,and), (|,or), (^,xor)
//   (&&,logical_action<and_action>), (||,or)
//   (<,relational_action<less_action>), (>,greater), (<=,lessorequal), (>=,greaterorequal), (==,equal), (!=,notequal)
//   (,,other_action<comma_action>)
const lambda_functor<lambda_functor_base<ACTION, tuple<lambda_functor<Arg>, const_copy_argument<const B>:type>>>
  operator OP<Arg,B>(const lambda_functor<Arg>& a, const B& b);
const lambda_functor<lambda_functor_base<ACTION, tuple<const_copy_argument<const A>::type, lambda_functor<Arg>>>>
  operator OP<A,Arg>(const A& a, const lambda_functor<Arg>& b);
const lambda_functor<lambda_functor_base<ACTION, tuple<lambda_functor<ArgA>, lambda_functor<ArgB>>>>
  operator OP<ArgA,ArgB>(const lambda_functor<ArgA>& a, const lambda_functor<ArgB>& b);

// following (OP, ACTION):
//   (+=,arithmetic_assignment_action<plus_action>), (-,minus), (*,multiply), (/,divide), (%,remainder)
//   (<<,bitwise_assignment_action<leftshift_action>), (>>,rightshift), (&,and), (|,or), (^,xor)
const lambda_functor<lambda_functor_base<ACTION, tuple<lambda_functor<Arg>, const_copy_argument<const B>:type>>>
  operator OP<Arg,B>(const lambda_functor<Arg>& a, const B& b);
const lambda_functor<lambda_functor_base<ACTION, tuple<reference_argument<A>::type, lambda_functor<Arg>>>>
  operator OP<A,Arg>(A& a, const lambda_functor<Arg>& b);
const lambda_functor<lambda_functor_base<ACTION, tuple<lambda_functor<ArgA>, lambda_functor<ArgB>>>>
  operator OP<ArgA,ArgB>(const lambda_functor<ArgA>& a, const lambda_functor<ArgB>& b);

struct convert_ostream_to_ref_others_to_c_plain_by_default<T>;
const lambda_functor<lambda_functor_base<bitwise_action<leftshift_action>, tuple<convert_ostream_to_ref_others_to_c_plain_by_default<A>::type, lambda_functor<Arg>>>>
  operator << <A,Arg>(A& a, const lambda_functor<Arg>& b);
struct convert_istream_to_ref_others_to_c_plain_by_default<T>;
const lambda_functor<lambda_functor_base<bitwise_action<rightshift_action>, tuple<convert_istream_to_ref_others_to_c_plain_by_default<A>::type, lambda_functor<Arg>>>>
  operator >> <A,Arg>(A& a, const lambda_functor<Arg>& b);

const lambda_functor<lambda_functor_base<bitwise_action<leftshift_action>, tuple<lambda_functor<Arg>,Ret(&)(ManipArg)>>>
  operator << <Arg,Ret,ManipArg>(const lambda_functor<Arg>& a, Ret(&b)(ManipArg));
const lambda_functor<lambda_functor_base<bitwise_action<rightshift_action>, tuple<lambda_functor<Arg>,Ret(&)(ManipArg)>>>
  operator >> <Arg,Ret,ManipArg>(const lambda_functor<Arg>& a, Ret(&b)(ManipArg));

// following (OP,ACTION) : (+, arithmetic_action<plus_action>)
const lambda_functor<lambda_functor_base<ACTION, tuple<lambda_functor<Arg>, <const> B(&)[n]>>>
  operator OP<Arg,n,B>(const lambda_functor<Arg>& a, <const> B(&b)[n]);

// following (OP,ACTION) : (+, arithmetic_action<plus_action>), (-, minus)
const lambda_functor<lambda_functor_base<ACTION, tuple<<const> A(&)[n], lambda_functor<Arg>>>>
  operator OP<n,A,Arg>(<const> A(&a)[n], const lambda_functor<Arg>& b);

// following (OP,ACTION):
//   (+, unary_arithmetic_action<plus_action>), (-, minus)
//   (~, bitwise_action<not_action>), (!, logical)
//   (++, pre_increment_decrement_action<increment_action>), (--, decrement)
//   (*, other_action<contentsof_action>), (&, addressof)
const lambda_functor<lambda_functor_base<ACTION, tuple<lambda_functor<Arg>>>>
  operator OP<Arg>(const lambda_functor<Arg>& a);

// following (OP,ACTION): (++, post_increment_decrement_action<increment_action>), (--, decrement)
const lambda_functor<lambda_functor_base<ACTION, tuple<lambda_functor<Arg>>>>
  operator OP<Arg>(const lambda_functor<Arg>& a, int);

class member_pointer_action{};
class other_action<member_pointer_action>;
struct result_type_2<other_action<member_pointer_action>, A, B>;
struct result_type_N<other_action<member_pointer_action>, Args>;
const lambda_functor<lambda_functor_base<action<2,other_action<member_pointer_action>>, tuple<lambda_functor<Arg1>, const_copy_argument<Arg2>::type>>>
  operator ->* <Arg1,Arg2>(const lambda_functor<Arg1>& a1, const Arg2& a2);
const lambda_functor<lambda_functor_base<action<2,other_action<member_pointer_action>>, tuple<lambda_functor<Arg1>, lambda_functor<Arg2>>>>
  operator ->* <Arg1,Arg2>(const lambda_functor<Arg1>& a1, const lambda_functor<Arg2>& a2);
const lambda_functor<lambda_functor_base<action<2,other_action<member_pointer_action>>, tuple<const_copy_argument<Arg1>::type, lambda_functor<Arg2>>>>
  operator ->* <Arg1,Arg2>(const Arg1& a1, const lambda_functor<Arg2>& a2);

class cast_action<T>;
class static_cast_action<T>; class dynamic_cast_action<T>; class const_cast_action<T>; class reinterpret_cast_action<T>;
class cast_action<static_cast_action<T>>;
class cast_action<dynamic_cast_action<T>>;
class cast_action<const_cast_action<T>>;
class cast_action<reinterpret_cast_action<T>>;
class typeid_action;
class sizeof_action;
struct return_type_N<cast_action<cast_type<T>>, A> { using type = T; };
struct return_type_N<typeid_action, A> { using type = std::type_info const&; };
struct return_type_N<sizeof_action, A> { using type = size_t; };
const lambda_functor<lambda_functor_base<action<1,cast_action<static_cast_action<T>>>, tuple<const_copy_argument<const Arg1>::type>>>
  ll_static_cast(const Arg1& a1);
const lambda_functor<lambda_functor_base<action<1,cast_action<dynamic_cast_action<T>>>, tuple<const_copy_argument<const Arg1>::type>>>
  ll_dynamic_cast(const Arg1& a1);
const lambda_functor<lambda_functor_base<action<1,cast_action<const_cast_action<T>>>, tuple<const_copy_argument<const Arg1>::type>>>
  ll_const_cast(const Arg1& a1);
const lambda_functor<lambda_functor_base<action<1,cast_action<reinterpret_cast_action<T>>>, tuple<const_copy_argument<const Arg1>::type>>>
  ll_reinterpret_cast(const Arg1& a1);
const lambda_functor<lambda_functor_base<action<1,typeid_action>, tuple<const_copy_argument<const Arg1>::type>>>
  ll_typeid(const Arg1& a1);
const lambda_functor<lambda_functor_base<action<1,sizeof_action>, tuple<const_copy_argument<const Arg1>::type>>>
  ll_sizeof(const lambda_functor<Arg1>& a1);
```

-----
#### Control Structures

```c++
class tagged_lambda_functor<Tag,LambdaFunctor>;
class tagged_lambda_functor<Tag,lambda_functor<Args>> : lambda_functor<Args>;

class ifthen_action{}; class ifthenelse_action{}; class ifthenelsereturn_action{};
class lambda_functor_base<ifthen_action,Args>;
const lambda_functor_base<ifthen_action,tuple<lambda_functor<Arg1>, lambda_functor<Arg2>>>
  if_then<Arg1,Arg2>(const lambda_functor<Arg1>& a1, const lambda_functor<Arg2>& a2);
class lambda_functor_base<ifthenelse_action,Args>;
const lambda_functor_base<ifthenelse_action,tuple<lambda_functor<Arg1>, lambda_functor<Arg2>, lambda_functor<Arg3>>>
  if_then_else<Arg1,Arg2,Arg3>(const lambda_functor<Arg1>& a1, const lambda_functor<Arg2>& a2, const lambda_functor<Arg3>& a3);
const lambda_functor_base<ifthenelsereturn_action,tuple<lambda_functor<Arg1>, const_copy_argument<Arg2>::type, const_copy_argument<Arg3>::type>>
  if_then_else_return<Arg1,Arg2,Arg3>(const lambda_functor<Arg1>& a1, const Arg2& a2, const Arg3& a3);
struct return_type_2<other_action<ifthenelsereturn_action>, A, B>;
class lambda_functor_base<ifthenelsereturn_action,Args>;
struct if_then_else_composite<CondT,ThenT,ElseT>;
struct else_gen<CondT,ThenT>;
struct if_then_composite<CondT,ThenT>;
struct if_gen<CondT>;
if_gen<CondT> if_<CondT>(CondT const& cond);

class forloop_action{}; class forloop_no_body_action{};
class whileloop_action{}; class whileloop_no_body_action{};
class dowhileloop_action{}; class dowhileloop_no_body_action{};
const lambda_functor_base<forloop_action,tuple<lambda_functor<Arg1>, lambda_functor<Arg2>, lambda_functor<Arg3>, lambda_functor<Arg4>>>
  for_loop<Arg1,Arg2,Arg3,Arg4>(const lambda_functor<Arg1>& a1, const lambda_functor<Arg2>& a2, const lambda_functor<Arg3>& a3, const lambda_functor<Arg4>& a4);
const lambda_functor_base<forloop_no_body_action,tuple<lambda_functor<Arg1>, lambda_functor<Arg2>, lambda_functor<Arg3>>>
  for_loop<Arg1,Arg2,Arg3>(const lambda_functor<Arg1>& a1, const lambda_functor<Arg2>& a2, const lambda_functor<Arg3>& a3);
const lambda_functor_base<whileloop_action,tuple<lambda_functor<Arg1>, lambda_functor<Arg2>>>
  while_loop<Arg1,Arg2>(const lambda_functor<Arg1>& a1, const lambda_functor<Arg2>& a2);
const lambda_functor_base<whileloop_no_body_action,tuple<lambda_functor<Arg1>, lambda_functor<Arg2>>>
  while_loop<Arg1>(const lambda_functor<Arg1>& a1);
const lambda_functor_base<dowhileloop_action,tuple<lambda_functor<Arg1>, lambda_functor<Arg2>>>
  do_while_loop<Arg1,Arg2>(const lambda_functor<Arg1>& a1, const lambda_functor<Arg2>& a2);
const lambda_functor_base<dowhileloop_no_body_action,tuple<lambda_functor<Arg1>, lambda_functor<Arg2>>>
  do_while_loop<Arg1>(const lambda_functor<Arg1>& a1);
class lambda_functor_base<forloop_action,Args>;
class lambda_functor_base<forloop_no_body_action,Args>;
class lambda_functor_base<whileloop_action,Args>;
class lambda_functor_base<whileloop_no_body_action,Args>;
class lambda_functor_base<dowhileloop_action,Args>;
class lambda_functor_base<dowhileloop_no_body_action,Args>;
struct while_composite<CondT,DoT>;
struct while_gen<CondT>;
while_gen<CondT> while_<CondT>(CondT const& cond);
struct do_composite<DoT,CondT>;
struct do_gen2<DoT>;
struct do_gen;
do_gen const do_ = do_gen{};
struct for_composite<InitT,CondT,StepT,DoT>;
struct for_gen<InitT,CondT,StepT>;
for_gen<InitT,CondT,StepT> for_<InitT,CondT,StepT>(InitT const& init, CondT const& cond, StepT const& step);

struct switch_action<n,...S>{};
struct detail::case_label<v>{}; struct detail::default_label{};
struct detail::switch_case_tag<T>{};
const tagged_lambda_functor<switch_case_tag<case_label<caseValue>>, lambda_functor<Arg>>
  case_statement<caseValue,Arg>(const lambda_functor<Arg>& a);
const tagged_lambda_functor<switch_case_tag<case_label<caseValue>>, lambda_functor<lambda_functor_base<do_nothing_action, null_type>>>
  case_statement<caseValue>();
const tagged_lambda_functor<switch_case_tag<default_label>, lambda_functor<Arg>>
  default_statement<Arg>(const lambda_functor<Arg>& a);
const tagged_lambda_functor<switch_case_tag<default_label>, lambda_functor<lambda_functor_base<do_nothing_action, null_type>>>
  default_statement();
class lambda_functor_base<switch_action<n,A...,[default_label]>,Args>;

using placeholderE_type = lambda_functor<placeholder<EXCEPTION>>;
struct detail::catch_block{}; struct detail::catch_all_block{}; struct exception_catch_tag<T>{};
struct catch_action<...Catch>{}; // at least 1
struct catch_all_action{};
struct return_try_catch_action<CatchActions>{};
struct try_catch_action<CatchActions>{};
struct throw_new_action{};
struct rethrow_action{};
struct throw_action<ThrowType>;
struct throw_action<rethrow_action>;
struct throw_action<throw_new_action>;
struct return_type_N<throw_action<T>,Any>{ using type = void; };
const lambda_functor<lambda_functor_base<action<0,throw_action<rethrow_action>>,null_type>
  rethrow();
const lambda_functor<lambda_functor_base<action<1,throw_action<rethrow_action>>,tuple<const_copy_argument<const Arg1>::type>>
  throw_exception<Arg1>(const Arg1&...a1);
const tagged_lambda_functor<exception_catch_tag<catch_block<CatchType>>,lambda_functor<Arg>>
  catch_exception<CatchType,Arg>(const lambda_functor<Arg>& a);
const tagged_lambda_functor<exception_catch_tag<catch_block<CatchType>>,lambda_functor<lambda_functor_base<do_nothing_action,null_type>>>
  catch_exception<CatchType>();
const tagged_lambda_functor<exception_catch_tag<catch_all_block>,lambda_functor<Arg>>
  catch_all<Arg>(const lambda_functor<Arg>& a);
const tagged_lambda_functor<exception_catch_tag<catch_all_block>,lambda_functor<lambda_functor_base<do_nothing_action,null_type>>>
  catch_all(const lambda_functor<Arg>& a);
const lambda_functor<lambda_functor_base<action<2,try_catch_action<catch_action<Catch>...,CatchN>>, tuple<lambda_functor<TryArg>, LF1...>>>
  try_catch<TryArg,...Catch,CatchN,...LF,LFN>(const lambda_functor<TryArg>& a1,
    const tagged_lambda_functor<exception_catch_tag<catch_block<Catch>>, LF>&...a,
    const tagged_lambda_functor<exception_catch_tag<CatchN>, LFN>& an);
class lambda_functor_base<action<n+1,try_catch_action<catch_action<catch_block<Catch>...>>>,Args>;
class lambda_functor_base<action<n+2,try_catch_action<catch_action<catch_block<Catch>...,catch_all_block>>>,Args>;
```

------
#### Utilities

```c++
// algorithms (wraps std algorithms)
struct ll::for_each;
struct ll::find; struct ll::find_if; struct ll::find_end; struct ll::find_first_of; struct ll::adjacent_find;
struct ll::count; struct ll::count_if;
struct ll::mismatch; struct ll::equal;
struct ll::search;
struct ll::copy; struct ll::copy_backward;
struct ll::swap; struct ll::swap_ranges; struct ll::iter_swap;
struct ll::transform;
struct ll::replace; struct ll::replace_if; struct ll::replace_copy; struct ll::replace_copy_if;
struct ll::fill; struct ll::fill_n;
struct ll::generate; struct ll::generate_n;
struct ll::remove; struct ll::remove_if; struct ll::remove_copy; struct remove_copy_if;
struct ll::unique; struct ll::unique_copy;
struct ll::reverse; struct ll::reverse_copy;
struct ll::rotate; struct ll::rotate_copy;
struct ll::random_shuffle;
struct ll::partition; struct ll::stable_partition;
struct ll::sort; struct ll::stable_sort; struct ll::partial_sort;
struct ll::nth_element;
struct ll::lower_bound; struct ll::upper_bound; struct ll::equal_range; struct ll::binary_search;
struct ll::merge; struct ll::inplace_merge;
struct ll::includes; struct ll::set_union; struct ll::set_intersection; struct ll::set_difference; struct set_symmetric_difference;
struct ll::push_heap; struct ll::pop_heap; struct ll::make_heap; struct ll::sort_heap;
struct ll::min; struct ll::max; struct ll::min_element; struct ll::max_element;
struct ll::lexicographical_compare;
struct ll::next_permutation; struct ll::prev_permutation;

struct ll::accumulate;
struct ll::inner_product;
struct ll::partial_sum;
struct ll::adjacent_difference;

// bind
const lambda_functor<lambda_functor_base<action<n+1,function_action<n+1,Result>>, bind_tuple_mapper<Result(&)(P...), const A...>::type>>
  bind<Result,...P,...A>(Result(&a1)(P...), const A...);
const lambda_functor<lambda_functor_base<action<n,function_action<n>>, bind_tuple_mapper<const Args...>::type>>
  bind<...Args>(const Args&... a);
const lambda_functor<lambda_functor_base<action<n,function_action<n,Result>>, bind_tuple_mapper<const Args...>::type>>
  bind<Result,...Args>(const Args&... a);

// closures
class closure_frame<ClosureT> : public ClosureT::tuple_t {
  closure_frame* save; closure_frame*& frame; // no copy
public: ctor(ClosureT& clos); ctor<TupleT>(ClosureT& clos, TupleT const& init); ~dtor();
};
class closure_member<n,ClosureT> {
  ClosureT::closure_frame_t*& frame;
public: using tuple_t = ClosureT::tuple_t;
  ctor();
  struct sig<TupleT>;
  Ret call<Ret,A,B,C>(A&, B&, C&) const;
};
class closure<T0=null_type,...,T4=null_type> {
  closure_frame_t* frame; // no copy
public: using tuple_t = tuple<T0,...,T4>; using closure_frame_t = closure_frame<self>;
  ctor();
  closure_frame_t<const>& context() <const>;
  using lambda_functor<closure_member<0,self>> member1; // 1~5
};

// construct
struct constructor {
  struct sig<T>{using type=void;};
  void operator()<A>(A a) const;
};
struct new_ptr<T> {
  struct sig<U> {using type = T*;};
  T* operator()<...Args>(Args&&...args)const;
};
struct delete_ptr {
  struct sig<U> {using type = void;};
  void operator()<A>(A&& a) const;
};
struct new_array<T> {
  struct sig<U> {using type = T*;};
  T* operator()<...Args>(Args&&...args)const;
};
struct delete_array {
  struct sig<U> {using type = void;};
  void operator()<A>(A&& a) const;
};
```

-----
### Dependencies

#### Boost.Bind

* `<boost/is_placeholder.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`

#### Boost.Detail

* `<boost/detail/container_fwd.hpp>`

#### Boost.Iterator

* `<boost/indirect_reference.hpp>`

#### Boost.MPL

* `<boost/mpl/has_xxx.hpp>`
* `<boost/mpl/or.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/*.hpp>`

#### Boost.Tuple

* `<boost/tuple/tuple.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

#### Boost.Utility

* `<boost/utility/result_of.hpp>`

------
### Standard Facilities

Language: lambda expression (C++11)
