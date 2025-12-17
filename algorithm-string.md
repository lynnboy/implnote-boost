# Boost.Algorithm/String

* lib: `boost/libs/algorithm`
* repo: `boostorg/algorithm`
* commit: `0edbfe8`, 2025-08-25

------
#### Case conversion

```c++
struct detail::to_lowerF<Ch>; // call std::tolower<Ch>() with wrapped std::locale
struct detail::to_upperF<Ch>; // call std::toupper<Ch>() with wrapped std::locale
OutIt detail::transform_range_copy<OutIt,R,F>(OutIt out, const R& in, F f) { return std::transform(begin(in),end(in),out,f); }
void detail::transform_range<R,F>(const R& in, F f) { std::transform(begin(in),end(in),begin(in),f); }
Seq detail::transform_range_copy<Seq,R,F>(const R& in, F f) { return {make_transform_iterator(begin(in),f),make_transform_iterator(end(in),f)}; }
OutIt to_lower_copy<OutIt,R>(OutIt out, const R& in, const std::locale& loc={})
{ return transform_range_copy(out, as_literal(in), to_lowerF<range_value<R>::type>{loc}); }
Seq to_lower_copy<Seq>(const Seq& in, const std::locale& loc={})
{ return transform_range_copy<Seq>(in, to_lowerF<range_value<Seq>::type>{loc}); }
void to_lower<WR>(WR& in, const std::locale& loc={})
{ transform_range(as_literal(in), to_lowerF<range_value<WR>::type>{loc}); }
OutIt to_upper_copy<OutIt,R>(OutIt out, const R& in, const std::locale& loc={})
{ return transform_range_copy(out, as_literal(in), to_upperF<range_value<R>::type>{loc}); }
Seq to_upper_copy<Seq>(const Seq& in, const std::locale& loc={})
{ return transform_range_copy<Seq>(in, to_upperF<range_value<Seq>::type>{loc}); }
void to_upper<WR>(WR& in, const std::locale& loc={})
{ transform_range(as_literal(in), to_upperF<range_value<WR>::type>{loc}); }
```

#### Compare & Predicates

```c++
struct is_equal; // arg1==arg2
struct is_iequal; // toupper(arg1,m_loc) == toupper(arg2,m_loc)
struct is_less; // arg1<arg2
struct is_iless; // toupper(arg1,m_loc) < toupper(arg2,m_loc)
struct is_not_greater; // arg1 <= arg2
struct is_not_igreater; // toupper(arg1,m_loc) <= toupper(arg2,m_loc)

bool detail::ends_with_iter_select<FwdIt1,FwdIt2,Pred>(FwdIt1 b, FwdIt1 e, FwdIt2 subb, FwdIt2 sube, Pred comp, std::bidirectional_iterator_tag); // loop
bool detail::ends_with_iter_select<FwdIt1,FwdIt2,Pred>(FwdIt1 b, FwdIt1 e, FwdIt2 subb, FwdIt2 sube, Pred comp, std::forward_iterator_tag); // call last_finder
bool starts_with<R1,R2,Pred>(const R1& in, const R2& test, Pred comp); // loop
bool starts_with<R1,R2>(const R1& in, const R2& test) { return starts_with(in, test, is_equal{}); }
bool istarts_with<R1,R2>(const R1& in, const R2& test, std::locale& loc={}) { return starts_with(in, test, is_iequal{loc}); }
bool contains<R1,R2,Pred>(const R1& in, const R2& test, Pred comp); // first_finder
bool contains<R1,R2>(const R1& in, const R2& test) { return contains(in, test, is_equal{}); }
bool icontains<R1,R2>(const R1& in, const R2& test, std::locale& loc={}) { return contains(in, test, is_iequal{loc}); }
bool equals<R1,R2,Pred>(const R1& in, const R2& test, Pred comp); // loop
bool equals<R1,R2>(const R1& in, const R2& test) { return equals(in, test, is_equal{}); }
bool iequals<R1,R2>(const R1& in, const R2& test, std::locale& loc={}) { return equals(in, test, is_iequal{loc}); }
bool lexicographical_compare<R1,R2,Pred>(const R1& in, const R2& test, Pred comp); // std::lexicographical_compare
bool lexicographical_compare<R1,R2>(const R1& in, const R2& test) { return lexicographical_compare(in, test, is_equal{}); }
bool ilexicographical_compare<R1,R2>(const R1& in, const R2& test, std::locale& loc={}) { return lexicographical_compare(in, test, is_iequal{loc}); }
bool all<R,Pred>(const R& in, Pred comp); // loop
```

#### Classification

```c++
struct predicate_facade<D>{};
struct detail::is_classifiedF : public predicate_facade<self> { /*...*/ }; // std::use_facet<std::ctype<Ch>>(m_loc).is(m_type, ch)
struct detail::is_any_ofF<Ch> : public predicate_facade<self> { /*...*/ }; // SBO, test ch is in array
struct detail::is_from_rangeF<Ch> : public predicate_facade<self> { /*...*/ }; // ch in range [m_from,m_to]
struct detail::pred_andF<Pred1,Pred2> : public predicate_facade<self> { /*...*/ }; // m_pred1(ch) && m_pred2(ch)
struct detail::pred_orF<Pred1,Pred2> : public predicate_facade<self> { /*...*/ }; // m_pred1(ch) || m_pred2(ch)
struct detail::pred_notF<Pred> : public predicate_facade<self> { /*...*/ }; // !m_pred(ch)
is_classifiedF is_classified(std::ctype_base::mask type, const std::locale& loc={}) { return {type,loc}; }
is_classifiedF is_space(const std::locale& loc={}) { return {space,loc}; }
is_classifiedF is_alnum(const std::locale& loc={}) { return {alnum,loc}; }
is_classifiedF is_alpha(const std::locale& loc={}) { return {alpha,loc}; }
is_classifiedF is_blank(const std::locale& loc={}) { return {blank,loc}; }
is_classifiedF is_cntrl(const std::locale& loc={}) { return {cntrl,loc}; }
is_classifiedF is_digit(const std::locale& loc={}) { return {digit,loc}; }
is_classifiedF is_graph(const std::locale& loc={}) { return {graph,loc}; }
is_classifiedF is_lower(const std::locale& loc={}) { return {lower,loc}; }
is_classifiedF is_print(const std::locale& loc={}) { return {print,loc}; }
is_classifiedF is_punct(const std::locale& loc={}) { return {punct,loc}; }
is_classifiedF is_upper(const std::locale& loc={}) { return {upper,loc}; }
is_classifiedF is_xdigit(const std::locale& loc={}) { return {xdigit,loc}; }
is_any_ofF is_any_of<R>(const R& set) { return {as_literal(set)}; }
is_from_rangeF is_from_rangeF<Ch>(Ch from Ch to) { return {from,to}; }
pred_andF operator&&<Pred1,Pred2>(const predicate_facade<Pred1>& pred1, const predicate_facade<Pred2>& pred2)
{ return {*(const Pred1*)&pred1, *(const red2*)&pred2}; }
pred_orF operator||<Pred1,Pred2>(const predicate_facade<Pred1>& pred1, const predicate_facade<Pred2>& pred2)
{ return {*(const Pred1*)&pred1, *(const red2*)&pred2}; }
pred_notF operator!<Pred>(const predicate_facade<Pred>& pred){ return {*(const Pred*)&pred}; }
```

#### Trimming

```c++
FwdIt detail::trim_end_iter_select<FwdIt,Pred>(FwdIt b, FwdIt e, Pred isSpace, std::forward_iterator_tag); // loop with 2
FwdIt detail::trim_end_iter_select<FwdIt,Pred>(FwdIt b, FwdIt e, Pred isSpace, std::bidirectional_iterator_tag); // loop
FwdIt detail::trim_begin<FwdIt,Pred>(FwdIt b, FwdIt e, Pred isSpace); // loop
FwdIt detail::trim_end<FwdIt,Pred>(FwdIt b, FwdIt e, Pred isSpace); // trim_end_iter_select
OutIt trim_left_copy_if<OutIt,R,Pred>(OutIt out, const R& in, Pred isSpace); // std::copy(trim_begin,end)
Seq trim_left_copy_if<Seq,Pred>(const Seq& in, Pred isSpace); // {trim_begin,end}
Seq trim_left_copy<Seq>(const Seq& in, const std::locale& loc={}) { trim_left_copy_if(in, is_space(loc)); }
void trim_left_if<Seq,Pred>(Seq& in, Pred isSpace); // in.erase(begin,trim_begin)
void trim_left<Seq>(Seq& in, const std::locale& loc={}); // trim_left_if(in,is_space(loc))
OutIt trim_right_copy_if<OutIt,R,Pred>(OutIt out, const R& in, Pred isSpace); // std::copy(begin,trim_end)
Seq trim_right_copy_if<Seq,Pred>(const Seq& in, Pred isSpace); // {begin,trim_end}
Seq trim_right_copy<Seq>(const Seq& in, const std::locale& loc={}) { trim_right_copy_if(in, is_space(loc)); }
void trim_right_if<Seq,Pred>(Seq& in, Pred isSpace); // in.erase(trim_end,end)
void trim_right<Seq>(Seq& in, const std::locale& loc={}); // trim_right_if(in,is_space(loc))
OutIt trim_copy_if<OutIt,R,Pred>(OutIt out, const R& in, Pred isSpace); // std::copy(trim_begin,trim_end)
Seq trim_copy_if<Seq,Pred>(const Seq& in, Pred isSpace); // {trim_begin,trim_end}
Seq trim_copy<Seq>(const Seq& in, const std::locale& loc={}) { trim_copy_if(in, is_space(loc)); }
void trim_if<Seq,Pred>(Seq& in, Pred isSpace); // trim_right_if, trim_left_if
void trim<Seq>(Seq& in, const std::locale& loc={}); // trim_if(in,is_space(loc))

Seq trim_all_copy_if<Seq,Pred>(const Seq& in, Pred isSpace)
{ return find_format_all_copy(trim_copy_if(in,isSpace), token_finder(isSpace, token_compress_on), dissect_formatter(head_finder(1))); }
Seq trim_all_copy<Seq>(const Seq& in, const std::locale& loc={}) { return trim_all_copy_if(in, is_space(loc)); }
void trim_all_if<Seq,Pred>(Seq& in, Pred isSpace)
{ trim_if(in,isSpace); find_format_all(in, token_finder(isSpace,token_compress_on), dissect_formatter(head_finder(1))); }
void trim_all<Seq>(Seq& in, const std::locale& loc={}) { trim_all_if(in, is_space(loc)); }
Seq trim_fill_copy_if<Seq,R,Pred>(const Seq& in, const R& fill, Pred isSpace)
{ return find_format_all_copy(trim_copy_if(in,isSpace), token_finder(isSpace,token_compress_on), const_formatter(as_literal(fill))); }
Seq trim_fill_copy<Seq,R>(const Seq& in, const R& fill, const std::locale& loc={}) { return trim_fill_copy_if(in, fill, is_space(loc)); }
void trim_fill_if<Seq,R,Pred>(Seq& in, const R& fill, Pred isSpace)
{ trim_if(in,isSpace); find_format_all(in, token_finder(isSpace, token_compress_on), const_formatter(as_literal(fill))); }
void trim_fill<Seq,R>(Seq& in, const R& fill, const std::locale& loc={}) { trim_fill_if(in, fill, is_space(loc)); }
```

#### Finder

```c++
enum token_compress_mode_type { token_compress_on, token_compress_off };
struct detail::first_finderF<It,Pred>;
struct detail::last_finderF<It,Pred>;
struct detail::nth_finderF<It,Pred>; // supports -n
struct detail::head_finderF;
struct detail::tail_finderF;
struct detail::token_finderF<Pred>; // concate adjacent tokens when token_compress_on
struct detail::range_finderF<FwdIt>; // noop, just return wrapped range

first_finderF<range_const_iterator<R>::type,is_equal> first_finder<R>(const R& s) { return {as_literal(s),{}}; }
first_finderF<range_const_iterator<R>::type,Pred> first_finder<R,Pred>(const R& s, Pred comp) { return {as_literal(s), comp}; }
last_finderF<range_const_iterator<R>::type,is_equal> last_finder<R>(const R& s) { return {as_literal(s),{}}; }
last_finderF<range_const_iterator<R>::type,Pred> last_finder<R,Pred>(const R& s, Pred comp) { return {as_literal(s), comp}; }
nth_finderF<range_const_iterator<R>::type,is_equal> nth_finder<R>(const R& s, int n) { return {as_literal(s),n,{}}; }
nth_finderF<range_const_iterator<R>::type,Pred> nth_finder<R,Pred>(const R& s, int n, Pred comp) { return {as_literal(s), n, comp}; }
head_finderF head_finder(int n) { return {n}; }
tail_finderF tail_finder(int n) { return {n}; }
token_finderF<Pred> token_finder(Pred pred, token_compress_mode_type m=token_compress_off) { return {pred,m}; }
range_finderF<FwdIt> range_finder<FwdIt>(FwdIt b, FwdIt e) { return {b,e}; }
range_finderF<FwdIt> range_finder<FwdIt>(iterator_range<FwdIt> r) { return {r}; }
```

#### Find

```c++
iterator_range<range_iterator<R>::type> find(R& in, const Finder& f)
{ auto litin = as_literal(in); return f(begin(litin),end(litin)); }
iterator_range<range_iterator<R1>::type> find_first<R1,R2>(R1& in, const R2& s)
{ return find(in,first_finder(s)); }
iterator_range<range_iterator<R1>::type> ifind_first<R1,R2>(R1& in, const R2& s, const std::locale& loc={})
{ return find(in,first_finder(s,is_iequal(loc))); }
iterator_range<range_iterator<R1>::type> find_last<R1,R2>(R1& in, const R2& s)
{ return find(in,last_finder(s)); }
iterator_range<range_iterator<R1>::type> ifind_last<R1,R2>(R1& in, const R2& s, const std::locale& loc={})
{ return find(in,last_finder(s,is_iequal(loc))); }
iterator_range<range_iterator<R1>::type> find_nth<R1,R2>(R1& in, const R2& s, int n)
{ return find(in,nth_finder(s,n)); }
iterator_range<range_iterator<R1>::type> ifind_nth<R1,R2>(R1& in, const R2& s, int n, const std::locale& loc={})
{ return find(in,nth_finder(s,n,is_iequal(loc))); }
iterator_range<range_iterator<R>::type> find_head<R>(R& in, int n) { return find(in,head_finder(n)); }
iterator_range<range_iterator<R>::type> find_tail<R>(R& in, int n) { return find(in,tail_finder(n)); }
iterator_range<range_iterator<R>::type> find_token<R,Pred>(R& in, Pred pred, token_compress_mode_type e=token_compress_off)
{ return find(in,token_finder(pred,e)); }
```

detail/: find_format_<all|store>, find_iterator, finder_regex, formatter_<regex>, replace_storage, sequence, util
std/: list_traits, rope_traits, slist_traits, string_traits
concept, config, erase, find_format, find_iterator, formatter, iter_find, join,
regex_find_format, regex, replace, sequence_traits, split, std_containers_traits, yes_no_type

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Function

* `<boost/function.hpp>`

#### Boost.Iterator

* `<boost/iterator/transform_iterator.hpp>`
* `<boost/iterator/iterator_facade.hpp>`
* `<boost/iterator/iterator_categories.hpp>`

#### Boost.MPL

* `<boost/mpl/bool.hpp>`
* `<boost/mpl/logical.hpp>`

#### Boost.Range

* `<boost/range/as_literal.hpp>`
* `<boost/range/begin.hpp>`, `<boost/range/end.hpp>`
* `<boost/range/distance.hpp>`
* `<boost/range/empty.hpp>`
* `<boost/range/iterator_range_core.hpp>`
* `<boost/range/iterator.hpp>`, `<boost/range/const_iterator.hpp>`
* `<boost/range/value_type.hpp>`

#### Boost.Regex

* `<boost/regex.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/make_unsigned.hpp>`
* `<boost/type_traits/remove_const.hpp>`

------
### Standard Facilities
