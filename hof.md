# Boost.HOF

* lib: `boost/libs/hof`
* repo: `boostorg/hof`
* commit: `1fa2229`, 2024-08-19

------
### Common

#### Definitions

* Function Adaptor
* Static Function Adaptor
* Decorator

```c++
template<class...F> class FunctionAdaptor_adaptor;
template<class...F> FunctionAdaptor_adaptor<F...> FunctionAdaptor(F...f);

template<class...F> class StaticFunctionAdaptor; // default_constructible

template<class...T> FunctionAdaptor Decorator(T...t);
```

------
#### Concepts

```c++
concept ConstFunctionObject<F, ...Ts> = std::is_object_v<F> && requires(const F f, Ts&&...ts) { f(ts...); };
concept NullaryFunctionObject<F> = ConstFunctionObject<F>;
concept UnaryFunctionObject<F,T> = ConstFunctionObject<F,T>;
concept BinaryFunctionObject<F,T,U> = ConstFunctionObject<F,T,U>;
concept MutableFunctionObject<F, ...Ts> = std::is_object_v<F> && requires(F f, Ts&&...ts) { f(ts...); };
concept EvaluatableFunctionObject<F> = NullaryFunctionObject<F> || UnaryFunctionObject<T,identity>;

concept Invocable<F, ...Ts> = std::invocable<F,Ts...>;
concept ConstInvocable<F, ...Ts> = std::invocable<const F,Ts...>;
concept UnaryInvocable<F,T> = ConstInvocable<F,T>;
concept BinaryInvocable<F,T,U> = ConstInvocable<F,T,U>;

concept Metafunction<TT> = requires { typename TT::type; };
concept MetafunctionClass<TT> = requires { typename TT::apply::type; };
concept MetafunctionClass<TT,...Ts> = requires { typename TT::apply<Ts...>::type; };
```

------
### Common parts

```c++
struct detail::holder<...Ts>{using type=void;};
struct detail::template_holder<TT<...>>{using type=void;};
struct detail::is_constructible<T>; // also supports default constructible
struct detail::is_default_constructible<...Xs>;
struct detail::remove_rvalue_reference<T>;
struct detail::enable_if_constructible<C,X,...Xs>;
using detail::and_<...Ts>;
constexpr T&& detail::forward<T>(std::remove_reference<T>::type& t) noexcept;
constexpr T&& detail::forward<T>(std::remove_reference<T>::type&& t) noexcept;
struct detail::unwrap_reference<T> { using type=T; };
struct detail::unwrap_reference<std::reference_wrapper<T>> { using type=T&; };

struct detail::static_const_storage<T> { static constexpr T value={}; };
struct detail::static_const_var_factory
{ constexpr const T& operator= <T> (const T&) const { return static_const_storage<T>::value; } };
const T& static_const_var<T>() { return static_const_storage<T>::value; }

struct detail::seq<n...>{using type = self;};
struct detail::merge_seg<seq<m...>,seq<n...>> : seq<m...,(sizeof...(m)+n)...> {};
using detail::gens<0> = seq<>; using detail::gens<1> = seq<0>;

using detail::can_be_called<F,...Ts> = can_be_called_impl<F,called_args<Ts...>>; // SFINAE-based

struct detail::non_class_function<F> {
    F f;
    ctor<...Xs>(Xs&&...xs) : f{fwd<Xs>(xs)...} {}
    constexpr auto operator() <...Ts> (Ts&&...xs) const noexcept(...) -> ... { return apply(f, fwd<Ts>(xs)...); }
};
struct detail::callable_base_type<F> : conditional_t<(is_class_v<F> && !is_final_v<F> && !is_polymorphic_v<F>), F, non_class_function<F>>{};
using detail::callable_base<F> = callable_base_type<F>::type;

struct detail::make<Adaptor<...>> {
    constexpr ctor() noexcept {}
    constexpr R operator() <...Fs, R=JOIN(Adaptor,Fs...)> (Fs... fs) const noexcept(...) { return R(fwd<Fs>(fs)...); }
};
std::remove_reference_t<T>&& move<T>(T&& x) noexcept { return (...)x; }

struct alias_tag<T>;
struct has_tag<T,Tag>; // trait
constexpr <const> T& detail::lvalue<T>(<const> T& x) noexcept { return x; }
struct alias<T,Tag=void> { T value; };
constexpr auto alias_value<Tag,T,...Ts> (alias<T, Tag> {const&|&} a, Ts&&...)
    noexcept(...) -> decltype(...) { return lvalue(a.value); }
constexpr auto alias_value<Tag,T,...Ts>(alias<T, Tag> && a, Ts&&...)
    noexcept(...) -> decltype(...) { return move(a.value); }
struct alias_tag<alias<T,Tag>> { using type = Tag; };

struct alias_inherit<T,Tag=void> : T {};
constexpr T const& alias_value<Tag,T,...Ts>(alias_inherit<T, Tag> const& a, Ts&&...)
    noexcept(...) requires is_class_v<T> { return lvalue(a); }
constexpr T & alias_value<Tag,T,...Ts>(alias_inherit<T, Tag> & a, Ts&&...)
    noexcept(...) requires is_class_v<T> { return lvalue(a); }
constexpr T && alias_value<Tag,T,...Ts>(alias_inherit<T, Tag> && a, Ts&&...)
    noexcept(...) requires is_class_v<T> { return move(a); }
struct alias_tag<alias_inherit<T,Tag>> { using type = Tag; };

struct detail::alias_static_storage<T,Tag> { static constexpr T value=T{}; };
struct alias_static<T,Tag=void> { constexpr ctor<...Ts>(Ts&&...) noexcept requires(...){} };
constexpr const T& alias_value<Tag,T,...Ts>(const alias_static<T,Tag>&, Ts&&...) noexcept
{ return alias_static_storage<T,Tag>::value; }
struct alias_tag<alias_static<T,Tag>> { using type = Tag; };

struct detail::alias_try_inherit<T,Tag> : std::conditional_t<is_class_v<T>&&!is_final_v<T>&&!is_polymorphic_v<T>, alias_inherit<T,Tag>, alias<T,Tag>> {};
struct detail::alias_empty<T,Tag> : std::conditional_t<is_empty_v<T>, alias_try_inherit<T,Tag>::type, alias<T,Tag>>;

struct detail::pair_tag<i,T,U>{};
struct detail::is_same_template<T,U> : std::false_type{}; // trait
struct detail::is_same_template<X<Ts...>,X<Us...>> : std::true_type{}; // trait
struct detail::is_related_template<U,T> : is_same_template<T,U>{};
struct detail::is_related<T,U> : std::bool_constant<is_base_of_v<T,U>||is_base_of_v<U,T>||is_related_template<T,U>::value> {};
struct detail::pair_holder<i,T,U> : std::conditional_t<is_related<T,U>::value, alias_empty<T,pair_tag<i,T,U>>, alias_try_inherit<T,pair_tag<i,T,U>>> {};
struct detail::compressed_pair<First,Second> : pair_holder<0,First,Second>::type, pair_holder<1,Second,First>::type {
    using first_base = ...; using second_base = ...;
    constexpr ctor<X,Y>(X&& x, Y&& y) noexcept(...) requires(...) {}
    constexpr const Base& get_alias_base<Base,...Xs> get_alias_base(Xs&&...xs) const noexcept { return always_ref(*this)(xs...); }
    constexpr const First& first<...Xs>(Xs&&...xs) const noexcept { return always_value(get_alias_base<first_base>(xs...), xs...); }
    constexpr const Second& second<...Xs>(Xs&&...xs) const noexcept { return always_value(get_alias_base<second_base>(xs...), xs...); }
};
constexpr compressed_pair<T,U> detail::make_compressed_pair<T,U>(T x, U y) noexcept(...) { return {(T&&)x, (U&&)y}; }
```

#### Macros:
##### Language macros
* `USING(n,...)`, `USING_TEMPLATE(n,...)`
* `NOEXCEPT(...)`, `NOEXCEPT_CONSTRUCTIBLE(...)`
* `STATIC_CONSTEXPR`, `STATIC_AUTO_REF`,
* `STATIC_CONST_VAR(v)`, `DECLARE_STATIC_VAR(v,...)`
* `INHERIT_DEFAULT(C,...)`, `INHERIT_DEFAULT_EMPTY(C,...)` define default ctor
* `DELEGATE_CONSTRUCTOR(C,T,v)`, `INHERIT_CONSTRUCTOR(Derived,Base)`
* `SFINAE_<MANUAL>_RESULT(...)` = `auto`, `SFINAE_<MANUAL>_RETURNS` = `RETURNS`
* `JOIN(c,...)` = `c<__VA_ARGS__>`
##### Library macros
* `IS_XXX(...)` = `std::is_xxx<__VA_ARGS>::value` for type traits
* `FORWARD(...)`
* `ENABLE_IF_CONVERTIBLE`, `ENABLE_IF_CONVERTIBLE_UNPACK`, `ENALBE_IF_BASE_OF`, `ENABLE_IF_CONSTRUCTIBLE`
##### Other macros

------
### Functions

##### `always`

```c++
struct detail::always_base<T> { T x;
    constexpr ctor(T xp) noexcept(...) x:{xp} {}
    constexpr T operator() <...As>(As&&...) { return x;}
};
struct detail::always_base<void> {
    constexpr ctor() noexcept {}
    constexpr T operator() <...As>(As&&...) { }
};
struct detail::always_f {
    always_base<T> operator() <T> (T x) const noexcept(...) { return {x}; }
    always_base<void> operator() () const noexcept(...) { return {}; }
};
struct detail::always_ref_f {
    always_base<T> operator() <T> (T& x) const noexcept(...) { return {x}; }
};

constexpr always_f const& always = always_f{};
constexpr always_ref_f const& always_ref = always_ref{};
```

Usage
```c++
auto f = always(x); // f always returns x
assert(x == f(a,b,c,d));
```

##### `arg`

```c++
struct detail::perfect_ref<T>{
    using type = T; using value_type = remove_reference_t<T>;
    T&& value;
    constexpr ctor(value_type& x) noexcept : value{std::forward<T>(x)} {}
};
struct detail::ignore<n>{ constexpr ignore<T>(T&&...) noexcept{} };
struct detail::args_at<...n>
{ constexpr auto operator() <T,...Ts>(ignore<n>..., T x, Ts...) const RETURNS(forword<T::type>(x.value)); };
constexpr args_at<n...> detail::make_args_at<...n>(seq<n...>) noexcept { return {}; };
constexpr auto detail::get_args<n,...Ts>(Ts&&...xs) RETURNS( make_args_at(gens<n>::type()) (nullptr, RETURNS_CONSTRUCT(perfect_ref<Ts>)(xs)...) );
struct detail::make_args_f<T n>{ constexpr auto operator() <...Ts>(Ts&&...xs) const RETURNS(get_args<n>(FORWARD(Ts)(xs)...)) };
struct detail::arg_f { constexpr make_args_f<size_t,I::value> operator() <I> (I) const noexcept { return {}; } };

constexpr make_args_f<size_t,n> arg_c = {};
constexpr arg_f const& arg = arg_f{};
```

Usage
```c++
auto arg3 = arg(std::integral_constant<int,3>{}); // returns 3rd arg
auto arg4 = arg(4); // returns 4th arg
assert(arg3(1,2,3,4,5)==3);
assert(arg4('a','b','c','d','e')=='d');
```

##### `construct`

```c++
struct detail::construct_f<T> {
    using storage = std::aligned_storage<sizeof(T)>::type;
    struct storage_holder { storage* s; // ctor
        T& data() noexcept { return *(T*)s; }
        ~dtor() noexcept(...) { this->data().~T(); }
    };
    constexpr T operator() <...Ts> (Ts&&...xs) const noexcept(...) requires(...);
    constexpr T operator() <X> (std::initializer_list<X>{const &|&|&&} x) const noexcept(...) requires(...);
};
struct detail::construct_template_f<Template<...>, D<...>>
{ constexpr R operator() <...Ts, R=JOIN(Template,D<Ts>::type...)> (Ts&&...xs) const noexcept(...) requires(...); };
struct detail::construct_meta_f<Metafun> {
    struct apply<...Ts> : MetafunctionClass::apply<Ts...> {};
    constexpr R operator()<...Ts, Metafun=JOIN(apply,Ts...), R=Metafun::type> (Ts&&...xs) const
        noexcept(...) requires(...) { return construct_f<R>()(fwd<Ts>(xs)...); }
};
struct detail::construct_meta_template_f<MetafunTemplate<...>> {
    constexpr R operator()<...Ts, Metafun=JOIN(MetafunTemplate,Ts...), R=Metafun::type> (Ts&&...xs) const
        noexcept(...) requires(...) { return construct_f<R>()(fwd<Ts>(xs)...); }
};
struct detail::construct_id<T> {using type=T;};

constexpr auto construct <T> () ->construct_f<T> noexcept { return {}; }
constexpr auto construct <Temp<...>> () ->construct_template_f<Temp,decay_mf> noexcept { return {}; }
constexpr auto construct_forward <T> () ->construct_f<T> noexcept { return {}; }
constexpr auto construct_forward <Temp<...>> () ->construct_template_f<Temp,construct_id> noexcept { return {}; }
constexpr auto construct_basic <T> () ->construct_f<T> noexcept { return {}; }
constexpr auto construct_basic <Temp<...>> () ->construct_template_f<Temp,remove_rvalue_reference> noexcept { return {}; }
constexpr auto construct_meta <T> () ->construct_meta_f<T> noexcept { return {}; }
constexpr auto construct_meta <Temp<...>> () ->construct_meta_template_f<Temp> noexcept { return {}; }
```

##### `decay`

```c++
struct detail::decay_mf<T> : unwrap_reference<std::decay_t<T>> {};
struct detail::decay_f {
    constexpr R operator() <T,R=unwrap_reference<std::decay_t<T>>::type> (T&& x) const
        noexcept(...) requires(...) { return fwd<T>(x); }
};
constexpr decay_f const& decay = decay_f{};
```

##### `identity`

```c++
struct detail::identity_base{
    constexpr T operator() <T> (T&& x) const noexcept(...) { return fwd<T>(x); }
    constexpr <const> std::initializer_list<T>& operator() <T> (<const> std::initializer_list<T>& x) const noexcept(...) { return x; }
    constexpr std::initializer_list<T> operator() <T> (std::initializer_list<T>&& x) const noexcept(...) { return fwd<std::initializer_list<T>(x); }
};
constexpr identity_base const& identity = identity_base{};
```

##### Placeholders

```c++
struct detail::simple_placeholder<n>{};
struct std::is_placeholder<simple_placeholder<n>> : std::integral_constant<int, n> {};

struct operators::call
{ constexpr auto operator() <F,...Ts> (F&& f, Ts&&...xs) const noexcept(...) ->decltype(...) { return f(fwd<Ts>(xs)...); } };

// add, subtract, multiply, divide, remainder, shift_right, shift_left
// greater_than, less_than, less_than_equal, greater_than_equal, equal, not_equal
// bit_and, xor_, bit_or, and_, or_
// assign_add, assign_subtract, assign_multiply, assign_divide, assign_remainder
// assign_shift_right, assign_shift_left, assign_bit_and, assign_bit_or, assign_xor
struct operators::NAME {
    constexpr auto operator() <T,U> (T&& x, U&& y) const noexcept(...) -> decltype(...)
    { return boost::hof::forward<T>(x) BIN_OP boost::hof::forward<U>(y); }
};
// not_, compl_, unary_plus, unary_subtract, dereference, increment, decrement
struct operators::NAME {
    constexpr auto operator() <T> (T&& x) const noexcept(...) -> decltype(...)
    { return UN_OP boost::hof::forward<T>(x); }
};

struct placeholder<n> {
    constexpr auto operator() <...Ts> (Ts&&...xs) const noexcept(...) -> decltype(...)
    { return lazy(operators::call())(simple_placeholder<n>{}, fwd<Ts>(xs)...); }
    // not_, compl_, unary_plus, unary_subtract, dereference, increment, decrement
    constexpr auto operator UN_OP () const noexcept(...) -> decltype(...)
    { return lazy(operators::NAME{})(simple_placeholder<n>{}); }
    // assign_add, assign_subtract, assign_multiply, assign_divide, assign_remainder
    // assign_shift_right, assign_shift_left, assign_bit_and, assign_bit_or, assign_xor
    constexpr auto operator ASN_OP <T> (T&& x) const noexcept(...) -> decltype(...)
    { return lazy(operators::NAME{})(simple_placeholder<n>{}, fwd<T>(x)); }
};
// add, subtract, multiply, divide, remainder, shift_right, shift_left
// greater_than, less_than, less_than_equal, greater_than_equal, equal, not_equal
// bit_and, xor_, bit_or, and_, or_
constexpr auto operator BIN_OP <T,n> (const placeholder<n>&, T&& x) noexcept(...) -> decltype(...)
{ return lazy(operators::NAME{})(simple_placeholder<n>{}, fwd<T>(x)); }
constexpr auto operator BIN_OP <T,n> (T&& x, const placeholder<n>&) noexcept(...) -> decltype(...)
{ return lazy(operators::NAME{})(fwd<T>(x), simple_placeholder<n>{}); }
constexpr auto operator BIN_OP <n,m> (const placeholder<n>&, const placeholder<m>&) noexcept(...) -> decltype(...)
{ return lazy(operators::NAME{})(simple_placeholder<n>&, simple_placeholder<m>{}); }

constexpr static auto const & placeholders::_1 = placeholder<1>{}; // up to _9
using placeholders::_1; // up to _9


struct detail::unnamed_placeholder {
    struct partial_op<T,Invoker> { T val;
        constexpr ctor<X,...Xs>(X&& x, Xs&&...xs) : val(fwd<X>(x), fwd<Xs>(xs)...) {}
        struct partial_ap_failure{
            struct apply<Failure> { struct of<...Xs>;struct of<X> : Failure::of<const T, X>{}; };
        };
        struct failure : failure_map<partial_ap_failure, Invoker> {};
        constexpr auto operator() <X> (X&& x) const noexcept(...) ->decltype(...) { return Invoker{}(val, fwd<X>(x)); }
    };
    static constexpr partial_ap<T,Invoker> make_partial_ap<Invoker,T>(T&& x) { return {fwd<T>(x)}; }
    struct left<Op> {
        struct failure : failure_for<Op> {};
        constexpr auto operator() <T,X> (T&& val, X&& x) const noexcept(...) ->decltype(...) { return Op{}(fwd<T>(val), fwd<X>(x)); }
    };
    struct right<Op> {
        struct right_failure{
            struct apply<Failure> { struct of<T,U,...Ts> : Failure::of<U, T, Ts...>{}; };
        };
        struct failure : failure_map<right_failre,Op> {};
        constexpr auto operator() <T,X> (T&& val, X&& x) const noexcept(...) ->decltype(...) { return Op{}(fwd<X>(x), fwd<T>(val)); }
    };
    // not_, compl_, unary_plus, unary_subtract, dereference, increment, decrement
    constexpr auto operator UN_OP () const noexcept(...) -> decltype(...) { return operators::NAME{}; }
    // assign_add, assign_subtract, assign_multiply, assign_divide, assign_remainder
    // assign_shift_right, assign_shift_left, assign_bit_and, assign_bit_or, assign_xor
    constexpr auto operator ASN_OP <T> (T&& x) const noexcept(...) -> decltype(...)
    { partial_ap<T,left<operators::NAME>>{x}; }
};
// add, subtract, multiply, divide, remainder, shift_right, shift_left
// greater_than, less_than, less_than_equal, greater_than_equal, equal, not_equal
// bit_and, xor_, bit_or, and_, or_
constexpr auto operator BIN_OP <T,n> (const placeholder<n>&, T&& x) noexcept(...) -> decltype(...)
{ return unnamed_placeholder::make_partial_ap<unnamed_placeholder::right<operators::NAME>>{decay(x)}; }
constexpr auto operator BIN_OP <T,n> (T&& x, const placeholder<n>&) noexcept(...) -> decltype(...)
{ return unnamed_placeholder::make_partial_ap<unnamed_placeholder::left<operators::NAME>>{decay(x)}; }
constexpr auto operator BIN_OP <n,m> (const placeholder<n>&, const placeholder<m>&) noexcept(...) -> decltype(...)
{ return oerators::NAME{}; }

constexpr static auto const & placeholders::_ = unnamed_placeholder{};
using placeholders::_;

struct std::is_placeholder<placeholder<n>> : std::integral_constant<int,n>{};
```

------
### Function Adaptors

##### `combine`

```c++
struct detail::combine_adaptor_base<seq<n...>,F,Gs...> : F, pack_base<seq<n...>,Gs...> {
    // ctors
    constexpr const F& base_function<...Ts>(Ts&&...xs) const { return always_ref(*this)(xs...); }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...)->decltype(...)
    { return base_function(xs...)(alias_value<pack_tag<seq<ns>,Gs...>,Gs>(*this, xs)(fwd<Ts>(xs))...); }
};
struct combine_adaptor<F,...Gs> : combine_adaptor_base<gens<sizeof...(Gs)>::type, callable_base<F>, callable_base<Gs>...> {};
constexpr combine_adaptor& combine{};
// assert( combine(f,gs...)(xs...) == f(gs(xs)...) )
```

##### `lazy`

```c++
struct detail::placeholder_transformer; // std::is_placeholder -> make_args_f
struct detail::bind_transformer; // std::is_bind_expression
struct detail::ref_transformer; // std::reference_wrapper -> always_ref
struct detail::id_transformer;
constexpr auto& detail::pick_transformer = first_of_adaptor<placeholder_transformer, bind_transformer, ref_transformer, id_transformer>{};
constexpr auto detail::lazy_transform<T,Pack>(T&& x, const Pack& p) ->decltype(...) { return p(pick_transformer(fwd<T>(x))); }

struct detail::lazy_unpack<F,Pack> {
    const F& f; const Pack& p; // ctor(f,p)
    constexpr auto operator()<...Ts>(Ts&&...xs) const ->decltype(...) { return f(lazy_transform(fwd<Ts>(xs), p)...); }
};
constexpr lazy_unpack<F,Pack> detail::make_lazy_unpack<F,Pack>(const F& f, const Pack& p) noexcept { return {f,p}; }

struct detail::lazy_invoker<F,Pack> : compressed_pair<F,Pack> {
    using base_type = ...; using fit_rewritable1_tag = lazy_invoker;
    constexpr const F& base_function<...Ts>(Ts&&...xs) const noexcept { return first(xs...); }
    constexpr const Pack& get_pack<...Ts>(Ts&&...xs) const noexcept { return second(xs...); }
    constexpr auto operator() <...Ts> (Ts&&...xs) const noexcept(...) ->decltype(...)
        { return make_lazy_unpack(base_function(xs...), pack_forward(fwd<Ts>(xs)...)); }
};
constexpr lazy_invoker<F,Pack> detail::make_lazy_invoker<F,Pack>(F f, Pack pack) noexcept(...) { return {(F&&)f, (Pack&&)pack}; }

struct detail::lazy_nullary_invoker<F> : F {
    constexpr const F& base_function<...Ts>(Ts&&...xs) const noexcept { return always_ref(*this)(xs...); }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) ->decltype(...) { return base_function(xs...)(); }
};
constexpr lazy_nullary_invoker<F> make_lazy_nullary_invoker(F f) noexcept(...) { return {(F&&)f}; }

struct lazy_adaptor<F> : callable_base<F> {
    constexpr const base& base_function<...Ts>(Ts&&...xs) const noexcept { return always_ref(*this)(xs...); }
    constexpr auto operator() <T,...Ts>(T x, Ts...xs) const ->decltype(...)
        { return make_lazy_invoker(base_function(x,xs...), pack_basic((T&&)x, (Ts&&)xs...)); }
    constexpr lazy_nullary_invoker<F> operator() <Unused=int> () const noexcept(...) -> decltype(...)
        { return make_lazy_nullary_invoker((callable_base<F>&&)base_function(Unused{})); }
};
constexpr auto const& lazy = make<lazy_adaptor>{};

struct std::is_bind_expression<lazy_invoker<F,Pack>> : std::true_type {};
struct std::is_bind_expression<lazy_nullary_invoker<F>> : std::true_type {};
```

##### `protect`

```c++
struct protect_adaptor<F> : callable_base<F> { using fit_rewritable1_tag = self; }
constexpr auto const& protect = make<protect_adaptor>{};
```

##### `reveal`

```c++
struct detail::has_failure<T>; // trait for T::failure
struct detail::identity_failure {
    T operator()<T>(T&& x);
    static T&& val<T>();
    using defer<Template<...>,...Ts> = Template<Ts...>;
};
struct get_failure<F> { struct of<...Ts>{ using apply<Id> = ...; }; };
struct as_failure<Template<...>> { struct of<...Ts>{ using apply<Id> = ...; }; };
using detail::apply_failure<Failure,...Ts> = Failure::of<Ts...>;
struct detail::reveal_failure<F,Failure> {
    constexpr auto operator() <...Ts> (Ts&&...xs) const ->apply_failure<Failure,Ts...>::apply<identity_failure>;
};
struct detail::traverse_failure<F,Failure=get_failure<F>,_=void> : reveal_failure<F,Failure>{};
struct detail::traverse_failure<F,Failure,holder<Failure::children>::type> : Failure::children::overloads<F>{};
struct detail::transform_failures<Failure,Transform,_=void> : Transform::apply<Failure>{};
struct detail::transform_failures<Failure,Transform,holder<Failure::children>> : Failure::children::transform<Transform>{};
struct with_failures<...Fs>{ using children = JOIN(failures,Fs...); };
struct failures<Failure,...Failures> {
    using transform<Transform> = with_failures<transform_failures<Failure,Transform>, transform_failures<Failures,Transform>...>;
    struct overloads<F,FailureBase=JOIN(failures,Failures...)> : traverse_failure<F,Failure>, FailureBase::overloads<F> { using base::operator(); };
};
struct failures<Failure> {
    using transform<Transform> = with_failures<transform_failures<Failure,Transform>>;
    using overloads<F> = traverse_failure<F,Failure>;
};
struct failure_map<Transform,...Fs> : with_failures<transform_failures<get_failure<Fs>,Transform>...>{};
struct failure_for<...Fs> : with_failures<get_failure<Fs>...>{};
struct reveal_adaptor<F,Base=callable_base<F>> : traverse_failure<Base>, Base
{ using fit_rewritable1_tag = self; using base::operator(); };
struct reveal_adaptor<reveal_adaptor<F>> : reveal_adaptor<F>{};

constexpr auto const& reveal = make<reveal_adaptor>();
```

------
### Type Traits

##### `is_invocable`

```c++
struct is_invocable<F,...Ts> : can_be_called<apply_f, F, Ts...>{};
struct is_invocable<F(Ts...), Us...> requires !std::is_same_v<F,F> {};
```

##### `unpack_sequence`

```c++
struct unpack_sequence<Seq,_=void> { using not_unpackable=void; };
```

------
### Utilities

#### Macors

* `RETURNS(...)` : `decltype(__VA_ARGS__){return __VA_ARGS__; }`
* `RETURNS_CLASS(C)`, `THIS`, `CONST_THIS` : checking facility for old compilers.
* `MANGLE_CAST`, `RETURNS_C_CAST`, `RETURNS_REINTERPRET_CAST`, `RETURNS_STATIC_CAST`, `RETURNS_CONSTRUCT`
* `AUTO_FORWARD(...)` : `static_cast<decltype(__VA_ARGS__)>(__VA_ARGS__)`

#### Functions

##### `apply`, `apply_eval`

```c++
struct detail::apply_mem_fn;
struct detail::apply_mem_data;
struct detail::apply_f {
    constexpr auto operator() <F,T,...Ts> (F&& f, T&& obj, TS&&...xs) const
        noexcept(...) ->decltype(...) requires (...)
    { return apply_mem_fn{}(f,*fwd<T>(obj), fwd<Ts>(xs)...); }
    constexpr auto operator() <F,T,...Ts> (F&& f, T&& obj, TS&&...xs) const
        noexcept(...) ->decltype(...) requires (...)
    { return apply_mem_fn{}(f,fwd<T>(obj), fwd<Ts>(xs)...); }
    constexpr auto operator() <F,T,...Ts> (F&& f, const std::reference_wrapper<T>& ref, TS&&...xs) const
        noexcept(...) ->decltype(...) requires (...)
    { return apply_mem_fn{}(f,ref.get(), fwd<Ts>(xs)...); }
    constexpr auto operator() <F,T> (F&& f, T&& obj) const
        noexcept(...) ->decltype(...) requires (...)
    { return apply_mem_data{}(f, fwd<T>(obj)); }
    constexpr auto operator() <F,T> (F&& f, T&& obj) const
        noexcept(...) ->decltype(...) requires (...)
    { return apply_mem_data{}(f, *fwd<T>(obj)); }
    constexpr auto operator() <F,T> (F&& f, const std::reference_wrapper<T>& ref) const
        noexcept(...) ->decltype(...) requires (...)
    { return apply_mem_data{}(f, ref.get()); }
};

struct detail::eval_helper<R>;
struct detail::apply_eval_f {
    constexpr R operator() <F,...Ts,R=decltype(...)> (const F& f, Ts&&...xs) const
        noexcept(...) requires (...)
    { return eval_ordered<R>(f, pack(), fwd<Ts>(xs)...); }
    constexpr holder<Ts...>::type operator() <F,...Ts,R=decltype(...)> (const F& f, Ts&&...xs) const
        noexcept(...) requires (...)
    { return {eval_ordered<R>(f, pack(), fwd<Ts>(xs)...)}; }
};

constexpr apply_f const& apply = apply_f{}; // INVOKE
constexpr apply_eval_f const& apply_eval = apply_eval_f{}; // = f(arg()...), ordered
```

##### `pack`

```c++
struct detail::pack_tag<...>{};
struct detail::pack_holder<T,Tag> : alias_empty<T,Tag>{};
struct detail::is_copyable<T> : std::bool_constant<IS_CONSTRUCTIBLE(T, const T&)>{};
struct detail::is_copyable<T{&|&&}> : std::true_type{};
constexpr T detail::pack_get<T,Tag,X,...Ts>(X&&x, TS&&...xs) noexcept(...) requires(!is_lvalue_reference_v<T>)
{ return (T)alias_value<Tag,T>(fwd<X>(x), xs...); }
constexpr T detail::pack_get<T,Tag,X,...Ts>(X&&x, TS&&...xs) noexcept requires(is_lvalue_reference_v<T>)
{ return alias_value<Tag,T>(x, xs...); }
constexpr auto detail::pack_get<T,Tag,X,...Ts>(X&& x, Ts&&...xs) noexcept(...) ->decltype(...)
{ return alias_value<Tag,T>(fwd<X>(x), xs...); }

struct detail::pack_base<Seq,...Ts>;
struct detail::pack_base<seq<ns...>,Ts...> : pack_holder<Ts,pack_tag<seq<ns>,Ts...>>::type... {
    constexpr ctor<...Xs>(Xs&&...xs) noexcept(...) requires(...) : base(fwd<xs>(xs)...) {}
    constexpr auto operator() <F> (F&& f) const noexcept(...) -> decltype(...)
    { return f(pack_get<Ts, pack_tag<seq<ns>, Ts...>>(*this, f)...); }
    using fit_function_param_limit = std::integral_constant<size_t, sizeof...(Ts)>;
    struct apply<F> : F::apply<Ts...> {};
};

constexpr auto detail::unpack_pack_base<F,...ns,...Ts>(F&& f, pack_base<seq<Ns...>, Ts...> const& x) noexcept(...) -> decltype(...)
{ return f(alias_value<pack_tag<seq<Ns>, Ts...>, Ts>(boost::hof::detail::lvalue(x), f)...); }
constexpr auto detail::unpack_pack_base<F,...ns,...Ts>(F&& f, pack_base<seq<Ns...>, Ts...> & x) noexcept(...) -> decltype(...)
{ return f(alias_value<pack_tag<seq<Ns>, Ts...>, Ts>(boost::hof::detail::lvalue(x), f)...); }
constexpr auto detail::unpack_pack_base<F,...ns,...Ts>(F&& f, pack_base<seq<Ns...>, Ts...> && x) noexcept(...) -> decltype(...)
{ return f(alias_value<pack_tag<seq<Ns>, Ts...>, Ts>(boost::hof::move(x), f)...); }

struct detail::pack_join_base<pack_base<seq<ns1...>,Ts1...>,pack_base<seq<ns2...>,Ts2...>> {
    using result_type = pack_base<gens<sizeof...(Ts1)+sizeof...(Ts2)>::type, Ts1..., Ts2...>;
    static constexpr result_type call<P1,P2>(P1&& p1, P2&& p2) noexcept(...)
    { return {pack_get<Ts1, pack_tag<seq<ns1>, Ts1...>>(fwd<P1>(p1))..., pack_get<Ts1, pack_tag<seq<ns2>, Ts2...>>(fwd<P2>(p2))...}}
};
struct detail::pack_join_result<P1,P2> : pack_join_base<std::remove_cvref_t<P1>, std::remove_cvref_t<P2>> {};
struct detail::pack_basic_f {
    constexpr auto operator() <...Ts> (Ts&&...xs) const noexcept(...) ->decltype(...)
    { return pack_base<gens<sizeof...(Ts)>::type, remove_rvalue_reference<Ts>::type...>(fwd<Ts>(xs)...); }
};
struct detail::pack_forward_f {
    constexpr auto operator() <...Ts> (Ts&&...xs) const noexcept(...) ->decltype(...)
    { return pack_base<gens<sizeof...(Ts)>::type, Ts&&...>(fwd<Ts>(xs)...); }
};
struct detail::pack_f {
    constexpr auto operator() <...Ts> (Ts&&...xs) const noexcept(...) ->decltype(...)
    { return pack_basic_f{}(decay(fwd<Ts>(xs))...); }
};
constexpr pack_join_result<P1,P2>::result_type detail::make_pack_join_dual<P1,P2>(P1&& p1, P2&& p2) noexcept(...) ->decltype(...)
{ return pack_join_result<P1,P2>::call(fwd<P1>(p1), fwd<P2>(p2)); }

struct detail::join_type<...Ts>;
struct detail::join_type<T> { using type = T; };
struct detail::join_type<T,Ts...> { using type = pack_join_result<T,join_type<Ts...>::type>::result_type; };
constexpr P1 detail::make_pack_join<P1>(P1&& p1) noexcept(...) { return fwd<P1>(p1); }
constexpr join_type<P1,Ps...>::type detail::make_pack_join<P1,...Ps>(P1&& p1, Ps&&...ps) noexcept(...) ->decltype(...)
{ return make_pack_join_dual(fwd<P1>(p1), make_pack_join(fwd<Ps>(ps)...)); }
struct pack_join_f {
    constexpr auto operator()<...Ps>(Ps&&...ps) const noexcept(...) ->decltype(...) { return make_pack_join(fwd<Ps>(ps)...); }
};

constexpr auto const& pack_basic = pack_basic_f{};
constexpr auto const& pack_forward = pack_forward_f{};
constexpr auto const& pack = pack_f{};
constexpr auto const& pack_join = pack_join_f{};

struct unpack_sequence<pack_base<T,Ts...>> {
    constexpr static auto apply<F,P>(F&& f, P&& p) noexcept(...) ->decltype(...)
    { return unpack_pack_base(fwd<F>(f), fwd<P>(p)); }
};
```


constexpr_deduce, pp, recursive_consexpr_depth, result_type, unpack_tuple

capture, compose, decorate, eval, first_of, fix, flip, flow, fold, function_param_limit, function,
if, implicit, indirect, infix, is_unpackable, lambda, lift, limit
match, mutable, partial, pipable, proj, repeat_while, repeat
result, reverse_fold, rotate, static, tap, unpack, version
