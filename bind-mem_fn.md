# Boost.Bind/MemFn

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

-------
#### Configuration

* Macros `BOOST_MEM_FN_ENABLE_CDECL`, `BOOST_MEM_FN_ENABLE_STDCALL`, `BOOST_MEM_FN_ENABLE_FASTCALL` adds member function pointer overloads with specified calling convention.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`, `<boost/config/pragma_message.hpp>`

#### Boost.Core

* `<boost/get_pointer.hpp>`

------
### Standard Facilities

Library: `mem_fn` (C++11)
