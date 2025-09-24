# Boost.TypeOf

* lib: `boost/libs/parameter_python`
* repo: `boostorg/parameter_python`
* commit: `800edc0`, 2025-05-03

------
#### Synopsis

* `<boost/parameter/python/python.hpp>`

```c++
// Actually MPL vector based.
struct aux::invoker<Arity,M,R,...Args>
{ static R execute(Args&&...args)          { return M()(type<R>(), forward<Args>(args)...); } };
struct aux::member_invoker<Arity,M,R,T,...Args>
{ static R execute(T& self, Args&&...args) { return M()(type<R>(), self, forward<Args>(args)...); } };
struct aux::call_invoker<Arity,T,R,...Args>
{ static R execute(T& self, Args&&...args) { return self(forward<Args>(args)...); } };
struct aux::init_invoker<Arity,T,...Args>
{ static T* execute(Args&&...args)         { return new T(forward<Args>(args)...); } };
using aux::make_invoker<M,R>::apply<Args>::type = invoker<sizeof...(Args),M,R,Args>;
using aux::make_member_invoker<M,R,T>::apply<Args>::type = member_invoker<sizeof...(Args),M,R,T,Args>;
using aux::make_call_invoker<T,R>::apply<Args>::type = call_invoker<,T,R,Args>;
using aux::make_init_invoker<T>::apply<Args>::type = init_invoker<sizeof...(Args),T,Args>;

PyObject* aux::unspecified_type(); // singleton instance, named "Boost.Parameter.Unspecified"
struct aux::empty_tag{};
PyObject* aux::empty_tag_to_python::convert(empty_tag) { return python_::xincref(unspecified_type()); }
void aux::initialize_converter()
{ static python_::to_python_converter<empty_tag, empty_tag_to_python> x; } // singleton instance

struct arg_from_python<maybe<T>> : arg_from_python<T> { // maybe<T> support
    bool empty; // whether is unspecified_type
    ctor(PyObject* p)
    bool convertible() const; maybe<T> operator()();
};

struct aux::arg_spec<K,Required,Optimized,T> {
    using keyword = K; using required = Required; using type = T; using optimized_default = Optimized;
};
using aux::is_optional<Spec> = !(Spec::required || Spec::optimized_default);
using aux::make_arg_spec<K,T>::type = arg_spec<K::first, K::second, has<K::third> ? K::third : false_, T>;
struct aux::no_keywords { T const& operator, <T> (T const& x) const { return x; } };
// recursively iterate over [It,End), makeup `arg()` combined with comma
void aux::def_combination_aux<Def,F,It,End,Keywords>(Def def, F f, It, End, Keywords const& keywords) {
    using spec = deref<It>::type; using kw = spec::keyword;
    if constexpr (It == End) { if constexpr (Keywords == no_keywords) def(f); else def(f, keywords); }
    else if constexpr (spec::optimized_default and not spec::required)
        def_combination_aux(def, f, next<It>::type(), End(), ( keywords, arg(kw::keyword_name()) = empty_tag()) );
    else def_combination_aux(def, f, next<It>::type(), End(), ( keywords, arg(kw::keyword_name()) ) );
}
struct aux::combinations_op<Spec,State> {
    using result = (Spec::required || Spec::optimized_default || (State::second & 1)) ?
        push_back<State::first, Spec> : Spec;
    using next_bits = (Spec::required || Spec::optimized_default) ? State::second : (State::second >> 1);
    using type = pair<result, next_bits>;
};
void aux::def_combination<Def,Specs,Bits,Invoker> (Def def, Specs*, Bits, Invoker*) {
    using combination0 = fold<Specs, pair<vector0<>,Bits>, combinations_op<_2,_1>>, 
    using combination = combination0::first;
    using invoker = apply_wrap<Invoker, combination0::first>;
    def_combination_aux(def, &invoker::execute, begin<combination>::type(), end<combination>::type(), no_keywords());
}
// recursively iterate over Specs and Bits, call Def with Invoker for each parameter specifier
void aux::def_combinations<Def,Specs,Bits,End,Invoker> (Def def, Specs*, Bits, End, Invoker*) {
    if constexpr (Bits == End) return;
    initialize_converter();
    def_combination(def, nullptr, Bits(), nullptr);
    def_combinations(def, nullptr, long_<Bits::value+1>(), End(), nullptr);
}

void aux::not_specified{};
struct aux::call_policies_as_options<CallPolicies> {
    CallPolicies call_policies;
    ctor(CallPolicies const&); // init
    CallPolicies const& policies() const; char const* doc() const;
};
struct aux::def_class<Class,Options=not_specified> { // wrapper for class def
    Class& cl; char const* name; Options options;
    ctor(Class& cl, char const* name, Options options={}); // init
    void def<F,[Keywords]> (F f, <Keywords const& keywords>, not_specified const*) const
    { cl.def(name, f, <keywords>); }
    void def<F,[Keywords]> (F f, <Keywords const& keywords>, void const*) const
    { cl.def(name, f, <keywords>, options.doc(), options.policies()); }

    void operator() <F,[Keywords]> (F f, <Keywords const& keywords>) const
    { def(f, <keywords>, &options); }
};
struct aux::def_init<Class,CallPolicies=default_call_policies> { // wrapper for __init__ def
    Class& cl; CallPolicies call_policies;
    ctor(Class& cl, CallPolicies call_policies={}); // init

    void operator() <F,[Keywords]> (F f, <Keywords const& keywords>) const
    { cl.def("__init__", make_constructor(f, call_policies, <keywords>)); }
};
struct aux::def_function { // wrapper for function def
    char const* name;
    ctor(char const* name);

    void operator() <F,[Keywords]> (F f, <Keywords const& keywords>) const
    { python::def(name, f, <keywords>); }
};


void def<M,Signature> (char const* name, Signature) { // def function
    using arg_types = iterator_range<next<begin<Signature>>, end<Signature>>;
    using arg_specs = transform<M::keywords, arg_types, make_arg_spec<_1,_2>>;
    using optional_arity = count_if<arg_specs, is_optional<_1>>;
    def_combinations(def_function(name),
        (arg_specs*)0, long_<0>(), long_<(1<<optional_arity)>(), (make_invoker<M,front<Signature>>*)0);
}
void def<M,Class,Signature> (Class& cl, char const* name, Signature) { // def class
    using arg_types = iterator_range<next<begin<Signature>>, end<Signature>>;
    using arg_specs = transform<M::keywords, arg_types, make_arg_spec<_1,_2>>;
    using optional_arity = count_if<arg_specs, is_optional<_1>>;
    def_combinations(def_class<Class>(cl, name),
        (arg_specs*)0, long_<0>(), long_<(1<<optional_arity)>(), (make_invoker<M,front<Signature>>*)0);
}

using aux::keyword<K>::type = is_pointer<K> ? keyword<remove_pointer<K>>::type : K;
using aux::required<K>::type = is_pointer<K> ? false_ : true_;
using aux::optimized<K>::type = is_pointer<K> && is_pointer<remove_pointer<K>> ? false_ : true_;
using aux::make_kw_spec<K(T)>::type = arg_spec<keyword<K>::key, required<K>::type, optimized<K>::type, T>;

struct init<Specs,CallPolicies=default_call_policies> : def_visitor<init> {
    CallPolicies call_policies;
    ctor(CallPolicies call_policies={});
    init<Specs,Pol> operator[] <Pol> (Pol const& call_policies) const; // change to other policies
    void visit <Class> (Class& cl) const {
        if constexpr (empty<Specs>) cl.def(python::init<>(){call_policies});
        else {
            using arg_specs = transform<Specs, make_kw_spec<_1>>;
            using optional_arity = count_if<arg_specs, is_optional<_1>>;
            def_combinations(def_init{cl, call_policies},
                (arg_specs*)0, long_<0>(), long_<(1<<optional_arity)>(),
                (make_init_invoker<Class::wrapped_type>)0);
        }
    }
};
struct call<Specs,CallPolicies=default_call_policies> : def_visitor<call> {
    CallPolicies call_policies;
    ctor(CallPolicies call_policies={});
    call<Specs,Pol> operator[] <Pol> (Pol const& call_policies) const; // change to other policies
    void visit <Class> (Class& cl) const {
        using arg_types = iterator_range<next<begin<Signature>>, end<Signature>>;
        using arg_specs = transform<Specs, make_kw_spec<_1>>;
        using optional_arity = count_if<arg_specs, is_optional<_1>>;
        def_combinations(def_class{cl, "__call__", call_policies_as_options{call_policies}},
            (arg_specs*)0, long_<0>(), long_<(1<<optional_arity)>(),
            (make_call_invoker<Class::wrapped_type, front<Specs>>)0);
    }
};
struct function<Fwd,Specs> : def_visitor<function> {
    void visit <Class,Options> (Class& cl, char const* name, Options const& options) const {
        using arg_types = iterator_range<next<begin<Signature>>, end<Signature>>;
        using arg_specs = transform<Specs, make_kw_spec<_1>>;
        using optional_arity = count_if<arg_specs, is_optional<_1>>;
        def_combinations(def_class{cl, name, options},
            (arg_specs*)0, long_<0>(), long_<(1<<optional_arity)>(),
            (make_member_invoker<Fwd, front<Specs>, Class::wrapped_type>)0);
    }
};
```

#### Usage

* Each parameter specifier is an usage of the keyword tag:
 - `tag` for required keyword parameter.
 - `tag*` for optional keyword.
 - `tag**` for special keyword.
* Arity (parameter count) excludes the special parameters.
* For each special keyword, overloads with and without that parameter will both be generated, thus for N special keywords, there will be 2^N overloads.

```c++
using namespace boost::python; namespace param=boost::parameter; namespace py = para::python; namespace mpl = boost::mpl;
BOOST_PARAMETER_KEYWORD(tag, x)
BOOST_PARAMETER_KEYWORD(tag, y)
using call_params = param::parameters<param::required<tag::x>, param::optional<tag::y>>;

BOOST_PARAMETER_FUNCTION((void), f, tag, (required (x,*)) (optional (y,*,1))) {}
struct X {
    BOOST_PARAMETER_CONSTRUCTOR(X, (X), tag, (required (x,*)) (optional (y, *)))
    template<class ArgPack> X(ArgPack const& args) { args[x]; args[y]; }
    BOOST_PARAMETER_MEMBER_FUNCTION((void), f, tag, (required (x,*)) (optional (y,*,1))) {}
    template <class...As> int operator() (As const& ... as) { return call(call_parameters{}(a0,a1)); }
    template <class ArgPack> call (ArgPack const& args) { args[x]; args[y]; }
};
BOOST_PYTHON_MODULE(mod) {

    using xf_fwd = decltype([](boost::type<void>, X& self, auto const& a0, auto const& a1)->void { self.f(a0, a1);});
    using f_fwd = decltype([](boost::type<void>, auto const& a0, auto const& a1)->void { f(a0, a1);});
    py::def<f_fwd, mpl::vector<void, tag::x(int), tag::y*(int)>>("f");
    class_<X>("X", no_init)
        .def(     py::init<            mpl::vector<tag::x(int), tag::y*(int)>>())
        .def(     py::call<            mpl::vector<int, tag::x(int), tag::y*(int)>>)
        .def("f", py::function<xf_fwd,  mpl::vector<void, tag::x(int), tag::y*(int)>>());
}
```

------
### Dependency

#### Boost.MPL

* `<boost/mpl/*.hpp>`

#### Boost.Parameter

* `<boost/parameter/aux_/maybe.hpp>`
* `<boost/parameter/aux_/python/invoker.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/*.hpp>`

#### Boost.Python

* `<boost/python/def.hpp>`
* `<boost/python/make_constructor.hpp>`
* `<boost/python/init.hpp>`
* `<boost/python/to_python_converter.hpp>`

------
### Standard Facilities

* Language: `decltype` (C++11), `auto` (C++11).
