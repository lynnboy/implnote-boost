# Boost.Assignment

* lib: `boost/libs/bind`
* repo: `boostorg/bind`
* commit: `a541a8d`, 2024-12-13

------
#### `mem_fn`

* Header `<boost/mem_fn.hpp>`

```c++
class _mfi::mf<Pm,R,T,...A> { // wrapper for member function pointer
    Pm pm_;
public:
    using result_type = R;
    mf(Pm pm): pm_(pm) {}
    // operator == and !=

    R operator() <U> (U&& u, A... a) const {
        if constexpr (std::is_base_of_v<T, std::remove_cvref_t<U>>) (std::forward<U>(u).*pm_)(std::forward<A>(a)...);
        else return (boost::get_pointer(std::forward<U>(u))->*pm_)(std::forward<A>(a)...);
    }
};
auto mem_fn <R,T,...A> (R(T::*pmf)(A...)[const][noexcept]) -> mf<decltype(pmf),R,T,A...>;

class _mfi::dm<R,T> { // wrapper for data member pointer
    using Pm = R T::*; Pm pm_;
public:
    using result_type = R const&; using argument_type = T const*;
    dm(Pm pm): pm_(pm) {}
    // operator == and !=

    auto operator() <U> (U&& u) const {
        if constexpr (std::is_base_of_v<T, std::remove_cvref_t<U>>) return std::forward<U>(u).*pm_;
        else return boost::get_pointer(std::forward<U>(u))->*pm_;
    }
};
dm<R,T,...A> mem_fn <R,T> (R T::* pm) requires !std::is_function_v<R>;
```

------
#### Placeholders

* Header `<boost/is_placeholder.hpp>`
* Header `<boost/bind/placeholders.hpp>`
* Header `<boost/bind/std::placeholders.hpp>`

```c++
struct is_placeholder<T> { enum _vt{ value = 0 }; }; // primary

struct arg<I> { // constructible from any I'th placeholder
    constexpr arg(){}
    constexpr arg <T> (T const&) requires I==is_placeholder<T>::value {}
};
constexpr bool operator== <I> (arg<I> const&, arg<I> const&) { return true; }

struct is_placeholder<arg<I>> { enum _vt{ value = I }; }; // arg<I> is I'th placeholder

inline constexpr arg<1> placeholders::_1; // up to _9;

struct is_placeholder<std::decay_t<decltype(std::placeholders::_1)>> { enum _vt{ value = 1 }; }; // up to _9
```

-----
#### `bind`

* Header `<boost/bind.hpp>`

```c++
using _bi::result_traits<R,F>::type = R; // primary
using _bi::result_traits<unspecified,F>::type = F::result_type;
// specialized for reference_wrapper<F>, std::plus<T>, std::minus<T>, std::multiplies<T>, ...

template<class T, T...I> struct _bi::integral_sequence{};
using _bi::make_integer_sequence<T,N> = integral_sequence<T,0,...,N-1>;
using _bi::index_sequence<...I> = integral_sequence<size_t,I...>;
using _bi::make_index_sequence<N> = make_integer_sequence<size_t,N>;
using _bi::index_sequence_for<...T> = make_index_sequence<sizeof...(T)>;

F _bi::tuple_for_each<F,Tp> (F&& f, TP&& tp)
{ template for (auto x : tp) f(std::forward<decltype(x)>(x)); return std::forward<F>(f); }
F _bi::tuple_for_each<F,Tp1,Tp2> (F&& f, Tp1&& tp1, Tp2&& tp2) {
    template for (size_t i : make_index_sequence<std::tuple_size_v<std::remove_reference_t<Tp1>>)
    { auto x = std::get<i>(tp1); auto y = std::get<i>(tp2);
      f(std::forward<decltype(x)>(x), std::forward<decltype(y)>(y)); }
    return std::forward<F>(f);
}

class _bi::value<T> {
    T t_;
public:
    value(T const&);
    T[const]& get()[const];
    bool operator==(value const&) const;
};

class _bi::type<T> {};

bool _bi::ref_compare<T> (T const& a, T const& b) { return a == b; } // common
bool _bi::ref_compare<I> (arg<I>(*)(), arg<I>(*)()) { return true; } // function returning same placeholder
bool _bi::ref_compare<T> (reference_wrapper<T> const& a, reference_wrapper<T> const& b) { return a.get_pointer() == b.get_pointer(); }
bool _bi::ref_compare<R,F,L> (bind_t<R,F,L> const& a, bind_t<R,F,L> const& b) { return a.compare(b); }
bool _bi::ref_compare<T> (value<weak_ptr<T>> const& a, value<weak_ptr<T>> const&) { return !(a.get() < b.get()) && !(b.get() < a.get()); } // special for weak_ptr

// list:
struct _bi::unwrapper<F> {
    static F& unwrap<F> (F& f, long) { return f; } // common
    static F2& unwrap<F2> (reference_wrapper<F2> rf, int) { return rf.get(); }
    static _mfi::dm<R,T> unwrap<R,T> (R T::* pm, int) { return dm<R,T>{pm}; } // data member ptr
};
template struct _bi::equal_lambda {
    bool result{true};
    void operator() <A1,A2> (A1& a1, A2& a2) { result = result && ref_compare(a1,a2); }
};
class _bi::list<...A> {
    std::tuple<A...> data_;
public: list(A...a): data_(a...){}
    R operator() <R,F,A2> (type<R>, F& f, A2& a2)[const] {return unwrapper<F>::unwrap(f,0)( a2[data_]... )}
    bool operator() <A2> (type<bool>, logical_and[const]&, A2& a2) [const] {return a2[data_...[0]] && a2[data_...[1]];}
    bool operator() <A2> (type<bool>, logical_or[const]&, A2& a2) [const] {return a2[data_...[0]] || a2[data_...[1]];}
    void accept <V> (V& v) const { tuple_for_each([&v](auto& a){visit_each(v,a,0);}, data); }
    bool operator==(list const& rhs) const
    { bool result{true}; tuple_for_each([](auto& a1, auto& a2){result &&= ref_compare(a1,a2);}); return result; }
};

class _bi::rrlist<...A> {
    std::tuple<A&...> data_;
public: rrlist(A&...a): data(a...){}
    explicit rrlist<...B> (rrlist<B...> const& r): data_(r.data_) {} // compatible
    A...[I-1]&& operator[] <I> (arg<I>) const {return std::get<I-1>(data_);} // indexing by placeholder
    A...[I-1]&& operator[] <I> (arg<I>(*)()) const; // indexing by placeholder function
    T[const]& operator[] <T> (value<T>[const]& v) const { return v.get(); }
    T& operator[] <T> (reference_wrapper<T>& v) const { return v.get(); }
    result_traits<R,F>::type operator[] <R,F,L> (bind_t<R,F,L>[const]& b) const { return b.eval(rrlist<A&...>{*this}); }
};

// bind_t:
class _bi::bind_t<R,F,L> {
    F f_; L l_; // function and list
public: using result_type = result_traits<R,F>::type;
    bind_t(F f, L const& l): f_(std::move(f)), l_(l) {}
    result_type operator() <...A> (A&&...a) [const] { return l_(type<result_type>{}, f_, rrlist<A...>{a...}); }
    result_type eval <A> (A& a) [const] { return l_(type<result_type>{}, f_, a); }
    void accept <V> (V& v) const { visit_each(v, f_, 0); l_.accept(v); }
    bool compare(bind_t const& rhs) const { return ref_compare(f_, rhs.f_) && l_ == rhs.l_; }
};

bool _bi::function_equal<R,F,L> (bind_t<R,F,L> const& a, bind_t<R,F,L> const& b) { return a.compare(b); }

using _bi::add_value<T>::type = is_placeholder<T> ? T : value<T>; // primary
using _bi::add_value<value<T>>::type = value<T>;
using _bi::add_value<reference_wrapper<T>>::type = reference_wrapper<T>;
using _bi::add_value<arg<I>(*)()>::type = arg<I>(*)();
using _bi::add_value<bind_t<R,F,L>>::type = bind_t<R,F,L>;
using _bi::list_av<...A>::type = list<add_value<A>::type...>; // wrap each bare T with value<T>

// bind expression:
bind_t<bool, std::logical_not<>, list<bind_t<R,F,L>>>
    operator! <R,F,L> (bind_t<R,F,L> const&);
bind_t<bool, std::equal_to<>, list<bind_t<R,F,L>,add_value<A2>::type>>
    operator== <R,F,L,A2> (bind_t<R,F,L> const&, A2 a2);
// also != (not_equal_to), < (less), > (greater), <= (less_equal), >= (greater_equal), && (logical_and), || (logical_or)

// boost::visit_each support
void _bi::visit_each<V,T> (V& v, value<T> const& t, int) { using boost::visit_each; visit_each(v, t.get(), 0); }
void _bi::visit_each<V,R,F,L> (V& v, bind_t<R,F,L> const& t, int) { t.accept(v); }

struct is_bind_expression<T> { enum _vt{value = 0}; }; // primary
struct is_bind_expression<bind_t<R,F,L>> { enum _vt{value = 1}; };

// bind API:
bind_t<R,F,list_av<A...>::type>
    bind <R,F,...A> (F f, A... a) { return { std::move(f), list_av<A...>::type { a... } }; }
bind_t<R,F,list_av<A...>::type>
    bind <R,F,...A> (type<R>, F, A... a);
bind_t<unspecified,F,list_av<A...>::type>
    bind <F,...A> (F f, A... a);

bind_t<R, R(*)(B...), list_av<A...>::type>
    bind <R,...B,...A> (R (*f)(B...b)[noexcept], A...a); // function pointer, 0~9 parameters
auto // -> decltype(bind(mem_fn(f), a1, a...)) == bind_t<R, mf<decltype(f),R,T,A...>, list_av<A...>::type>
    bind <R,T,A1,...A> (R (T::*f)(A...)[const][noexcept], A1 a1, A... a); // member function pointer, 0~8 parameters
auto // requires R2!=R, -> decltype(bind<R2>(mem_fn(f), a1, a...))
    bind <R2,R,T,A1,...A> (R (T::*f)(A...)[const][noexcept], A1 a1, A... a);
auto // -> decltype(bind(type<Rt2>{}, mem_fn(f), a1, a...))
    bind <R2,R,T,A1,...A> (type<Rt2>, R (T::*f)(A...)[const][noexcept], A1 a1, A... a);

using _bi::add_cref<M,I>::type = I ? M : M const&;
using _bi::isref<R>::value = is_reference_v<R> || is_pointer<R>;
using _bi::dm_result<M,A1>::type = M const&;
using _bi::dm_result<M,bind_t<R,F,L>>::type = add_cref<M,isref<bind_t<R,F,L>::result_type>>::type;

bind_t<dm_result<M,A1>::type, dm<M,T>, list_av<A1>::type> // data member pointer
    bind <A1,M,T> (M T::*f, A1 a1); // requires !is_function<M>
```

------
#### Misc

* Header `<boost/bind/apply.hpp>`

```c++
struct apply<R> {
    using result_type = R;
    result_type operator() <F,...A> (F&& f, A&&... a) const { return std::forward<F>(f)(std::forward<A>(a)...); }
};
```

* Header `<boost/bind/protect.hpp>`

```c++
class _bi::protected_bind_t<F>;
protected_bind_t<F> protect<F> (F f);
```

-------
#### Configuration

* Macro `BOOST_BIND` define the name of the function `bind`, by default is just `bind`.
* Macros `BOOST_BIND_ENABLE_STDCALL`, `BOOST_BIND_ENABLE_FASTCALL`, `BOOST_BIND_ENABLE_PASCAL` adds function pointer overloads with specified calling convention.
* Macros `BOOST_MEM_FN_ENABLE_CDECL`, `BOOST_MEM_FN_ENABLE_STDCALL`, `BOOST_MEM_FN_ENABLE_FASTCALL` adds member function pointer overloads with specified calling convention.
* Macro `BOOST_BIND_NO_PLACEHOLDERS` excludes placeholders.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`, `<boost/config/pragma_message.hpp>`

#### Boost.Core

* `<boost/core/get_pointer.hpp>`
* `<boost/core/ref.hpp>`
* `<boost/core/type.hpp>`
* `<boost/core/visit_each.hpp>`

------
### Standard Facilities

Language: lambda expression (C++11)
Library: `<bind>` (C++11), `mem_fn` (C++11)
