# Boost.Algorithm/String

* lib: `boost/libs/algorithm`
* repo: `boostorg/algorithm`
* commit: `0edbfe8`, 2025-08-25

------
#### Concept

```c++
concept Finder<F,It> = requires(iterator_range<It> r, It i, F* pF) { r=(*pF)(i,i); };
concept Formatter<Fmtr,Fndr,It> = requires(Fmtr* pFmtr, Fndr* pF, It i) { begin((*pFmtr)( (*pF)(i,i) )); end((*pFmtr)( (*pF)(i,i) )); };
```

#### Container Traits

```c++
struct size_descriptor<i> { using type = char(&)[i]; };
using yes_type = size_descriptor<1>::type; using no_type = size_descriptor<2>::type;
struct has_native_replace<T> : false_type{}
struct has_stable_iterators<T> : false_type{}
struct has_const_time_insert<T> : false_type{}
struct has_const_time_erase<T> : false_type{}
struct has_stable_iterators<std::list<T,A>> : true_type{}
struct has_const_time_insert<std::list<T,A>> : true_type{}
struct has_const_time_erase<std::list<T,A>> : true_type{}
struct has_native_replace<std::basic_string<Ch,Tr,A>> : true_type{}
// and slist, rope (not standardized)
```

#### Case Conversion

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

#### Formatter

```c++
struct detail::empty_container<Ch>;
OutIt detail::bounded_copy<InIt,OutIt>(InIt f, InIt l, OutIt df, OutIt dl);
struct detail::copy_iterator_rangeF<Seq,It=Seq::const_iterator>; // copy_range<Seq>(r)
struct detail::const_formatF<R>; // iterator_range<range_const_iterator<R>::type>
struct detail::identity_formatF<R>; // R{begin(r),end(r)}
struct detail::empty_formatF<Ch>; // empty_container<Ch>{}
struct detail::dissect_formatF<Finder>; // m_finder(begin(r),end(r))

const_formatF<iterator_range<range_const_iterator<R>::type>> const_formatter<R>(const R& format) { return {as_literal(format)}; }
identity_formatF<iterator_range<range_const_iterator<R>::type>> identity_formatter<R>() { return {}; }
empty_formatF<range_value<R>::type> empty_formatter<R>(const R&) { return {}; }
dissect_formatF<Finder> dissect_formatter<Finder>(const Finder& finder) { return {finder}; }
```

#### Find Iterator

```c++
class detail::find_iterator_base<It> {
protected: using match_type=iterator_range<It>; using finder_type = function<match_type, It, It>;
    match_type do_find(It b, It e) const { if (!m_finder.empty()) return m_finder(b,e); else return {e,e}; }
    bool is_null() const { return m_finder.empty(); }
private: finder_type m_finder;
};

class find_iterator<It> : public iterator_facade<self, const iterator_range<It>, forward_traversal_tag>, private find_iterator_base<It> {
    match_type m_match; It m_end;
    const match_type& dereference() const { return m_match; }
    void increment() { m_match=do_find(m_match.end(),m_end); }
    bool equal(const self& other) const { return eof() || other.eof() ? eof()==other.eof() : (m_match==other.m_match&&m_end==other.m_end); }
public: ctor<Finder>(It b, It e, Finder finder): base{finder,0}, m_match{b,b}, m_end{e}{increment();}
    ctor<Finder,R>(R& col, Finder finder): base{finder,0} // and other ctor
    { auto lit_col=as_literal(col); m_match=make_iterator_range(begin(lit_col),end(lit_col)); m_end=end(lit_col); increment();}
    bool eof() const { return is_null() || (m_match.begin()==m_end && m_match.end()==m_end); }
};
find_iterator<range_iterator<R>::type> make_find_iterator<R,Finder>(R& col, Finder finder) { return {col,finder}; }

class split_iterator<It> : public iterator_facade<self, const iterator_range<It>, forward_traversal_tag>, private find_iterator_base<It> {
    match_type m_match; It m_next, m_end; bool m_eof;
    const match_type& dereference() const { return m_match; }
    void increment() { auto m=do_find(m_next,m_end); if(m.begin()==m_end&&m.end()==m_end) if(m_match.end()==m_end) m_eof=true; m_match={m_next,m.begin()}; m_next=m.end(); }
    bool equal(const self& other) const { return eof() || other.eof() ? eof()==other.eof() : (m_match==other.m_match&&m_next==other.m_next&&m_end==other.m_end); }
public: ctor<Finder>(It b, It e, Finder finder): base{finder,0}, m_match{b,b}, m_next{b}, m_end{e}, m_eof{false}, {if (b!=e) increment();}
    ctor<Finder,R>(R& col, Finder finder): base{finder,0}, m_eof{false} // and other ctor
    { auto lit_col=as_literal(col); m_match=make_iterator_range(begin(lit_col),end(lit_col)); m_next=begin(lit_col); m_end=end(lit_col); if (m_next!=m_end) increment(); }
    bool eof() const { return is_null() || m_eof; }
};
split_iterator<range_iterator<R>::type> make_split_iterator<R,Finder>(R& col, Finder finder) { return {col,finder}; }
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

#### Find Format

```c++
struct detail::find_format_store<FwdIt,Formatter,FormatRes> : iterator_range<FwdIt> { format_result_type m_formatResult; const Formatter& m_formatter; /*...*/};
bool detail::check_find_result<In,FindRes>(In&, FindRes& findResult) { return !iterator_range<range_const_iterator<In>::type>{findResult}.empty(); }

void detail::insert<In,FwdIt>(In& in, In::iterator at, FwdIt b, FwdIt e) { in.insert(at, b, e); }
In::iterator detail::erase<In>(In& in, In::iterator from, In::iterator to) { return in.erase(from, to); }

struct detail::replace_const_time_helper<hasConstTimeOperations>; // optimized upon hasConstTimeOperations
struct detail::replace_native_helper<hasNative>; // call in.replace when hasNative, otherwise replace_const_time_helper
void detail::replace<In,FwdIt>(In& in, In::iterator from, In::iterator to, FwdIt b, FwdIt e)
{ replace_native_helper<has_native_replace<In>::value>{}(in, from, to, b, e); }
void detail::replace<In,R>(In& in, In::iterator from, In::iterator to, const R& insert)
{ if (from!=to) replace(in, from, to, begin(insert), end(insert)) else insert(in, from, begin(insert), end(insert)); }

OutIt detail::move_from_storage<St,OutIt>(St& storage, OutIt b, OutIt e)
{ OutIt out=b; while (!storage.empty()&&out!=e) {*out=storage.front(); storage.pop_front(); ++out;} return out; }
void detail::copy_to_storage<St,What>(St& storage, const What& what) { storage.insert(storage.end(),begin(what),end(what)); }
struct detail::process_segment_helper<hasStableIterators>;
FwdIt detail::process_segment<St,In,FwdIt>(St& storage, In& in, FwdIt insert, FwdIt b, FwdIt e)
{ return process_segment_helper<has_stable_iterators<In>::value>{}(storage, input, insert, b, e); }

OutIt detail::find_format_copy_impl2<OutIt,In,Formatter,FindRes,FormatRes>(OutIt out, const In& in, Formatter formatter, const FindRes& findResult, const FormatRes& formatResult)
{ find_format_store<range_const_iterator<In>::type, Formatter, FormatResult> M(findResult,formatResult,formatter);
  if (!M) { out=std::copy(begin(in),end(in),out); return out; }
  out = std::copy(begin(in),M.begin(),out); out=std::copy(begin(M.format_result()),end(M.format_result()),out); out=std::copy(M.end(),end(in),out); return out; }
OutIt detail::find_format_copy_impl<OutIt,In,Formatter,FindRes>(OutIt out, const In& in, Formatter formatter, const FindRes& findResult)
{ if (check_find_result(in,findResult)) return find_format_copy_impl2(out,in,formatter,findResult,formatter(findResult)); else return std::copy(begin(in),end(in),out); }
In detail::find_format_copy_impl2<In,Formatter,FindRes,FormatRes>(const In& in, Formatter formatter, const FindRes& findResult, const FormatRes& formatResult)
{ find_format_store<range_const_iterator<In>::type, Formatter, FormatResult> M(findResult,formatResult,formatter); if (!M) return {in};
  In out; insert(out,end(out),begin(in),M.begin()); insert(out,end(out),M.format_result()); insert(out,end(out),M.end(),end(in)); return out; }
In detail::find_format_copy_impl<In,Formatter,FindRes>(const In& in, Formatter formatter, const FindRes& findResult)
{ if (check_find_result(in,findResult)) return find_format_copy_impl2(in,formatter,findResult,formatter(findResult)); else return in; }
void detail::find_format_impl2<In,Formatter,FindRes,FormatRes>(In& in, Formatter formatter, const FindRes& findResult, const FormatRes& formatResult)
{ find_format_store<range_iterator<In>::type, Formatter, FormatResult> M(findResult,formatResult,formatter); if (!M) return;
  replace(in,M.begin(),M.end(),M.format_result()); }
void detail::find_format_impl<In,Formatter,FindRes>(In& in, Formatter formatter, const FindRes& findResult)
{ if (check_find_result(in,findResult)) return find_format_impl2(in,formatter,findResult,formatter(findResult)); }

OutIt detail::find_format_all_copy_impl2<OutIt,In,Finder,Formatter,FindRes,FormatRes>(OutIt out, const In& in, Finder finder, Formatter formatter, const FindRes& findResult, const FormatRes& formatResult)
{ find_format_store<range_const_iterator<In>::type, Formatter, FormatResult> M(findResult,formatResult,formatter); auto lastMatch=begin(in);
  while (M) { out=std::copy(lastMatch,M.begin(),out); out=std::copy(begin(M.format_result()),end(M.format_result()),out); lastMatch=M.end(); M=finder(lastMatch,end(in)); }
  out=std::copy(lastMatch,end(in),out); return out; }
OutIt detail::find_format_all_copy_impl<OutIt,In,Finder,Formatter,FindRes>(OutIt out, const In& in, Finder finder, Formatter formatter, const FindRes& findResult)
{ if (check_find_result(in,findResult)) return find_format_all_copy_impl2(out,in,finder,formatter,findResult,formatter(findResult)); else return std::copy(begin(in),end(in),out); }
In detail::find_format_all_copy_impl2<In,Finder,Formatter,FindRes,FormatRes>(const In& in, Finder finder, Formatter formatter, const FindRes& findResult, const FormatRes& formatResult)
{ find_format_store<range_const_iterator<In>::type, Formatter, FormatResult> M(findResult,formatResult,formatter); auto lastMatch=begin(in);
  In out; while(M) { insert(out,end(out),lastMatch,M.begin()); insert(out,end(out),M.format_result()); lastMatch=M.end(); M=finder(lastMatch,end(in)); }
  insert(out,end(out),lastMatch,end(in)); return out; }
In detail::find_format_all_copy_impl<In,Finder,Formatter,FindRes>(const In& in, Finder finder Formatter formatter, const FindRes& findResult)
{ if (check_find_result(in,findResult)) return find_format_all_copy_impl2(in,finder,formatter,findResult,formatter(findResult)); else return in; }
void detail::find_format_all_impl2<In,Finder,Formatter,FindRes,FormatRes>(In& in, Finder finder, Formatter formatter, const FindRes& findResult, const FormatRes& formatResult)
{ find_format_store<range_iterator<In>::type, Formatter, FormatResult> M(findResult,formatResult,formatter);
  std::deque<range_value<In>::type> storage; auto insertIt=begin(in), searchIt=begin(in);
  while (M) { insertIt = process_segment(storage,in,insertIt,searchIt,M.begin()); searchIt=M.end(); copy_to_storage(storage,M.format_result()); M=finder(searchIt,end(in)); }
  insertIt=process_segment(storage,in,insertIt,searchIt,end(in));
  if (storage.empty) erase(in,insertIt,end(in)); else insert(in,end(in),storage.begin(),storage.end()); }
void detail::find_format_all_impl<In,Finder,Formatter,FindRes>(In& in, Finder finder, Formatter formatter, const FindRes& findResult)
{ if (check_find_result(in,findResult)) return find_format_all_impl2(in,finder,formatter,findResult,formatter(findResult)); }

OutIt find_format_copy<OutIt,R,Finder,Formatter>(OutIt out, const R& in, Finder finder, Formatter formatter)
    requires(Finder<Finder,range_const_iterator<R>::type> && Formatter<Formatter<Formatter,Finder,range_const_iterator<R>::type>>)
{ auto lit_in = as_literal(in); return find_format_copy_impl(out,lit_in,formatter,finder(begin(lit_in),end(lit_in))); }
Seq find_format_copy<Seq,Finder,Formatter>(const Seq& in, Finder finder, Formatter formatter) requires(...)
{ return find_format_copy_impl(in,formatter,finder(begin(in),end(in))); }
void find_format<Seq,Finder,Formatter>(Seq& in, Finder finder, Formatter formatter) requires(...)
{ find_format_impl(input,formatter,finder(begin(in),end(in))); }

OutIt find_format_all_copy<OutIt,R,Finder,Formatter>(OutIt out, const R& in, Finder finder, Formatter formatter) requires(...)
{ auto lit_in = as_literal(in); return find_format_all_copy_impl(out,lit_in,finder,formatter,finder(begin(lit_in),end(lit_in))); }
Seq find_format_all_copy<Seq,Finder,Formatter>(const Seq& in, Finder finder, Formatter formatter) requires(...)
{ return find_format_all_copy_impl(in,finder,formatter,finder(begin(in),end(in))); }
void find_format_all<Seq,Finder,Formatter>(Seq& in, Finder finder, Formatter formatter) requires(...)
{ find_format_all_impl(input,finder,formatter,finder(begin(in),end(in))); }
```

#### Erase

```c++
OutIt erase_range_copy<OutIt,R>(OutIt out, const R& in, const iterator_range<range_const_iterator<R>::type>& search)
{ return find_format_copy(out,in, range_finder(search), empty_formatter(in)); }
Seq erase_range_copy<Seq>(const Seq& in, const iterator_range<range_const_iterator<Seq>::type>& search)
{ return find_format_copy(in, range_finder(search), empty_formatter(in)); }
void erase_range<Seq>(Seq& in, const iterator_range<range_iterator<Seq>::type>& search)
{ find_format(in, range_finder(search), empty_formatter(in)); }

OutIt erase_first_copy<OutIt,R1,R2>(OutIt out, const R1& in, const R2& search)
{ return find_format_copy(out, in, first_finder(search), empty_formatter(in)); }
Seq erase_first_copy<Seq,R>(const Seq& in, const R& search)
{ return find_format_copy(in, first_finder(search), empty_formatter(in)); }
void erase_first<Seq,R>(Seq& in, const R& search)
{ find_format(in, first_finder(search), empty_formatter(in)); }
OutIt ierase_first_copy<OutIt,R1,R2>(OutIt out, const R1& in, const R2& search, std::locale& loc={})
{ return find_format_copy(out, in, first_finder(search, is_iequal{loc}), empty_formatter(in)); }
Seq ierase_first_copy<Seq,R>(const Seq& in, const R& search, std::locale& loc={})
{ return find_format_copy(in, first_finder(search, is_iequal{loc}), empty_formatter(in)); }
void ierase_first<Seq,R>(Seq& in, const R& search, std::locale& loc={})
{ find_format(in, first_finder(search, is_iequal{loc}), empty_formatter(in)); }

OutIt erase_last_copy<OutIt,R1,R2>(OutIt out, const R1& in, const R2& search)
{ return find_format_copy(out, in, last_finder(search), empty_formatter(in)); }
Seq erase_last_copy<Seq,R>(const Seq& in, const R& search)
{ return find_format_copy(in, last_finder(search), empty_formatter(in)); }
void erase_last<Seq,R>(Seq& in, const R& search)
{ find_format(in, last_finder(search), empty_formatter(in)); }
OutIt ierase_last_copy<OutIt,R1,R2>(OutIt out, const R1& in, const R2& search, std::locale& loc={})
{ return find_format_copy(out, in, last_finder(search, is_iequal{loc}), empty_formatter(in)); }
Seq ierase_last_copy<Seq,R>(const Seq& in, const R& search, std::locale& loc={})
{ return find_format_copy(in, last_finder(search, is_iequal{loc}), empty_formatter(in)); }
void ierase_last<Seq,R>(Seq& in, const R& search, std::locale& loc={})
{ find_format(in, last_finder(search, is_iequal{loc}), empty_formatter(in)); }

OutIt erase_nth_copy<OutIt,R1,R2>(OutIt out, const R1& in, const R2& search, int n)
{ return find_format_copy(out, in, nth_finder(search,n), empty_formatter(in)); }
Seq erase_nth_copy<Seq,R>(const Seq& in, const R& search, int n)
{ return find_format_copy(in, nth_finder(search,n), empty_formatter(in)); }
void erase_nth<Seq,R>(Seq& in, const R& search, int n)
{ find_format(in, nth_finder(search,n), empty_formatter(in)); }
OutIt ierase_nth_copy<OutIt,R1,R2>(OutIt out, const R1& in, const R2& search, int n, std::locale& loc={})
{ return find_format_copy(out, in, nth_finder(search,n, is_iequal{loc}), empty_formatter(in)); }
Seq ierase_nth_copy<Seq,R>(const Seq& in, const R& search, int n, std::locale& loc={})
{ return find_format_copy(in, nth_finder(search,n, is_iequal{loc}), empty_formatter(in)); }
void ierase_nth<Seq,R>(Seq& in, const R& search, int n, std::locale& loc={})
{ find_format(in, nth_finder(search,n, is_iequal{loc}), empty_formatter(in)); }

OutIt erase_all_copy<OutIt,R1,R2>(OutIt out, const R1& in, const R2& search)
{ return find_format_all_copy(out, in, first_finder(search), empty_formatter(in)); }
Seq erase_all_copy<Seq,R>(const Seq& in, const R& search)
{ return find_format_all_copy(in, first_finder(search), empty_formatter(in)); }
void erase_all<Seq,R>(Seq& in, const R& search)
{ find_format_all(in, first_finder(search), empty_formatter(in)); }
OutIt ierase_all_copy<OutIt,R1,R2>(OutIt out, const R1& in, const R2& search, std::locale& loc={})
{ return find_format_all_copy(out, in, first_finder(search, is_iequal{loc}), empty_formatter(in)); }
Seq ierase_all_copy<Seq,R>(const Seq& in, const R& search, std::locale& loc={})
{ return find_format_all_copy(in, first_finder(search, is_iequal{loc}), empty_formatter(in)); }
void ierase_all<Seq,R>(Seq& in, const R& search, std::locale& loc={})
{ find_format_all(in, first_finder(search, is_iequal{loc}), empty_formatter(in)); }

OutIt erase_head_copy<OutIt,R>(OutIt out, const R& in, int n) { return find_format_copy(out,in,head_finder(n), empty_formatter(in)); }
Seq erase_head_copy<Seq>(const Seq& in, int n) { return find_format_copy(in,head_finder(n), empty_formatter(in)); }
void erase_head<Seq>(Seq& in, int n) { find_format(in,head_finder(n), empty_formatter(in)); }
OutIt erase_tail_copy<OutIt,R>(OutIt out, const R& in, int n) { return find_format_copy(out,in,tail_finder(n), empty_formatter(in)); }
Seq erase_tail_copy<Seq>(const Seq& in, int n) { return find_format_copy(in,tail_finder(n), empty_formatter(in)); }
void erase_tail<Seq>(Seq& in, int n) { find_format(in,tail_finder(n), empty_formatter(in)); }
```

#### Replace

```c++
OutIt replace_range_copy<OutIt,R1,R2>(OutIt out, const R1& in, const iterator_range<range_const_iterator<R1>::type>& search, const R2& format)
{ return find_format_copy(out,in, range_finder(search), const_formatter(format)); }
Seq replace_range_copy<Seq,R>(const Seq& in, const iterator_range<range_const_iterator<Seq>::type>& search, const R& format)
{ return find_format_copy(in, range_finder(search), const_formatter(format)); }
void replace_range<Seq,R>(Seq& in, const iterator_range<range_iterator<Seq>::type>& search, const R& format)
{ find_format(in, range_finder(search), const_formatter(format)); }

OutIt replace_first_copy<OutIt,R1,R2,R3>(OutIt out, const R1& in, const R2& search, const R3& format)
{ return find_format_copy(out, in, first_finder(search), const_formatter(format)); }
Seq replace_first_copy<Seq,R1,R2>(const Seq& in, const R1& search, const R2& format)
{ return find_format_copy(in, first_finder(search), const_formatter(format)); }
void replace_first<Seq,R1,R2>(Seq& in, const R1& search, const R2& format)
{ find_format(in, first_finder(search), const_formatter(format)); }
OutIt ireplace_first_copy<OutIt,R1,R2,R3>(OutIt out, const R1& in, const R2& search, const R3& format, std::locale& loc={})
{ return find_format_copy(out, in, first_finder(search, is_iequal{loc}), const_formatter(format)); }
Seq ireplace_first_copy<Seq,R1,R2>(const Seq& in, const R1& search, const R2& format, std::locale& loc={})
{ return find_format_copy(in, first_finder(search, is_iequal{loc}), const_formatter(format)); }
void ireplace_first<Seq,R1,R2>(Seq& in, const R1& search, const R2& format, std::locale& loc={})
{ find_format(in, first_finder(search, is_iequal{loc}), const_formatter(format)); }

OutIt replace_last_copy<OutIt,R1,R2,R3>(OutIt out, const R1& in, const R2& search, const R3& format)
{ return find_format_copy(out, in, last_finder(search), const_formatter(format)); }
Seq replace_last_copy<Seq,R1,R2>(const Seq& in, const R1& search, const R2& format)
{ return find_format_copy(in, last_finder(search), const_formatter(format)); }
void replace_last<Seq,R1,R2>(Seq& in, const R1& search, const R3& format)
{ find_format(in, last_finder(search), const_formatter(format)); }
OutIt ireplace_last_copy<OutIt,R1,R2,R3>(OutIt out, const R1& in, const R2& search, const R3& format, std::locale& loc={})
{ return find_format_copy(out, in, last_finder(search, is_iequal{loc}), const_formatter(format)); }
Seq ireplace_last_copy<Seq,R1,R2>(const Seq& in, const R1& search, const R2& format, std::locale& loc={})
{ return find_format_copy(in, last_finder(search, is_iequal{loc}), const_formatter(format)); }
void ireplace_last<Seq,R1,R2>(Seq& in, const R1& search, const R2& format, std::locale& loc={})
{ find_format(in, last_finder(search, is_iequal{loc}), const_formatter(format)); }

OutIt replace_nth_copy<OutIt,R1,R2,R3>(OutIt out, const R1& in, const R2& search, const R3& format, int n)
{ return find_format_copy(out, in, nth_finder(search,n), const_formatter(format)); }
Seq replace_nth_copy<Seq,R1,R2>(const Seq& in, const R1& search, const R2& format, int n)
{ return find_format_copy(in, nth_finder(search,n), const_formatter(format)); }
void replace_nth<Seq,R1,R2>(Seq& in, const R1& search, const R2& format, int n)
{ find_format(in, nth_finder(search,n), const_formatter(format)); }
OutIt ireplace_nth_copy<OutIt,R1,R2,R3>(OutIt out, const R1& in, const R2& search, const R3& format, int n, std::locale& loc={})
{ return find_format_copy(out, in, nth_finder(search,n, is_iequal{loc}), const_formatter(format)); }
Seq ireplace_nth_copy<Seq,R1,R2>(const Seq& in, const R1& search, const R2& format, int n, std::locale& loc={})
{ return find_format_copy(in, nth_finder(search,n, is_iequal{loc}), const_formatter(format)); }
void ireplace_nth<Seq,R1,R2>(Seq& in, const R1& search, const R2& format, int n, std::locale& loc={})
{ find_format(in, nth_finder(search,n, is_iequal{loc}), const_formatter(format)); }

OutIt replace_all_copy<OutIt,R1,R2,R3>(OutIt out, const R1& in, const R2& search, const R3& format)
{ return find_format_all_copy(out, in, first_finder(search), const_formatter(format)); }
Seq replace_all_copy<Seq,R1,R2>(const Seq& in, const R1& search, const R2& format)
{ return find_format_all_copy(in, first_finder(search), const_formatter(format)); }
void replace_all<Seq,R1,R2>(Seq& in, const R1& search, const R2& format)
{ find_format_all(in, first_finder(search), const_formatter(format)); }
OutIt ireplace_all_copy<OutIt,R1,R2,R3>(OutIt out, const R1& in, const R2& search, const R3& format, std::locale& loc={})
{ return find_format_all_copy(out, in, first_finder(search, is_iequal{loc}), const_formatter(format)); }
Seq ireplace_all_copy<Seq,R1,R2>(const Seq& in, const R1& search, const R2& format, std::locale& loc={})
{ return find_format_all_copy(in, first_finder(search, is_iequal{loc}), const_formatter(format)); }
void ireplace_all<Seq,R1,R2>(Seq& in, const R1& search, const R2& format, std::locale& loc={})
{ find_format_all(in, first_finder(search, is_iequal{loc}), const_formatter(format)); }

OutIt replace_head_copy<OutIt,R1,R2>(OutIt out, const R1& in, int n, const R2& format) { return find_format_copy(out,in,head_finder(n), const_formatter(format)); }
Seq replace_head_copy<Seq,R>(const Seq& in, int n, const R& format) { return find_format_copy(in,head_finder(n), const_formatter(format)); }
void replace_head<Seq,R>(Seq& in, int n, const R& format) { find_format(in,head_finder(n), const_formatter(format)); }
OutIt replace_tail_copy<OutIt,R1,R2>(OutIt out, const R1& in, int n, const R2& format) { return find_format_copy(out,in,tail_finder(n), const_formatter(format)); }
Seq replace_tail_copy<Seq,R>(const Seq& in, int n, const R& format) { return find_format_copy(in,tail_finder(n), const_formatter(format)); }
void replace_tail<Seq,R>(Seq& in, int n, const R& format) { find_format(in,tail_finder(n), const_formatter(format)); }
```

#### Split

```c++
Seq& iter_find<Seq,R,Finder>(Seq& result, R&& in, Finder finder) requires(...)
{ auto lit_in=as_literal(in); auto e=end(lit_in);
  using FindIt = find_iterator<range_iterator<R>::type>; using CopyR = copy_iterator_rangeF<range_value<Seq>::type,range_iterator<R>::type>;
  auto itb = make_transform_iterator(FindIt{begin(lit_in),e,finder},CopyR{}), ite=make_transform_iterator(FindIt{},CopyR{});
  result.swap(Seq{itb,ite}); return result; }
Seq& iter_split<Seq,R,Finder>(Seq& result, R&& in, Finder finder) requires(...)
{ auto lit_in=as_literal(in); auto e=end(lit_in);
  using FindIt = split_iterator<range_iterator<R>::type>; using CopyR = copy_iterator_rangeF<range_value<Seq>::type,range_iterator<R>::type>;
  auto itb = make_transform_iterator(FindIt{begin(lit_in),e,finder},CopyR{}), ite=make_transform_iterator(FindIt{},CopyR{});
  result.swap(Seq{itb,ite}); return result; }

Seq& find_all<Seq,R1,R2>(Seq& result, R1&& in, const R2& search)
{ return iter_find(result,in,first_finder(search)); }
Seq& ifind_all<Seq,R1,R2>(Seq& result, R1&& in, const R2& search, const std::locale& loc={})
{ return iter_find(result,in,first_finder(search,is_iequal(loc))); }
Seq& split<Seq,R,Pred>(Seq& result, R&& in, Pred pred, token_compress_mode_type e=token_compress_off)
{ return iter_split(result,in,token_finder(pred,e)); }
```

#### Join

```c++
range_value<Seq>::type join<Seq,R>(const Seq& in, const R& sep)
{ auto b=begin(in), e=end(in); range_value<Seq>::type result;
  if (b!=e) { insert(result, end(result), *b); ++b; }
  for (; b!=e; ++b) { insert(result,end(result), as_literal(sep)); insert(result, end(result), *b); }
  return result; }
range_value<Seq>::type join_if<Seq,R,Pred>(const Seq& in, const R& sep, Pred pred)
{ auto b=begin(in), e=end(in); range_value<Seq>::type result;
  while (b!=e&&!pred(*b)) ++b; if (b!=e) { insert(result, end(result), *b); ++b; }
  for (; b!=e; ++b) { if (pred(*b)) { insert(result,end(result), as_literal(sep)); insert(result, end(result), *b); } }
  return result; }
```

#### Regex

```c++
struct detail::regex_search_result<It> : public iterator_range<It>; // wrap match_results
struct detail::find_regexF<Regex>; // regex_search(b,e,res, m_regex,m_matchflags)
struct detail::regex_formatF<Str>; // search_res.match_results().format(m_fmt, m_flags)

find_regexF<basic_regex<Ch,Tr>> regex_finder<Ch,Tr>(const basic_regex<Ch,Tr>& re, match_flag_type mf=match_default) { return {re,mf}; }
regex_formatF<std::basic_string<Ch,Tr,A>> regex_formatter<Ch,Tr,A>(const std::basic_string<Ch,Tr,A>& format, match_flag_type mf=format_default) { return {format,mf}; }

iterator_range<range_iterator<R>::type> find_regex<R,Ch,Tr>(R& in, const basic_regex<Ch,Tr>& re, match_flag_type mf=match_default)
{ auto lit_in=as_literal(in); return regex_finder(re,mf)(begin(lit_in),end(lit_in)); }

OutIt erase_regex_copy<OutIt,R,Ch,Tr>(OutIt out, const R& in, const basic_regex<Ch,Tr>& re, match_flag_type mf=match_default)
{ return find_format_copy(out,in,regex_finder(re,mf),empty_formatter(in)); }
Seq erase_regex_copy<Seq,Ch,Tr>(const Seq& in, const basic_regex<Ch,Tr>& re, match_flag_type mf=match_default)
{ return find_format_copy(in,regex_finder(re,mf),empty_formatter(in)); }
void erase_regex<Seq,Ch,Tr>(Seq& in, const basic_regex<Ch,Tr>& re, match_flag_type mf=match_default)
{ find_format(in,regex_finder(re,mf),empty_formatter(in)); }

OutIt erase_all_regex_copy<OutIt,R,Ch,Tr>(OutIt out, const R& in, const basic_regex<Ch,Tr>& re, match_flag_type mf=match_default)
{ return find_format_all_copy(out,in,regex_finder(re,mf),empty_formatter(in)); }
Seq erase_all_regex_copy<Seq,Ch,Tr>(const Seq& in, const basic_regex<Ch,Tr>& re, match_flag_type mf=match_default)
{ return find_format_all_copy(in,regex_finder(re,mf),empty_formatter(in)); }
void erase_all_regex<Seq,Ch,Tr>(Seq& in, const basic_regex<Ch,Tr>& re, match_flag_type mf=match_default)
{ find_format_all(in,regex_finder(re,mf),empty_formatter(in)); }

OutIt replace_regex_copy<OutIt,R,Ch,RTr,STr,StrA>(OutIt out, const R& in,
    const basic_regex<Ch,RTr>& re, const std::basic_string<Ch,STr,A>& format, match_flag_type mf=match_default|format_default)
{ return find_format_copy(out,in,regex_finder(re,mf),regex_formatter(format,mf)); }
Seq replace_regex_copy<Seq,Ch,RTr,STr,StrA>(const Seq& in,
    const basic_regex<Ch,RTr>& re, const std::basic_string<Ch,STr,A>& format, match_flag_type mf=match_default|format_default)
{ return find_format_copy(in,regex_finder(re,mf),regex_formatter(format,mf)); }
void replace_regex<Seq,Ch,RTr,STr,StrA>(Seq& in,
    const basic_regex<Ch,RTr>& re, const std::basic_string<Ch,STr,A>& format, match_flag_type mf=match_default|format_default)
{ find_format(in,regex_finder(re,mf),regex_formatter(format,mf)); }

OutIt replace_all_regex_copy<OutIt,R,Ch,RTr,STr,StrA>(OutIt out, const R& in,
    const basic_regex<Ch,RTr>& re, const std::basic_string<Ch,STr,A>& format, match_flag_type mf=match_default|format_default)
{ return find_format_all_copy(out,in,regex_finder(re,mf),regex_formatter(format,mf)); }
Seq replace_all_regex_copy<Seq,Ch,RTr,STr,StrA>(const Seq& in,
    const basic_regex<Ch,RTr>& re, const std::basic_string<Ch,STr,A>& format, match_flag_type mf=match_default|format_default)
{ return find_format_all_copy(in,regex_finder(re,mf),regex_formatter(format,mf)); }
void replace_all_regex<Seq,Ch,RTr,STr,StrA>(Seq& in,
    const basic_regex<Ch,RTr>& re, const std::basic_string<Ch,STr,A>& format, match_flag_type mf=match_default|format_default)
{ find_format_all(in,regex_finder(re,mf),regex_formatter(format,mf)); }

Seq& find_all_regex<Seq,R,Ch,Tr>(Seq& res, const R& in, const basic_regex<Ch,Tr>& re, match_flag_type mf=match_default)
{ return iter_find(result,in,regex_finder(re,mf)); }
Seq& split_regex<Seq,R,Ch,Tr>(Seq& res, const R& in, const basic_regex<Ch,Tr>& re, match_flag_type mf=match_default)
{ return iter_split(result,in,regex_finder(re,mf)); }
range_value<Seq>::type join_if<Seq,R,Ch,Tr>(const Seq& in, const R& sep, const basic_regex<Ch,Tr>& re, match_flag_type mf=match_default)
{ auto b=begin(in), e=end(in); range_value<Seq>::type result;
  while (b!=e && !regex_match(begin(*b),end(*b),re,mf)) ++b;
  if (b!=e) { insert(result, end(result),*b); ++b; }
  for (; b!=e; ++b) { if (regex_match(begin(*b), end(*b), re, mf)) {insert(result,end(result),as_literal(sep)); insert(result,end(result),*b); } }
  return result; }
```

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

Library:
* `starts_with`, `ends_with` (C++20); `contains` (C++23),
* `ranges::starts_with`, `ranges::ends_with`, `ranges::contains` (C++23),
* `regex_search`, `regex_replace` (C++23),
