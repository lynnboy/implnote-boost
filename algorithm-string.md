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

#### Predicates and Classification

```c++
```

detail/: classification, find_format_<all|store>, find_iterator, finder_<regex>, formatter_<regex>, predicate, replace_storage, sequence, trim, util
std/: list_traits, rope_traits, slist_traits, string_traits
classification, compare, concept, config, constants, erase, find_format, find_iterator, find, finder, formatter,
iter_find, join, predicate_facade, predicate, regex_find_format, regex, replace, sequence_traits, split, std_containers_traits, trim_all, trim, yes_no_type

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
