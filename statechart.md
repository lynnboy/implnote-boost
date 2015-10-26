# Boost.StateChart

* lib: `boost/libs/statechart`
* repo: `boostorg/statechart`
* commit: `21225fa0`, 2015-01-25

------
### State Chart

#### Concepts

```c++
concept bool Scheduler<S> = requires {
    typename S::processor_context; typename S::processor_handle;
    requires(S::processor_context const cpc) {
        { cpc.my_scheduler() } -> S &;
        { cpc.my_handle() } -> S::processor_handle;
    }
};

concept bool FifoWorker<F, W> = requires (F f, F const cf, W w, unsigned long n){
    requires is_same_v<F::work_item, function<void()>>;
    F(); F(false);  // non-blocking
    F(true);        // blocking
    w(); f.queue_work_item(w);
    f.terminate();  { cf.terminated() } -> bool;
    { f(n) } -> unsigned long; { f() } -> unsigned long; // loop for n items
};

concept bool ExceptionTranslator<EH> = requires (EH eh, A a, EH eh){
    { a() } -> result; { eh(declval<event_base>()) } -> result;
    { et(a, eh) } -> result;    // catch ex from a() and call eh(event)
};

concept bool StateBase<S> = requires (S const cs){
    { cs.outer_state_ptr() } -> const S*;
    { cs.dynamic_type() } -> typename S::id_type;
    { cs.custom_dynamic_type_ptr<Type>() } -> Type const*;
};

concept bool SimpleState<S> = is_base_of_v<simple_state<S,C,I,h>, S> &&
default_constructible_v<S> && destructible_v<S> && requires (S* pS) {
    pS->exit();
    typename S::reactions; // mpl::list<...>
};

concept bool State<S> = is_base_of_v<state<S,C,I,h>, S> &&
requires {
    S(declval<typename S::my_context>());
};

concept bool Event<E> = is_base_of_v<event<E>, E> && copy_constructible_v<E>;
```
