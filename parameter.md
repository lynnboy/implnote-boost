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

struct default_<KW, V> {
  V& value;
  default_(V& x): value(x) {} // implicit
}
struct lazy_default<KW, DefComputer> {
  DefComputer const& compute_default;
  lazy_default(const DefComputer& x) : compute_default(x) {} // implicit
}

struct aux::tagged_argument_base {};
struct aux::tagged_argument<Keyword, Arg> : tagged_argument_base {
  using key_type = Keyword, value_type = Arg, reference = Arg&;
  reference value;  // stored argument value
    
  tagged_argument(reference x) : value(x) {}
    
  arg_list<tagged_argument<Keyword, Arg>, arg_list<tagged_argument<KW2, Arg2>>
  operator,<KW2, Arg2> (tagged_argument<KW2, Arg2> x) const; // combine at call site arg list
    
  // value fetching
  reference operator[](keyword<KW> const&) const { return value; }
  reference operator[]<Def>(default_<KW, Def> const&) const { return value; }
  reference operator[]<F>(lazy_default<KW, F> const&) const { return value; }
  Def& operator[]<KW2,Def>(default_<KW2, Def> const& x) const { return x.value; }
  result_of_t<F()> operator[]<KW2,F>(lazy_default<KW2, F> const& x) const { return x.compute_default(); }
};
using aux::is_tagged_argument<T> = is_base_of<tagged_argument_base, remove_const_t<remove_reference_t<T>>>;
struct aux::tag<Keyword, ActualArg> {
  using Arg = unwrap_cv_reference<ActualArg>; using ConstArg = const Arg; using MutArg = remove_const_t<Arg>;
  using type = (is_lvalue_reference<ActualArg> || is_cv_reference_wrapper<ActualArg>) ?
    tagged_argument_list_of<tagged_argument<Keyword, Arg>> : is_scalar<MutArg> ?
      tagged_argument_list_of<tagged_argument<Keyword, ConstArg>>
      tagged_argument_list_of<tagged_argument_rref<Keyword, Arg>>
};

struct parameter_requirements<KW, Pred, HasDef> { using keyword = KW, predicate = Pred, has_default = HasDef; }

struct maybe_base {};
struct maybe<T> : maybe_base {
  using reference = T const &, non_cv_value = remove_cv_t<remove_reference_t<reference>>

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
using is_maybe<T> = is_base_and_derived<maybe_base, T>;

struct empty_arg_list {
  struct binding {
    using apply<KW,Def,Ref> = Def; // not matching anything
  }
  empty_arg_list(void_, void_, ...);

  Def& operator[] <K, Def> (default_<K, Def> x) const { return x.value; }
  result_of_t<F()> operator[] <K, F> (lazy_default<K,F> x) const { return x.compute_default(); }
  static ParReq::has_default satisfies<ParReq,ArgPack>(ParReq*, ArgPack*);
}
struct arg_list<TaggedArg, Next=empty_arg_list> : Next {
  using self = arg_list<TaggedArg,Next>, key_type = TaggedArg::key_type;
  using reference = is_maybe<TaggedArg::value_type> ? TaggedArg::value_type::reference : TaggedArg::reference;
  using value_type = is_maybe<TaggedArg::value_type> ? reference : TaggedArg::value_type;
  
  struct binding {
    using apply<KW,Def,Ref> = is_same<KW, key_type> ? // recursive lookup for match type
        (Ref ? reference : value_type) : Next::binding::apply<KW,Def,Ref>
  }

  TaggedArg arg; // storage of arg

  arg_list<A...>(A & a0, a1, ...) : Next(a1, ..., void_reference(), arg(a0) {} // init base and 'arg'
  Def& operator[] (keyword<key_type> const&) const { return arg.value; }
  Def& operator[] <Def> (default_<key_type, Def> d) const {
    return is_maybe<TaggedArg::value_type> ?
        arg.value ? arg.value.get() : arg.value.construct(d.value) : // arg is maybe
        arg.value;                                                   // arg is not maybe
  }
  reference operator[] <Def> (lazy_default<key_type,Def>) const { return arg.value; }
  
  arg_list<tagged_argument<KW,T2>, self> operator, <KW, T2> (tagged_argument<KW,T2> x) const {
    return arg_list<tagged_argument<KW,T2>, self>(x, *this);
  }
}
arg_list<...> operator() <...A> (A const & ... a) { arg_list<...>(a...); }

using binding<Args,KW,Def=void_> = ...::reference;
using lazy_binding<Args,KW,F> = binding<Args,KW,result_of_t<F()>>;
using value_type<Args,KW,Def=void_> = ...::value_type;
using lazy_value_type<Args,KW,F> = value_type<Args,KW,result_of_t<F()>>;
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
