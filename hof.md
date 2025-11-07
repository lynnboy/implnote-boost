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
### Function Adaptors



------
### Dependency

------
### Standard Facilities
