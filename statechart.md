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

struct rtti_policy {    // simplified, for native RTTI version
    class id_type; // wrapper for std::type_info&
    using id_provider_type = bool;
    struct rtti_base_type<B> : B {
        id_type dynamic_type() const;   // typeid(*this)
    };
    struct rtti_drived_type<D,B> : B {
        static id_type static_type();   // typeid(const D)
    };
};

struct counted_base<NeedsLocking=true> {
    using count = NeedsLocking ? atomic_count : long;
    mutable count count_;
    // add_ref, release, supporting intrusive_ptr
};

using orthogonal_position_type = uint8_t;
```

* `simple_state`, `state`, and `event` are mandatory base classes for state and event classes.
* Statemachine structure and behavior is configured via template arguments and nested types.
* States and events are defined with RTTI, through `rtti_policy` type
* States and events are reference counted, using `intrusive_ptr`, to manage lifetime within statemachine instance

------
### State Machine Structure

```c++
struct state_base<Alloc> : rtti_base_type<counted_base<false>> {
    using state_list_type = list<intrusive_ptr<leaf_state>, Alloc>;

    bool deferredEvents = false;
    void defer_events(); bool deferred_events() const; // set/fetch 'deferredEvents'

    void exit() {}  // default empty exit()
    virtual state_base const* outer_state_ptr() const = 0;
    void set_context<Context>(uint8_t position, Context* p) {
        p->add_inner_state(position, this);
    }
    virtual void remove_from_state_list(state_list_type::iterator& statesEnd,
        intrusive_ptr<node_state_base>& pOutermostUnstableState, bool performFullExit) = 0;
};

struct node_state_base<Alloc> : state_base<Alloc> {
    virtual void exit_impl(intrusive_ptr<node_state_base> & pself,
        intrusive_ptr<node_state_base>& pOutermostUnstableState, bool performFullExit) = 0;
};
struct node_state<OrthogonalRegionCount,Alloc> : node_state_base<Alloc> {
    array<state_base*, OrthogonalRegionCount::value> pInnerStates; // state pointers
    void add_inner_state(uint8_t, state_base*); void remove_inner_state(uint8_t);

    virtual void remove_from_state_list(state_list_type::iterator& statesEnd,
        intrusive_ptr<node_state_base>& pOutermostUnstableState, bool performFullExit) override {
        if (none_of(pInnerStates, [](auto p){return p != nullptr;})) {
            assert(this == pOutermostUnstableState);
            auto pSelf = pOutermostUnstableState;
            pSelf->exit_impl(pSelf, pOutermostUnstableState, performFullExit);
        } else {
            for (auto state : reversed(pInnerStates))
                if (state != nullptr) state->remove_from_state_list(statesEnd,
                                                    pOutermostUnstableState, performFullExit);
        }
    }
};

struct leaf_state<Alloc> : state_base<Alloc> {
    state_list_type::iterator listPosition_;    // remember position within list
    
    virtual void exit_impl(intrusive_ptr<node_state_base> & pself,
        intrusive_ptr<node_state_base>& pOutermostUnstableState, bool performFullExit) = 0;
    virtual void remove_from_state_list(state_list_type::iterator& statesEnd,
        intrusive_ptr<node_state_base>& pOutermostUnstableState, bool performFullExit) override {
        swap(*listPosition_, *--statesEnd); (*listPosition_)->listPosition_ = listPosition_;
        this->exit_impl(*statesEnd, pOutermostUnstableState, performFullExit);
    }
};

using simple_state_base_type<MostDerived,Context,...InnerInitial> = rtti_derived_type<MostDerived,
    sizeof...InnerInitial == 0 ? leaf_state<> : node_state<sizeof...InnerInitial>>;

struct simple_state<MostDerived, Context, history_mode hm=has_no_history, ...InnerInitial>
    : simple_state_base_type<MostDerived,Context::inner_context_type, InnerInitial...> {
    using orthogonal_position = Context::inner_orthogonal_position;
    using context_type = Context::inner_context_type;
    using outermost_context_type = context_type::outermost_context_type;
    struct orthogonal<uint8_t innerOrthogonalPosition> {    // bundle pos and context
        using inner_orthogonal_position = integral_c<uint8_t, innerOrthogonalPosition>;
        using inner_context_type = MostDerived;  // actual context
    };
    using inner_orthogonal_position = integral_c<uint8_t, 0>;   // for single orthogonal
    using inner_context_type = MostDerived; // for single orthogonal
    using inner_context_ptr_type = intrusive_ptr<inner_context_type>;
    using context_ptr_type = context_type::inner_context_ptr_type;
    using context_type_list = mpl::list<context_type, context_type::context_type_list...>;

    context_ptr_type pContext_;

    outermost_context[_base]_type [const]& outermost[_base]_context() [const]; // forward to pContext_
    OtherContext [const]& context<OtherContext>() [const]; // return this or forward to pContext_
    // from outermost: state_cast, state_downcast, state_begin, state_end

    simple_state() : pContext_(0) {}
    ~simple_state() {
        if (pContext_ && deferred_events()) outermost_context_base().release_events();
        pContext_ && pContext_->remove_inner_state(orthogonal_position::value);
    }
    state_base const* outer_state_ptr() const override; // pContext_, or 0 if it is outermost

    void exit_impl(intrusive_ptr<node_state_base> & pself,
        intrusive_ptr<node_state_base>& pOutermostUnstableState, bool performFullExit) {
        inner_context_ptr_type pMostDerivedSelf = this; pSelf = 0; // move reference to stack obj
        switch (this->ref_count()) {
        case 2: // have one reference
            if (pOutermostUnstableState == this) pOutermostUnstableState = pContext_; // ref becomes 1
            else break;
        case 1: // only ref-ed by pself
            if (pOutermostUnstableState == 0) pOutermostUnstableState = pContext_;
            if (performFullExit) { pSelf->exit(); check_store }
            auto pContext = pContext_; pself = 0;       // save ref to outer context/state, destroy self
            pContext->exit_impl(pContext, pOutermostUnstableState, performFullExit); // exit the context
        }
    }
};
```
