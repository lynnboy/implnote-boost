# Boost.MetaParse

* lib: `boost/libs/metaparse`
* repo: `boostorg/metaparse`
* commit: `0c7c2dd`, 2025-04-15

------
### Parsers

```c++
struct impl::is_char_c<ch> { using type = self; struct apply<Ch>: bool_<Ch::type::value == ch>{}; };
struct impl::is_none_c_impl<d,...cs>; struct impl::is_none_c_impl<d> : bool<true>{}; struct impl::is_none_c_impl<d,d,cs...> : bool<false>{}; struct impl::is_none_c_impl<d,c,cs...> : is_none_c_impl<d,cs...>{};
struct impl::is_none_c<...cs> { using type = self; struct apply<Ch>: is_none_c_impl<C::type::value,cs...>{}; };
struct impl::next_digit { using type = self; struct apply<PartialResult,NextDigit>: mpl::int_<PartialResult::type::value * 10 + NextDigit::type::value>{}; };
struct impl::void_{ using type = self; };

// character parsers
using alphanum = one_of<letter,digit>;
using digit = accept_when<change_error_message<one_char, digit_expected>, is_digit<>, digit_expected>;
using letter = accept_when<one_char, is_letter<>, letter_expected>;
struct lit_c<ch> : accept_when<change_error_message<one_char, literal_expected<ch>>, is_char_c<ch>, literal_expected<ch>>{};
struct lit<Ch> : lit_c<Ch::type::value>{};
struct one_char {
private: struct next_pos<Ch,Pos>; // EOL ? next_line<Pos,Ch> ? next_char<Pos,Ch>
	struct unchecked<S,NextPos> : accept<front<S>::type, pop_front<S>, NextPos>{};
	using type = self; struct apply<S,Pos> : eval_if<empty<S>, reject<unexpected_end_of_input,Pos>, unchecked<S,next_pos<front<S>,Pos>>>{};
};
struct one_char_except<...Cs> : accept_when<one_char, is_none_c<Cs::type::value...>, unexpected_character>{};
struct one_char_except_c<...cs> : accept_when<one_char,is_none_c<cs...>, unexpected_character>{};
struct range<From,To> : accept_when<one_char, in_range_c<char,From::type::value,To::type::value>, unexpected_character>{};
struct range_c<from,to> : accept_when<one_char, in_range_c<char,from,to>, unexpected_character>{};
using space = accept_when<one_char, is_whitespace<>, whitespace_expected>{};

// numeric parsers
using digit_val = transform<digit, digit_to_int<>>;
using int_ = foldl1<digit_val, mpl::int_<0>, next_digit>;

// space parsers
using spaces = repeated1<space>;

// validation and error reporting
struct empty<Result> { using type = empty; struct apply<S,Pos> : mpl::if_<mpl::empty<S>, accept<Result,S,Pos>, reject<end_of_input_expected,Pos>>{}; };
struct fail<Msg> { using type = fail; struct apply<S,Pos> : reject<Msg,Pos>{}; };

// miscellanous
struct keyword<Kw,ResultType=void_> {
private: struct nonempty {
	private: using next_char_parser = lit<mpl::front<Kw>::type>; using rest_parser = keyword<mpl::pop_front<Kw>::type,ResultType>;
		struct apply_unchecked<S,Pos> : rest_parser::apply<get_remaining<next_char_parser::apply<S,Pos>>::type, get_position<next_char_parser::apply<S,Pos>>::type>{};
	public: struct apply<S,Pos> : mpl::eval_if<is_error<next_char_parser::apply<S,Pos>>::type, next_char_parser::apply<S,Pos>, apply_unchecked<S,Pos>>{};
	};
public: using type = self; struct apply<S,Pos> : mpl::if_<mpl::empty<Kw>, return_<ResultType>, nonempty>::type::apply<S,Pos>{};
};
struct return_<C> { using type = self; struct apply<S,Pos> : accept<C,S,Pos>{}; };
```

------
### Parser Combinators

```c++
struct impl::iterate_impl_unchecked<n,P,Accum,S,Pos> : iterate_impl<n-1,P, mpl::push_back<Accum,get_result<P::apply<S,Pos>>::type>::type>
	::apply<get_remaining<P::apply<S,Pos>>::type,get_position<P::apply<S,Pos>>::type> {};
struct impl::iterate_impl<n,P,Accum> { using type=self; struct apply<S,Pos> : mpl::eval_if<is_error<P::apply<S,Pos>>::type, P::apply<S,Pos>, iterate_impl_unchecked<n,P,Accum,S,Pos>>{}; };
struct impl::iterate_impl<0,P,Accum> : return_<Accum>{};
struct impl::back_inserter { using type=self; struct apply<T0,T1> : mpl::push_back<T0,T1>{}; };
struct impl::front_inserter { using type=self; struct apply<T0,T1> : mpl::push_front<T0,T1>{}; };
struct impl::eval_later_result<R1,R2> : mpl::eval_if<mpl::less<get_position<R2>::type,get_position<R1>::type>::type, R1,R2>{};
struct impl::any_of_c<...cs> { using type=self; static constexpr bool run(char c_); struct apply<Ch> : mpl::bool_<run(Ch::type::value)>{}; };
struct impl::any_of_c<> { using type=self; struct apply<_> : mpl::false_{}; };
struct impl::nth_of_c_skip_remaining<FRes,S,Pos> : return_<FRes>::apply<S,Pos>{};
struct impl::nth_of_c_skip_remaining<FRes,S,Pos,P,Ps..>{
private: struct apply_unchecked<NextRes> : nth_of_c_skip_remaining<FRes,get_remaining<NextRes>::type,get_position<NextRes>::type,Ps...>{};
public: using type = std::conditional< is_error<P::apply<S,Pos>>::type::value, P::apply<S,Pos>, apply_unchecked<P::apply<S,Pos>>>::type::type;
};
struct impl::nth_of_c<n,S,Pos,P,Ps...> {
private: struct apply_unchecked<NextRes> : nth_of_c<n-1, get_remaining<NextRes>::type, get_position<NextRes>::type, Ps...>{};
public: using type = std::conditional< is_error<P::apply<S,Pos>>::type::value, P::apply<S,Pos>, apply_unchecked<P::apply<S,Pos>>>::type::type;
};
struct impl::nth_of_c<0,S,Pos,P,Ps...> {
private: struct apply_unchecked<NextRes> : nth_of_c_skip_remaining<get_result<NextRes>::type, get_remaining<NextRes>::type, get_position<NextRes>::type, Ps...>{};
public: using type = std::conditional< is_error<P::apply<S,Pos>>::type::value, P::apply<S,Pos>, apply_unchecked<P::apply<S,Pos>>>::type::type;
};
struct impl::push_front_result<Value> { using type=self; struct apply<Seq> : mpl::push_front<Seq,get_result<Value>::type>{}; };
struct impl::sequence_apply_transformN<T<...>> { using type=self; struct apply<V> { using type=T<V...>; };
};

/// validation and error reporting
struct accept_when<P,Pred,Msg> {
private: struct unchecked { struct apply<S,Pos> : mpl::eval_if<Pred::apply<get_result<P::apply<S,Pos>>::type>::type, P::apply<S,Pos>, reject<Msg,Pos>>{}; };
public: using type=self; struct apply<S,Pos> : mpl::if_<is_error<P::apply<S,Pos>>, P, unchecked>::type::apply<S,Pos>{};
};
struct change_error_message<P,Msg> { using type=self; struct apply<S,Pos> : eval_if<is_error<P::apply<S,Pos>>::type, reject<Msg,Pos>, P::apply<S,Pos>>{}; };
struct entire_input<P,Msg=end_of_input_expected> : first_of<P,change_error_message<empty<void>,Msg>> {};
struct except<P,Result,Msg> { using type=self; struct apply<S,Pos>: mpl::if_<is_error<P::apply<S,Pos>>, accept<Result,S,Pos>, reject<Msg,Pos>>{}; };
struct fail_at_first_char_expected<P> {
private: struct apply_err<S,Pos> : eval_if<mpl::equal_to<Pos,get_position<P::apply<S,Pos>>::type>::type, accept<void_,S,Pos>, P::apply<S,Pos>>{};
public: using type=self; struct apply<S,Pos>: mpl::eval_if<is_error<P::apply<S,Pos>>::type, apply_err<S,Pos>, reject<expected_to_fail,Pos>>{};
};
struct transform_error<P,F> { using type=self; struct apply<S,Pos> : mpl::eval_if<is_error<P::apply<S,Pos>>::type, F::apply<P::apply<S,Pos>::type>, P::apply<S,Pos>>{}; };
struct transform_error_message<P,F> {
	struct rejection<R> : reject<F::apply<get_message<R>::type>::type, get_position<R>>{};
	using type=self; struct apply<S,Pos> : mpl::eval_if<is_error<P::apply<S,Pos>>::type, rejection<P::apply<S,Pos>>, P::apply<S,Pos>>{};
};

// repetation
struct foldl<P,State,ForwardOp> {
private: struct apply_unchecked<Res> : self<P,ForwardOp::apply<State::type,get_result<Res>::type>, ForwardOp>::apply<get_remaining<Res>::type,get_position<Res>::type>{};
	struct next_iteration<S,Pos> : accept<State::type,S,Pos>{};
public: using type=self; struct apply<S,Pos> : mpl::eval_if<is_error<P::apply<S,Pos>>::type, next_iteration<S,Pos>, apply_unchecked<P::apply<S,Pos>>>{};
};
struct foldl1<P,State,ForwardOp> { using type=self; struct apply<S,Pos> : mpl::if_<is_error<P::apply<S,Pos>>, P, foldl<P,State,ForwardOp>>::type::apply<S,Pos>{}; };
struct foldl_reject_incomplete<P,State,ForwardOp> {
private: struct apply_unchecked<Res> : self<P,ForwardOp::apply<State::type,get_result<Res>::type>, ForwardOp>::apply<get_remaining<Res>::type,get_position<Res>::type>{};
	struct accept_state<S,Pos> : accept<State::type,S,Pos>{};
	struct end_of_folding<S,Pos> : mpl::eval_if<equal_to<Pos::type, get_position<P::apply<S,Pos>>::type>::type, accept_state<S,Pos>, P::apply<S,Pos>>{};
public: using type=self; struct apply<S,Pos> : mpl::eval_if<is_error<P::apply<S,Pos>>::type, end_of_folding<S,Pos>, apply_unchecked<P::apply<S,Pos>>>{};
};
struct foldl_reject_incomplete1<P,State,ForwardOp> { using type=self; struct apply<S,Pos> : mpl::if_<is_error<P::apply<S,Pos>>, P, foldl_reject_incomplete<P,State,ForwardOp>>::type::apply<S,Pos>{}; };
struct foldl_reject_incomplete_start_with_parser<P,StateP,ForwardOp> {
private: struct apply_unchecked<Res> : foldl_reject_incomplete<P,get_result<Res>::type,ForwardOp>::apply<get_remaining<Res>::type,get_position<Res>::type>{};
public: using type=self; struct apply<S,Pos> : mpl::eval_if<is_error<StateP::apply<S,Pos>>::type, StateP::apply<S,Pos>, apply_unchecked<StateP::apply<S,Pos>>>{};
};
struct foldl_start_with_parser<P,StateP,ForwardOp> {
private: struct apply_unchecked<Res> : foldl<P,get_result<Res>::type,ForwardOp>::apply<get_remaining<Res>::type,get_position<Res>::type>{};
public: using type=self; struct apply<S,Pos> : mpl::eval_if<is_error<StateP::apply<S,Pos>>::type, StateP::apply<S,Pos>, apply_unchecked<StateP::apply<S,Pos>>>{};
};
struct foldr<P,State,BackwardOp> : foldr_start_with_parser<P,return_<State>,BackwardOp>{};
struct foldr1<P,State,BackwardOp> { using type=self; struct apply<S,Pos> : mpl::if_<is_error<P::apply<S,Pos>>, P, foldr<P,State,BackwardOp>>::type::apply<S,Pos>{}; };
struct foldr_reject_incomplete<P,State,BackwardOp> : foldr_start_with_parser<P,first_of<return_<State>, fail_at_first_char_expected<P>>, BackwardOp>{};
struct foldr_reject_incomplete1<P,State,BackwardOp> { using type=self; struct apply<S,Pos> : mpl::if_<is_error<P::apply<S,Pos>>, P, foldr_reject_incomplete<P,State,BackwardOp>>::type::apply<S,Pos>{}; };
struct foldr_start_with_parser<P,StateP,BackwardOp> {
private: struct apply_unchecked1<Res,Rem> : accept<BackwardOp::apply<get_result<Rem>::type,get_result<Res>::type>::type, get_remaining<Rem>::type,get_position<Rem>::type>{};
	struct apply_unchecked<Res> {
	private: using parsed_remaining = foldr_start_with_parser::apply<get_remaining<Res>::type,get_position<Res>::type>;
	public: using type = mpl::eval_if<is_error<parsed_remaining>::type, parsed_remaining, apply_unchecked1<Res,parsed_remaining>>::type;
	};
public: using type=self; struct apply<S,Pos> : mpl::eval_if<is_error<P::apply<S,Pos>>::type, StateP::apply<S,Pos>, apply_unchecked<P::apply<S,Pos>>>{};
};

struct iterate<P,N> : iterate_c<P,N::type::value>{};
struct iterate_c<P,n> : iterate_impl<n,P,mpl::deque<>>{};

struct repeated<P> : foldl<P,mpl::vector<>, impl::back_inserter>{};
struct repeated1<P> : foldl1<P,mpl::vector<>, impl::back_inserter>{};
struct repeated_reject_incomplete<P> : foldl_reject_incomplete<P,mpl::vector<>, impl::back_inserter>{};
struct repeated_reject_incomplete1<P> : foldl_reject_incomplete1<P,mpl::vector<>, impl::back_inserter>{};
using repeated_one_of<...Ps> = repeated<one_of<Ps...>>;
using repeated_one_of1<...Ps> = repeated1<one_of<Ps...>>;

// selection
struct if_<P,T,F> { using type=self; struct apply<S,Pos> : accept<mpl::if_<is_error<P::apply<S,Pos>>,F,T>::type, S,Pos>{}; };
struct one_of<P,Ps...> {
	using type=self;
	struct apply<S,Pos> : eval_if< is_error<P::apply<S,Pos>>::type, eval_if< is_error<one_of<Ps...>::apply<S,Pos>>::type,
								eval_later_result< P::apply<S,Pos>, one_of<Ps...>::apply<S,Pos>>, one_of<Ps...>::apply<S,Pos>>, P::apply<S,Pos>>{};
};
struct one_of<>: fail<none_of_the_expected_cases_found>{};
struct one_of_c<...cs> : accept_when<one_char, any_of_c<cs...>, none_of_the_expected_cases_found>{};
struct optional<P,Def=void> { using type=self; struct apply<S,Pos> : mpl::if_<is_error<P::apply<S,Pos>>, accept<Def,S,Pos>, P::apply<S,Pos>::type>{}; };

//sequence
struct first_of<...Ps> { using type=self; struct apply<S,Pos> : impl::nth_of_c<0,S,Pos,Ps...>{}; };
struct first_of<> : fail<index_out_of_range<0,-1,0>>{};
struct last_of<...Ps> { using type=self; struct apply<S,Pos> : impl::nth_of_c<sizeof...(Ps)-1,S,Pos,Ps...>{}; };
struct last_of<> : fail<index_out_of_range<0,-1,0>>{};
struct middle_of<P1,P2,P3> {
	using type=self;
	struct apply<S,Pos> : impl::nth_of_c<1,P1,P2,transform_error_message<P3,error::unpaired<get_line<Pos>::type::value, get_col<Pos>::type::value>>>::apply<S,Pos>{};
};
struct nth_of<K,...Ps> : impl::nth_of_c<K::type::value,Ps...>{};
struct nth_of_c<n,...Ps> {
	using type=self;
	struct apply<S,Pos> : std::conditional<(0<=n && n<sizeof...(Ps)), mpl::nth_of_c<n,S,Pos,Ps...>, fail<index_out_of_range<0,sizeof...(Ps)-1,n>>::apply<S,Pos>>::type{};
};
struct nth_of_c<n> : fail<index_out_of_range<0,-1,n>>{};
struct sequence<> : return_<mpl::vector<>>{};
struct sequence<P,Ps...> {
private: struct apply_unchecked<Res> : transform<sequence<Ps...>, impl::push_front_result<Res>>::apply<get_remaining<Res>::type, get_position<Res>::type>{};
public: using type=self; struct apply<S,Pos> : mpl::eval_if<is_error<P::apply<S,Pos>>::type, P::apply<S,Pos>, apply_unchecked<P::apply<S,Pos>>>{};
};
struct sequence_applyN<T<...>,...Ps> : transform<sequence<Ps...>, sequence_apply_transformN<T>>{}; // N from 1 upto LIMIT_SEQUENCE_SIZE

// result transformation (semantic action)
struct always<P,Result> {
private: struct apply_unchecked<Res> : accept<Result,get_remaining<Res>::type,get_position<Res>::type>{};
public: using type=self; struct apply<S,Pos> : mpl::eval_if<is_error<P::apply<S,Pos>>::type, P::apply<S,Pos>, apply_unchecked<P::apply<S,Pos>>>{};
};
struct always_c<ch,Result> : always<lit_c<Ch>,Result>{};
struct transform<P,T> {
private: struct no_error<S,Pos> : accept<T::apply<get_result<P::apply<S,Pos>>::type>::type, get_remaining<P::apply<S,Pos>>, get_position<P::apply<S,Pos>>>{};
public: using type=self; struct apply<S,Pos> : unless_error<P::apply<S,Pos>, no_error<S,Pos>>{};
};
```

------
### Grammar

```c++
namespace grammar_util {
struct repeated_apply_impl<op,FState> { using type=self; struct apply<G>: repeated<FState::apply<G>::type>{}; };
struct repeated_apply_impl<'+',FState> { using type=self; struct apply<G>: repeated1<FState::apply<G>::type>{}; };
struct build_repeated { using type=self; struct apply<FState,T>: repeated_apply_impl<T::type::value, FState>{}; };
struct build_sequence { using type=self;
	struct apply_impl<FState,FP> { using type=self; struct apply<G>: sequence<FState::apply<G>::type, FP::apply<G>::type>{}; };
	struct apply<FState,FP>: apply_impl<FState,FP>{};
};
struct build_selection { using type=self;
	struct apply_impl<FState,FP> { using type=self; struct apply<G>: one_of<FState::apply<G>::type, FP::apply<G>::type>{}; };
	struct apply<FState,FP>: apply_impl<FState,FP>{};
};
struct get_parser<G,Name> { using P=mpl::at<G::rules,Name>::type::apply<G>;
	struct impl<Actions> : transform<P::type, Actions::type>{};
	using type = mpl::eval_if<mpl::has_key<G::actions,Name>::type, impl<mpl::at<G::actions,Name>>, P>::type;
};
struct build_name { using type=self;
	struct apply_impl<Name> { using type=self; struct apply<G> : get_parser<G,Name>{}; };
	struct apply<Name> : apply_impl<Name>{};
};
struct build_char { using type=self;
	struct apply_impl<Ch> { using type=self; struct apply<G> : lit<Ch>{}; };
	struct apply<Ch> : apply_impl<Ch>{};
};
using repeated_token = token<lit_c<'*'>>;
using repeated1_token = token<lit_c<'+'>>;
using or_token = token<lit_c<'|'>>;
using open_bracket_token = token<lit_c<'('>>;
using close_bracket_token = token<lit_c<')'>>;
using define_token = token<keyword<string<':',':','='>>>;
using char_token = middle_of<lit_c<'\''>,
	one_of< last_of< lit_c<'\\'>,
				one_of< always<lit_c<'n'>,mpl::char_<'\n'>>, always<lit_c<'r'>,mpl::char_<'\r'>>, always<lit_c<'t'>,mpl::char_<'\t'>>, lit_c<'\\'>, lit_c<'\''> >>,
			one_char_except_c<'\''>>,
	token<lit_c<'\''>>>;
using name_token = token<foldr1<one_of<alphanum, lit_c<'_'>>, string<>, impl::front_inserter>>;
using bracket_expression = middle_of<open_bracket_token, expression, close_bracket_token>;
using name_expression = one_of<transform<char_token,build_char>, transform<name_token,build_name>, bracket_expression>;
using repeated_expression = foldl_start_with_parser<one_of<repeated_token,repeated1_token>, name_expression, build_repeated>;
using seq_expression = foldl_start_with_parser<repeated_expression,repeated_expression, build_sequence>;
struct expression : foldl_start_with_parser<last_of<or_token,seq_expression>, seq_expression,build_selection>{};
using rule_definition = sequence<name_token, define_token, expression>;
using parser_parser = build_parser<entire_input<rule_definition>>;

struct build_native_parser<P> { using type=self; struct apply<G> { using type=P; }; };
struct build_parsed_parser<S> {
	using p=parser_parser::apply<S>::type; using name = mpl::front<p>::type; using exp = mpl::back<p>::type;
	struct the_parser { using type=self; struct apply<G>: exp::apply<G>{}; };
	using type = mpl::pair<name,the_parser>;
};
using name_parser = build_parser<name_token>;
struct rebuild<S> : name_parser::apply<S>{};

struct no_action;
struct grammar_builder<Start,Rules,Actions> {
	using type=grammar_builder; using rules=Rules; using actions=Actions;
	struct apply<S,Pos> : get_parser<grammar_builder,rebuild<Start>::type>::type::apply<S,Pos>{};
	struct import<Name,P> : add_import<grammar_builder,rebuild<Name>::type,P>{};
	struct rule<Def,Action=no_action> :add_rule<grammar_builder, build_parsed_parser<Def>, Action>{};
};
struct add_rule<grammar_builder<Start,Rules,Actions>,P,no_action> : grammar_builder<Start,mpl::insert<Rules,P::type>::type,Actions>{};
struct add_rule<grammar_builder<Start,Rules,Actions>,P,F>
	: grammar_builder<Start,mpl::insert<Rules,P::type>::type,mpl::insert<Actions,mpl::pair<P::name,mpl::lambda<F>::type>>::type>{};
struct add_import<grammar_builder<Start,Rules,Actions>,Name,P> : grammar_builder<Start,mpl::insert<Rules,mpl::pair<Name,build_native_parser<P>>>::type,Actions>{};
}

struct grammar<Start=string<'S'>> : grammar_builder<Start,mpl::map<>,mpl::map<>>{};

struct look_ahead<P> {
private: struct no_error<S,Pos> : accept<get_result<P::apply<S,Pos>>::type,S,Pos>{};
public: using type=self; struct apply<S,Pos> : mpl::eval_if<is_error<P::apply<S,Pos>>::type, P::apply<S,Pos>, no_error<S,Pos>>{};
};
struct token<P> : first_of<P,repeated<space>>{};
```

------
### Compile-time Data Structures and Values

```c++
// result of parsing
struct accept<Result,Remaining,Pos> { using tag=accept_tag; using type=self; using result=Result; using remaining=Remaining; using source_position = Pos; };
struct get_message<T> : get_message_impl<T::type::tag>::apply<T::type>{};
struct get_position<T> : get_position_impl<T::type::tag>::apply<T::type>{};
struct get_remaining<T> : get_remaining_impl<T::type::tag>::apply<T::type>{};
struct get_result<T> : get_result_impl<T::type::tag>::apply<T::type>{};
struct is_error<T=mpl::na> : is_same<fail_tag,mpl::tag<T::type>::type>{};
struct is_error<mpl::na> { using type=self; struct apply<T=mpl::na> : is_error<T>{}; };
struct reject<Msg,Pos> { using tag=fail_tag; using type=self; using source_position=Pos; using message=Msg; };

// source position
struct get_col<T> : get_col_impl<T::type::tag>::apply<T::type>{};
struct get_line<T> : get_line_impl<T::type::tag>::apply<T::type>{};
struct get_prev_char<T> : get_prev_char_impl<T::type::tag>::apply<T::type>{};
struct next_char<P,Ch> : next_char_impl<mpl::tag<P::type>::type>::apply<P::type,Ch::type>{};
struct next_line<P,Ch> : next_line_impl<mpl::tag<P::type>::type>::apply<P::type,Ch::type>{};
struct source_position<Line,Col,PrevChar> { using tag=source_position_tag; using type=source_position; using line=Line; using col=Col; using prev_char=PrevChar; };
struct mpl::equal_to_impl<source_position_tag,source_position_tag>; // line, col, prev_char
struct mpl::not_equal_to_impl<source_position_tag,source_position_tag>;
struct mpl::less_impl<source_position_tag,source_position_tag>; // and less_equal_impl, greater_impl, greater_equal_impl,

using start = source_position<mpl::int_<1>, mpl::int_<1>, mpl::char_<0>>;
```

------
### String

```c++
struct impl::empty_string<Ignore=int> { using type=self; static constexpr char value[1]={0}; };
struct impl::size<string<cs...>> : int_<sizeof...(cs)>{};
struct impl::push_front_c<string<cs...>,c> : string<c,cs...>{};
struct impl::push_back_c<string<cs...>,c> : string<cs...,c>{};
struct impl::pop_front<string<c,cs...>> : string<cs...>{};
struct impl::pop_back<string<c>> : mpl::clear<string<c>>{};
struct impl::pop_back<string<c,cs...>> : push_front_c<pop_back<string<cs...>>::type,c>{};
constexpr T impl::string_at<maxLen,len,T>(const T (&s)[len], int n) { return n >= len-1 ? T{} : s[n]; }

struct string<...cs> { using type=self; using tag=string_tag; };

struct string_tag { using type=self; };
struct mpl::push_back_impl<string_tag>;
struct mpl::pop_back_impl<string_tag>;
struct mpl::push_front_impl<string_tag>;
struct mpl::pop_front_impl<string_tag>;
struct mpl::clear_impl<string_tag>;
struct mpl::begin_impl<string_tag>;
struct mpl::end_impl<string_tag>;
struct mpl::equal_to_impl<string_tag,string_tag>;
struct mpl::equal_to_impl<string_tag,T>;
struct mpl::equal_to_impl<T,string_tag>;
struct mpl::c_str<string<cs...>>;

#define STRING(...) string<__VA_ARGS__>
#define STRING_AT impl::string_at<LIMIT_STRING_SIZE>
#define STRING_VALUE(s) (STRING(s){})
```

------
### Errors

```c++
namespace error;
DEFINE_ERROR(digit_expected, "Digit expected");
DEFINE_ERROR(end_of_input_expected, "End of input expected");
DEFINE_ERROR(expected_to_fail, "Parser expected to fail");
struct index_out_of_range<from,to,n> { using type=self; static std::string get_value(); }; // "index (<n>) out of range [<from>-<to>]"
DEFINE_ERROR(letter_expected, "Letter expected");
struct literal_expected<c> { using type=self; static std::string get_value(); }; // "Expected: <c>"
DEFINE_ERROR(none_of_the_expected_cases_found, "None of the expected cases found");
DEFINE_ERROR(unexpected_character, "Unexpected character");
DEFINE_ERROR(unexpected_end_of_input, "Unexpected end of input");
struct unpaired<line,col,Msg=mpl::na> { using type=self; static std::string get_value(); }; // "<Msg::get_value()> (see <line>:<col>)"
struct unpaired<line,col,mpl::na> { using type=self; struct apply:self<line,col,Msg>{}; };
DEFINE_ERROR(whitespace_expected, "Whitespace expected");
```

------
### Tags

```c++
struct accept_tag { using type=self; };
struct get_position_impl<accept_tag> { struct apply<A> : A::source_position{}; };
struct get_remaining_impl<accept_tag> { struct apply<A> : A::remaining{}; };
struct get_result_impl<accept_tag> { struct apply<A> { using type=A::result; }; };

struct fail_tag { using type=self; };
struct get_message_impl<fail_tag> { struct apply<A> { using type=A::message; }; };
struct get_position_impl<fail_tag> { struct apply<A> : A::source_position{}; };

struct source_position_tag {using type=self;};
struct get_col_impl<source_position_tag> { struct apply<P> : P::col{}; };
struct get_line_impl<source_position_tag> { struct apply<P> : P::line{}; };
struct get_prev_char_impl<source_position_tag> { struct apply<P> : P::prev_char{}; };
struct next_char_impl<source_position_tag> { struct apply<P,Ch> : source_position<get_line<P>::type, mpl::int_<get_col<P>::type::value+1>,Ch>; };
struct next_line_impl<source_position_tag> { struct apply<P,Ch> : source_position<mpl::int_<get_line<P>::type::value+1>, mpl::int_<1>, Ch>; };
```

------
### Utilities

```c++
struct x__________________PARSING_FAILED__________________x<line,col,Msg> {}; // fail
struct parsing_failed<P,S> : x__________________PARSING_FAILED__________________x<
	get_line<get_position<P::apply<S,start>>>::type::value,
	get_col<get_position<P::apply<S,start>>>::type::value,
	get_message<P::apply<S,start>>::type>{};
struct build_parser<P> { using type=self;
	struct apply<S> : mpl::eval_if<is_error<P::aply<S,start>>::type, parsing_failed<P,S>, get_result<P::apply<S,start>>>{};
};
struct debug_parsing_error<P,S> {
private: struct display_error<Result> { static void run(); }; // "Parsing failed:\nline <line>, col <col>: <message>\n"
	struct display_no_error<Result> { static void run(); }; // "Parsing was successful. Remaining string is: <remaining>\n"
	struct display<Result> : mpl::if_<is_error<Result>::type, display_error<Result>, display_no_error<Result>>::type{};
public: using type=self;
	ctor() { using runner = display<P::apply<S,start>::type>;
		std::cout << "Compile-time parsing results\n-----------------------\nInput text:\n";
		std::cout << mpl::c_str<S>::type::value << "\n\n";
		runner::run();
		std::exit(0);
	}
};
class debug_parsing_error<build_parser<P>,S> : debug_parsing_error<P,S>{};
struct unless_error<T,NotErrorCase> : mpl::eval_if<is_error<T>::type, T,NotErrorCase>{};

#define DEFINE_ERROR(name,msg) struct { using type = name; static std::string get_value() { return msg; } }
#define VERSION BOOST_VERSION_NUMBER(1, 0, 0)

namespace util {
struct digit_to_int_c<c> : digit_expected{};
struct digit_to_int_c<'0'> : mpl::int_<0>{}; // up to '9'
struct digit_to_int<D=mpl::na> : digit_to_int_c<D::type::value>{};
struct digit_to_int<mpl::na> { using type=self; struct apply<D=mpl::na> : digit_to_int<D>{}; };
struct int_to_digit_c<n>;
struct int_to_digit_c<0> : mpl::char_<'0'>{}; // up to 9
struct int_to_digit<N=mpl::na> : int_to_digit_c<N::type::value>{};
struct int_to_digit<mpl::na> { using type=self; struct apply<N=mpl::na> : int_to_digit<N>{}; };
struct in_range<LB=mpl::na,UB=mpl::na,Item=mpl::na> : mpl::bool_<mpl::less_equal<LB,Item>::type::value && mpl::less_equal<Item,UB>::type::value>{};
struct in_range<LB,UB,mpl::na> { using type=self; struct apply<Item=mpl::na> : in_range<LB,UB,Item>{}; };
struct in_range<LB,mpl::na,mpl::na> { using type=self; struct apply<UB=mpl::na,Item=mpl::na> : in_range<LB,UB,Item>{}; };
struct in_range<mpl::na,mpl::na,mpl::na> { using type=self; struct apply<LB=mpl::na,UB=mpl::na,Item=mpl::na> : in_range<LB,UB,Item>{}; };
struct in_range_c<T,T lb, T ub> { using type=self; struct apply<Item> : mpl::bool_<(lb<=Item::type::value && Item::type::value<=ub)>{}; };
struct is_digit<C=mpl::na> : in_range_c<char,'0','9'>::apply<C>{};
struct is_digit<mpl::na> { using type=self; struct apply<C=mpl::na> : is_digit<C>{}; };
struct is_lcase_letter<C=mpl::na> : in_range_c<char,'a','z'>::apply<C>{};
struct is_lcase_letter<mpl::na> { using type=self; struct apply<C=mpl::na> : is_lcase_letter<C>{}; };
struct is_ucase_letter<C=mpl::na> : in_range_c<char,'A','Z'>::apply<C>{};
struct is_ucase_letter<mpl::na> { using type=self; struct apply<C=mpl::na> : is_ucase_letter<C>{}; };
struct is_letter<C=mpl::na> : mpl::bool_<(is_lcase_letter<C>::type::value || is_ucase_letter<C>::type::value)>{};
struct is_letter<mpl::na> { using type=self; struct apply<C=mpl::na> : is_letter<C>{}; };
struct is_whitespace_c<c> : mpl::false_{};
struct is_whitespace_c<' '> : mpl::true_{}; // and '\r', '\n', '\t'
struct is_whitespace<C=mpl::na> : is_whitespace_c<C::type::value>{};
struct is_whitespace<mpl::na> { using type=self; struct apply<C=mpl::na> : is_whitespace<C>{}; };
}

struct swap<F> { using type=self; struct apply<A,B> : F::apply<B,A>{}; };
```

------
### Terms

* boxed value
* currying
* lazy template metafunction
* nullary template metafunction
* parser
* parser combinator
* parsing error message
* predicate
* tag
* template metafunction
* template metafunction class
* template metaprogramming value

------
### Configuration

* `STD`: detected standard version.
* `LIMIT_ONE_CHAR_EXCEPT_SIZE`: default 10. (C++98 preprocessor-based config)
* `LIMIT_ONE_OF_SIZE`: defualt 20. (C++98 preprocessor-based config)
* `LIMIT_SEQUENCE_SIZE`: default 5.
* `LIMIT_STRING_SIZE`: default 32.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Config

* `<boost/mpl/**.hpp>`

#### Boost.Predef

* `<boost/predef/version_number.h>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_same.hpp>`

------
### Standard Facilities

Language: Rvalue-reference, move-semantic (C++11)
Library: `unique_ptr` (C++11)
