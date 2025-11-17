# Boost.Fiber

* lib: `boost/libs/fiber`
* repo: `boostorg/fiber`
* commit: `238487b`, 2025-02-21

------
### Fiber Management API

Main Header `<boost/fiber/all.hpp>`

```c++
```

##### Exceptions

```c++
struct fiber_error : public std::system_error {
  explicit ctor(std::error_code ec);
  ctor(std::error_code ec, {const char*|std::string const&} what_arg);
};
struct lock_error : public std::fiber_error {}; // 3 base::ctor

enum class future_errc { broken_promise=1,future_already_retrieved,promise_already_satisfied,no_state};
std::error_catgory const& future_category() noexcept;
struct std::is_error_code_enum<future_errc> : true_type{};
std::error_code std::make_error_code(future_errc e) noexcept;
std::error_condition std::make_error_condition(future_errc e) noexcept;

struct future_error : fiber_error { explicit ctor(std::error_code ec); };
struct future_uninitialized : future_error {};
struct future_already_retrieved : future_error {};
struct broken_promise : future_error {};
struct promise_already_satisfied : future_error {};
struct promise_uninitialized : future_error {};
struct packaged_stack_uninitialized : future_error {};
```

##### Context

```c++
using detail::ready_hook = intrusive::list_member_hook<tag<struct ready_tag>, link_mode<auto_unlink>>;
using detail::sleep_hook = intrusive::set_member_hook<tag<struct sleep_tag>, link_mode<auto_unlink>>;
using detail::worker_hook = intrusive::list_member_hook<tag<struct worker_tag>, link_mode<auto_unlink>>;
using detail::terminated_hook = intrusive::slist_member_hook<tag<struct terminated_tag>, link_mode<safe_link>>;
using detail::remote_ready_hook = intrusive::slist_member_hook<tag<struct remote_ready_tag>, link_mode<safe_link>>;
class context {
  struct fss_data { void* vp{nullptr};, fss_cleanup_function::ptr_t cleanup_function{}
    ctor()=default; ctor(void* vp_, fss_cleanup_function::ptr_t fn) noexcept;
    void do_cleanup(){ (*cleanup_function)(vp); }
  }; using fss_data_t = std::map<uintptr_t, fss_data>;

  std::atomic<size_t> use_count_;
  remote_ready_hook remote_ready_hook_{}; spinlock splk_{};
  bool terminated_{false}; wait_queue wait_queue_{}; std::atomic<size_t> waker_epoch_{0};
  scheduler* scheduler_{nullptr};
  fss_data_t fss_data_{};
  sleep_hook sleep_hook_{}; waker sleep_waker_{}; ready_hook ready_hook_{}; terminated_hook terminated_hook_{}; worker_hook worker_hook_{};
  fiber_properties* properties_{nullptr};
  context::fiber c{}; steady_clock::time_point tp_; type type_; launch policy_;
  ctor(size_t initial_count, type t, launch policy) noexcept;
public:
  class id { context* impl_{nullptr};
  public: ctor()=default; explicit ctor(context* impl) noexcept;
    bool operator==(self const& other) const noexcept; // and != <, >, <=, >=
    friend std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, self const& id);
    explicit operator bool() const noexcept; bool operator!() const noexcept;
  };
  static bool initialized_thread(algorithm::ptr_t algo, stack_allocator_wrapper&& salloc) noexcept;
  static context* active() noexcept;
  static void reset_active() noexcept;
  ctor(self {const&|&&})=delete; self& operator=(self {const&|&&})=delete;

  friend bool operator==(self const& l, self const& r) noexcept;
  virtual ~dtor();
  scheduler* get_scheduler() const noexcept { return scheduler_; }
  id get_id() const noexcept;
  bool is_resumable() const noexcept { return (bool)c_; }
  void resume()noexcept; void resume(spinlock_lock&) noexcept; void resume(context*) noexcept;
  void suspend() noexcept; void suspend(spinlock_lock&) noexcept;
  context::fiber suspend_with_cc() noexcept;
  context::fiber terminate() noexcept;
  void join();
  void yield() noexcept;
  bool wait_until(steady_clock::time_point const&) noexcept;
  bool wait_until(steady_clock::time_point const&, spinlock_lock&, waker&&) noexcept;
  bool wait(size_t) noexcept;
  waker create_waker() noexcept { return {this, **waker_epoch_}; }
  void schedule(context*) noexcept;
  bool is_context(type t) const noexcept{ return (type_&t)!=none; }
  void* get_fss_data(void const* vp) const;
  void set_fss_data(void const* vp, fss_cleanup_function::ptr_t const& cleanup_fn, void* data, bool cleanup_existing);
  void set_properties(fiber_properties* props) noexcept;
  fiber_properties* get_properties() const noexcept { return properties_; }
  launch get_policy() const noexcept { return policy_; }
  bool worker_is_linked() const noexcept; void worker_link<List>(List& lst) noexcept; void worker_unlink() noexcept
  bool ready_is_linked() const noexcept; void ready_link<List>(List& lst) noexcept; void ready_unlink() noexcept;
  bool remote_ready_is_linked() const noexcept; void remote_ready_link<List>(List& lst) noexcept; void sleep_unlink() noexcept;
  bool sleep_is_linked() const noexcept; void sleep_link<Set>(Set& set) noexcept;
  bool terminated_is_linked() const noexcept; void terminated_link<List><List>(List& lst) noexcept;
  void detach() noexcept; void attach(context*) noexcept;
  friend void intrusive_ptr_add_ref(self*) noexcept;
  friend void intrusive_ptr_release(self*) noexcept;
};
bool operator<(context const& l, context const& r) noexcept { return l.get_id() < r.get_id(); }

class wroker_context<Fn,...Arg> final : public context {
  std::decay_t<Fn> fn_; std::tuple<Arg...> arg_;
  context::fiber run_(context::fiber&& c);
public:
  ctor<StackAlloc>(launch policy, <fiber_properties* properties>, context::preallocated const& palloc, StackAlloc&& salloc, Fn&& fn, Arg...arg);
};

static intrusive_ptr<context> make_worker_context_with_properties<StackAlloc,Fn,...Arg>(launch policy, fiber_properties* properties, StackAlloc&& salloc, Fn&& fn, Arg...arg);
static intrusive_ptr<context> make_worker_context<StackAlloc,Fn,...Arg>(launch policy, StackAlloc&& salloc, Fn&& fn, Arg...arg);

struct std::hash<context::id> { size_t operator()(context::id const& id) const noexcept; };
```

##### Fiber

```c++
class fiber { using ptr_t = intrusive_ptr<context>;
  ptr_t impl_{};
  void start_() noexcept;
public: using id = context::id;
  ctor()=default;
  explicit ctor<Fn,...Arg>(<fiber_properties* properties>, Fn&& fn, Arg&&...arg) requires(disable_overload<{fiber|launch|std::allocator_arg_t},Fn>);
  ctor<Fn,...Arg>(launch policy, <fiber_properties* properties>, Fn&& fn, Arg&&...arg) requires disable_overload<fiber,Fn>;
  ctor<StackAlloc,Fn,...Arg>(<launch policy>, std::allocator_arg_t, StackAlloc&& salloc, Fn&& fn, Arg&&...arg);
  ~dtor(); ctor(self const&)=delete; self& operator=(self const&)=delete;
  ctor(self &&); self& operator=(self &&);
  void swap(fiber& other) noexcept { impl_.swap(other.impl_); }
  id get_id() const noexcept { return impl_? impl_->get_id() : id{}; }
  bool joinable() const noexcept { return impl_!=nullptr; }
  void join(); void detach();
  PROPS& properties<PROPS>() { return dynamic_cast<PROPS&>(*impl_get_properties()); }
};
bool operator<(fiber const& l, fiber const& r) noexcept { return l.get_id() < r.get_id(); }
void swap(fiber& l, fiber& r) noexcept { return l.swap(r); }

fiber::id this_fiber::get_id() noexcept;
void this_fiber::yield() noexcept;
void this_fiber::sleep_until<Clock,Duration>(std::chrono::time_point<Clock,Duration> const& sleep_time_);
void this_fiber::sleep_for<Rep,Period>(std::chrono::duration<Rep,Period> const& timeout_duration);
PROPS& this_fiber::properties<PROPS>();

bool has_ready_fibers() noexcept;
bool initialize_thread(algorithm::ptr_t algo, stack_allocator_wrapper&& salloc) noexcept;
void use_scheduling_algorithm<SchedAlgo,...Args>(Args&&...args) noexcept;
```

##### Scheduler

```c++
class scheduler {
  struct timepoint_less { bool operator()(context const& l, context const& r) const noexcept; };
  using ready_queue_type = intrusive::list<context, member_hook<context,ready_hook,&context::ready_hook_>, constant_time_size<false>>;
  using sleep_queue_type = intrusive::multiset<context, member_hook<context,sleep_hook,&context::sleep_hook_>, constant_time_size<false>, compare<timepoint_less>>;
  using worker_queue_type = intrusive::list<context, member_hook<context,worker_hook,&context::worker_hook_>, constant_time_size<false>>;
  using terminated_queue_type = intrusive::slist<context, member_hook<context,terminated_hook,&context::terminated_hook_>, linear<true>, cache_last<true>>;
  using remote_ready_queue_type = intrusive::slist<context, member_hook<context,remote_ready_hook,&context::remote_ready_hook_>, linear<true>, cache_last<true>>;
  spinlock remote_ready_splk_{}; remote_ready_queue_type remote_ready_queue_{};
  algo::ptr_t algo_; sleep_queue_type sleep_queue_{}; worker_queue_type worker_queue_{}; terminated_queue_type terminated_queue_{};
  intrusive_ptr<context> dispatcher_ctx_{}; context* main_ctx_{nullptr}; bool shutdown_{false};
  void release_terminated_() noexcept;
  void remote_ready2ready_() noexcept;
  void sleep2ready_() noexcept;
public: ctor(algorithm::ptr_t algo) noexcept; ctor(self const&)=delete; self& operator=(self const&)=delete; virtual ~dtor();
  void schedule(context *) noexcept; void schedule_from_remote(context*) noexcept;
  context::fiber dispatch() noexcept; context::fiber terminate(spinlock_lock&, context*) noexcept;
  void yield(context*) noexcept;
  bool wait_until(context* steady_clock::time_point const&) noexcept;
  bool wait_until(context* steady_clock::time_point const&, spinlock_lock&, waker&&) noexcept;
  void suspend() noexcept; void suspend(spinlock_lock&) noexcept;
  bool has_ready_fibers() const noexcept;
  void set_algo(algorithm::ptr_t) noexcept;
  void attach_main_context(context*) noexcept;
  void attach_dispatcher_context(intrusive_ptr<context>) noexcept;
  void attach_worker_context(context*) noexcept; void detach_worker_context(context*) noexcept;
};
```

##### Fiber Specific Storage

```c++
class detail::fss_cleanup_function {
  std::atomic<size_t> use_count_{0};
public: using ptr_t = intrusive_ptr<self>;
  ctor()=default; virtual ~dtor()=default;
  virtual void operator()(void* data) =0;
  friend void intrusive_ptr_add_ref(self* p) noexcept;
  friend void intrusive_ptr_release(self* p) noexcept;
};

class fiber_specific_ptr<T> {
  struct default_cleanup_function : fss_cleanup_function
  { void operator()(void* data) noexcept override { delete (T*)data; } };
  struct custom_cleanup_function : fss_cleanup_function {
    void (*fn)(T*); explicit ctor(void(*fn_)(T*)) noexcept;
    void operator()(void* data) noexcept override { if (fn) fn((T*)data); }
  };
  fss_cleanup_function::ptr_t cleanup_fn_;
public: using element_type = T; // no copy
  ctor() :cleanup_fn_{new default_cleanup_function{}}{}
  explicit ctor(void(*fn)(T*)) :cleanup_fn_{new custom_cleanup_function{fn}}{}
  ~dtor() { auto active_ctx=context::active(); if (active_ctx) active_ctx->set_fss_data(this, cleanup_fn_, nullptr, true); }
  T* get() const noexcept { (T*)context::active()->get_fss_data(this); }
  T* operator->() const noexcept; T& operator*() const noexcept;
  T* release() { T* tmp=get(); context::active()->set_fss_data(this, cleanup_fn_, nullptr, false); return tmp; }
  void reset(T* t) { T* c=get() if (c!=t) context::active()->set_fss_data(this, cleanup_fn_, t, true); }
};
```

##### Threadings

```c++
class mutex {
  spinlock wait_queue_splk_{}; wait_queue wait_queue_{}; context* owner_{nullptr};
public: ctor()=default; dtor();
  void lock(); bool try_lock(); void unlock();
};
class recursive_mutex {
  spinlock wait_queue_splk_{}; wait_queue wait_queue_{}; context* owner_{nullptr}; size_t count{0};
public: ctor()=default; dtor(); // no copy
  void lock(); bool try_lock(); void unlock();
};
class timed_mutex {
  spinlock wait_queue_splk_{}; wait_queue wait_queue_{}; context* owner_{nullptr};
public: ctor()=default; dtor(); // no copy
  void lock(); bool try_lock(); void unlock();
  bool try_lock_until<Clock,Duration>(time_point<Clock,Duration> const& timeout_time_);
  bool try_lock_for<Rep,Period>(duration<Rep,Period> const& timeout_duration);
};
class recursive_timed_mutex {
  spinlock wait_queue_splk_{}; wait_queue wait_queue_{}; context* owner_{nullptr}; size_t count{0};
public: ctor()=default; dtor(); // no copy
  void lock(); bool try_lock(); void unlock();
  bool try_lock_until<Clock,Duration>(time_point<Clock,Duration> const& timeout_time_);
  bool try_lock_for<Rep,Period>(duration<Rep,Period> const& timeout_duration);
};

enum class cv_status { no_timeout=1, timeout };
class condition_variable_any {
  spinlock wait_queue_splk_{}; wait_queue wait_queue_{};
public: ctor()=default; ~dtor(); // no copy
  void notify_one() noexcept; void notify_all() noexcept;
  void wait<LockType,[Pred]>(LockType& lt, <Pred pred>);
  cv_status wait_until<LockType,Clock,Duration>(LockType& lt, time_point<Clock,Duration> const& timeout_time);
  bool wait_until<LockType,Clock,Duration,Pred>(LockType& lt, time_point<Clock,Duration> const& timeout_time, Pred pred);
  cv_status wait_for<LockType,Rep,Period>(LockType& lt, duration<Rep,Period> const& timeout_time);
  bool wait_for<LockType,Rep,Period,Pred>(LockType& lt, duration<Rep,Period> const& timeout_time, Pred pred);
};
class condition_variable {
  condition_variable_any cnd_;
public: ctor()=default; // no copy
  void notify_one() noexcept; void notify_all() noexcept;
  void wait<[Pred]>(std::unique_lock<mutex>& lt, <Pred pred>);
  cv_status wait_until<Clock,Duration>(std::unique_lock<mutex>& lt, time_point<Clock,Duration> const& timeout_time);
  bool wait_until<Clock,Duration,Pred>(std::unique_lock<mutex>& lt, time_point<Clock,Duration> const& timeout_time, Pred pred);
  cv_status wait_for<Rep,Period>(std::unique_lock<mutex>& lt, duration<Rep,Period> const& timeout_time);
  bool wait_for<Rep,Period,Pred>(std::unique_lock<mutex>& lt, duration<Rep,Period> const& timeout_time, Pred pred);
};

class barrier {
  size_t initial_, current_, cycle_{0}; mutex mtx_{}; conditional_variable cond_{};
public:
  explicit ctor(size_t);
  bool wait();
};

enum class channel_op_status { success, empty, full, closed, timeout };
class buffered_channel<T> { using slot_type = value_type;
  mutable spinlock splk_{}; wait_queue waiting_producers_{}, waiting_consumers_{};
  slot_type* slots_; size_t pidx_{0}, cidx_{0}, capacity_; bool closed_{false};
  bool is_full_() const noexcept { return cidx_==((pidx_+1)%capacity_); }
  bool is_empty_() const noexcept { return cidx_==pidx_; }
  bool is_closed_() const noexcept { return closed_; }
public: using value_type = std::remove_reference_t<T>;
  explicit ctor(size_t capacity); ~dtor(); // no copy
  bool is_closed() const noexcept; void close() noexcept;
  channel_op_status <try>_push(value_type {const&|&&} value);
  channel_op_status push_wait_for<Rep,Period>(value_type {const&|&&} value, duration<Rep,Period> const& timeout_duration);
  channel_op_status push_wait_until<Clock,Duration>(value_type {const&|&&} value, time_point<Clock,Duration> const& timeout_time);
  channel_op_status <try>_pop(value_type& value); value_type value_pop();
  channel_op_status pop_wait_for<Rep,Period>(value_type& value, duration<Rep,Period> const& timeout_duration);
  channel_op_status pop_wait_until<Clock,Duration>(value_type& value, time_point<Clock,Duration> const& timeout_time);
  class iterator {
    using storage_type = std::aligned_storage<sizeof(value_type), alignof(value_type)>::type;
    buffered_channel* chan_{nullptr}; storage_type storage_;
    void increment_(bool initial=false);
  public: using iterator_category = std::input_iterator_tag;
    using difference_type = std::ptrdiff_t; using pointer = value_type*; using reference = value_type&;
    ctor()=default; ctor(self const&) noexcept; self& operator=(self const&) noexcept;
    explicit ctor(buffered_channel<T>* chan) noexcept;
    bool operator==(self const&) const noexcept // and !=
    self& operator++(); const self operator++(int)=delete;
    reference operator*() noexcept; pointer operator->() noexcept;
  };
  friend iterator begin(self& chan); friend iterator end(self&);
};
class unbuffered_channel<T> {
  struct slot { value_type value; waker w; ctor(value_type {const&|&&} value_, waker&& w); };
  std::atomic<slot*> slot_{nullptr}; std::atomic_bool closed_{false};
  mutable spinlock splk_producers_{}, splk_consumers_{};
  wait_queue waiting_producers_{}, waiting_consumers_{}; char pad_[cacheline_length];
  bool is_empty_() const noexcept { return cidx_==pidx_; }
  bool try_push_(slot* own_slot); slot* try_pop_();
public: using value_type = std::remove_reference_t<T>;
  ctor()=default; ~dtor(); // no copy
  bool is_closed() const noexcept; void close() noexcept;
  channel_op_status push(value_type {const&|&&} value);
  channel_op_status push_wait_for<Rep,Period>(value_type {const&|&&} value, duration<Rep,Period> const& timeout_duration);
  channel_op_status push_wait_until<Clock,Duration>(value_type {const&|&&} value, time_point<Clock,Duration> const& timeout_time);
  channel_op_status pop(value_type& value); value_type value_pop();
  channel_op_status pop_wait_for<Rep,Period>(value_type& value, duration<Rep,Period> const& timeout_duration);
  channel_op_status pop_wait_until<Clock,Duration>(value_type& value, time_point<Clock,Duration> const& timeout_time);
  class iterator {
    using storage_type = std::aligned_storage<sizeof(value_type), alignof(value_type)>::type;
    unbuffered_channel* chan_{nullptr}; storage_type storage_;
    void increment_(bool initial=false);
  public: using iterator_category = std::input_iterator_tag;
    using difference_type = std::ptrdiff_t; using pointer = value_type*; using reference = value_type&;
    ctor()=default; ctor(self const&) noexcept; self& operator=(self const&) noexcept;
    explicit ctor(unbuffered_channel<T>* chan) noexcept;
    bool operator==(self const&) const noexcept // and !=
    self& operator++(); const self operator++(int)=delete;
    reference operator*() noexcept; pointer operator->() noexcept;
  };
  friend iterator begin(self& chan); friend iterator end(self&);
};
```

##### Futures, Promises, Tasks

```c++
class detail::shared_state_base {
  std::atomic<size_t> use_count_{0}; mutable condition_variable waiters_{};
protected:
  mutable mutex mtx_{}; bool ready_{false}; std::exception_ptr except_{};
  void mark_ready_and_notify_(std::unique_lock<mutex>& lk) noexcept;
  void owner_destroyed_(std::unique_lock<mutex>& lk);
  void set_exception_(std::exception_ptr except, std::unique_lock<mutex>& lk);
  std::exception_ptr get_exception_ptr_(std::unique_lock<mutex>& lk);
  void wait_(std::unique_lock<mutex>& lk) const;
  future_state wait_for_<Rep,Period>(std::unique_lock<mutex>& lk, duration<Rep,Period> const& timeout_duration) const;
  future_state wait_until_<Clock,Duration>(std::unique_lock<mutex>& lk, time_point<Clock,Duration> const& timeout_time) const;
  virtual void deallocate_future() noexcept =0;
public: ctor()=default; virtual ~dtor()=default; // no copy
  void owner_destroyed();
  void set_exception(std::exception_ptr except);
  std::exception_ptr get_exception_ptr_();
  void wait_() const;
  future_state wait_for<Rep,Period>(duration<Rep,Period> const& timeout_duration) const;
  future_state wait_until<Clock,Duration>(time_point<Clock,Duration> const& timeout_time) const;
  friend void intrusive_ptr_add_ref(self*) noexcept;
  friend void intrusive_ptr_release(self*) noexcept;
};
class detail::shared_state<R> : public shared_state_base {
  alignas(alignof(R)) unsigned char storage_[sizeof(R)]{};
  void set_value_(R {const&|&&} value, std::unique_lock<mutex>& lk);
  R& get_(std::unique_lock<mutex>& lk);
public: using ptr_type = intrusive_ptr<shared_state>;
  void set_value(R {const&|&&} value); R& get();
};
class detail::shared_state<R&> : public shared_state_base {
  R* value_{nullptr};
  void set_value_(R& value, std::unique_lock<mutex>& lk);
  R& get_(std::unique_lock<mutex>& lk);
public: using ptr_type = intrusive_ptr<shared_state>;
  void set_value(R& value); R& get();
};
class detail::shared_state<void> : public shared_state_base {
  void set_value_(std::unique_lock<mutex>& lk);
  R& get_(std::unique_lock<mutex>& lk);
public: using ptr_type = intrusive_ptr<shared_state>;
  void set_value(); void get();
};

class detail::shared_state_object<R,Allocator> : public shared_state<R> {
public: using allocator_type = std::allocator_traits<Allocator>::rebind_alloc<self>;
  ctor(allocator_type const& alloc);
protected: void deallocate_future() noexcept override final;
private: allocator_type alloc_;
  static void destroy_(allocator_type const& alloc, shared_state_object* p) noexcept;
};

enum class future_status { ready=1, timeout, deferred };
struct detail::future_base<R> {
  using ptr_type = shared_state<R>::ptr_type;
  ptr_type state_{};
  ctor()=default; ~dtor()=default; explicit ctor(ptr_type p);
  ctor(self const&); ctor(self&&) noexcept; self& operator=(self const&) noexcept; self& operator=(self&&) noexcept;
  bool valid() const noexcept;
  std::exception_ptr get_exception_ptr();
  void wait() const;
  future_status wait_for<Rep,Period>(duration<Rep,Peroid> const& timeout_duration) const;
  future_status wait_until<Clock,Duration>(time_point<Clock,Duration> const& timeout_time) const;
};

class future<R> : future_base<R> {
  explicit ctor(base::ptr_type const& p) noexcept;
public: ctor()=default; ctor(self&&) noexcept; self& operator=(self&&) noexcept; // no copy
  shared_future<R> share(); R get();
};
class future<R&> : future_base<R&> {}; // same members as future<R>
class future<void> : future_base<void> {}; // same members as future<R>

class shared_future<R> : future_base<R> {
  explicit ctor(base::ptr_type const& p) noexcept;
public: ctor()=default; ~dtor()=default;
  ctor(self {const&|&&}) noexcept; self& operator=(self {const&|&&}) noexcept;
  ctor(future<R>&&) noexcept; self& operator=(future<R>&&) noexcept;
  R get();
};
class shared_future<R&> : future_base<R&> {}; // same members as shared_future<R>
class shared_future<void> : future_base<void> {}; // same members as shared_future<R>

struct detail::promise_base<R> {
  using ptr_type = shared_state<R>::ptr_type;
  bool obtained_{false}; ptr_type future_{};
  ctor(); ctor<Alloc>(std::allocator_arg_t, Alloc alloc); ~dtor();
  ctor(self&&) noexcept; self& operator=(self&&) noexcept; // no copy
  future<R> get_future();
  void swap(promise_base& other) noexcept;
  void set_exception(std::exception_ptr p);
};
struct promise<R> : promise_base<R> {
  ctor()=default; ctor<Alloc>(std::allocator_arg_t, Alloc alloc); // no copy, move=default
  void set_value(R {const&|&&} value);
  void swap(self& other) noexcept;
};
struct promise<R&> : promise_base<R&> {}; // same members as promise<R>
struct promise<void> : promise_base<void> {}; // same members as promise<R>
void swap<R>(promise<R>& l, promise<R>& r) noexcept;

struct detail::task_base<R,...Args> : shared_state<R> {
  using ptr_type = intrusive_ptr<task_base>;
  virtual ~dtor(){}
  virtual void run(Args&&...args)=0;
  virtual ptr_type reset()=0;
};
class detail::task_object<Fn,Alloc,R,...Args> : public task_base<R,Args...> {
  Fn fn_; allocator_type alloc_;
  static void destroy_(allocator_type const& alloc, task_object* p) noexcept;
protected: void deallocate_future() noexcept override final;
public: using allocator_type = std::allocator_traits<Alloc>::rebind_alloc<self>;
  ctor(allocator_type const& alloc, Fn {const&|&&} fn);
  void run(Args&&...args) override final;
  ptr_type reset() override final;
};
class detail::task_object<Fn,Alloc,void,Args...> : public task_base<void,Args...> {}; // same members as primary

class packaged_task<Signature>;
class packaged_task<R(Args...)> {
  using ptr_type = task_base<R,Args...>::ptr_type;
  bool obtained_{false}; ptr_type task_{};
public: ctor()=default; ~dtor(){} ctor(self&&) noexcept; self& operator=(self&&) noexcept; // no copy
  explicit ctor<Fn>(Fn&& fn) requires(...);
  explicit ctor<Fn>(std::allocator_arg_t, Alloc const& alloc, Fn&& fn);
  void swap(self& other) noexcept;
  bool valid() const noexcept;
  future<R> get_future();
  void operator()(Args...args);
  void reset();
};
void swap<Signature>(packaged_task<Signature>& l, packaged_task<Signature>& r) noexcept;

auto async<[Policy],Fn,...Args>(<Policy policy>, Fn&& fn, Args...args)
  -> future<result_of_t<decay_t<Fn>(decay_t<Args>...)>>;
auto async<Policy,StackAlloc,[Alloc],Fn,...Args>(Policy policy, std::allocator_arg_t, StackAlloc salloc, <Alloc alloc>, Fn&& fn, Args...args)
  -> future<result_of_t<decay_t<Fn>(decay_t<Args>...)>>;
```

##### Common Bits

```c++
enum class launch { dispatch, post };
struct detail::is_launch_policy<Fs> : std::false_type{};
struct detail::is_launch_policy<launch> : std::true_type{};

class fiber_properties {
protected: context* ctx_; algo::algorithm* algo_{nullptr};
  void notify() noexcept;
public: explicit ctor(context* ctx) noexcept; virtual ~dtor()=default;
  void set_algorithm(algo::algorithm* algo) noexcept;
  void set_context(context* ctx) noexcept;
};

enum class type { none=0, main_context=2, dispatcher_context=4, worker_context=8, pinned_context=main_context|dispatcher_context };
constexpr type operator&(type l, type r); // and |, ^, ~, &=, |=, ^=

using detail::waker_queue_hook = intrusive::slist_member_hook<>;
class waker { context* ctx_{}, size_t epoch_{};
public: ctor()=default; ctor(context* ctx, const size_t epoch);
  bool wake() const noexcept;
};
struct waker_with_hook : waker {
  waker_queue_hook waker_queue_hook_{};
  explicit ctor(waker&& w);
  bool is_linked() const noexcept;
  friend bool operator==(waker const& l, waker const& r) noexcept;
};
using detail::waker_slist_t = intrusive::slist<waker_with_hook,
  intrusive::member_hook<waker_with_hook, waker_queue_hook, &waker_with_hook::waker_queue_hook_>,
  constant_time_size<false>, cache_last<true>>;

class wait_queue {
  waker_slist_t slist_{};
public: void suspend_and_wait(spinlock_lock&, context*);
  bool suspend_and_wait_until(spinlock_lock&, context*, std::chrono::steady_clock::time_point const&);
  void notify_one(); void notify_all(); bool empty() const;
};

// stack_allocator_wrapper
struct detail::polymorphic_stack_allocator_base {
  ctor()=default; virtual ~dtor()=default; // delete copy/move ctor and assign
  virtual context::stack_context allocate() = 0;
  virtual void deallocate(context::stack_context& sctx) = 0;
};
class polymorphic_stack_allocator_impl<StackAllocator> final : public polymorphic_stack_allocator_base {
  StackAllocator _allocator;
public: ctor<...Args>(Args&&...args); ~dtor()=default;
  context::stack_context allocate() override { return _allocator.allocate(); }
  void deallocate(context::stack_context& sctx) override { _allocate.deallocate(sctx); }
};
class stack_allocator_wrapper final { // delete copy/move ctor and assign
  std::unique_ptr<polymorphic_stack_allocator_base> _allocator;
public: ctor(std::unique_ptr<polymorphic_stack_allocator_base> allocator); ~dtor()=default;
  context::stack_context allocate() { return _allocator->allocate(); }
  void deallocate(context::stack_context& sctx) override { _allocate->deallocate(sctx); }
};
stack_allocator_wrapper make_stack_allocator_wrapper<StackAllocator,...Args>(Args&&...args)
{ return {std::make_unique_ptr<polymorphic_stack_allocator_impl<StackAllocator>>(std::forward<Args>(args)...)}; }

class detail::context_spinlock_queue { using slot_type = context*;
  mutable spinlock splk_{};
  size_t pidx_{0}, cidx_{0}, capacity_; slot_type* slots_;
  void resize_(); bool is_full_() const noexcept; bool is_empty_() const noexcept;
public: ctor(size_t capacity=4096); ~dtor(); // no copy
  bool empty() const noexcept;
  void push(context* c); context* pop();
  context* steal();
};
class detail::context_spmc_queue {
  class array { using atomic_type = std::atomic<context*>; using storage_type = atomic_type;
    std::size_t capacity_; storage_type* storage_;
  public: ctor(size_t capacity); ~dtor();
    size_t capacity() const noexcept;
    void push(size_t bottom, context* ctx) noexcept;
    context* pop(size_t top) noexcept;
    array* resize(size_t bottom, size_t top);
  };
  std::atomic<size_t> top_{0}, bottom_{0};
  std::atomic<array*> array_; std::vector<array*> old_arrays_{}; char padding_{cacheline_length};
public: ctor(size_t capacity=4096); ~dtor(); // no copy
  bool empty() const noexcept;
  void push(context* c); context* pop();
  context* steal();
};
```

------
### Algorithms

```c++
class algo::algorithm {
  std::atomic<size_t> use_count_{0};
public: using ptr_t = intrusive_ptr<self>;
  virtual ~dtor()=default;
  virtual void awakened(context*) noexcept=0;
  virtual context* pick_next() noexcept=0;
  virtual bool has_ready_fibers() const noexcept=0;
  virtual void suspend_until(steady_clock::time_point const&) noexcept=0;
  virtual void notify() noexcept=0;
  friend void intrusive_ptr_add_ref(self*) noexcept;
  friend void intrusive_ptr_release(self*) noexcept;
};
class algo::algorithm_with_properties_base : public algorithm {
public:
  virtual void property_change_(context* ctx, fiber_properties* props) noexcept=0;
protected:
  static fiber_properties* get_properties(context* ctx) noexcept;
  static void set_properties(context* ctx, fiber_properties* p) noexcept;
};
struct algo::algorithm_with_properties<PROPS> : algorithm_with_properties_base {
  void awakened(context* ctx) noexcept final;
  virtual void awakened(context*, PROPS&) noexcept=0;
  PROPS& properties(context* ctx) noexcept { return (PROPS&)(*get_properties(ctx)); }
  virtual void property_change(context*, PROPS&) noexcept{}
  void property_change_(context* ctx, fiber_properties* props) noexcept final { property_change(ctx, *(PROPS*)(props)); }
  virtual fiber_properties* new_properties(context* ctx) { return new PROPS{ctx}; }
};

class algo::round_robin : public algorithm {
  scheduler::ready_queue_type rqueue_{};
  std::mutex mtx_{}; std::condition_variable cnd_{}; bool flag_{false};
public: ctor()=default; // no copy, implement: awkened, pick_next, has_ready_fibers, suspend_until, notify
};

class algo::shared_robin : public algorithm {
  std::deque<context*> rqueue_{}; std::mutex rqueue_mtx_{};
  scheduler::ready_queue_type lqueue_{};
  std::mutex mtx_{}; std::condition_variable cnd_{}; bool flag_{false}; bool suspend_{false}
public: ctor()=default; ctor(bool suspend); // no copy/move, implement: awkened, pick_next, has_ready_fibers, suspend_until, notify
};

class algo::work_stealing : public algorithm {
  std::atomic<uint32_t> counter_; std::vector<intrusive_ptr<self>> schedulers_;
  std::uint32_t id_, thread_count_; context_spinlock_queue rqueue_{};
  std::mutex mtx_{}; std::condition_variable cnd_{}; bool flag_{false}; bool suspend_;
  static void init_(uint32_t, std::vector<intrusive_ptr<self>>&);
public: ctor()=default; // no copy, implement: awkened, pick_next, has_ready_fibers, suspend_until, notify
};

class algo::shared_robin : public algorithm {
  std::deque<context*> rqueue_{}; std::mutex rqueue_mtx_{};
  scheduler::ready_queue_type lqueue_{};
  std::mutex mtx_{}; std::condition_variable cnd_{}; bool flag_{false}; bool suspend_{false}
public: ctor()=default; ctor(bool suspend); // no copy/move, implement: awkened, pick_next, steal, has_ready_fibers, suspend_until, notify
};
```

------
### NUMA

```c++
namespace boost::fibers::numa;
void pin_thread(uint32_t, std::thread::native_handle_type);
void pin_thread(uint32_t cpuid);

struct node { uint32_t id; std::set<uint32_t> logical_cpus; std::vector<uint32_t> distance; };
bool operator<(node const& lhs, node const& rhs) noexcept;
std::vector<node> topology();

class algo::work_stealing : public algorithm {
  static std::vector<intrusive_ptr<work_stealing>> schedulers_;
  uint32_t cpu_id_; std::vector<uint32_t> local_cpus_, remote_cpus_;
  context_spinlock_queue rqueue_{};
  std::mutex mtx_{}; std::condition_variable cnd_{};
  bool flag_{false}, suspend_;
  static void init_(std::vector<node> const&, std::vector<intrusive_ptr<work_stealing>>&);
public: ctor(uint32_t, uint32_t, std::vector<node> const&, bool=false); // no copy/move
  virtual void awakened(context*) noexcept;
  virtual context* pick_next() noexcept;
  virtual context* steal() noexcept;
  virtual bool has_ready_fibers() const noexcept;
  virtual bool suspend_until(steady_clock::time_point const&) noexcept;
  virtual void notify() noexcept;
};
```

------
### GPU

##### CUDA

```c++
namespace boost::fibers::cuda;
static void detail::trampoline<Rendezvous>(cudaStream_t st, cudaError_t status, void* vp);
class detail::single_stream_rendezvous {
  mutex mtx_{}; condition_variable cv_{};
  cudaStream_t st_{}; cudaError_t status_{cudaErrorUnknown}; bool done_{false};
public: ctor(cudaStream_t st);
  void notify(cudaStream_t st, cudaError_t status) noexcept;
  std::tuple<cudaStream_t, cudaError_t> wait();
};
class detail::many_streams_rendezvous {
  mutex mtx_{}; condition_variable cv_{};
  std::set<cudaStream_t> stx_; std::vector<std::tuple<cudaStream_t,cudaError_t>> results_;
public: ctor(std::initializer_list<cudaStream_t> l);
  void notify(cudaStream_t st, cudaError_t status) noexcept;
  std::vector<std::tuple<cudaStream_t, cudaError_t>> wait();
};
void waitfor_all();
std::tuple<cudaStream_t, cudaError_t> waitfor_all(cudaStream_t st);
std::vector<std::tuple<cudaStream_t, cudaError_t>> waitfor_all<...STP>(cudaStream_t st, STP...stx);
```

##### ROCm/HIP

```c++
namespace boost::fibers::cuda;
static void detail::trampoline<Rendezvous>(hipStream_t st, hipError_t status, void* vp);
class detail::single_stream_rendezvous {
  mutex mtx_{}; condition_variable cv_{};
  hipStream_t st_{}; hipError_t status_{hipErrorUnknown}; bool done_{false};
public: ctor(hipStream_t st);
  void notify(hipStream_t st, hipError_t status) noexcept;
  std::tuple<hipStream_t, hipError_t> wait();
};
class detail::many_streams_rendezvous {
  mutex mtx_{}; condition_variable cv_{};
  std::set<hipStream_t> stx_; std::vector<std::tuple<hipStream_t,hipError_t>> results_;
public: ctor(std::initializer_list<hipStream_t> l);
  void notify(hipStream_t st, hipError_t status) noexcept;
  std::vector<std::tuple<hipStream_t, hipError_t>> wait();
};
void waitfor_all();
std::tuple<hipStream_t, hipError_t> waitfor_all(hipStream_t st);
std::vector<std::tuple<hipStream_t, hipError_t>> waitfor_all<...STP>(hipStream_t st, STP...stx);
```

------
### Implementation Details

##### Spinlock

```c++
enum class detail::spinlock_status { locked, unlocked };
#define cpu_relax() ... // platform-specific, `YieldProcessor();` on Win, `asm volatile (...)`

class detail::spinlock_ttas {
  std::atomic<spinlock_status> state_{unlocked};
public: ctor()=default; ctor(self const&)=delete; self& operator=(self const&)=delete;
  void lock() noexcept; bool try_lock() noexcept; void unlock() noexcept;
};
class detail::spinlock_ttas_adaptive {
  std::atomic<spinlock_status> state_{unlocked}; std::atomic<size_t> retries_{0};
public: ctor()=default; ctor(self const&)=delete; self& operator=(self const&)=delete;
  void lock() noexcept; bool try_lock() noexcept; void unlock() noexcept;
};

int detail::sys_futex(void* addr, int32_t op, int32_t x) { // OS_LINUX || OS_BSD_OPEN
  if constexpr (OS_BSD_OPEN) return futex((volatile uint32_t*)addr, (int)op, x, nullptr, nullptr);
  else return syscall(SYS_futex, addr, op, x, nullptr, nullptr, 0);
}
int detail::futex_wake(std::atomic<int32_t>* addr) {
  if constexpr (OS_WINDOWS) { WakeByAddressSingle((void*)addr); return 0; }
  else if constexpr (OS_LINUX||OS_BSD_OPEN) return 0 <= sys_futex((void*)addr, FUTEX_WAIT_PRIVATE, 1) ? 0 : -1;
}
int detail::futex_wait(std::atomic<int32_t>* addr, int32_t x) {
  if constexpr (OS_WINDOWS) { WaitOnAddress((volatile void*)addr, &x, sizeof(x), INFINITE); return 0; }
  else if constexpr (OS_LINUX||OS_BSD_OPEN) return 0 <= sys_futex((void*)addr, FUTEX_WAIT_PRIVATE, x) ? 0 : -1;
}

class detail::spinlock_ttas_futex {
  std::atomic<int32_t> value_{0};
public: ctor()=default; ctor(self const&)=delete; self& operator=(self const&)=delete;
  void lock() noexcept; bool try_lock() noexcept; void unlock() noexcept;
};
class detail::spinlock_ttas_adaptive_futex {
  std::atomic<int32_t> value_{0}; std::atomic<int32_t> retries_{0};
public: ctor()=default; ctor(self const&)=delete; self& operator=(self const&)=delete;
  void lock() noexcept; bool try_lock() noexcept; void unlock() noexcept;
};

class detail::spinlock_rtm<FBSplk> {
  FBSplk splk_{};
public: ctor()=default; ctor(self const&)=delete; self& operator=(self const&)=delete;
  void lock() noexcept; bool try_lock() noexcept; void unlock() noexcept;
};

using detail::spinlock = ...;
using detail::spinlock_lock = std::unique_lock<spinlock>;
```

##### Thread barrier

```c++
class detail::thread_barrier {
  size_t initial_, current_; bool cycle_{true};
  std::mutex mtx_{}; std::condition_variable cond_{};
public: explicit ctor(size_t initial); // no copy
  bool wait();
};
```

##### RTM

```c++
struct detail::rtm_status {
  enum {none=0, explicit_abort=1, may_retry=2, memory_conflict=4, buffer_overflow=8, debug_hit=16, nested_abort=32};
  static constexpr uint32_t success = ~uint32_t{0};
};
static uint32_t detail::rtm_begin() noexcept;
static void detail::rtm_end() noexcept;
static void detail::rtm_abort_lock_not_free() noexcept;
static bool detail::rtm_test() noexcept;
```

##### Misc

```c++
struct detail::data_t {
  spinlock_lock * lk{nullptr};
  context * ctx{nullptr}, *from;
  explicit ctor(context* from_) noexcept;
  explicit ctor(spinlock_lock* lk_, context* from_) noexcept;
  explicit ctor(context* ctx_, context* from_) noexcept;
};

struct detail::is_all_same<X,...Y>; // reduce by std::is_same

std::decay_t<T> detail::decay_copy<T>(T&& t) { return std::forward<T>(t); }

std::chrono::steady_clock::time_point convert(std::chrono::steady_clock::time_point const& timeout_time) noexcept;
std::chrono::steady_clock::time_point convert<Clock,Duration>(std::chrono::time_point<Clock,Duration> const& timeout_time) noexcept;
```

------
### Configuration

* `CONTENTION_WINDOW_THRESHOLD`: default 16
* `RETRY_THRESHOLD`: default 64
* `SPIN_BEFORE_SLEEP0`: default 32
* `SPIN_BEFORE_YIELD`: default 64
* `NO_ATOMICS`, `HAS_FUTEX`, `USE_TSX` for spinlock.

------
### Dependency

#### Boost.Algorithm

* `<boost/algorithm/string.hpp>` - on Linux for `topology`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/auto_link.hpp>`

#### Boost.Context

* `<boost/context/**.hpp>`

#### Boost.Core

* `<boost/core/ignore_unused.hpp>`
* `<boost/core/pointer_traits.hpp>`

#### Boost.FileSystem

* `<boost/filesystem.hpp>` - on Linux for `topology`
* `<boost/filesystem/fstream.hpp>` - on Linux for `topology`

#### Boost.Format

* `<boost/format.hpp>` - on Linux for `topology`

#### Boost.Intrusive

* `<boost/intrusive_ptr.hpp>`
* `<boost/intrusive/**.hpp>`

#### Boost.PreDef

* `<boost/predef.h>`

#### Boost.SmartPtr

* `<boost/make_shared.hpp>` - by import

------
### Standard Facilities
