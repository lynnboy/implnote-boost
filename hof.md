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

constexpr gens<std::tuple_size<Seq>::value>::type detail::make_tuple_gens<Seq>(const Seq&) { return {}; }
constexpr auto detail::unpack_tuple<F,T,...n>(F&& f, T&& t, seq<n...>) noexcept(...) ->decltype(...) { return f(std::get<n>(t)...); }
struct detail::unpack_tuple_apply {
    constexpr static auto apply<F,S>(F&& f, S&& t) noexcept(...) ->decltype(...) { return unpack_tuple(fwd<F>(f), fwd<S>(t), make_tuple_gens(t)); }
};
struct unpack_sequence<std::tuple<Ts...>> : unpack_tuple_apply {};
struct unpack_sequence<std::pair<T,U>> : unpack_tuple_apply {};
struct unpack_sequence<std::array<T,n>> : unpack_tuple_apply {};

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

struct detail::function_result_type<F,_=void>{};
struct detail::function_result_type<F,holder<F::result_type>>{ using result_type = F::result_type; };
struct detail::compose_function_result_type<F,G,_=void> : function_result_type<F>{};
struct detail::compose_function_result_type<F,G,holder<...>> { using result_type = decltype(declval<F>()(declval<G::result_type>())); };

struct detail::constexpr_deduce{ constexpr operator F() const { return F{}; } };
struct detail::constexpr_deduce_unique<T> { constexpr operator F() const { return F{}; } };
```

##### `alias` - tagged wrapper

```c++
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
```

#### Macros:
##### Language macros
* `USING(n,...)`, `USING_TEMPLATE(n,...)`
* `NOEXCEPT(...)`, `NOEXCEPT_CONSTRUCTIBLE(...)`
* `STATIC_CONSTEXPR`, `STATIC_AUTO_REF`,
* `STATIC_CONST_VAR(v)`, `DECLARE_STATIC_VAR(v,...)`
    * Variables are actually: `static constexpr auto& VAR = static_const_var_factory() = INITIALIZER;`
* `INHERIT_DEFAULT(C,...)`, `INHERIT_DEFAULT_EMPTY(C,...)` define default ctor
* `DELEGATE_CONSTRUCTOR(C,T,v)`, `INHERIT_CONSTRUCTOR(Derived,Base)`
* `SFINAE_<MANUAL>_RESULT(...)` = `auto`, `SFINAE_<MANUAL>_RETURNS` = `RETURNS`
* `JOIN(c,...)` = `c<__VA_ARGS__>`
* `RECURSIVE_CONSTEXPR_DEPTH` = 16
##### Library macros
* `IS_XXX(...)` = `std::is_xxx<__VA_ARGS>::value` for type traits
* `FORWARD(...)`
* `ENABLE_IF_CONVERTIBLE`, `ENABLE_IF_CONVERTIBLE_UNPACK`, `ENALBE_IF_BASE_OF`, `ENABLE_IF_CONSTRUCTIBLE`

------
### Functions

##### `always` - wrap and return fixed value

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

##### `arg` - forward return n-th arg

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

##### `construct` - factories

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

##### `decay` - forward decayed arg

```c++
struct detail::decay_mf<T> : unwrap_reference<std::decay_t<T>> {};
struct detail::decay_f {
    constexpr R operator() <T,R=unwrap_reference<std::decay_t<T>>::type> (T&& x) const
        noexcept(...) requires(...) { return fwd<T>(x); }
};
constexpr decay_f const& decay = decay_f{};
```

##### `identity` - forwards its arg

```c++
struct detail::identity_base{
    constexpr T operator() <T> (T&& x) const noexcept(...) { return fwd<T>(x); }
    constexpr <const> std::initializer_list<T>& operator() <T> (<const> std::initializer_list<T>& x) const noexcept(...) { return x; }
    constexpr std::initializer_list<T> operator() <T> (std::initializer_list<T>&& x) const noexcept(...) { return fwd<std::initializer_list<T>(x); }
};
constexpr identity_base const& identity = identity_base{};
```

##### Placeholders - supports operators to create bind expressions

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

##### `combine` - zip each function on each arg

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

##### `compose` - function composition

```c++
struct detail::compose_kernel<F1,F2> : compressed_pair<F1,F2>, compose_function_result_type<F1,F2> {
    constexpr auto operator()<...Ts> const noexcept(...) ->result_of<const F1&, result_of<const F2&,id_<Ts>...>>
    { return first(xs...)(second(xs...)(fwd<Ts>(xs)...)); }
};

struct compose_adaptor<F,...Fs> : compose_kernel<callable_base<F>,JOIN(compose_adaptor,callable_base<Fs>...)> {
    using fit_rewritable_tag = self; using tail = JOIN(compose_adaptor,callable_base<Fs>...);
    constexpr ctor<X,...Xs> (X&& f1, Xs&&...fs) noexcept(...) requires(...):base(fwd<X>(f1)),tail(fwd<Xs>(fs)...) {}
    constexpr ctor<X> (X&& f1) noexcept(...) requires(...) :base(fwd<X>(f1)) {}
};
struct compose_adaptor<F> : callable_base<F> { using fit_rewritable_tag = self;
    constexpr ctor<X> (X&& f1) noexcept(...) requires(...) :base(fwd<X>(f1)) {}
};
struct compose_adaptor<F1,F2> : compose_kernel<callable_base<F1>,callable_base<F2>> { using fit_rewritable_tag = self; };

constexpr auto const& compose = make<compose_adaptor>();
// assert( compose(f,g)(xs...) == f(g(xs...)) );
```

##### `decorate` - create decorator function

```c++
struct detail::decorator_invoke<D,T,F> : compressed_pair<compressed_pair<D,T>,F> {
    constexpr const compressed_pair<D,T>& get_pair<...Ts>(Ts&&...xs) const noexcept { return first(xs...); }
    constexpr const F& base_function<...Ts>(Ts&&...xs) const noexcept { return second(xs...); }
    constexpr const D& get_decorator<...Ts>(Ts&&...xs) const noexcept { get_pair(xs...).first(xs...); }
    constexpr const T& get_data<...Ts>(Ts&&...xs) const noexcept { get_pair(xs...).second(xs...); }
    struct decorator_invoke_failure {
        struct apply<Failure> { struct of<...Ts> : Failure::of<const T&, const F&, Ts...> {}; };
    };
    struct failure : failure_map<decorator_invoke_failure, D> {};
    constexpr auto operator() <...Ts> (Ts&&...xs) const noexcept(...) ->result_of<const D&, id_<const T&>, id_<const F&>, id_<Ts>...>
    { return get_decorator(xs...)(get_data(xs...), base_function(xs...), fwd<Ts>(xs)...); }
};
struct detail::decoration<D,T> : compressed_pair<D,T> {
    constexpr const D& get_decorator <...Ts> (Ts&&...xs) const noexcept { return first(xs...); }
    constexpr const T& get_data <...Ts> (Ts&&...xs) const noexcept { return second(xs...); }
    constexpr auto operator() <F> (F f) const noexcept(...) ->decorator_invoke<D, T, detail::callable_base<F>>
    { return {*this, (callable_base<F>&&)f}; }
};
struct decorate_adaptor<F> : callable_base<F> {
    using fit_rewritable1_tag = self;
    constexpr const base& base_function<...Ts>(Ts&&...xs) const noexcept { return always_ref(*this)(xs...); }
    constexpr auto operator() <T> (T f) const noexcept(...) ->decoration<base,T> { return {base_function(x), (T&&)x}; }
};

constexpr auto const& decorate = make<decorate_adaptor>();
// auto decorator = decorate([](auto arg, auto f, auto v...){/*do sth with arg*/ return f(v...);});
```

##### `first_of` - call first viable overloads

```c++
struct detail::basic_first_of_adaptor<F1,F2>: F1,F2 {
    constexpr ctor<A,B>(A&& f1, B&& f2) noexcept(...) requires(...);
    constexpr ctor<X>(X&& x) noexcept(...) requires(...);
    constexpr auto operator() <...Ts,F=select<Ts...>::type> (Ts&&...xs) const
        noexcept(...) -> result_of<select<Ts...>::type, id_<Ts>...> { return (*this)(fwd<Ts>(xs)...); }
};
struct detail::conditional_kernel<F1,F2> : compressed_pair<F1,F2> {
    constexpr auto operator() <...Ts> (Ts&&...xs) const noexcept(...) -> result_of<select<Ts...>::type, id_<Ts>...>
    { if constexpr (is_invocable<F1,Ts...>::value) return first(xs...)(fwd<Ts>(xs)...) : second(xs...)(fwd<Ts>(xs)...); }
};

struct first_of_adaptor<F,...Fs> : conditional_kernel<F,JOIN(first_of_adaptor,Fs...)> {
    using fit_rewritable_tag = self; using kernel_base = JOIN(self, Fs...);
    constexpr ctor<X,...Xs>(X&& f1, Xs&&...fs) noexcept(...) requires(...);
    constexpr ctor<X,...Xs>(X&& f1) noexcept(...) requires(...);
    struct failure : failure_for<F,Fs...> {};
};
struct first_of_adaptor<F> : F { using fit_rewritable_tag = self; struct failure : failure_for<F> {}; };
struct first_of_adaptor<F1,F2> : conditional_kernel<F1,F2> {
    using fit_rewritable_tag = self;
    struct failure : failure_for<F1,F2> {};
};

constexpr auto const& first_of = make<first_of_adaptor>();
// first_of([](int){return 1;},[](float){return 2;})(3.0) == 1
```

##### `fix` - fixed-point combinator

```c++
struct detail::compute_indirect_ref<F> { using type = indirect_adaptor<const F*>; };
struct detail::compute_indirect_ref<indirect_adaptor<F*>> { using type = indirect_adaptor<F*>; }
constexpr indirect_adaptor<const F*> detail::make_indirect_ref<F>(const F& f) noexcept { return {&f}; }
constexpr indirect_adaptor<const F*> detail::make_indirect_ref<F>(const indirect_adaptor<F*>& f) noexcept { return f; }

struct detail::fix_result<F,_=void>{ struct apply<...Ts> { using type = decltype(declval<F>()(delcval<Ts>()...)); }; };
struct detail::fix_result<F,holder<F::result_type>::type> { struct apply<...> { using type = F::result_type; }; };
struct detail::fix_adaptor_base<F,R,n> : F {
    using base_ref_type = compute_indirect_ref<F>::type; using derived = self<base_ref_type, R, n-1>; // recursion
    constexpr const F& base_function<...Ts>(Ts&&...xs) const noexcept { return always_ref(*this)(xs...); }
    constexpr derived derived_function<...Ts>(Ts&&...xs) const noexcept { return {make_indirect_ref(base_function(xs...))}; }
    struct fix_failure { struct apply<Failure> { struct of<...Ts> : Failure::of<derived, Ts...>{}; }; };
    struct failure : failure_map<fix_failure, F> {};
    constexpr auto operator() <...Ts> (Ts&&...xs) const noexcept(...) ->result_of<const F&, id_<derived>, id_<Ts>...>::type
    { return base_function(xs...)(derived_function(xs...), fwd<Ts>(xs)...); }
};
struct detail::fix_adaptor_base<F,R,0> : F {
    const F& base_function<...Ts>(Ts&&...) const noexcept { return *this; }
    struct fix_failure { struct apply<Failure> { struct of<...Ts> : Failure::of<self, Ts...>{}; }; };
    struct failure : failure_map<fix_failure, F> {};
    auto operator() <...Ts> (Ts&&...xs) const->R::apply<self,Ts...>::type
    { return base_function(xs...)(*this, fwd<Ts>(xs)...); }
};

struct fix_adaptor<F> : fix_adaptor_base<F, fix_result<F>, RECURSIVE_CONSTEXPR_DEPTH> { using fit_rewritable1_tag = self; };
struct result_adaptor<R,fix_adaptor<F>> : fix_adaptor<self<R,F>> {};

constexpr auto const& fix = make<fix_adaptor>();
// assert( fix(f)(xs...) == f(fix(f), xs...) );
```

##### `flip` - swap first two args

```c++
struct flip_adaptor<F> : callable_base<F> {
    using fit_rewritable1_tag self;
    base& base_function<...Ts>(Ts&&...xs) const { return always_ref(*this)(xs...); }
    struct flip_failure { struct apply<Failure> { struct of<T,U,...Ts> : Failure::of<U,T,Ts...>{}; }; };
    struct failure : failure_map<flip_failure, callable_base<F>>{};
    constexpr auto operator()<T,U,...Ts>(T&& x, U&& y, Ts&&...xs) const noexcept(...) ->decltype(...) requires(...)
    { base_function(xs...)(fwd<U>(y), fwd<T>(x), fwd<Ts>(xs)...); }
};

constexpr auto const& flip = make<flip_adaptor>();
// assert( flip(f)(x,y,xs...) == f(y,x,xs...) );
```

##### `flow` - invoke functions one by one

```c++
struct detail::flow_kernel<F1,F2> : compressed_pair<callable_base<F1>,callable_base<F2>>, compose_function_result_type<F2,F1> {
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) ->result_of<const callable_base<F2>&, result_of<const callable_base<F1>&, id_<Ts>...>>
    { return second(xs...)(first(xs...)(fwd<Ts>(xs)...)); }
};
struct flow_adaptor<F,...Fs> : flow_kernel<F,JOIN(flow_adaptor,Fs...)> {
    using fit_rewritable_tag self; using tail = JOIN(flow_adaptor,Fs...);
    constexpr ctor<X,...Xs>(X&& f1, Xs&&...fs) noexcept(...) requires(...) : base(fwd<X>(f1), tail{fwd<Xs>(fs)...}){}
    constexpr ctor<X>(X&& f1) noexcept(...) : base(fwd<X>(f1)) {}
};
struct flow_adaptor<F> : callable_base<F> {
    using fit_rewritable_tag self;
    constexpr ctor<X>(X&& f1) noexcept(...) requires(...) : base(fwd<X>(f1)) {}
};
struct flow_adaptor<F1,F2> : flow_kernel<callable_base<F1>,callable_base<F2>> { using fit_rewritable_tag self; };

constexpr auto const& flow = make<flow_adaptor>();
// assert( flow(f,g)(xs...) == g(f(xs...)) );
```

##### `fold` - invoke binary function on sequence

```c++
struct detail::v_fold {
    constexpr auto operator()<F,State,T,...Ts>(const F& f, State&& state, T&& x, Ts&&...xs) const noexcept(...)
        -> result_of<const v_fold&, id_<const F&>, result_of<const F&, id_<State>, id_<T>>, id_<Ts>...>
    { (*this)(f, f(fwd<State>(state), fwd<T>(x)), fwd<Ts>(xs)...); }
    constexpr State operator()<F,State>(const F&, State&& state) const noexcept { return fwd<State>(state); }
};
struct fold_adaptor<F,_=void> : compressed_pair<callable_base<F>, State> {
    constexpr const callable_base<F>& base_function<...Ts>(Ts&&...xs) const noexcept { return first(xs...); }
    constexpr State get_state<...Ts>(Ts&&...xs) const noexcept { return second(xs...); }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) -> result_of<v_fold, id_<const callable_base<F>&>, id_<State>, id_<Ts>...>
    { return v_fold{}(base_function(xs...), get_state(xs...), fwd<Ts>(xs)...); }
};
struct fold_adaptor<F,void> : callable_base<F> {
    constexpr const callable_base<F>& base_function<...Ts>(Ts&&...xs) const noexcept { return always_ref(*this)(xs...); }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) -> result_of<v_fold, id_<const callable_base<F>&>, id_<Ts>...>
    { return v_fold{}(base_function(xs...), fwd<Ts>(xs)...); }
};

constexpr auto const& fold = make<fold_adaptor>();
// assert( fold(f,z)() == z ); // z is init state
// assert( fold(f)(x, xs...) == fold(f,x)(xs...) );
// assert( fold(f,z)(x, xs...) == fold(f, f(z,x))(xs...) );
```

##### `implicit` - (static) deduce template parameter type

```c++
struct detail::is_implicit_callable<F,Pack,X>; // is_convertible< decltype(pack(f)), X >
struct implicit<F<...>> {
    struct invoker<Pack> { Pack p;
        constexpr ctor(Pack pp) noexcept(...) : p{move(pp)} {}
        constexpr operator X() const noexcept(...) requires (...) { return p(F<X>()); }
    };
    struct make_invoker { constexpr invoker<Pack> operator()<Pack>(Pack p) const noexcept(...) {return {move(p)}; } };
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) ->decltype(...)
    { return make_invoker{}(pack_basic(fwd<Ts>(xs)...)); }
};
```

##### `indirect` - dereference before call

```c++
struct indirect_adaptor<F> : F {
    using fit_rewritable1_tag = self;
    constexpr const F& base_function<...Ts>(Ts&&...xs) const noexcept { return always_ref(*this)(xs...); }
    struct failure : failure_for<decltype(*std::declval<F>())> {};
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) -> result_of<decltype(*std::declval<F>()), id_<Ts>...>
    { return (*base_function(xs...))(fwd<Ts>(xs)...); }
};
struct indirect_adaptor<F*> {
    using fit_rewritable1_tag = self; F* f;
    constexpr ctor() noexcept{}  constexpr ctor(F* x) noexcept :f{x}{}
    constexpr F& base_function<...Ts>(Ts&&...xs) const noexcept { return *f; }
    struct failure : failure_for<F> {};
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) -> result_of<F, id_<Ts>...>
    { return base_function(xs...)(fwd<Ts>(xs)...); }
};

constexpr auto const& indirect = make<indirect_adaptor>();
// assert( indirect(f)(xs...) == (*f)(xs...) );
```

##### `infix` - make function work as binary-operator

```c++
struct detail::postfix_adaptor<T,F> : F {
    T x;
    constexpr ctor<X,XF>(X&& xp, XF&& fp) noexcept(...);
    constexpr const F& base_function<...Ts>(Ts&&...xs) const noexcept { return always_ref(*this)(xs...); }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...)->result_of<const F&, id_<T&&>, id_<Ts>...> requires(...)
    { base_function(xs...)((T&&)x, fwd<Ts>(xs)...); }
    constexpr auto operator> <A> (A&& a) const noexcept(...) ->result_of<const F&, id_<T&&>, id_<A>> requires(...)
    { base_function(a)((T&&)x, fwd<A>(a)); }
};
constexpr postfix_adaptor<T,F> detail::make_postfix_adaptor<T,F>(T&& x, F f) noexcept(...) { return {fwd<T>(x), (F&&)f}; }

struct infix_adaptor<F> : callable_base<F> {
    using fit_rewritable1_tag = self;
    constexpr const callable_base<F>& <infix>_base_function<...Ts>(Ts&&...xs) const noexcept { return always_ref(*this)(xs...); }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...)->decltype(...) { return base_function(xs...)(fwd<Ts>(xs)...); }
};
constexpr auto operator< <T,F> (T&& x, const infix_adaptor<F>& i) noexcept(...) ->decltype(...)
{ return make_postfix_adaptor(fwd<T>(x), move(i.base_function(x))); }

auto detail::operator< <T,F>(T&& x, const static_function_wrapper<F>& f) noexcept(...) ->decltype(...)
{ return make_postfix_adaptor(fwd<T>(x), move(f.base_function().infix_base_function())); }
auto detail::operator< <T,F>(T&& x, const static_default_function<F>& f) noexcept(...) ->decltype(...)
{ return make_postfix_adaptor(fwd<T>(x), f.infix_base_function()); }

constexpr auto const& infix = make<infix_adaptor>{};
// assert( x <infix(f)> y == f(x,y) );
```

##### `lazy` - constexpr simple wrapper

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
// assert( lazy(f)(xs...) == std::bind(f, xs...) );
// assert( lazy(f)(xs...)() == f(xs...) );
// assert( lazy(f)(_1)(x) == f(x) );

struct std::is_bind_expression<lazy_invoker<F,Pack>> : std::true_type {};
struct std::is_bind_expression<lazy_nullary_invoker<F>> : std::true_type {};
```

##### `match` - overload resolution

```c++
struct match_adaptor<...Fs>;
struct match_adaptor<F,Fs...> : callable_base<F>, match_adaptor<Fs...> {
    using fit_rewritable_tag = self;
    struct failure : failure_for<callable_base<F>, Fs...>{};
    constexpr ctor<X,...Xs>(X&& f1, Xs&&...fs) requires(...) {}
};
struct match_adaptor<F> : callable_base<F> {};

constexpr auto const& match = make<match_adaptor>{};
```

##### `mutable` - wrap const function object

```c++
struct mutable_adaptor {
    mutable F f;
    auto operator()<...Ts>(Ts&&...xs) const noexcept(...)->result_of<F, id_<Ts>...> { return f(fwd<Ts>(xs)...); }
};

constexpr auto const& mutable_ = make<mutable_adaptor>{};
```

##### `partial` - partial application (currying)

```c++
struct detail::partial_adaptor_invoke<Derived,F,Pack> {
    constexpr const F& get_function<...Ts>(Ts&&...) const noexcept { return (const F&)(const Derived&)*this; }
    constexpr const Pack& get_pack<...Ts>(Ts&&...) const noexcept { return (const Pack&)(const Derived&)*this; }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) -> result_of<...>
    { return pack_join(get_pack(xs...), pack_forward(fwd<Ts>(xs)...)); }
};
struct detail::partial_adaptor_join<Derived,F,Pack> {
    constexpr const F& get_function<...Ts>(Ts&&...) const noexcept { return (const F&)(const Derived&)*this; }
    constexpr const Pack& get_pack<...Ts>(Ts&&...) const noexcept { return (const Pack&)(const Derived&)*this; }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) -> result_of<...> requires(...)
    { return partial((F&&)get_function(xs...), pack_join(get_pack(xs...), pack(fwd<Ts>)(xs)...)); }
};
struct detail::partial_adaptor_pack<Derived,F> {
    constexpr const F& get_function<...Ts>(Ts&&...) const noexcept { return (const F&)(const Derived&)*this; }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) -> decltype(...) requires(...)
    { return partial((F&&)get_function(xs...), pack(fwd<Ts>)(xs)...); }
};
struct detail::partial_adaptor_base<F,Pack> { using type = basic_first_of_adaptor<partial_adaptor_invoke<partial_adaptor<F,Pack>,F,Pack>, partial_adaptor_join<partial_adaptor<F,Pack>, F, Pack>>; };
struct detail::partial_adaptor_pack_base<Derived,F> { using type = basic_first_of_adaptor<F,partial_adaptor_pack<Derived,F>>; };

struct partial_adaptor<F,Pack=void> : partial_adaptor_base<F,Pack>::type, F, Pack {
    using fit_rewritable1_tag = self;
    constexpr ctor<X,S>(X&& x, S&& seq) noexcept(...) : F{fwd<X>(x)}, Pack{fwd<S>(seq)}{}
    constexpr const F& base_function<...Ts>(Ts&&...) const noexcept { return *this; }
    constexpr const Pack& get_pack() const noexcept { return *this; }
};
struct partial_adaptor<F,void> : partial_adaptor_pack_base<partial_adaptor<F,void>, callable_base<F>>::type {
    using fit_rewritable1_tag = self;
    constexpr const callable_base<F>& base_function<...Ts>(Ts&&...) const noexcept { return *this; }
};
struct partial_adaptor<pipable_adaptor<F>,void> : partial_adaptor<F,void> { using fit_rewritable1_tag self; };
struct partial_adaptor<static_<pipable_adaptor<F>>,void> : partial_adaptor<F,void> { using fit_rewritable1_tag self; };

constexpr auto& partial = make<partial_adaptor>();
// static( partial(f)(xs...)(ys...) == f(xs..., ys...) );
```

##### `pipable` - add pipe (`|`) chaining

```c++
struct detail::pipe_closure<F,Pack>: F, Pack {
    constexpr ctor<X,P>(X&& fp, P&& packp) noexcept(...) : F{fwd<X>(fp)}, Pack{fwd<P>(packp)} {}
    constexpr const F& base_function<...Ts>(Ts&&...) const noexcept { return *this; }
    constexpr const Pack& get_pack<...Ts>(Ts&&...) const noexcept { return *this; }
    struct invoke<A>{ A a; const pipe_closure* self; constexpr ctor<X>(X&& xp, const pipe_closure* selfp);
        constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) ->result_of<const F&,id_<A>,id_<Ts>...>::type
        { return ((const F&)(*self))(fwd<A>(a), fwd<Ts>(xs)...); }
    };
    constexpr auto operator()<A>(A&& a) const noexcept(...) ->result_of<const Pack&, id_<invoke<A&&>>>::type
    { return get_pack(a)(invoke<A&&>{fwd<A>(a), this}); }
};
constexpr auto detail::make_pipe_closure<F,Pack>(F f, Pack&& p) noexcept(...) ->decltype(...)
{ return pipe_closure<F,std::remove_reference_t<Pack>>{(F&&)f, fwd<Pack>(p)}; }

struct detail::pipe_pack<Derived,F> {
    constexpr const F& get_function<...Ts>(Ts&&...) const noexcept { return (const F&)(const Derived&)*this; }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) ->decltype(...) requires(...)
    { return make_pipe_closure((F&&)get_function(xs...), pack_forward(fwd<Ts>(xs)...)); }
};
constexpr auto detail::operator| <A,F,Pack>(A&& a, const pipe_closure<F,Pack>& p) noexcept(...) ->decltype(...) { return p(fwd<A>(a)); }

struct pipable_adaptor<F> : basic_first_of_adaptor<callable_base<F>, pipe_pack<pipable_adaptor<F>, callable_base<F>>> {
    using fit_rewritable_tag = self;
    constexpr const callable_base<F>& base_function() const noexcept { return *this; }
};
constexpr auto detail::operator| <A,F>(A&& a, const pipable_adaptor<F>& p) noexcept(...) ->decltype(...) { return p(fwd<A>(a)); }

constexpr auto detail::operator| <A,F>(A&& a, const static_function_wrapper<F>& p) noexcept(...) ->decltype(...) { return p(fwd<A>(a)); }
constexpr auto detail::operator| <A,F>(A&& a, const static_default_function<F>& p) noexcept(...) ->decltype(...) { return p(fwd<A>(a)); }
constexpr auto operator| <A,F>(A&& a, static_<F> f) noexcept(...) ->decltype(...) { return f.base_function().base_function()(fwd<A>(a)); }

constexpr auto& pipable = make<pipable_adaptor>();
// assert( x | pipable(f)(ys...) == f(x, ys...) );
```

##### `proj` - apply projection on each arg

```c++
struct detail::project_eval<T,Proj> { T&& x; const Proj& p; constexpr ctor<X,P>(X&& xp, const P& pp);
    constexpr auto operator()() const noexcept(...) ->decltype(...) { return p(fwd<T>(x)); }
};
constexpr project_eval<T,Proj> detail::make_project_eval<T,Proj>(T&& x, const Proj& p) { return {fwd<T>(x), p}; }

struct detail::project_void_eval<T,Proj> { T&& x; const Proj& p; constexpr ctor<X,P>(X&& xp, const P& pp);
    struct void_{}; constexpr void_ operator()() const { p(fwd<T>(x)); return void_{}; }
};
constexpr project_void_eval<T,Proj> detail::make_project_void_eval<T,Proj>(T&& x, const Proj& p) { return {fwd<T>(x), p}; }

constexpr R detail::by_eval<Proj,F,...Ts,R=decltype(...)>(const Proj& p, const F& f, Ts&&...xs)
{ return apply_eval(f, make_project_eval(fwd<Ts>(xs), p)...); }
constexpr void detail::by_void_eval<Proj,...Ts>(const Proj& p, Ts&&...xs)
{ return apply_eval(always(), make_project_void_eval(fwd<Ts>(xs), p)...); }

struct proj_adaptor<Proj,F=void> : compressed_pair<callable_base<Proj>,callable_base<F>>, function_result_type<F> {
    using fit_rewritable_tag = self;
    constexpr const callable_base<F>& base_function<...Ts>(Ts&&...xs) const { return second(xs...); }
    constexpr const callable_base<Proj>& base_projection<...Ts>(Ts&&...xs) const { return first(xs...); }
    struct by_failure { struct apply<Failure> { struct of<...Ts> : Failure::of<decltype(...)>{}; }; };
    struct failure : failure_map<by_failure, callable_base<F>> {};
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) ->result_of<...>::type
    { return by_eval(base_projection(xs...), base_function(xs...), fwd<Ts>(xs)...); }
};
struct proj_adaptor<Proj,void> : callable_base<Proj> {
    using fit_rewritable1_tag = self;
    constexpr const callable_base<Proj>& base_projection<...Ts>(Ts&&...xs) const { return always_ref(*this)(xs...); }
    constexpr void operator()<...Ts>(Ts&&...xs) const requires(...) { return by_void_eval(base_projection(xs...), fwd<Ts>(xs)...); }
};

constexpr auto& proj = make<proj_adaptor>();
// assert( proj(p,f)(xs...) == f(p(xs)...) );
// assert( proj(p)(xs...) == (void)p(xs)... );
```

##### `protect` - make bind/lazy expression being normal function

```c++
struct protect_adaptor<F> : callable_base<F> { using fit_rewritable1_tag = self; }
constexpr auto const& protect = make<protect_adaptor>{};
// assert( lazy(f)( project( lazy(g)(_1) ) ) == f(lazy(g)(_1)) );
```

##### `result` - (static) change function's result type

```c++
struct result_adaptor<Result,F> : callable_base<F> {
    using result_type = conditional_t<is_same_v<Result,void>,holder<Ts...>::type,Result>;
    struct failure : failure_for<callable_base<F>> {};
    constexpr const callable_base<F>& base_function<...Ts>(Ts&&...xs) const { return always_ref(*this)(xs...); }
    constexpr result_type operator()<...Ts>(Ts&&...xs) const { return base_function(xs...)(fwd<Ts>(xs)...); }
};
struct detail{ struct result_f<Result> { constexpr result_adaptor<Result,F> operator()(F f) const { return {move(f)}; } }; }

constexpr auto result<Result> = result_f<Result>;
// assert( std::is_same_v<decltype( result<int>{[]{return true;}}() ), int> );
```

##### `reveal` - report template substitution errors

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

##### `reverse_fold` - fold from right to left

```c++
struct detail::v_reverse_fold {
    constexpr auto operator()<F,State,T,...Ts>(const F& f, State&& state, T&& x, Ts&&...xs) const noexcept(...) -> result_of<...>
    { f((*this)(f, fwd<State>(state), fwd<Ts>(xs)...), fwd<T>(x)); }
    constexpr State operator()<F,State>(const F&, State&& state) const noexcept { return fwd<State>(state); }
};
struct reverse_fold_adaptor<F,_=void> : compressed_pair<callable_base<F>, State> {
    constexpr const callable_base<F>& base_function<...Ts>(Ts&&...xs) const noexcept { return first(xs...); }
    constexpr State get_state<...Ts>(Ts&&...xs) const noexcept { return second(xs...); }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) -> result_of<...>
    { return v_reverse_fold{}(base_function(xs...), get_state(xs...), fwd<Ts>(xs)...); }
};
struct reverse_fold_adaptor<F,void> : callable_base<F> {
    constexpr const callable_base<F>& base_function<...Ts>(Ts&&...xs) const noexcept { return always_ref(*this)(xs...); }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) -> result_of<...>
    { return v_reverse_fold{}(base_function(xs...), fwd<Ts>(xs)...); }
};

constexpr auto const& reverse_fold = make<reverse_fold_adaptor>();
// assert( reverse_fold(f,z)() == z ); // z is init state
// assert( reverse_fold(f)(x, xs...) == f(reverse_fold(f)(xs...), x) );
// assert( reverse_fold(f,z)(x, xs...) == f(reverse_fold(f, z)(xs...), x) );
```

##### `rotate` - move arg1 to end

```c++
struct rotate_adaptor<F> : callable_base<F> {
    using fit_rewritable1_tag = self;
    constexpr const callable_base<F>& base_function<...Ts>(Ts&&...xs) const noexcept { return always_ref(*this)(xs...); }
    struct rotate_failure { struct apply<Faiure> { struct of<T,...Ts> : Failure::of<Ts...,T>{}; }; };
    struct failure : failure_map<rotate_failure, callable_base<F>> {};
    constexpr auto operator()<T,...Ts>(T&& x, Ts&&...xs) const noexcept(...) ->result_of<...>::type
    { return base_function(xs...)(fwd<Ts>(xs)..., fwd<T>(x)); }
};

constexpr auto const& rotate = make<rotate_adaptor>();
// assert( rotate(f)(x, xs...) == f(xs..., x) );
```

##### `static` - (static) make function object as static-initialized

```c++
struct static_<F>{
    struct failure : failure_for<F>{};
    const F& base_function() const noexcept(...) { static F f; return f; }
    auto operator()<...Ts>(Ts&&...xs) const ->result_of<F,id_<Ts>...>::type { return base_function()(fwd<Ts>(xs)...); }
};
```

##### `unpack` - make function invocable for tuple (sequence) of args

```c++
constexpr auto detail::unpack_simple<F,Seq>(F&& f, Seq&& s) noexcept(...) ->decltype(...)
{ return unpack_impl(fwd<F>(f), fwd<Seq>(s)); }
constexpr auto detail::unpack_join<F,...Seq>(F&& f, Seq&&...s) noexcept(...) ->decltype(...)
{ return pack_join(unpack_simple(pack_forward, fwd<Seq>(s))...)(fwd<F>(f)); }

struct unpack_adaptor<F> : callable_base<F> {
    using fit_rewritable1_tag = self;
    constexpr const callable_base<F>& base_function<...Ts>(Ts&&...xs) const noexcept { return always_ref(*this)(xs...); }
    struct unpack_failure { struct apply<Failure> { struct of<...Ts> : ...{}; }; };
    struct failure : failure_map<unpack_failure, callable_base<F>> {};
    constexpr auto operator()<T>(T&& x) const noexcept(...) ->decltype(...) requires is_unpackable<T>::value
    { return unpack_simple(base_function(x), fwd<T>(x)); }
    constexpr auto operator()<T,...Ts>(T&& x) const noexcept(...) ->decltype(...) requires is_unpackable<T>::value && (is_unpackable<Ts>::value && ...)
    { return unpack_join(base_function(x), fwd<T>(x), fwd<Ts>(xs)...); }
};

constexpr auto const& unpack = make<unpack_adaptor>();
// assert( unpack(f)(make_tuple(xs...)) == f(xs...) );
```

------
### Decorators

##### `capture` - wrap args for pending function call

```c++
struct detail::capture_invoke<F,Pack> : compressed_pair<callable_base<F>, Pack>, function_result_type<F> {
    using fit_rewritable1_tag = self;
    constexpr const callable_base<F>& base_function<...Ts>(Ts&&...xs) const noexcept { return first(xs...); }
    constexpr const Pack& get_pack<...Ts>(Ts&&...xs) const noexcept { return second(xs...); }
    struct unpack_capture_failure<Failure,...Ts> { struct apply<...Us> { using type = Failure::of<Us..., Ts...>; }; };
    struct capture_failure { struct apply<Failure> { struct of<...Ts> : Pack::apply<unpack_capture_failure<Failure,Ts...>>::type{}; }; };
    struct failure : failure_map<capture_failure, callable_base<F>> {};
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) ->result_of<...>::type
    { return pack_join(get_pack(xs...), pack_forward(fwd<Ts>(xs)...))((callable_base<F>&&)base_function(xs...)); }
};
struct detail::capture_pack<Pack> : Pack {
    constexpr auto operator()<F>(F f) const noexcept(...) ->decltype(...)
    { return capture_invoke<F,Pack>{(F&&)f, (Pack&&)(const Pack&)*always(this)(f)}; }
};
struct detail::make_capture_pack_f {
    constexpr capture_pack<Pack> operator()<Pack>(Pack p) const noexcept(...) { return {(Pack&&)p}; }
};
struct detail::capture_f<F> {
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) -> decltype(...)
    { return make_capture_pack_f{}(F{}(fwd<Ts>(xs)...)); }
};

constexpr auto const& capture_basic = capture_f<pack_basic_f>();
constexpr auto const& capture_forward = capture_f<pack_forward_f>();
constexpr auto const& capture = capture_f<pack_f>();
// assert( capture(xs...)(f)(ys...) == f(xs..., ys...) );
```

##### `if` - make it callable conditionally

```c++
struct detail::if_depend<C,...> : C {};
struct detail::if_adaptor<cond,F> : callable_base<F> {};
struct detail::if_adaptor<false,F> { constexpr ctor<...Ts>(Ts&&...) noexcept {} };
struct detail::make_if_f<cond> { constexpr if_adaptor<cond,F> operator()<F>(F f) const noexcept(...) { return {(F&&)(f)}; } };
struct detail::if_f { cosntexpr make_if_f<Cond::type::value> operator()<Cond>(Cond) const noexcept { return {}; } };

constexpr make_if_f<b> if_c = {};
constexpr auto const& if_ = if_f{};
```

##### `limit` - set max number of args

```c++
struct detail::limit_adaptor<n,F> : callable_base<F> {
    using fit_function_param_limit = std::integral_constant<size_t,n>;
    constexpr const callable_base<F>& base_function<...Ts>(Ts&&...xs) const { return always_ref(*this)(xs...); }
    constexpr auto operator()<...Ts>(Ts&&...xs) const noexcept(...) ->result_of<...>::type requires ((sizeof...(Ts) <= n))
    { return base_function(xs...)(fwd<Ts>(xs)...); }
};
struct detail::make_limit_f<n> { constexpr limit_adaptor<n,F> operator()<F>(F f) const { return {(F&&)f}; } };
struct detail::limit_f { constexpr make_limit_f<N::value> operator()<N>(N) const { return {}; } };
constexpr limit_adaptor<n,F> limit_c<n,F>(F f) { return {(F&&)f}; }
constexpr auto const& limit = limit_f{};
```

##### `repeat`, `repeat_while` - repeatedly invoke function

```c++
struct detail::repeater<n> {
    constexpr auto operator()<F,...Ts>(const F& f, Ts&&...xs) const noexcept(...) ->result_of<...>::type
    { return repeater<n-1>{}(f, f(fwd<Ts>(xs)...)); }
};
struct detail::repeater<0> { constexpr T operator() <F,T> (const F&, T&& x) const noexcept(...) { return x; } };
struct detail::repeat_constant_decorator {
    constexpr auto operator()<Int,F,...Ts>(Int,const F& f, Ts&&...xs) const noexcept(...) ->decltype(...)
    { return repeater<Int::value>{}(f, fwd<Ts>(xs)...); }
};
struct detail::repeat_integral_decorator<depth> {
    constexpr auto operator()<Int,F,T,...Ts>(Int n, const F& f, T&& x, Ts&&...xs) const noexcept(...) ->decltype(...)
    { (n) ? self<depth-1>{}(n-1, f, f(fwd<T>(x), fwd<Ts>(xs)...)) : fwd<T>(x); }
};
struct detail::repeat_integral_decorator<0> {
    constexpr auto operator()<Int,F,T>(Int n, const F& f, T x) const noexcept(...) ->decltype(...)
    { while (n>0) { n--; x = f(fwd<T>(x)); } return x; }
};

constexpr auto const& repeat = decorate_adaptor<first_of_adaptor<repeat_constant_decorator, repeat_integral_decorator<1>>>{};

struct detail::compute_predicate<P,...Ts> { using type = decltype(...); };
struct detail::while_repeater<b> {
    constexpr auto operator()<F,P,...Ts>(const F& f, const P& p, Ts&&...xs) const noexcept(...) -> result_of<...>::type
    { return while_repeater<compute_predicate<P,decltype(f(fwd<Ts>(xs)...))>::value>{}(f, p, f(fwd<Ts>(xs)...)); }
};
struct detail::while_repeater<false> { constexpr T operator()<F,P,T>(const F&, const P&, T&& x) const noexcept(...) { return x; } };
struct detail::repeat_while_constant_decorator {
    constexpr auto operator()<P,F,...Ts>(const P& p, const F& f, Ts&&...xs) const noexcept(...) ->decltype(...)
    { return while_repeater<compute_predicate<P,decltype(f(fwd<Ts>(xs)...))>::value>{}(f, p, fwd<Ts>(xs)...); }
};
struct detail::repeat_while_integral_decorator<depth> {
    constexpr auto operator()<P,F,T,...Ts>(const P& p, const F& f, T&& x, Ts&&...xs) const noexcept(...) ->decltype(...)
    { p(x, fwd<Ts>(xs)...) ? self<depth-1>{}(p, f, f(x, fwd<Ts>(xs)...)) : fwd<T>(x); }
};
struct detail::repeat_while_integral_decorator<0> {
    constexpr auto operator()<P,F,T>(const P& p, const F& f, T x) const noexcept(...) ->decltype(...)
    { while (p(x)) { x = f(fwd<T>(x)); } return x; }
};

constexpr auto const& repeat_while = decorate_adaptor<first_of_adaptor<repeat_while_constant_decorator, repeat_while_integral_decorator<1>>>{};
```

------
### Type Traits

##### `function_param_limit`

```c++
struct function_param_limit<F,_=void> : std::integral_constant<size_t,SIZE_MAX>{};
struct function_param_limit<F> : F::fit_function_param_limit {};
```

##### `is_invocable`

```c++
struct is_invocable<F,...Ts> : can_be_called<apply_f, F, Ts...>{};
struct is_invocable<F(Ts...), Us...> requires !std::is_same_v<F,F> {};
```

##### `is_unpackable`

```c++
struct is_unpackable<Seq> : ... {};
```

##### `unpack_sequence`

```c++
struct unpack_sequence<Seq,_=void> { using not_unpackable=void; };
```

------
### Utilities

#### Macros

* `RETURNS(...)` : `decltype(__VA_ARGS__){return __VA_ARGS__; }`
* `RETURNS_CLASS(C)`, `THIS`, `CONST_THIS` : checking facility for old compilers.
* `MANGLE_CAST`, `RETURNS_C_CAST`, `RETURNS_REINTERPRET_CAST`, `RETURNS_STATIC_CAST`, `RETURNS_CONSTRUCT`
* `AUTO_FORWARD(...)` : `static_cast<decltype(__VA_ARGS__)>(__VA_ARGS__)`
* `STATIC_FUNCTION(name)` : statically initialized variable with `reveal_adaptor`
* `STATIC_LAMBDA`, `STATIC_LAMBDA_FUNCTION`
* `LIFT(...)`, `LIFT_CLASS(name,...)`, `LIFT_IS_NOEXCEPT(...)`
* `VERSION_MAJOR` (0), `VERSION_MINOR` (6), `VERSION_PATCH` (0), `VERSION`: `0xMM_mm_pppp`

#### Functions

##### `apply`, `apply_eval` - `INVOKE`

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

##### `eval`

```c++
struct detail::simple_eval {
    constexpr auto operator()<F,...Ts>(F&& f, Ts&&...xs) const noexcept(...) ->result_of<...>::type
    { return always_ref(f)(xs...)(); }
};
struct detail::id_eval {
    constexpr auto operator()<F,...Ts>(F&& f, Ts&&...xs) const noexcept(...) ->result_of<...>::type
    { return always_ref(f)(xs...)(identity); }
};

constexpr auto const& eval = first_of_adaptor<simple_eval, id_eval>{};
// assert( eval([]{return 1;}) == 1 );
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
##### `tap` - call function then return argument

```c++
struct detail::tap_f{
    constexpr T operator()<T,F>(T&& x, const F& f) const noexcept(...)
    { apply(f, x); return fwd<T>(x); }
};

constexpr auto const& tap = tap_f{};
```
