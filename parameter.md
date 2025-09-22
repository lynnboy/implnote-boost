# Boost.Parameter

* lib: `boost/libs/parameter`
* repo: `boostorg/parameter`
* commit: `2592296`, 2025-08-26

------
### Parameter

Header `<boost/parameter.hpp>`

------
#### Facilities

Decide `BOOST_PARAMETER_HAS_PERFECT_FORWARDING` and `BOOST_PARAMETER_CAN_USE_MP11` via config.

```c++
struct void_{};
void_& aux::void_reference(); // placeholder for (void)
char aux::yes_tag; char aux::no_tag[2];
yes_tag|no_tag aux::to_yesno(mpl::true_|false_);
yes_tag|no_tag aux::to_yesno(mp11::mp_true|mp_false);

using is_cv_reference_wrapper<T> = ; // reference_wrapper<T> const volatile or its reference
using unwrap_cv_reference<T> = is_cv_reference_wrapper<T> ? T::type : remove_reference_t<T>;

struct aux::template_keyword_base {};
using aux::is_template_keyword<T> = is_base_of<template_keyword_base, remove_const_t<remove_reference_T<T>>>;

struct aux::use_default_tag {};
struct aux::use_default{};

struct aux::default_<KW, V> { V& value; }; // wrapper for a lref to a value
struct aux::lazy_default<KW, DefComputer> { DefComputer const& compute_default; }; // wrapper for a func
struct aux::default_r_<KW, V> { V&& value; }; // wrapper for a rref to a value

using aux::result_of0<F>::type = result_of_t<F()> != void ? result_of_t<F()> : use_default_tag;

struct aux::name_tag<Tag>{};
struct aux::name_tag_base{};
struct aux::is_name_tag<T> : false_{}; // primary
struct aux::lambda_tag;
struct mpl::lambda<T,enable_if<is_name_tag<T>,lambda_tag>>{type;}; // MPL lambda support for name tags

struct aux::tagged_argument_base {};
using aux::is_tagged_argument<T> = is_base_of<tagged_argument_base, remove_const_t<remove_reference_t<T>>>;

using aux::has_nested_template_fn<T> = ...; // find T::fn (for MP11 usage)
using aux::is_mpl_placeholder<T> = ...; // detect if T is mpl::arg<n>
using aux::is_mp11_placeholder<T> = ...; // detect if T is mp_arg<n>

struct aux::tagged_argument<Keyword, Arg> : tagged_argument_base
  requires !(!is_scalar<Arg> && Keyword::qualifier == consume_reference) &&
    !(!is_const<Arg> && Key::qualifier == out_reference)
{
  using arg_type = remove_const_t<Arg>;

  using key_type = Keyword;
  using value_type = is_function<arg_type> ? std::function<arg_type> : Arg;
  using reference = is_function<arg_type> ? value_type const& : Arg&;

  (is_function<arg_type> ? value_type : reference) value;  // stored argument value
    
  explicit ctor(reference x) : value(x) {} // + copy-ctor

  struct binding {
    using apply<KW,Default,Ref> = KW == key_type ? (Ref ? reference : value_type) : Default; // MPL
    using fn<KW,Default,Ref> = KW == key_type ? (Ref ? reference : value_type) : Default; // MP11
  };

  // value fetching
  reference get_value() const { return value; }
  reference operator[](keyword<KW> const&) const { return get_value(); }
  reference operator[]<Def>(default_<KW, Def> const&) const { return get_value(); }
  reference operator[]<Def>(default_r_<KW, Def> const&) const { return get_value(); }
  reference operator[]<F>(lazy_default<KW, F> const&) const { return get_value(); }
  Def& operator[]<KW2,Def>(default_<KW2, Def> const& x) const { return x.value; }
  Def&& operator[]<KW2,Def>(default_r_<KW2, Def> const& x) const { return forward<Def>(x.value); }
  result_of_t<F()> operator[]<KW2,F>(lazy_default<KW2, F> const& x) const { return x.compute_default(); }

  ParameterRequirements::has_default satisfies(ParameterRequirements*);
  Pred<value_type> satisfies<HasDef, Pred>(parameter_requirements<key_type,Pred,HasDef>*);
  using type = tagged_argument; using tail_type = empty_arg_list; using tag = arg_list_tag;
};

struct aux::tagged_argument_rref<Keyword, Arg> : tagged_argument_base
  requires (Keyword::qualifier = out_reference)
{
  using key_type = Keyword; using value_type = Arg; using reference = Arg&&; // rvalue ref

  reference value; // stored reference

  explicit ctor(reference x) : value(forward<Arg>(x)) {} // + copy-ctor

  struct binding; // same as tagged_argument
  reference get_value() const { return forward<Arg>(value); }
  // all operator[], same as tagged_argument
  // satisfies, same as tagged_argument
  using type = tagged_argument_rref; using tail_type = empty_arg_list; using tag = arg_list_tag;
};

struct aux::tagged_argument_list_of_1<TaggedArg> : TaggedArg {
  using base_type = TaggedArg;
  explicit ctor(reference x); // + copy-ctor
  auto operator, <TA2> (TA2 const& x) const ->
    flat_like_arg_list<flat_like_arg_tuple<key_type,base_type>, // tuple for self
                       flat_like_arg_tuple<TA2::key_type,TA2::base_type>>; // tuple for TA2
};

struct aux::tag<Keyword, ActualArg> {
  using Arg = unwrap_cv_reference<ActualArg>; using ConstArg = const Arg; using MutArg = remove_const_t<Arg>;
  using type = (is_lvalue_reference<ActualArg> || is_cv_reference_wrapper<ActualArg>) ?
    tagged_argument_list_of_1<tagged_argument<Keyword, Arg>>
    : is_scalar<MutArg> ? tagged_argument_list_of_1<tagged_argument<Keyword, ConstArg>>
                        : tagged_argument_list_of_1<tagged_argument_rref<Keyword, Arg>>;
};

struct aux::parameter_requirements<KW, Pred, HasDef> { using keyword = KW; using predicate = Pred; using has_default = HasDef; }

struct aux::maybe_base {};
using aux::is_maybe<T> = is_base_and_derived<maybe_base, remove_const_t<T>>;
struct aux::maybe<T> : maybe_base {
  using reference = T const &;
  using non_cv_value = remove_cv_t<remove_reference_t<reference>>;

  optional<T> value;
  mutable bool constructed = false;
  aligned_storage<sizeof<non_cv_value>> m_storage;

  explicit maybe(T v) : value(v) {}
  maybe() {} // require construct()
  ~maybe() { if (constructed) destroy(); }
  reference construct(reference v) const { return v; }
  reference construct<U>(U const& v) const {
    new (m_storage.address()) non_cv_value(v); constructed = true;
    return *(non_cv_value*)m_storage.address();
  }
  void destroy() { ((non_cv_value*)m_storage.address())->~non_cv_value(); }
  explicit operator bool() const { return value.has_value(); }
  reference get() const { return value.get(); }
}

struct aux::empty_arg_list { // constexpr
  using struct tagged_arg::value_type = void_;
  using type = empty_arg_list; using tag = arg_list_tag;
  struct binding {
    using apply<KW,Def,Ref> = Def; using fn<KW,Def,Ref> = Def; // not matching anything
  };
  empty_arg_list<...Args>(Args&&...); // no-op, match-all ctor

  Def& operator[] <K, Def> (default_<K, Def> x) const { return x.value; }
  Def& operator[] <K, Def> (default_r_<K, Def> x) const { return forward<Def>(x.value); }
  result_of_t<F()> operator[] <K, F> (lazy_default<K,F> x) const { return x.compute_default(); }
  static no_tag has_key<KW>(KW*); 
  static ParReq::has_default satisfies<ParReq,ArgPack>(ParReq*, ArgPack*);
};
struct aux::arg_list<TaggedArg, Next=empty_arg_list, EmitErrors=true> : public Next {// constexpr
  using tagged_arg = TaggedArg; using self = arg_list; using key_type = TaggedArg::key_type;
  using reference = is_maybe<TaggedArg::value_type> ? TaggedArg::value_type::reference : TaggedArg::reference;
  using value_type = is_maybe<TaggedArg::value_type> ? reference : TaggedArg::value_type;
  struct binding {
    using apply<KW,Def,Ref> = is_same<KW, key_type> ? // recursive lookup for match type
        (Ref ? reference : value_type) : Next::binding::apply<KW,Def,Ref>; using fn = apply;
  };

  TaggedArg arg; // storage of arg

  arg_list<A...>(A0&& a0, A&&...args)
  if constexpr (value_type != void)
    : arg(forward<A0>(a0)), Next(forward<A>(args)...)
  else
    : arg(void_reference()), Next(forward<A0>(a0), forward<A>(args)...)
  {}
  reference operator[] (keyword<key_type> const&) const { return arg.get_value(); } // not maybe
  reference operator[] <Def> (default_<key_type, Def> const& d) const {
    return is_maybe<TaggedArg::value_type> ?
        arg.get_value() ? arg.get_value().get() : arg.get_value().construct(d.value) : // arg is maybe
        arg.get_value();                                                   // arg is not maybe
  }
  reference operator[] <Def> (lazy_default<key_type,Def> const&) const { return arg.get_value(); } // not maybe
  using Next::operator[]; // from base class

  auto satisfies<HasDef,Pred,ArgPack>(parameter_requirements<key_type,Pred,HasDef>*, ArgPack*);
  using Next::satisfies; // compile-time type checking
  arg_list<tagged_argument[_rref]<KW,T2>, self> operator, <KW,T2> (tagged_argument[_rref]<KW,T2> const& x) const { return {x, *this}; }
};

using aux::arg_list_cons<>::type = empty_arg_list;
using aux::arg_list_cons<ArgTuple0,...Tuples>::type = arg_list<ArgTuple0::tagged_arg, arg_list_cons<Tuples...>, ArgTuple0::emits_errors>;

struct aux::flat_like_arg_tuple<KW,TaggedArg,EmitsErrors=true_> { using tagged_arg = TaggedArg; using emits_errors = EmitsErrors; };
struct aux::flat_like_arg_list<> : empty_arg_list {
  ctor<...Args>(Args&&...); // forward to base
  flat_like_arg_list<flat_like_arg_tuple<TaggedArg::key_type, TaggedArg::base_type>, empty_arg_list>
    operator, (TaggedArg const&) const;
};
struct aux::flat_like_arg_list<...ArgTuples> : arg_list_const<ArgTuples...>::type {
  ctor(tagged_arg const& h, tail_type const& t);
  ctor<...Args>(Args&&...); // forward to base
  flat_like_arg_list<flat_like_arg_tuple<TaggedArg::key_type, TaggedArg::base_type>, ArgTuples...>
    operator, (TaggedArg const&) const;
};

// MPL list iterator support
struct aux::arg_list_iterator<ArgPack> {
  using category = mpl::forward_iterator_tag;
  using next = arg_list_iterator<ArgPack::tail_type>;
  using type = ArgPack::key_type;
};
struct aux::arg_list_iterator<>{};
struct mpl::has_key_impl<arg_list_tag>; // if value_type<ArgList,KW> != void
struct mpl::count_impl<arg_list_tag>; // value_type<ArgList,KW> != void ? 1 : 0
struct mpl::key_type_impl<arg_list_tag>; // value_type<ArgList,KW> == void ? void : KW
struct mpl::value_type_impl<arg_list_tag>; // value_type<ArgList,KW> == void ? void : KW
struct mpl::at_impl<arg_list_tag>; // value_type<ArgList,KW> == void ? void : KW
struct mpl::order_impl<arg_list_tag>; // value_type<ArgList,KW> == void ? void : (iter distance from KW to end)
```

`arg_list` is called a ArgumentPack, which can be indexed by keyword object, or tagged (lazy) default
object, to fetch for argument value (if any).

------
#### Keyword Object

```c++
struct keyword<Tag> {
  const tag<Tag, T> operator= <T> (T [const] & x) const { return x; }
  default_<Tag, Def> operator| <Def> (Def [const] & def) { return default_<Tag, Def>(def); }
  lazy_default<Tag, Def> operator|| <Def> (Def [const] & def) { return lazy_default<Tag, Def>(def); }
  
  static keyword<Tag> const instance; static keyword<Tag>& get() { return instance; }
}
BOOST_PARAMETER_KEYWORD(tag_ns, name) :=
namespace tag_ns { struct name {}; }
namespace { keyword<tag_ns::name> const & name = keyword<tag_ns::name>::instance; }
```

A keyword object is a constant with a tag type.

------
#### Parameter Declarations

```c++
struct required<Tag,Pred=use_default> { using key_type = Tag, predicate = Pred; }
struct optional<Tag,Pred=use_default> { using key_type = Tag, predicate = Pred; }
struct deduced<Tag> { using key_type = Tag; }
using is_required = ..., is_optional = ..., is_deduced = ...;
using tag_type<T> = ...;
using has_default = !is_required;
using predicate<T> = ...;
using satisfies<ArgList,ParReq> = ...
using deduce_tag<Arg,ArgList,Ded,UsedArgs,TagFn> = ...
using make_arg_list<List,DeduceArgs,TagFn,EmitErrors=true_> = ...

struct parameters<Ps...> {
  using deduced_list = ...;
  using match<As...> = ...;
  using bind<A...> = ...;
  
  empty_arg_list operator()() const;
  arg_list<...> operator()(A&...a) const;
}
```

* `BOOST_PARAMETER_FUNCTION(result,name,tag_ns,arguments)`
* `BOOST_PARAMETER_MEMBER_FUNCTION(result,name,tag_ns,arguments)`
* `BOOST_PARAMETER_CONSTRUCTOR(cls,impl,tag_ns,arguments)`
* `BOOST_PARAMETER_NAME(name)`
* `BOOST_PARAMETER_TEMPLATE_KEYWORD(name)`
* `BOOST_PARAMETER_KEYWORD(tag_ns,name)`
* `BOOST_PARAMETER_MATCH(param,args,x)`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.Core

* `<boost/core/enable_if.hpp>`

#### Boost.Function

* `<boost/function.hpp>` -- when no std `<functional>`

#### Boost.Fusion

* `<boost/fusion/*.hpp>`

#### Boost.MP11

* `<boost/mp11/*.hpp>`

#### Boost.MPL

* `<boost/mpl/*.hpp>`

#### Boost.Optional

* `<boost/optional.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/*>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

#### Boost.Utility

* `<boost/utility/result_of.hpp>`

------
### Standard Facilities
