# Boost.Utility/CallTraits

* lib: `boost/libs/utility`
* repo: `boostorg/utility`
* commit: `dda83c5`, 2025-08-26

------
### `call_traits`

* Header `<boost/call_traits.hpp>`

```c++
struct call_traits<T> {
    using value_type = T; using <const>_reference = <const> T&;
    using param_type = conditional_t<is_pointer_v<T>, const T,
        conditional_t<is_arithmetic_v<T>||is_enum_v<T>,
            conditional_t<sizeof(T)<=sizeof(void*), const T, const T&>,
            const T&>>
};
struct call_traits<T&> {
    using value_type = T&; using <const>_reference = <const> T&;
    using param_type = T&;
};
struct call_traits<T[n]> {
    using value_type = const T*; using <const>_reference = <const> T(&)[n];
    using param_type = const T* const;
};
struct call_traits<const T[n]> {
    using value_type = const T*; using <const>_reference = const T(&)[n];
    using param_type = const T* const;
};
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

------
### Standard Facilities
