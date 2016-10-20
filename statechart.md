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

    outermost_context[_base]_type [const]& outermost_context[_base]() [const]; // forward to pContext_
    OtherContext [const]& context<OtherContext>() [const]; // return this or forward to pContext_
    // from outermost: state_cast, state_downcast, state_begin, state_end

    simple_state() : pContext_{nullptr} {}
    ~simple_state() {
        if (pContext_ && deferred_events()) outermost_context_base().release_events();
        pContext_ && pContext_->remove_inner_state(orthogonal_position::value);
    }
    state_base const* outer_state_ptr() const override; // pContext_, or nullptr if it is outermost

    void set_context(const context_ptr_type & p) {
        pContext_ = p; set_context(orthogonal_position::value, p);
    }

    static inner_context_ptr_type shallow_construct(    // just create MostDerived and add to list
        const context_ptr_type & pContext, outermost_context_base_type & outerm) {
            inner_context_ptr_type s = new MostDerived; s.set_context(pContext); // set context
            outerm.add(s); return s;
    }
    static void deep_construct_inner<Head, ...Tail>(    // create each inner state in MostDerived
        const inner_context_ptr_type & pState, outermost_context_base_type & outerm) {
            Head::deep_construct(pState, outerm);  // recursive to child state
            deep_construct_inner<Tail...>(pState, outerm);  // recursive on state list
    }
    static void deep_construct(
        const context_ptr_type & pContext, outermost_context_base_type & outerm) {
            auto s = MostDerived::shallow_construct(pContext, outerm);     // MostDerived itself
            deep_construct_inner<InnerInitial...>(s, outerm);  // child states
    }
    void exit_impl override (intrusive_ptr<node_state_base> & pself,
        intrusive_ptr<node_state_base>& pOutermostUnstableState, bool performFullExit) {
        inner_context_ptr_type pMostDerivedSelf; swap(pMostDerivedSelf, pSelf); // moveto stack obj
        switch (this->ref_count()) {
        case 2: // have one reference
            if (pOutermostUnstableState == this) pOutermostUnstableState = pContext_ || nullptr;
            else break;
        case 1: // only ref-ed by pself
            if (pOutermostUnstableState == 0) pOutermostUnstableState = pContext_ || nullptr;
            if (performFullExit)
              { pSelf->exit(); check_store_shallow_history(); check_store_deep_history(); }
            auto pContext = pContext_; pself = 0;   // save ref to outer context/state, destroy self
            pContext->exit_impl(pContext, pOutermostUnstableState, performFullExit); // context exit
        }
    }
};
struct state<MostDerived, Context, history_mode hm=has_no_history, ...InnerInitial>
    : simple_state<MostDerived, Context, historyMode, InnerInitial...> {
    struct my_context { context_ptr_type pContext_; };      // transfer context to constructor
    state( my_context ctx) { set_context(ctx.pContext_); }
    static inner_context_ptr_type shallow_construct(    // just create MostDerived and add to list
        const context_ptr_type & pContext, outermost_context_base_type & outerm) {
            inner_context_ptr_type s = new MostDerived( my_context{pContext} ); 
            outerm.add(s); return s;
    }
};

struct state_machine<MostDerived, InitialState,
    Allocator=std::allocator<void>, ExceptionTranslator=null_exception_translator> {
    using allocator_type = Allocator;

    state_list_type currentStates_ = {};
    state_list_type::iterator currentStatesEnd_ = currentStates_.end();
    intrusive_ptr<node_state_base> pOutermostUnstableState_ = nullptr;
    state_base* pOutermostState = nullptr;
    bool isInnermostCommonOuter_ = false;

    using inner_orthogonal_position = integral_c<uint8_t, 0>;   // for single orthogonal
    using inner_context_type = MostDerived; // for single orthogonal
    using inner_context_ptr_type = state_machine*;
    using context_type_list = mpl::list<>;
    using outermost_context_base_type = state_machine;
    using outermost_context_type = MostDerived;

protected:
    state_machine(){}
    virtual ~state_machine(){ terminate_impl(false); }
public:
    struct state_iterator : std::iterator<std::forward_iterator_tag, state_base....> {..}

    outermost_context[_base]_type [const]& outermost_context[_base]() [const]; // return this
    Context [const]& context<Context>() [const]; // return this casted to Context

    state_iterator state_begin() const { return state_iterator{ currentStates_.begin() }; }
    state_iterator state_end() const { return state_iterator{ currentStatesEnd_ }; }
    Target state_cast<Target>() const {
        for (auto && it : { currentStates_.begin(), currentStatesEnd_ } ) {
            state_base* state = *it;
            while (state) {
                Target result = dynamic_cast<Target>(is_pointer_v<Target> ? state : *state);
                if (is_pointer_v<Target> && result != nullptr) return state;
                state = state->outer_state_ptr();
            }
        }
        if (is_pointer_v<Target>) return nullptr; else throw bad_cast();
    }
    Target state_downcast<Target>() const {
        // similar as state_cast, but test with (emulated) type_info comparasion and 'static_cast'
    }

    // outermost services
    void add<State>(intrusive_ptr<State> const& pState) { // state created (before child creation)
        if (leaf_state(pState)) { if (currentStatesEnd_ == currentStates_.end())
            { pState->set_list_position(currentStates_.insert(currentStateEnd_, pState)); }
            else        // this case is for reuse cached list items
            { *currentStatesEnd_ = pState; pState->set_list_position(currentStatesEnd_++); }
        }
        if (isInnermostCommonOuter_ ||
            is_highest<State>() && pOutermostUnstableState_ == pState->outer_state_ptr())
            pOutermostUnstableState_ = leaf_state(pState) ? nullptr : pState;
        isInnermostCommonOuter_ = false;
    }

    // context/state interface
    void add_inner_state(uint8_t, state_base* pOutermostState) { pOutermostState_ = pOutermostState; }
    void remove_inner_state(uint8_t) { pOutermostState_ = 0; }
    void exit_impl override (inner_context_ptr_type &, intrusive_ptr<node_state_base>&, bool) { }
};
```

* The structure is fully decided at compile-time: context and inner states are template type arguments.
  * `inner_context_type` and `inner_orthogonal_position` types are recursively available in each level of state.
  * `context_type_list` is a list of every outer state/context types.
  * `outermost_context` and `outermost_context_base` access the `state_machine`
  * `state_cast`, `state_downcast`, and `context` search and get state pointer from the current state tree.
* States are ref-counted, ref holded by `intrusive_ptr`.
  * State can be leaf/node, and each state remembers its outerstate's reference, and its orthogonal position
  * node state remembers all its inner state's pointer (not ref-counted)
  * Outermost `state_machine` only remember a `list` of leaf states
  * During 'unstable' (not fully constructed), the outermost unstable state is remembered by the machine
* `simple_state` don't have outer state reference during construction, while `state` takes its outer reference
  via constructor argument
* State construction is performed by `deep_construct`, which will construct itself, call `state_machine`'s `add`,
  then recursively construct all its inner states.
  * Leaf state `add` to `state_machine`'s list, which reuse any cached list node
  * If a newly added state is current 'outermost unstable's last inner, it will become the new 'outermost unstable'
  * `outermost unstable` is also updated when the root state is added (because of transition)
* State destruction is caused by removing from machine's leaf-state-list, and lost all references
  * `remove_from_state_list` is called recursively for all node-state's inner states.
  * `state_machine` cache allocated list nodes, and `remove_from_state_list` updates the end-marker.
  * if an empty node-state becomes the 'outermost unstable', it is explicitly called `exit_impl`,
    otherwise it will be called by last inner state's `exit_impl`.
  * `exit_impl` will maintain `state_machine` reference of 'outermost unstable' state.

------
### State Transition

```c++
struct event_base : rtti_base_type<counted_base> {
    intrusive_ptr<const event_base> intrusive_from_this() const; // either add-ref-count or clone
  protected:
    virtual ~event_base();
    virtual intrusive_ptr<const event_base> clone() const = 0;
};
struct event<MostDerived, Allocator> : rtti_derived_type<MostDerived, event_base> {
    // new, delete
    // protected ctor, dtor
  private: intrusive_ptr<const event_base> clone() const
  { return intrusive_ptr<const event_base>(new MostDerived(...)); }
};

enum reaction_result { no_reaction, do_forward_event, do_discard_event, do_defer_event, consumed };

struct reaction_dispatcher<Reactions,State,EventBase,Event,ActionContext,IdType> {
  static reaction_result react(State & stt, const EventBase& evt, const IdType& eventType) {
    if (Event <is> EventBase) {
      if (ActionContext <is> no_context<Event>) Reactions::react_without_action(stt);
      else                                      Reactions::react_with_action(stt, evt);
    } else {
      if (ActionContext <is> no_context<Event>) Reactions::react_without_action(stt);
      else
        Reactions::react_with_action(stt, *polymorphic_downcast<const Event*>(&evt));
    }
  }
};
struct transition<Event,Destination,
    TransitionContext=no_context<Event>,
    void (TransitionContext::*)(const Event&) pAction =no_context<Event>::no_function> {
  struct reactions<State> {
    static result react_without_action(State & stt) { return stt.transit<Destination>(); }
    static result react_with_action(State & stt, const Event & evt)
    { return stt.transit<Destination>(pAction, evt); }
  };
  static reaction_result react<State,EventBase,IdType>(State & stt, const EventBase & evt,
      const IdType& eventType) {
    return reaction_dispatcher<reactions<State>, State, EventBase, Event, TransitionContext, IdType>
      ::react(stt, evt, eventType);
  }
};

struct in_state_reaction<Event,
    ReactionContext=no_context<Event>,
    void (ReactionContext::*)(const Event&) pAction =no_context<Event>::no_function> {
  struct reactions<State> {
    static result react_without_action(State & stt) { return stt.discard_event(); }
    static result react_with_action(State & stt, const Event & evt)
    { stt.context<ReactionContext>().*pAction(evt); return react_without_action(stt); }
  };
  static reaction_result react<State,EventBase,IdType>(State & stt, const EventBase & evt,
      const IdType& eventType) {
    return reaction_dispatcher<reactions<State>, State, EventBase, Event, ReactionContext, IdType>
      ::react(stt, evt, eventType);
  }
};

struct deferral<Event> {
  static reaction_result react<State,EventBase,IdType>(State & stt, const EventBase & evt,
      const IdType& eventType) {
    if (eventType == Event::static_type()) return stt.defer_event();
    else return no_reaction;
  }
};

struct termination<Event> {
  static reaction_result react<State,EventBase,IdType>(State & stt, const EventBase & evt,
      const IdType& eventType) {
    if (eventType == Event::static_type()) return stt.terminate();
    else return no_reaction;
  }
};

struct custom_reaction<Event> {
  static reaction_result react<State,EventBase,IdType>(State & stt, const EventBase & evt,
      const IdType& eventType) {
    if (eventType == Event::static_type())
      return stt.react(*polymorphic_downcast<const Event*>(&evt));
    else return no_reaction;
  }
};
```

------
### Event Processor and Asynchronous

```c++
struct event_processor<Scheduler> {
  private:
    virtual void initiate_impl() = 0;
    virtual void process_event_impl(const event_base & evt) = 0;
    virtual void terminate_impl() = 0;
    
    Scheduler & sch_; const processor_handle handle_;
  protected:
    event_processor(Scheduler::processor_context ctx)
        : sch_(ctx.my_scheduler()), handle_(ctx.my_handle()) {}
  public:
    Scheduler & my_scheduler() const; processor_handle my_handle() const;
    void initiate(); void process_event(const event_base &evt); void terminate(); // call virtuals
    virtual ~event_processor();
};

struct processor_container<Scheduler, WorkItem, Allocator> {
    using processor_holder_type = auto_ptr<event_processor<Scheduler>>;
    using processor_handle = weak_ptr<processor_holder_type>;
    struct processor_context {
      private: Scheduler & sch_; processor_handle handle_;
      public: Scheduler& my_scheduler() const; processor_handle & my_handle() const;
    };
    set<processor_holder_ptr_type> processorSet_;

    WorkItem create_processor<Processor,...Arg>(processor_handle& handle, Scheduler& scheduler, Arg...arg) {
      shared_ptr<processor_holder_ptr_type> p = make_processor_holder();
      handle = p;
      processor_context ctx{scheduler, handle};
      auto impl = []{
        processorSet_.insert(p);
        processor_holder_type holder(new Processor(ctx, arg...));
        *p = holder;
      };
      return WorkItem(impl, Allocator());
    }
    WorkItem destroy_processor(const processor_handle& processor) {
      auto impl = [] {
        const processor_holder_ptr_type p = processor.lock();
        if (p) processorSet_.erase(p);
      };
      return WorkItem(impl, Allocator());
    }
    WorkItem initiate_processor(const processor_handle& processor) {
      auto impl = [] {
        const processor_holder_ptr_type p = processor.lock();
        if (p) (*p)->initiate();
      };
      return WorkItem(impl, Allocator());
    }
    WorkItem terminate_processor(const processor_handle& processor) {
      auto impl = [] {
        const processor_holder_ptr_type p = processor.lock();
        if (p) (*p)->terminate();
      };
      return WorkItem(impl, Allocator());
    }
    WorkItem queue_event(const processor_handle& processor, const intrusive_ptr<const event_base>& pEvent) {
      auto impl = [] {
        const processor_holder_ptr_type p = processor.lock();
        if (p) (*p)->process_event(*pEvent);
      };
      return WorkItem(impl, Allocator());
    }
};

struct fifo_worker<Allocator> {
  using work_item = function<void()>;

  bool terminated_;
  const bool waitOnEmptyQueue_;
  mutex mutex_; condition queueNotEmpty_;
  list<work_item> waitOnEmptyQueue_;

  fifo_worker(bool waitOnEmptyQueue=false) : waitOnEmptyQueue_(waitOnEmptyQueue), terminated_(false) {}

  void queue_work_item(work_item& item) {
    if (item.empty()) return;
    mutex::scoped_lock lock(mutex_);
    workQueue_.push_back(work_item()); workQueue_.back().swap(item);
    queueNotEmpty_.notify_one();
  }
  void queue_work_item(const work_item& item) { work_item copy{item}; queue_work_item(copy); }

  void terminate() {
    queue_work_item(work_item([]{ terminated_ = true; }));
  }
  bool terminated() const { return terminated_; }
  
  unsigned long operator()(unsigned long maxItemCount=0) {
    unsigned long itemCount = 0;
    while (!terimated() && (maxItemCount==0 || itemCount < maxItemCount)) {
        work_item item = dequeue_item(); if (item.empty()) break;
        item(); ++itemCount;
    }
    return itemCount;
  }
  work_item dequeue_item() {
    mutex::scoped_lock lock(mutex_);
    if (!waitOnEmptyQueue_ && workQueue_.empty()) return work_item(); // return empty item
    while (workQueue_.empty()) queueNotEmpty_.wait(lock);
    work_item result; result.swap(workQueue_.front()); workQueue_.pop_front();
    return result;
  }
};

struct fifo_scheduler<FifoWorker=fifo_worker<>, Allocator> {
  using container = processor_container<fifo_scheduler, FifoWorker::work_item, Allocator>;
  private: container container_; FifoWorker worker_;
  public:
    processor_handle create_processor<Processor,...Arg>(Arg...arg) {
      processor_handle result;
      work_item item = container_.create_processor<Processor>(result, *this, arg...);
      worker_.queue_work_item(item);
    }
    void destroy_processor(const processor_handle& processor) {
      auto item = container_.destroy_processor(processor);
      worker_.queue_work_item(item);
    }
    void initiate_processor(const processor_handle& processor) {
      auto item = container_.initiate_processor(processor);
      worker_.queue_work_item(item);
    }
    void terminate_processor(const processor_handle& processor) {
      auto item = container_.terminate_processor(processor);
      worker_.queue_work_item(item);
    }
    void queue_event(const processor_handle& processor, const intrusive_ptr<const event_base>& pEvent) {
      auto item = container_.queue_event(processor, pEvent);
      worker_.queue_work_item(item);
    }
    void queue_work_item([const] work_item & item) { worker_.queue_work_item(item); }
    void terminate() { worker_.terminate(); }
    bool terminated() { return worker_.terminated(); }
    unsigned long operator() (unsigned long maxEventCount = 0) { return worker_(maxEventCount); }
};

struct asynchronous_state_machine<MostDerived, InitialState,
    Scheduler=fifo_scheduler,
    Allocator=std::allocator<void>, ExceptionTranslator=null_exception_translator> :
  state_machine<MostDerived,InitialState,Allocator,ExceptionTranslator>,
  event_processor<Scheduler>
{
    void terminate() { event_processor::terminate(); }
    void initiate_impl() override { state_machine::initiate(); }
    void process_event_impl(const event_base& evt) override { state_machine::process_event(evt); }
    void terminate_impl() override { state_machine::terminate(); }
};
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`.

#### Boost.Assert

* `<boost/assert.hpp>`.

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`.

 #### Boost.ThrowException

* `<boost/throw_exception.hpp>`.

#### Boost.Core

* `<boost/noncopyable.hpp>`
* `<boost/ref.hpp>`
* `<boost/get_pointer.hpp>`

#### Boost.SmartPtr

* `<boost/intrusive_ptr.hpp>`
* `<boost/shared_ptr.hpp>`
* `<boost/weak_ptr.hpp>`
* `<boost/detail/atomic_count.hpp>`

#### Boost.Conversion

* `<boost/polymorphic_cast.hpp>`

#### Boost.Function

* `<boost/function/function0.hpp>`

#### Boost.Bind

* `<boost/bind.hpp>`

#### Boost.Detail

* `<boost/detail/allocator_utilities.hpp>`

#### Boost.Thread

* `<boost/thread/mutex.hpp>`, `<boost/thread/condition.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`, is_same, is_pointer, remove_reference, is_base_of

#### Boost.MPL

* `<boost/mpl/*.hpp>`

------
### Standard Facilities
