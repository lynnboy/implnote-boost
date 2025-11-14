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

##### Misc

```c++
struct detail::data_t {
  spinlock_lock * lk{nullptr};
  context * ctx{nullptr}, *from;
  explicit ctor(context* from_) noexcept;
  explicit ctor(spinlock_lock* lk_, context* from_) noexcept;
  explicit ctor(context* ctx_, context* from_) noexcept;
};

std::decay_t<T> detail::decay_copy<T>(T&& t) { return std::forward<T>(t); }

class detail::fss_cleanup_function {
  std::atomic<size_t> use_count_{0};
public: using ptr_t = intrusive_ptr<self>;
  ctor()=default; virtual ~dtor()=default;
  virtual void operator()(void* data) =0;
  friend void intrusive_ptr_add_ref(self* p) noexcept;
  friend void intrusive_ptr_release(self* p) noexcept;
};
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
