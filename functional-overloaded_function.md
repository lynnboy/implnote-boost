# Boost.Functional/OverloadedFunction

* lib: `boost/libs/functional/overloaded_function`
* repo: `boostorg/functional`
* commit: `3c468dd`, 2025-06-26

------
### Overloaded Function Wrapper

Header `<boost/functional/overloaded_function.hpp>`

```c++
template <typename ...F>
class overloaded_function {
public:
  overloaded_function(function<F> const&...);
  function_traits<F>::result_type operator()(function_traits<F>::arg_type...) const; // for each signature
};

template <typename ...F>
overloaded_function<...> make_overloaded_function(F...f);
```

* One instance can be constructed to hold multiple functors with different signatures.
* The implementation make preprocessor generated base classes, each wraps a `function<F>` as stored functor.
* Integrated with *Boost.TypeOf*

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`
* `<boost/cstdint.hpp>`
* `<boost/config/no_tr1/cmath.hpp>`

#### Boost.MPL

* `<boost/mpl/if.hpp>`, `<boost/mpl/identity.hpp>`, `<boost/mpl/pop_front.hpp>`, `<boost/mpl/push_front.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/remove_pointer.hpp>`, `<boost/type_traits/remove_reference.hpp>`

#### Boost.FunctionTypes

* `<boost/function_types/*.hpp>`

#### Boost.Function

* `<boost/function.hpp>`

#### Boost.TypeOf

* `<boost/typeof/typeof.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/*.hpp>`

------
### Standard Facilities

* Proposals:
  * P0051 - C++ generic overload functions
  * P0045 - Overloaded and qualified `std::function`
