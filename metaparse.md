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
iterate
iterate_c
repeated
repeated1
repeated_reject_incomplete
repeated_reject_incomplete1
repeated_one_of
repeated_one_of1

// selection
if_
one_of
one_of_c
optional
repated_one_of
repated_one_of1

//sequence
first_of
last_of
middle_of
nth_of
nth_of_c
sequence
sequence_apply

// result transformation (semantic action)
always
always_c
transform
```

#### Miscellaneous

```c++
grammar
look_ahead
token
```

------
### Compile-time Data Structures and Values

#### Result of parsing

```c++
accept
get_message
get_position
get_remaining
get_result
is_error
reject
```

#### Source position

```c++
get_col
get_line
get_prev_char
next_char
next_line
source_position
source_position_tag
start
```

------
### String

```c++
string
string_tag
#define STRING
#define STRING_VALUE
```

------
### Errors

```c++
digit_expected
end_of_input_expected
expected_to_fail
index_out_of_range
letter_expected
literal_expected
none_of_the_expected_cases_found
unexpected_character
unexpected_end_of_input
unpaired
whitespace_expected
```

------
### Tags

```c++
accept_tag
fail_tag
source_position_tag
```

------
### Utilities

```c++
build_parser
debug_parsing_error
#define DEFINE_ERROR(name,msg) struct { using type = name; static std::string get_value() { return msg; } }
#define VERSION
unless_error

digit_to_int
digit_to_int_c
int_to_digit
int_to_digit_c
in_range
in_range_c
is_digit
is_lcase_letter
is_letter
is_ucase_letter
is_whitespace
is_whitespace_c
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

v1/:
accept_tag, always_<c>, build_parser, debug_parsing_error, fail_tag, first_of,
get_{col,line,message,position,prev_char,remaining,result}, grammar, if_, is_error, iterate_<c>,
last_of, look_ahead, middle_of, next_{char,line}, nth_of_<c>, one_of_<c>, optional,
reject, repeated_<reject_incomplete|one_of><1>, sequence_<apply>, source_position_<tag>,
start, string_<tag>, swap, token, unless_error

v1/error/: {digit|end_of_input|letter|literal|whitespace}_expected, index_out_of_range,
	none_of_the_expected_cases_found, unexpected_{character|end_of_input}, unpaired
v1/fwd/: accept, buid_parser, get{line,message,position,prev_char,remaining,result}, next_{char,line}, reject, source_position, string
v1/impl/: apply_parser, assert_string_length, at_c, {back,front}inserter, has_type, iterate_impl_<unchecked>, no_char, returns, string_iterator_<tag>
v1/util/: digit_to_int_<c>, in_range_<c>, int_to_digit_<c>, is_digit, is_<lcase|ucase>_letter, is_whitespace_<c>
v1/util/fwd/iterate_impl

v1/cpp11/: {first,last}_of, nth_of_<c>, one_of_<c>, repeated_one_of<1>, sequence, string
v1/cpp11/impl/: any_of_c, at_c, concat, empty_string, eval_later_result, nth_of_c_<skip_remaining>,
	or_c, pop_{back,front}, push_{back,front}_c, push_front_result, size, string_<at>
v1/cpp11/fwd/string

v1/cpp14/one_of_c, v1/cpp14/impl/any_of_c

accept_tag, always_<c>, build_parser, config, debug_parsing_error, fail_tag, first_of,
get_{col,line,message,position,prev_char,remaining,result}, grammar, if_, is_error, iterate_<c>,
last_of, limit_{one_char_except_size,one_of_size,sequence_size,string_size},
look_ahead, middle_of, next_{char,line}, nth_of_<c>, one_of_<c>, optional,
reject, repeated_<reject_incomplete|one_of><1>, sequence_<apply>, source_position_<tag>,
start, string_<tag>, token, unless_error, version

error/: {digit|end_of_input|letter|literal|whitespace}_expected, index_out_of_range,
	none_of_the_expected_cases_found, unexpected_{character|end_of_input}, unpaired
util/: digit_to_int_<c>, in_range_<c>, int_to_digit_<c>, is_digit, is_<lcase|ucase>_letter, is_whitespace_<c>


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
