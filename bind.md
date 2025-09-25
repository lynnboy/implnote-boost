# Boost.Assignment

* lib: `boost/libs/bind`
* repo: `boostorg/bind`
* commit: `a541a8d`, 2024-12-13

------
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

* Header `<boost/bind.hpp>`

```c++
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
