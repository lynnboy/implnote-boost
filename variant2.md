# Boost.Variant2

* lib: `boost/libs/variant2`
* repo: `boostorg/variant2`
* commit: `8caa1a3`, 2025-01-12

------
#### `variant`

* Header `<boost/variant2.hpp>` or `<boost/variant2/variant.hpp>`

```c++
class bad_variant_access: public std::exception{};

struct monostate{}; // empty
constexpr bool operator< (monostate const&, monostate const&) noexcept; // also >, <=, >=, ==, !=

struct variant_size<T> {}; // primary
struct variant_size<variant<T...>> : mp_size<variant<T...>> {};
struct variant_size<const T> : variant_size<T> {}; // and specialize for cv T, T&, and T&&
constexpr size_t variant_size_v<T> = variant_size<T>::value;

struct variant_alternative<i,T> {}; // primary
struct variant_alternative<i,variant<T...>> : mp_defer<mp_at, variant<T...>, mp_size_t<i>> {}; // SFINAE type = T...[i]
struct variant_alternative<i,const T> {}; // and specialize for cv T, T&, and T&&
struct variant_alternative<i,const variant<T...>> {}; // and specialize for cv T, T&, and T&&, propagate cvref to T...[i]
using variant_alternative_t<i,T> = variant_alternative<i,T>::type;

constexpr size_t variant_npos = -1; // casted

constexpr bool holds_alternative <U,...T> (variant<T...> const& v) noexcept // assert mp_count<U,v<T...>> == 1
{ return v.index() == mp_find<variant<T...>, U>::value; }
constexpr auto get <i,...T> (variant<T...>[const]{&|&&} v) -> T...[i] [const]{&|&&}; // throw if i != v.index(), move on return for && overloads
constexpr auto unsafe_get <i,...T> (variant<T...>[const]{&|&&} v) -> T...[i] [const]{&|&&}; // don't throw
constexpr auto get <U,...T> (variant<T...>[const]{&|&&} v) -> U [const]{&|&&} { return get<mp_find<variant<T...>,U>::value>(v); }
constexpr auto get_if <i,...T> (variant<T...>[const]* v) noexcept -> T...[i] [const]*; // nullptr for !v or v->index() != i
constexpr auto get_if <U,...T> (variant<T...>[const]* v) noexcept -> U[const]*; // nullptr for !v or v->index() != i

union detail::variant_storage_impl<D>{}; // empty
union detail::variant_storage_impl<D,...T>; // primary, not def
union detail::variant_storage_impl<mp_false,T1,...T> { // not all trivally destructible
    T1 first_; variant_storage<T...> rest_;
    constexpr ctor <...A> (mp_size_t<0>, A&&... a) : first_(std::forward<A>(a)...) {}
    constexpr ctor <i,...A> (mp_size_t<i>, A&&... a) : rest_(mp_size_t<i-1>, std::forward<A>(a)...) {} // recursive
    ~dtor() {} // default, because non-trivally destructible member will delete dtor
    void emplace <...A> (mp_size_t<0>, A&&... a) { ::new(&first_) T1(std::forward<A>(a)...); }
    void emplace <i,...A> (mp_size_t<i>, A&&... a) { rest_.emplace(mp_size_t<i-1>{}, std::forward<A>(a)...); } // recursive
    constexpr T1[const]& get (mp_size_t<0>) [const]noexcept { return first_; }
    constexpr T...[i-1] [const]& get <i> (mp_size_t<i>) [const]noexcept { return rest_.get(mp_size_t<i-1>{}); }
};
union detail::variant_storage_impl<mp_true,T1,...T> { // all trivally destructible
    T1 first_; variant_storage<T...> rest_;
    // ctor, emplace, get are the same as above, no need provide dtor
    constexpr emplace <i,...A> (mp_size_t<i>, A&&... a) {
        if constexpr (mp_all<is_trivally_move_assignable<T1>, is_trivally_move_assignable<T>...>::value)
            *this = variant_storage_impl(pm_size_t<i>(), std::forward<A>(a)...); // trivally move assign
        else if constexpr (i == 0) { ::new(&first_) T1(std::forward<A>(a)...); }
        else { rest_.emplace(mp_size_t<i-1>{}, std::forward<A>(a)...); } // recursive
    }
};
union detail::variant_storage_impl<mp_false,T0,...,T9,...T>; // to speedup instantiation
union detail::variant_storage_impl<mp_true,T0,...,T9,...T>; // to speedup instantiation

using detail::variant_storage<...T> = variant_storage_impl<mp_all<is_trivally_destructible<T>...>, T...>;

struct detail::overload<...T>; // make overload for each type in T
struct detail::overload<> { void operator()() const; }; // base
struct detail::overload<T1,...T> : overload<T...> { using base::operator(); mp_identity<T1> operator()(T1) const; }; // recursive
using detail::resolve_overload_type<U,...T> = decltype(overload<T...>{}(declval<U>()))::type; // find best match for U
using detail::resolve_overload_index<U,...T> = mp_find<mp_list<T...>, resolve_overload_type<U,...T>>; // find I for type best matches U

using detail::get_index_type<isDouble,...T> = ; // smallest unsigned integral type to store value `sizeof...(T)` or `sizeof...(T) * 2`

struct detail::none{};

struct detail::variant_base_impl<is_trivally_destructible, is_single_buffered, ...T>; // primary
struct detail::variant_base_impl<true,true,T...> { // all trival-dtor, single buffer (all nothrow-move-ctor)
    using index_type = get_index_type<false, T...>;
    variant_storage<none, T...> st_;  index_type ix_; // additional `none` state
    constexpr ctor() : st_{mp_size_t<0>{}}, ix_{0} {} // store `none`
    constexpr explicit ctor <I,...A> (I, A&&...a) : st_(mp_size_t<I::value+1>{}, std::forward<A>(a)...), ix_(I::value+1) {} // off-by-1 for `none`
    void _replace <I,...A> (I, A&&...a); // placement-new, requires `none` state
    constexpr size_t index() const noexcept { return ix_ - 1; } // off-by-1
    constexpr T...[i] [const]& _get_impl <i> (mp_size_t<i>) [const]noexcept { return st_.get(mp_size_t<i+1>{}); } // off-by-1
    constexpr void emplace<i,...A> (A&&...a) { // strong ex-safety
        using U = I...[i];
        if constexpr (is_nothrow_constructible_v<U,A&&...>) { st_.emplace(mp_size_t<i+1>{}, std::forward<A>(a)...); }
        else                  { U tmp{std::forward<A>(a)...}; st_.emplace(mp_size_t<i+1>{}, std::move(tmp)); } // tmp's init may throw
        ix_ = i+1; // off-by-1
    }
    static constexpr bool uses_double_storage() noexcept { return false; }
};
struct detail::variant_base_impl<true,false,T...> { // all trival-dtor, double buffer (not-all nothrow-move-ctor)
    using index_type = get_index_type<true, T...>;
    variant_storage<none, T...> st_[2];  index_type ix_; // additional `none` state, `ix=2n` for st[0][n], `ix=2n+1` for st[1][n]
    constexpr ctor() : st_{mp_size_t<0>{},mp_size_t<0>{}}, ix_{0} {} // store `none`
    constexpr explicit ctor <I,...A> (I, A&&...a) : st_{{mp_size_t<I::value+1>{}, std::forward<A>(a)...},mp_size_t<0>{}}, ix_((I::value+1)*2) {} // off-by-1 for `none`
    void _replace <I,...A> (I, A&&...a); // placement-new on st_[0], requires `none` state
    constexpr size_t index() const noexcept { return ix_/2 - 1; } // off-by-1
    constexpr T...[i] [const]& _get_impl <i> (mp_size_t<i>) [const]noexcept { return st_[ix_%2].get(mp_size_t<i+1>{}); } // off-by-1
    constexpr void emplace<i,...A> (A&&...a) { // strong ex-safety
        unsigned i2 = 1 - ix_%2; // peer storage
        st_[i2].emplace(mp_size_t<i+1>{}, std::forward<A>(a)...); // may throw
        ix_ = (i+1)*2 + i2; // off-by-1, switch storage
    }
    static constexpr bool uses_double_storage() noexcept { return true; }
};
struct detail::variant_base_impl<false,true,T...> : variant_base_impl<true,true,T...> { // not all trival-dtor, single buffer
    using base::ctor;
    ~dtor() noexcept { _destroy(); }
    void _destroy() noexcept { // call dtor for current alternative
        template for (size_t i : std::make_index_sequence<sizeof...(T)>)
        { using U = T...[i]; if (ix_-1 == i) st_.get(mp_size_t<i>{}).~U(); }
    }
    constexpr void emplace<i,...A> (A&&...a) { // strong ex-safety
        using U = I...[i];
        U tmp{std::forward<A>(a)...}; // may throw
        _destroy();
        st_.emplace(mp_size_t<i+1>{}, std::move(tmp));
        ix_ = i+1; // off-by-1
    }
};
struct detail::variant_base_impl<false,false,T...> : variant_base_impl<true,false,T...> { // not all trival-dtor, double buffer
    using base::ctor;
    ~dtor() noexcept { _destroy(); }
    void _destroy() noexcept { // call dtor for current alternative
        template for (size_t i : std::make_index_sequence<sizeof...(T)>)
        { using U = T...[i]; if (ix_/2-1 == i) st_[ix_%2].get(mp_size_t<i>{}).~U(); }
    }
    constexpr void emplace<i,...A> (A&&...a) { // strong ex-safety
        unsigned i2 = 1 - ix_%2; // peer storage
        st_[i2].emplace(mp_size_t<i+1>{}, std::forward<A>(a)...); // may throw
        _destroy();
        ix_ = (i+1)*2 + i2; // off-by-1, switch storage
    }
};
using detail::variant_base<...T> = variant_base_impl<mp_all<is_trivally_destructible<T>...>::value,
                                                     mp_all<is_nothrow_move_constructible<T>...>::value, T...>;

struct in_place_type_t<T>{};
constexpr in_place_type_t<T> in_place_type<T>{}; // var template
struct in_place_index_t<i>{};
constexpr in_place_index_t<i> in_place_index<i>{}; // var template
using detail::is_in_place_type<T> = ...;
using detail::is_in_place_index<T> = ...;

class variant<...T> : private variant_base<T...> {
public:
    ctor(self {const&|&&} r) requires(is_trivally_{copy|move}_constructible_v<T> && ...) = default;
    ctor(self {const&|&&} r) noexcept(is_nothrow_{copy|move}_constructible_v<T> && ...) requires(is_{copy|move}_constructible_v<T> && ...)
    {
        template for (size_t i : std::make_index_sequence<sizeof...(T)>)
            if (i == r.index()) _replace(mp_size_t<i>{}, <std::move>(r._get_impl(mp_size_t<i>{})));
    }
    ctor(self {const&|&&} r) = delete;
    self& operator=(self {const&|&&} r) requires((is_trivally_{copy|move}_constructible_v<T> && is_trivally_{copy|move}_assignable_v<T> && is_trivally_destructible_v<T>) && ...) = default;
    self& operator=(self {const&|&&} r) noexcept(is_nothrow_{copy|move}_constructible_v<T> && ...) requires ((is_{copy|move}_constructible_v<T> && is_{copy|move}_assignable_v<T>) && ...)
    {
        template for (size_t i : std::make_index_sequence<sizeof...(T)>)
            if (i == r.index()) emplace<i>(<std::move>(r._get_impl(mp_size_t<i>{})));
        return *this;
    }
    self& operator=(self {const&|&&} r) = delete;

    constexpr ctor() noexcept(is_nothrow_default_constructible_v<T...[0]>)
        requires is_default_constructible_v<T...[0]> : base(mp_size_t<0>{}) {}
    constexpr ctor <U, Ud=std::decay_t<U>, V=resolve_overload_type<U&&,T...>> (U&& u) noexcept(is_nothrow_constructible_v<V,U&&>)
        requires !is_base_of_v<variant, Ud> && !is_in_place_index<Ud> && !is_in_place_type<Ud> && is_constructible_v<V,U&&>
        : base(resolve_overload_index<U&&, T...>(), std::forward<U>(u)) {}
    constexpr explicit ctor <U, [V], ...A, i=mp_find<variant<...>,U>> (in_place_type_t<U>, <std::initializer_list<V> il>, A&&... a)
        requires is_constructible_v<U,[std::initializer_list<V>&],A&&> : base(i{}, [il], std::forward<A>(a)...) {}
    constexpr explicit ctor <i, [V], ...A> (in_place_index_t<i>, <std::initializer_list<V> il>, A&&... a)
        requires is_constructible_v<T...[i],[std::initializer_list<V>&],A&&> : base(mp_size_t<i>{}, [il], std::forward<A>(a)...) {}

    constexpr self& operator= <U, V=resolve_overload_type<U,T...>> (U&& u) noexcept(is_nothrow_constructible_v<V,U&&>)
        requires !is_same_v<std::decay_t<U>, variant> && is_assignable_v<V&,U&&> && is_constructible_v<V,U&&>
    { emplace<resolve_overload_index<U, T...>::value> (std::forward<U>(u)); return *this; }

    constexpr U& emplace<U,[V],...A> (<std::initializer_list<V> il>, A&&...a) requires mp_count<variant,U>::value == 1 && is_constructible_v<U,<std::initializer_list<V>&>,A&&...>
    { using I=mp_find<variant,U>; base::emplace<I::value>(<il>, std::forward<A>(a)...); return _get_impl(I{}); }
    constexpr U& emplace<i,[V],...A> (<std::initializer_list<V> il>, A&&...a) requires is_constructible_v<T...[i],<std::initializer_list<V>&>,A&&...>
    { base::emplace<i>(<il>, std::forward<A>(a)...); return _get_impl(mp_size_t<i>{}); }

    constexpr bool valueless_by_exception() const noexcept { return false; } // always ex-safe

    void swap(variant& r) noexcept ((is_nothrow_move_constructible_v<T> && is_nothrow_swappable_v<T>) && ...) {
        if (index() == r.index()) {
            template for (size_t i : std::make_index_sequence<sizeof...(T)>)
                if (i == r.index()) { using std::swap; swap(_get_impl(i), r._get_impl(i)); }
        } else { variant tmp{std::move(*this)}; *this=std::move(r); r=std::move(tmp); }
    }

    // converting ctors
    ctor <...U> (variant<U...> {const&|&&} r) noexcept(is_nothrow_copy_constructible_v<U> && ...)
        requires ((is_copy_constructible_v<U> && mp_contains<variant,U>::value) && ...)
    {
        template for (size_t i : std::make_index_sequence<sizeof...(U)>)
            if (i == r.index()) { _replace(mp_find<variant,U...[i]>{}, r._get_impl(i)); }
    }
    ctor <...U> (variant<U...> && r) noexcept(is_nothrow_move_constructible_v<U> && ...)
        requires ((is_move_constructible_v<U> && mp_contains<variant,U>::value) && ...)
    {
        template for (size_t i : std::make_index_sequence<sizeof...(U)>)
            if (i == r.index()) { _replace(mp_find<variant,U...[i]>{}, std::move(r._get_impl(i))); }
    }

    // subset
    constexpr variant<U...> subset <...U> () [const]{&|&&}
        requires ((is_copy_constructible_v<U> && mp_contains<variant,U>::value) && ...)
    {
        template for (size_t i : std::make_index_sequence<sizeof...(T)>)
            if (i == index()) {
                using V = T...[i]; using J = mp_find<variant<U...>,V>;
                if (J::value == sizeof...(U)) throw bad_variant_access(); // not found
                return variant<U...>{in_place_index_t<J::value>, [std::move](_get_impl(i))};
            }
    }
};

constexpr bool operator== <...T> (variant<T...> const& v, variant<T...> const& w)
{ return v.index() == w.index() && mp_with_index<sizeof...(T)>(v.index(), [v,w](auto i){return v._get_impl(i) == w._get_impl(i);}); }
constexpr bool operator!= <...T> (variant<T...> const& v, variant<T...> const& w)
{ return v.index() != w.index() || mp_with_index<sizeof...(T)>(v.index(), [v,w](auto i){return v._get_impl(i) != w._get_impl(i);}); }
constexpr bool operator< <...T> (variant<T...> const& v, variant<T...> const& w)
{ return v.index() < w.index() || mp_with_index<sizeof...(T)>(v.index(), [v,w](auto i){return v._get_impl(i) < w._get_impl(i);}); }
constexpr bool operator> <...T> (variant<T...> const& v, variant<T...> const& w) { return w < v; }
constexpr bool operator<= <...T> (variant<T...> const& v, variant<T...> const& w)
{ return v.index() <= w.index() && mp_with_index<sizeof...(T)>(v.index(), [v,w](auto i){return v._get_impl(i) <= w._get_impl(i);}); }
constexpr bool operator>= <...T> (variant<T...> const& v, variant<T...> const& w) { return w <= v; }

// visit
variant<T...> const& detail::extract_variant_base_<...T>(variant<T...> const&);
using detail::extract_variant_base<V> = remove_cv_ref_t<decltype(extract_variant_base_(declval<V>()))>;
using detail::variant_base_size<V> = variant_size<extract_variant_base<V>>;
using detail::copy_cv_ref_t<T,[const][volatile]U>= T [const][volatile];
using detail::copy_cv_ref_t<T,U{&|&&}> = copy_cv_ref_t<T,U> {&|&&};
using detail::apply_cv_ref<V> = mp_product<copy_cv_ref_t, extract_variant_base<V>, mp_list<V>>;

struct detail::deduced{};

constexpr auto visit <R=deduced, F> (F&& f) -> Vret<R,F> { return std::forward<F>(f)(); }
constexpr auto visit <R=deduced, F,V1> (F&& f, V1&& v1) -> Vret<R,F,V1>
{ return mp_with_index<variant_base_size<V1>>( v1.index(), [&](auto i) {
    return std::forward<F>(f)(unsafe_get<i.value>(std::forward<V1>(v1))>);
    });
}
constexpr auto visit <R=deduced, F,V1,V2,...V> (F&& f, V1&& v1, V2&& v2, V&&... v) -> Vret<R,F,V1,V2,V...>
{ return mp_with_index<variant_base_size<V1>>( v1.index(), [&](auto i) {
    auto f2 = [&](auto&&...a) { return std::forward<F>(f)(unsafe_get<i.value>(std::forward<(decltype(a)>(a)...)>)) };
    return visit<R>(f2, std::forward<V2>(v2), std::forward<V>(v)... ); // recursion 
}); }


void swap<...T> (variant<T...>& v, variant<T...>& w) noexcept(noexcept(v.swap(w)))
    requires ((is_move_constructible_v<T> && is_swappable_v<T>) && ...)
{ v.swap(w); }

constexpr auto visit_by_index <R=deduced,V,...F> (V&& v, F&&... f) -> Vret2<R,V,F...>
    requires variant_size_v<V> == sizeof...(F)
{
    auto tp = make_tuple(f...);
    return mp_with_index<variant_size_v<V>>(v.index(), // call f...[i](get<i>(v))
    [&](auto i){return (std::move(get<i.value>(tp))(unsafe_get<i.value>(std::forward<V>(v))));});
}

// stream output
using detail::is_output_streamable<Os,T> = {SFINAE declval<Os&>() << declval<T const&>()};
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>&, monostate const&); // "monostate"
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,T1,...T> (std::basic_ostream<Ch,Tr>&, variant<T1,T...> const&); // os << get<i>(v)

// hash
size_t hash_value(monostate const&) { return 0xA7EE4757u; }
size_t hash_value<...T> (variant<T...> const& v)
{ mp_with_index<sizeof...(T)>(v.index(), [&](auto i){return hash_value_impl(i.value, boost::hash<T...[i.value]>(unsafe_get<i.value>(v)));}); }
size_t detail::hash_value_std<...T> (variant<T...> const& v)
{ mp_with_index<sizeof...(T)>(v.index(), [&](auto i){return hash_value_impl(i.value, std::hash<T...[i.value]>(unsafe_get<i.value>(v)));}); }
struct std::hash<monostate> { size_t operator() (monostate const& v) const { return hash_value(v); } };
struct std::hash<variant<T...>> requires (is_default_constructible_v<std::hash<remove_const_t<T>>> && ...)
{ size_t operator() (variant<T...> const& v) const { return hash_value_std(v);} };

// json
struct boost::json::is_null_like<monostate> : true_type {};
void tag_invoke<...T> (json::value_from_tag const&, json::value& v, variant<T...> const& w)
{ visit([&](auto const& t){ json::value_from(t, v); }, w); } // value_from(get<w.index()>(w), v)
auto tag_invoke<...T> (json::try_value_to_tag<variant<T...>> const&, json::value& v) -> json::result_for<variant<T...>, json::value>::type
{
    auto r = json::result_from_errno<variant<T...>>(EINVAL, BOOST_SOURCE_LOCATION); // system::result<variant>
    mp_for_each<mp_iota_c<sizeof...(T)>>([&](auto i){ // for each index
        if (!r) if (auto r2 = json::try_value_to<T...[i.value]>(v), r2) r.emplace(in_place_index<i.value>, std::move(*r2));
    });
    return r;
}
```

* Always strong-exception-safety guarantee. No `stateless_by_exception`.
* Use double storage when any of `T` isn't nothrow move constructible.

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`
* `<boost/assert/source_location.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.MP11

* `<boost/mp11.hpp>`

------
### Standard Facilities

Library: `<bind>` (C++11), `mem_fn` (C++11)
