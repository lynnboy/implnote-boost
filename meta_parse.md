# Boost.MetaParse

* lib: `boost/libs/metaparse`
* repo: `boostorg/metaparse`
* commit: `0c7c2dd`, 2025-04-15

------
#### 

```c++
```

accept_<tag,when>, alphanum, always_<c>, repeated_<reject_incomplete|one_of><1>, build_parser,
change_error_message, config, debug_parsing_error, define_error, digit_<val>, empty, entire_input,
except, fail_<at_first_char_expected,tag>, first_of, fold{l,r}_<reject_incomplete>_<start_with_parser><1>,
get_{col,line,message,position,prev_char,remaining,result}, grammar, if_, int_, is_error, iterate_<c>,
keyword, last_of, letter, limit_{one_char_except_size,one_of_size,sequence_size,string_size}, lit_<c>,
look_ahead, middle_of, next_{char,line}, nth_of_<c>, one_char_<except>_<c>, one_of_<c>, optional,
range_<c>, reject, return_, sequence_<apply>, source_position_<tag>, space<s>, start, string_<tag>,
token, transform_<error>_<message>, unless_error, version

error/: {digit|end_of_input|letter|literal|whitespace}_expected, index_out_of_range,
	none_of_the_expected_cases_found, unexpected_{character|end_of_input}, unpaired
util/: digit_to_int_<c>, in_range_<c>, int_to_digit_<c>, is_digit, is_<lcase|ucase>_letter, is_whitespace_<c>

-------
#### Configuration

* `DISABLE_FORCEINLINE`, `FORCEINLINE_IS_BOOST_FORCELINE`

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
