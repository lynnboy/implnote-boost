# Boost.Cobalt

* lib: `boost/libs/cobalt`
* repo: `boostorg/cobalt`
* commit: `694d9c4`, 2025-09-11

------
#### Concepts

```c++
concept awaitable_type<Awaitable,Promise=void> = requires (Awaitable aw, std::coroutine_handle<Promise> h)
{ {aw.await_ready()}->std::convertible_to<bool>; {aw.await_suspend(h)}; {aw.await_resume()}; };
concept awaitable<Awaitable,Promise=void> = awaitable_type<Awaitable,Promise>
  || requires (Awaitable&& aw) { {std::forward<Awaitable>(aw).operator co_await()}-> awaitable_type<Promise>;}
  || requires (Awaitable&& aw) { {operator co_await(std::forward<Awaitable>(aw))}-> awaitable_type<Promise>;};
struct enable_awaitables<Promise=void> { Aw&& await_transform<awaitable<Promise> Aw>(Aw&& aw, const source_location& loc=CURRENT_LOCATION); };

concept with_get_executor<T> = requires (T& t) { {t.get_executor()}->asio::execution::executor; };

concept interruptible<Awaitable> = (std::is_rvalue_reference_v<Awaitable> && requires (Awaitable&& t) {std::move(t).interrupt_await();})
  || (!std::is_rvalue_reference_v<Awaitable> && requires (Awaitable t) {t.interrupt_await();});
struct detail::forward_cancellation {
  asio::cancellation_signal& cancel_signal;
  void operator()(asio::cancellation_type ct) const { cancel_signal.emit(ct); }
};
```

#### Errors & Exceptions

```c++
enum class error { moved_from, detached, completed_unexpected, wait_not_ready, already_awaited, allocation_failed };
struct cobalt_category_t final: system::error_category {
  ctor() : base{0x7d4c7b49d8a4fdULL}{}
  std::string message(int ev) const override;
  char const* message(int ev, char*, size_t) const noexcept override; // name of error{ev}
  const char* name() const noexcept override;
};
system::error_category& cobalt_category();
system::error_code make_error_code(error e);
struct system::is_error_code_enum<error> { static constexpr bool value = true; };

std::exception_ptr detail::moved_from_exception();
std::exception_ptr detail::detached_exception();
std::exception_ptr detail::completed_unexpected();
std::exception_ptr detail::wait_not_ready();
std::exception_ptr detail::already_awaited();
std::exception_ptr detail::allocation_failed();
void detail::throw_bad_executor(const source_location& loc=CURRENT_LOCATION);
std::exception_ptr detail::wait_not_ready<_>() { return wait_not_ready(); }
```

#### Common Parts

```c++
// variant utils
std::size_t detail::variadic_first<T>(size_t=0u);
std::size_t detail::variadic_first<T,First,...Args>(size_t pos=0u);
constexpr bool detail::variadic_has<T,...Args> = variadic_first<T,Args...>() < sizeof(Args);
constexpr decltype(auto) detail::get_variadic<idx,First,...Args>(First&& first, Args&&...args) requires idx<=sizeof(Args);
struct detail::variadic_element<idx,...Args> { using type = ... };
using detail::variadic_element_t<idx,...Args> = variadic_element<idx,Args...>::type;
struct detail::variadic_last<...Args> { using type=...; };
using detail::variadic_last_t<...Args> = variadic_last<Args...>::type;
constexpr decltype(auto) detail::get_last_variadic<First>(First&& first);
constexpr decltype(auto) detail::get_last_variadic<First,...Args>(First&&, Args&&...args);
auto detail::get_resume_result<Awaitable>(Awaitable& aw) -> system::result<decltype(aw.await_resume()), std::exception_ptr>;
void detail::self_destroy(std::coroutine_handle<void> h, const executor& exec) noexcept;
void detail::self_destroy<T>(std::coroutine_handle<T> h) noexcept;
using detail::void_as_monostate<T> = std::conditional_t<std::is_void_v<T>, monostate, T>;
using detail::monostate_as_void<T> = std::conditional_t<std::is_same_v<T, monostate>, void, T>;

class detail::sbo_resource final : pmr::memory_resource {
  struct block_{ void* p{nullptr}; size_t avail{0u}, size{0u}; bool fragmented{false}; };
  block_ buffer; pmr::memory_resource* upstream_;
  constexpr size_t align_as_max_(size_t size);
  constexpr void align_as_max_();
public: constexpr ctor(<void* buffer, size_t size>, pmr::memory_resource* upstream=pmr::get_default_resource());
  constexpr void* do_allocate(size_t size, size_t align) override;
  constexpr void do_deallocate(void* p, size_t size, size_t align) override;
  constexpr bool do_is_equal(memory_resource const& other) const noexcept override;
};
sbo_resource* detail::get_null_sbo_resource();
struct detail::sbo_allocator<T> {
  using value_type = T; using size_type = size_t; using difference_type = ptrdiff_t;
  using propagate_on_container_move_assignment = std::true_type;
  sbo_resource* resource_{nullptr};
  ctor<U>(self<U> alloc); ctor(sbo_resource* resource);
  [[nodiscard]] constexpr T* allocate(size_t n);
  constexpr void deallocate(T* p, size_t n);
  sbo_resource* resource() const;
};

struct detail::monotonic_resource {
private:
  struct block_{ void*p; size_t avail, size; std::align_val_t aligned; block_* next; };
  block_ buffer_; block_* head_=&buffer_; size_t chunk_size_{buffer_.size};
public: constexpr ctor(void* buffer, size_t size); constexpr ctor(size_t chunk_size=1024);
  ctor(self&& lhs) noexcept=delete; constexpr ~dtor()
  constexpr void release();
  constexpr void* allocate(size_t size, std::align_val_t align_=alignof(std::max_align_t));
};
struct monotonic_allocator<T> {
  ctor<U>(self<U> alloc): resource_{alloc.resource_}{}
  using value_type=T; using size_type = size_t; using difference_type = ptrdiff_t;
  using propagate_on_container_move_assignment=std::true_type;
  [[nodiscard]] constexpr T* allocate(size_t n);
  constexpr void deallocate(T* p, size_t n);
  ctor(monotonic_resource* resource=nullptr);
private: monotonic_resource* resource_{nullptr};
};
```

#### Result

```c++
concept detail::result_error<T> = requires (const T& t, const source_location& loc) { <system::>throw_exception_from_error(t,loc); }; // qualified or ADL
constexpr auto interpret_as_result(std::tuple<> &&) { return system::result<void>{}; }
auto interpret_as_result<Arg(std::tuple<Arg> && args); // system::result<void,Arg> or result<Arg>
auto interpret_as_result<First,...Args>(std::tuple<First,Args...>&& args) -> system::result<std::tuple<First,Args...>> requires(...);
auto interpret_as_result<result_error Error,...Args>(std::tuple<Error,Args...>&& args) -> system::result<std::tuple<Args...>,Error>;
auto interpret_as_result<result_error Error,Arg>(std::tuple<Error,Arg>&& args) -> system::result<Arg,Error>;

struct as_result_t<awaitable_type Aw> {
  ctor(Aw&& aw); ctor<Aw_>(Aw_&& aw) requires(...);
  bool await_ready() { return aw_.await_ready(); }
  auto await_suspend<T>(std::coroutine_handle<T> h) { return aw_.await_suspend(h); }
  auto await_resume();
private: Aw aw_;
};
as_result_t<awaitable_type Aw>(Aw&&) -> as_result_t<Aw>;
as_result_t<Aw> requires(...) -> as_result<decltype(...)>;
auto as_result<awaitable_type Aw>(Aw&& aw) -> as_result_t<Aw> { return {std::forward<Aw>(aw)}; }
auto as_result<Aw>(Aw&& aw) requires(...) {
  struct lazy_tuple{ Aw aw; auto operator co_await() { return {std::forward<Aw>(aw)}; } };
  return lazy_tuple{std::forward<Aw>(aw)};
}

struct as_tuple_t<awaitable Aw> {
  ctor(Aw&& aw); ctor<Aw_>(Aw_&& aw) requires(...);
  bool await_ready() { return aw_.await_ready(); }
  auto await_suspend<T>(std::coroutine_handle<T> h) { return aw_.await_suspend(h); }
  auto await_resume();
private: Aw aw_;
};
as_tuple_t<awaitable_type Aw>(Aw&&) -> as_tuple_t<Aw>;
as_tuple_t<Aw> requires(...) -> as_tuple_t<decltype(...)>;
auto as_tuple<awaitable_type Aw>(Aw&& aw) -> as_tuple_t<Aw> { return {std::forward<Aw>(aw)}; }
auto as_tuple<Aw>(Aw&& aw) requires(...) {
  struct lazy_tuple{ Aw aw; auto operator co_await() { return {std::forward<Aw>(aw)}; } };
  return lazy_tuple{std::forward<Aw>(aw)};
}
```

#### Main Entrance, Thread Entrance

```c++
struct detail::signal_helper { asio::cancellation_signal signal; };
struct detail::main_promise : signal_helper, promise_cancellation_base<cancellation_slot,enable_total_cancellation>,
  promise_throw_if_cancelled_base, enable_awaitables<main_promise>, enable_await_allocator<main_promise>,
  enable_await_executor<main_promise>, enable_await_deferred {
    ctor(int, char**);
    static pmr::memory_resource* my_resource = pmr::get_default_resource();
    void* operator new(size_t size); void operator delete(void* raw, size_t size);
    std::suspend_always initial_suspend() noexcept { return {}; }
    auto final_suspend() noexcept -> std::suspend_never { error_code ec; if (signal_set) signal_set->cancel(ec); return {}; }
    void unhandled_exception() { throw; }
    void return_value(int res=0) { if (result) *result=res; }
    static int run_main(main mn) {
      asio::io_context ctx{CONCURRENCY_HINT_1};
      this_thread::set_executor(ctx.get_executor());
      int res=-1;
      mn.promise->result=&res; mn.promise->exec.emplace(ctx.get_executor()); mn.promise->exec_ = mn.promise->exec->get_executor();
      auto p = std::coroutine_handle::from_promise(*mn.promise); asio::basic_signal_set<executor_type> ss{ctx,SIGNINT,SIGTERM};
      mn.promise->signal_set = &ss;
      struct work { void operator()(error_code ec, int sig) const { if (sig==SIGINT) signal.emit(total); if (sig==SIGTERM) signal.emit(terminal); if (!ec) ss.async_wait(*this); } };
      ss.async_wait(work{ss, signal=mn.promise->signal});
      asio::post(ctx.get_executor(), [p]{ p.resume(); });
      ctx.run(); return res;
    }
    friend int main(int argc, char* argv[]){ // standard entrance
      pmr::unsynchronized_pool_resource root_resource;
      struct reset_res { void operator()(pmr::memory_resource* res) { this_thread::set_default_resource(res); } };
      std::unique_ptr<pmr::memory_resource,reset_res> pr{this_thread::set_default_result(&root_resource)};
      char buffer[8096]; pmr::monotonic_buffer_resource main_res{buffer, 8096, &root_resource};
      my_resource = &main_res;
      return run_main(co_main(argc, argv));
    }
    using executor_type = executor;
    const executor_type& get_executor() const { return *exec_; }
    using allocator_type = pmr::polymorphic_allocator<void>; using resource_type = pmr::unsynchronized_pool_resource;
    mutable resource_type resource{my_resource};
    allocator_type get_allocator() const { return allocator_type{&resource}; }
    using base::await_transform;
private:
    int* result;
    std::optional<asio::executor_work_guard<executor_type>> exec; std::optional<executor_type> exec_;
    asio::basic_signal_set<executor_type>* signal_set;
    main get_return_object() { return {this}; }
};
struct std::coroutine_traits<main,int,Ch> { using promise_type = main_promise; };
class main{ main_promise* promise; };
auto co_main(int argc, char* argv[]) -> main; // entrance

struct detail::signal_helper_2{asio::cancellation_signal signal;};
struct detail::thread_state { asio::io_context ctx{1u}; asio::cancellation_signal signal;
  std::mutex mtx; std::optional<completion_handler<std::exception_ptr>> waitor; std::atomic<bool> done=false; };
struct detail::thread_promise : signal_helper_2, promise_cancellation_base<cancellation_slot,enable_total_cancellation>,
  promise_throw_if_cancelled_base, enable_awaitables<thread_promise>, enable_await_allocator<thread_promise>,
  enable_await_executor<thread_promise>, enable_await_deferred {
    ctor();
    struct initial_awaitable {
      bool await_ready() const {return false;}
      void await_suspend(std::coroutine_handle<thread_promise> h) {h.promise().mtx.unlock();}
      void await_resume(){}
    };
    initial_awaitable initial_suspend() noexcept { return {}; }
    std::suspend_never final_suspend() noexcept { wexec_.reset(); return {}; }
    void unhandled_exception() { throw; }
    void return_void() {}
    using executor_type = executor;
    const executor_type& get_executor() const { return *exec_; }
    using allocator_type = pmr::polymorphic_allocator<void>; using resource_type = pmr::unsynchronized_pool_resource;
    resource_type* resource;
    allocator_type get_allocator() const { return allocator_type{resource}; }
    using base::await_transform;
    thread get_return_object();
    void set_executor(asio::io_context::executor_type exec) { wexec_.emplace(exec); exec_.emplace(exec); }
    std::mutex mtx;
private:
    std::optional<asio::executor_work_guard<asio::io_context::executor>> wexec; std::optional<executor> exec_;
};
struct detail::thread_awaitable {
  asio::cancellation_slot cl;
  std::optional<std::tuple<std::exception_ptr>> res;
  bool await_ready(const source_location& loc=CURRENT_LOCATION) const;
  bool await_suspend<Promise>(std::coroutine_handle<Promise> h);
  void await_resume();
  system::result<void,std::exception_ptr> await_resume(const as_result_tag&);
  std::tuple<std::exception_ptr> await_resume(const as_tuple_tag&);
  explicit ctor(<std::thread thread>, std::shared_ptr<thread_state> state);
private: std::optional<std::thread> thread_; std::shared_ptr<thread_state> state_;
};

struct thread {
  void cancel(asio::cancellation_type type=all);
  thread_awaitable operator co_wait() {&|&&};
  ~dtor(); ctor(self&&) noexcept=default;
  using executor_type = executor;
  using id = std::thread::id;
  id get_id() const noexcept;
  void join(); bool joinable() const; void detach();
  executor_type get_executor(const source_location& loc=CURRENT_LOCATION) const;
  using promise_type = thread_promise;
private: ctor(std::thread thr, std::shared_ptr<thread_state> state);
  std::thread thread_; std::shared_ptr<thread_state> state_;
};
```

#### Promise

```c++
struct detail::partial_promise_base<Alloc> {
  void* operator new<[Executor],Token>(size_t size, <Executor&>, Token& token) { allocate_coroutine(size, asio::get_associated_allocator(token)); }
  void operator delete(void* raw, size_t size) { deallocate_coroutine<Alloc>(raw, size); }
};
struct detail::partial_promise_base<void>{};
struct detail::partial_promise_base<std::allocator<void>>{};
struct detail::partial_promise_base<std::allocator<T>>{};
struct detail::partial_promise<Alloc=void> : partial_promise_base<Alloc> {
  auto initial_suspend() noexcept { return std::suspend_always{}; }
  auto final_suspend() noexcept { return std::suspend_never{}; }
  void return_void() {}
};
struct detail::post_coroutine_promise<Alloc=void> : partial_promise_base<Alloc> {
  auto yield_value<Token>(Token cpl) {
    struct awaitable_t { Token cpl;
      constexpr bool await_ready() noexcept { return false; }
      auto await_suspend(std::coroutine_handle<void> h) noexcept { auto c = std::move(cpl);
        if (this_thread::has_executor()) self_destroy(h, asio::get_associated_executor(c, this_thread::get_executor()));
        else self_destroy(h, asio::get_associated_executor(c));
        asio::post(std::move(c));
      }
    };
    return awaitable_t{std::move(cpl)};
  }
  std::coroutine_handle<self> get_return_object() { return std::coroutine_handle<self>::from_promise(*this); }
  void unhandled_exception() { self_destroy(get_return_object()); throw; }
};
struct std::coroutine_traits<coroutine_handle<post_coroutine_promise<T>>,Args...> { using promise_type = post_coroutine_promise<T>; };
auto detail::post_coroutine<Token>(Token token) -> std::coroutine_handle<post_coroutine_promise<asio::associated_allocator_t<Token>>> { co_yield std::move(token); }
auto detail::post_coroutine<asio::execution::executor Executor, Token>(Executor exec, Token token)
  -> std::coroutine_handle<post_coroutine_promise<asio::associated_allocator_t<Token>>> { co_yield asio::bind_executor(exec, std::move(token)); }
auto detail::post_coroutine<with_get_executor Context, Token>(Context& ctx, Token token)
  -> std::coroutine_handle<post_coroutine_promise<asio::associated_allocator_t<Token>>> { co_yield asio::bind_executor(ctx.get_executor(), std::move(token)); }

struct as_tuple_tag; struct as_result_tag;
struct detail::promise_value_holder<T> {
  std::optional<T> result; bool result_taken = false;
  constexpr ctor()=default; constexpr ctor(noop<T> value) noexcept(...);
  system::result<T,std::exception_ptr> get_result_value() { result_token=true; return {in_place_value, std::move(*result)}; }
  void return_value(T {const&|&&} ret) { result.emplace(<std::move>(ret)); ((promise_receiver<T>*)this)->set_done(); }
};
struct detail::promise_value_holder<void> {
  bool result_taken = false;
  constexpr ctor()=default; constexpr ctor(noop<void> value){}
  system::result<void,std::exception_ptr> get_result_value() { result_token=true; return {in_place_value}; }
  void return_void() { ((promise_receiver<void>*)this)->set_done(); }
};
struct detail::promise_receiver<T> : promise_value_holder<T> {
  struct awaitable { promise_receiver* self; std::exception_ptr ex; asio::cancellation_slot cl;
    ctor(promise_receiver* self); ctor(self&& aw); ~dtor();
    bool await_ready() const { return self->done; }
    bool await_suspend<Promise>(std::coroutine_handle<Promise> h) {
      if (self->done || ex) return false; if (self->awaited_from) { ex=already_awaited(); return false; }
      if constexpr (requires{...}) if ((cl = h.promise().get_cancellation_slot()).is_connected()) cl.emplace<forward_cancellation>(*self->cancel_signal);
      self->awaited_from.reset(h.address()); return true;
    }
    T await_resume(const source_location& loc=CURRENT_LOCATION)
    { if (cl.is_connected()) cl.clear(); if (ex) std::rethrow_exception(ex); return self->get_result().value(loc); }
    system::result<T,std::exception_ptr> await_resume(const as_result_tag&)
    { if (cl.is_connected()) cl.clear(); if (ex) return {in_place_error, std::move(ex)}; return self->get_result(); }
    auto await_resume(const as_tuple_tag&) { if (cl.is_connected()) cl.clear();
      if constexpr (std::is_void_t<T>) { if (ex) return std::move(ex); return self->get_result().error(); }
      else { if (ex) return std::make_tuple(std::move(ex),T{});
        auto res = self->get_result(); if (res.has_error()) return std::make_tuple(res.error(), T{});
        return std::make_tuple(std::exception_ptr{}, std::move(*res)); }
    }
    void interrupt_await()& { if (!self) return; ex=detached_exception(); if (self->awaited_from) self->awaited_from.release().resume(); }
  };
  std::exception_ptr exception; bool done = false; unique_handle<void> awaited_from{nullptr};
  self** reference=nullptr; asio::cancellation_signal* cancel_signal=nullptr;

  ctor()=default; ctor(noop<T> value): base{std::move(value)},done{true}{}
  ctor(self*& reference, asio::cancellation_signal& cancel_signal); // set reference=this;
  ~dtor() {if(!done&&*reference==this) *reference=nullptr;}
  ctor(self&& lhs) noexcept; self& operator=(self&& lhs) noexcept; // set lhs.exception with moved_from, and lhs.done
  system::result<T,std::exception_ptr> get_result() {
    if (exception&&!done) return {in_place_error,std::exchange(exception,nullptr)};
    else if (exception) { this->result_taken = true; return {in_place_error, exception}; }
    else return this->get_result_value(); }
  void unhandled_exception() { exception = std::current_exception(); set_done(); }
  void set_done() { done=true; }
  awaitable get_awaitable() { return {this}; }
  void interrupt_await()& { exception=detached_exception(); awaited_from.release().resume(); }
};

struct detail::cobalt_promise_result<Return> {
  promise_receiver<Return>* receiver{nullptr};
  void return_value(Return {const&|&&} ret) { if(receiver) receiver->return_value(<std::move>(ret)); }
};
struct detail::cobalt_promise_result<void> {
  promise_receiver<void>* receiver{nullptr};
  void return_void() { if(receiver) receiver->return_void(); }
};

struct detail::cobalt_promise<Return> : promise_memory_resource_base,
  promise_cancellation_base<asio::cancellation_slot, asio::enable_total_cancellation>, promise_throw_if_cancelled_base,
  enable_awaitables<self>, enable_await_allocator<self>, enable_await_executor<self>, enable_await_deferred, cobalt_promise_result<Return> {
    mutable asio::cancellation_signal signal;
    using executor_type = executor; executor_type exec;
    ctor<...Args>(Args&...args): promise_cancellation_base(get_memory_resource_from_args(args...)),
      exec{get_executor_from_args(args...)} { reset_cancellation_source(signal.slot()); }
    ~dtor() { if(receiver) {
      if(!receiver->done && !receiver->exception) receiver->exception = completed_unexpected();
      receiver->set_done(); receiver->awaited_from.reset(nullptr);
    } }
    using base::await_transform;
    [[nodiscard]] promise<Return> get_return_object() { return {this}; }
    const executor_type& get_executor() const { return exec; }
    std::suspend_never initial_suspend() noexcept {return{};}
    auto final_suspend() noexcept { return final_awaitable{this}; }
    void unhandled_exception() { if (receiver) receiver->unhandled_exception(); else throw;}
private:  struct final_awaitable { cobalt_promise* promise;
    bool await_ready() const noexcept { return promise->receiver && promise->receiver->awaited_from.get()==nullptr; }
    std::coroutine_handle<void> await_suspend(std::coroutine_handle<cobalt_promise> h) noexcept {
      std::coroutine_handle<void> res = std::noop_coroutine{};
      if (promise->receiver && promise->receiver->awaited_from.get()) res = promise->receiver->awaited_from.release();
      if (auto& rec=h.promise().receiver; rec) { if (!rec->done&&!rec->exception) rec->exception = completed_unexpected();
        rec->set_done(); rec->awaited_from.reset(nullptr); rec=nullptr; }
      self_destroy(h); return res;
    }
    void await_resume() noexcept{}
  };
};

struct [[nodiscard]] promise<Return> {
  using promise_type = cobalt_promise<Return>;
  ctor(self&& lhs) noexcept; self& operator=(self&& lhs) noexcept; // no copy
  constexpr ctor(noop<Return> n); ~dtor() { if(attached_) cancel(); }
  auto operator co_await() { return receiver_.get_awaitable(); }
  void operator+()&& { detach(); }
  void cancel(asio::cancellation_type ct=all) { if(!receiver_.done&&*receiver_.reference ==&receiver_) receiver_.cancel_signal->emit(ct); }
  bool ready() const { return receiver_.done; }
  explicit operator bool() const { return !receiver_.done||!receiver_.return_taken; }
  bool attached() const { return attached_; }
  void detach() { attached_=false; }
  void attach() { attached_=true; }
  Result get(const source_location& loc=CURRENT_LOCATION) { return receiver_.get_result().value(loc); }
private: promise_receiver<Return> receiver_; bool attached_;
  ctor(cobalt_promise<Return>* promise): receiver_{promise->receiver,promise->signal}, attached_{true}{}
};
```

### Task, Async Operation, Spawn, Run

```c++
struct unique_handle<T> {
  ctor() noexcept=default; explicit ctor(T* promise) noexcept; ctor(nullptr_t) noexcept;
  std::coroutine_handle<T> release() { return std::coroutine_handle<T>::from_promise(*handle_.release()); }
  void* address() const noexcept { return get_handle_().address(); }
  static self from_address(void* a) noexcept { self res; res.handle_.reset(&std::coroutine_handle<T>::from_address(a).promise()); return res; }
  bool done() const noexcept { return get_handle_().done(); }
  explicit operator bool() const { return (bool)handle_; }
  void destroy() { handle_.reset(); }
  void resume() const& { get_handle_().resume(); }
  void resume() && { release().resume(); }
  void operator()() {const&|&&} { resume(); }
  T& promise() { return *handle_;}
  constexpr static self from_promise(T& p) noexcept { self res; res.handle_.reset(&p); return res; }
  <const> T& operator*() <const> {return *handle_;}
  <const> T* operator->() <const> {return handle_.get();}
  <const> T* get() <const> {return handle_.get();}
  void reset(T* handle=nullptr) {handle_.reset(handle);}
  friend auto operator{==|!=}(const self& h, nullptr_t) {return h.handle_{==|!=}nullptr;}
private: struct deleter_{ void operator()(T* p) { from_promise(*p).destroy(); } };
  std::unique_ptr<T,deleter_> handle_; source_location loc_;
  std::coroutine_handle<T> get_handle_() const { return from_promise(*handle_); }
};
struct unique_handle<void> {
  ctor() noexcept=default; ctor(nullptr_t) noexcept{} explicit ctor(void* handle) noexcept;
  std::coroutine_handle<void> release() { return ::from_address(handle_.release()); }
  void* address() const noexcept { return get_handle_().address(); }
  static self from_address(void* a) noexcept { self res; res.handle_.reset(&std::coroutine_handle<void>::from_address(a).address()); return res; }
  // same: done, op bool, destroy, resume, op(), get, reset, op==, op!=
private: struct deleter_{ void operator()(void* p){ from_address(p).destroy(); } };
  std::unique_ptr<T,deleter_> handle_; source_location loc_;
  std::coroutine_handle<T> get_handle_() const { return from_address(handle_.get()); }
};
struct unique_handle<std::noop_coroutine_promise> {
  ctor() noexcept=default; ctor(nullptr_t) noexcept{}
  std::coroutine_handle<void> release() { return std::noop_coroutine(); }
  void* address() const noexcept { return std::noop_coroutine().address(); }
  bool done() const noexcept { return true; }
  explicit operator bool() const { return true; }
  void operator()() const {} void resume() const {} void destroy() {}
  struct executor_type { void execute<Fn>(Fn&&) const{} }; executor_type get_executor() const { return {}; }
  friend auto operator==(const self&, nullptr_t) { return false; }
  friend auto operator!=(const self&, nullptr_t) { return true; }
};
struct asio::associator<Associator,unique_handle<Promise>,DefaultCandidate> : Associator<Promise,DefaultCandidate> {
  static base::type get(const unique_handle<Promise>& h) noexcept { return base::get(*h); }
  static auto get(const unique_handle<Promise>& h, const DefaultCandidate& c) noexcept -> decltype(...) { return Base::get(*h, c); }
};

enum class detail::completed_immediately_t { no, maybe, yes, initiating };
struct detail::completion_handler_noop_executor {
  executor exec; completed_immediately_t* completed_immediately=nullptr;
  ctor(executor iner, completed_immediately_t* completed_immediately);
  void execute<Fn>(Fn&& fn) const;
  friend bool operator{==|!=}(const self&, const self&) noexcept; // always equal
};
executor detail::get_executor<Promise>(std::coroutine_handle<Promise> h); // try h.promise().get_executor() or this_thread::get_executor()
executor detail::get_executor(std::coroutine_handle<>) { return this_thread::get_executor(); }
struct detail::completion_handler_base {
  using cancellation_slot_type = asio::cancellation_slot; cancellation_slot_type cancellation_slot;
  cancellation_slot_type get_cancellation_slot() const noexcept { return cancellation_slot; }
  using executor_type = executor; executor_type executor_;
  const executor_type& get_executor() const noexcept { return executor_; }
  using allocator_type = pmr::polymorphic_allocator<void>; allocator_type allocator;
  allocator_type get_allocator() const noexcept { return allocator; }
  using immediate_executor_type = completion_handler_noop_executor; completed_immediately_t* completed_immediately = nullptr;
  immediate_executor_type get_immediate_executor() const noexcept { return {get_executor(), completed_immediately}; }
  ctor<Promise>(std::coroutine_handle<Promise> h, <pmr::memory_resource* resource>, completed_immediately_t* completed_immediately=nullptr);
};
void detail::assign_cancellation<Handler>(std::coroutine_handle<void>, Handler&&){}
void detail::assign_cancellation<Promise,Handler>(std::coroutine_handle<Promise> h, Handler&& func)
{ if constexpr (requires{...}) if ((auto c = h.promise().get_cancellation_slot()).is_connected()) c.assign(std::forward<Handler>(func)); }

struct handler<...Args> { std::optional<std::tuple<Args...>>& result; void operator()(Args...args) {result.emplace(args...);} };
handler<...Args>(std::optional<std::tuple<Args...>>& result)->handler<Args...>;

struct completion_handler<...Args> : completion_handler_base {
  using result_type = std::optional<std::tuple<Args...>>;
  ctor(self&&)=default;
  ctor<Promise>(std::coroutine_handle<Promise> h, result_type& result, <pmr::memory_resource* resource>,
    completed_immediately_t* completed_immediately=nullptr, const source_location& loc=CURRENT_LOCATION);
  void operator()(Args...args) { result.emplace(std::move(args)...); auto p = self.release();
    if (completed_immediately && *completed_immediately==maybe) { *completed_immediately = yes; return;}
    std::move(p)();
  }
  ~dtor() { if (self && completed_immediately&&*completed_immediately==initiating && std::uncaught_exceptions()) self.release(); }
private: unique_handle<void> self; result_type& result; source_location loc_;
};

struct detail::composition_promise<...Args> : promise_cancellation_base<asio::cancellation_slot, asio::enable_total_cancellation>,
  enable_await_allocator<composition_promise<Args...>>, enable_await_executor<composition_promise<Args...>> {
    void get_return_object(){}
    using base::await_transform;
    using handler_type = completion_handler<Args...>;
    using allocator_type = handler_type::allocator_type;
    allocator_type get_allocator() const {return handler.get_allocator();}
    using resource_type = sbo_resource;
    auto await_transform<...Args_,Initiation,...InitArgs>(asio::deferred_async_operation<void(Args_...),Initiation,InitArgs...> op_);
    auto await_transform<Op>(Op&& op_) requires(...);
    using executor_type = handler_type::executor_type;
    const executor& get_executor() const {return handler.get_executor();}
    static void* operator new<...Ts>(size_t size, Ts&...args);
    static void operator delete(void* raw) noexcept;
    completion_handler<Args...> handler;
    ctor<...Ts>(Ts&&...args);
    void unhandled_exception(){throw;}
    constexpr static std::suspend_never initial_suspend() {return{};}
    void return_value(std::tuple<Args...> args) { handler.result.emplace(std::move(args)); }
    struct final_awaitable {
      constexpr bool await_ready() noexcept {return false;}
      completion_handler<Args...> handler;
      std::coroutine_handle<void> await_suspend(std::coroutine_handle<composition_promise> h) noexcept;
      constexpr void await_resume() noexcept{}
    };
    final_awaitable final_suspend() noexcept { return{std::move(handler)}; }
};

struct std::coroutine_traits<void,T...,completion_handler<Args...>>
{ using promise_type = composition_promise<Args...>; }

struct noop<T=void> { T value;
  constexpr static bool await_ready() { return true; }
  constexpr static void await_suspend<P>(std::coroutine_handle<P>){}
  constexpr T await_resume(){return std::move(value);}
};
struct noop<void> {
  constexpr static bool await_ready() { return true; }
  constexpr static void await_suspend<P>(std::coroutine_handle<P>){}
  constexpr static void await_resume(){}
};
noop<T>(T {const&|&&})->noop<T>;
noop()->noop<void>;

struct op<...Args> {
  virtual void ready(handler<Args...>){}
  virtual void initiate(completion_handler<Args...> complete)=0;
  virtual ~dtor()=default;
  struct awaitable_base { op<Args...>& op_; std::optional<std::tuple<Args...>> result;
    using resource_type = pmr::memory_resource;
    ctor(op<Args...>* op_, resource_type* resource); ctor(self&&)noexcept=default;
    bool await_ready() { op_.ready(handler<Args...>{result}); return result.has_value(); }
    completed_immediately_t completed_immediately=no; std::exception_ptr init_ep; resource_type* resource;
    bool await_suspend<Promise>(std::coroutine_handle<Promise> h, const source_location& loc=CURRENT_LOCATION) {
      try { completed_immediately=initiating; op_.initiate(completion_handler<Args...>(h, result, resource, &completed_immediately));
        if (completed_immediately==initiating) completed_immediately=no; return completed_immediately!=yes;
      } catch(...) { init_ep = std::current_exception(); return false; }
    }
    auto await_resume(const source_location& loc=CURRENT_LOCATION) { if (init_ep) std::rethrow_exception(init_ep); return await_resume(as_result_tag{}).value(loc); }
    auto await_resume(const as_tuple_tag&) { if (init_ep) std::rethrow_exception(init_ep); return *std::move(result); }
    auto await_resume(const as_result_tag&) { if (init_ep) std::rethrow_exception(init_ep); return interpret_as_result(*std::move(result)); }
  };
  struct awaitable: awaitable_base { char buffer[SBO_BUFFER_SIZE]; sbo_resource resource{buffer, sizeof(bufffer)};
    ctor(op<Args...>* op_); ctor(self&& rhs) : base{std::move(rhs)}{this->base::resource=&resource;}
    base replace_resource(resource_type* resource)&& { auto nw=std::move(*this); nw.resource=resource; return nw; }
  };
  awaitable operator co_await() { return {this}; }
};

struct use_op_t {
  struct executor_with_default<InnerExecutor> : InnerExecutor { using default_completion_token_type = use_op_t;
    ctor(const InnerExecutor& ex) noexcept; ctor<IE1>(const IE1& ex) requires(); };
  using as_default_on_t<T> = T::rebind_executor<executor_with_default<T::executor_type>>::other;
  static as_default_on_t<std::decay_t<T>> as_default_on(T&& object) { return {std::forward<T>(object)}; }
};
constexpr use_op_t use_op{};

struct enable_await_deferred {
  auto await_transform<...Args,Initiation,...InitArgs>(asio::deferred_async_operation<void(Args...),Initiation,InitArgs...> op_) {
    struct deferred_op : op<Args...> {
      asio::deferred_async_operation<void(Args...),Initiation,InitArgs...> op_;
      void initiate(completion_handler<Args...> complete) override { std::move(op_)(std::move(complete)); }
    };
    return deferred_op{std::move(op_)};
  }
};
struct asio::async_result<use_op_t, void(Args...)> { using return_type = op<Args...>;
  struct op_impl<Initiation,...InitArgs> final : op<Args...> {
    Initiation initiation; std::tuple<InitArgs...> args;
    ctor<I_,...IA>(I initiation, IA&&...args);
    void initiation(completion_handler<Args...> complete) final override
    { std::apply([&](InitArgs&&...args){ std::move(initiation)(std::move(complete), std::move(args)...);}, std::move(args)); }
  };
  static auto initiate<Initiation,...InitArgs>(Initiation&& initiation, use_opt_t, InitArgs&&...args) ->op_impl<std::decay_t<Initiation>, std::decay_t<InitArgs>...>
  { return {std::forward<Initiation>(initiation), std::forward<InitArgs>(args)...}; }
};

struct detail::task_value_holder<T> {
  std::optional<T> result; bool result_taken = false;
  system::result<T,std::exception_ptr> get_result_value() { result_taken = true; return {in_place_value, std::move(*result)}; }
  void return_value(T {const&|&&} ret) { result.emplace(<std::move>(ret)); ((task_receiver<T>*)this)->set_done(); }
  constexpr ctor() noexcept; constexpr ctor(noop<T> n) noexcept(...);
};
struct detail::task_value_holder<void> {
  bool result_taken=false;
  system::result<void,std::exception_ptr> get_result_value() { result_taken = true; return {in_place_value}; }
  void return_void() { ((task_receiver<void>*)this)->set_done(); }
  constexpr ctor() noexcept; constexpr ctor(noop<void>) noexcept;
};
struct task_receiver<T> : task_value_holder<T> {
  std::exception_ptr exception;
  system::result<T,std::exception_ptr> get_result() {
    if (exception && !done) return {in_place_error, std::exchange(exception,nullptr)};
    else if (exception) { this->result_taken=true; return {in_place_error, exception}; }
    return get_result_value(); }
  void unhandled_exception() { exception = std::current_exception(); set_done(); }
  bool done = false; unique_handle<void> awaited_from{nullptr};
  void set_done() { done=true; }
  void cancel(asio::cancellation_type ct) const { if (!done) promise->signal.emit(ct); }
  ctor(noop<T> n); ctor()=default; ctor(self&& lhs);
  ~dtor() { if(!done&&promise&&promise->receiver==this) { promise->receiver=nullptr; if(!promise->started) from_promise(*promise).destroy(); } }
  ctor(task_promise<T>* promise) : promise{promise} { promise->receiver=this; }
  struct awaitable { task_receiver* self, asio::cancellation_slot cl;
    ctor(task_receiver* self); ctor(self&&); ~dtor();
    bool await_ready() const { return self->done; }
    std::coroutine_handle<void> await_suspend<Promise>(std::coroutine_handle<Promise> h) {
      if (self->done) return std::coroutine_handle<void>::from_address(h.address());
      if constexpr (requires{...}) if ((cl=h.promise().get_cancellation_slot()).is_connected()) cl.emplace<forward_cancellation>(self->promise->signal);
      if constexpr (requires{...}) self->promise->exec.emplace(h.promise().get_executor())
      else self->promise->exec.emplace(this_thread::get_executor())
      self->promise->exec_ = self->promise->exec->get_executor(); self->awaited_from.reset(h.address());
      return std::coroutine_handle<task_promise<T>>::from_promise(*self->promise);
    }
    T await_resume(const source_location& loc=CURRENT_LOCATION) { if (cl.is_connected()) cl.clear(); return self->get_result().value(loc); }
    system::result<T,std::exception_ptr> await_resume(const as_result_tag&) { if (cl.is_connected()) cl.clear(); return self->get_result(); }
    auto await_resume(const as_tuple_tag&) {
      if (cl.is_connected()) cl.clear();
      auto res = self->get_result();
      if constexpr (std::is_void_v<T>) return res.error();
      else { if (res.has_error()) return std::make_tuple(res.error(),T{}); else return std::make_tuple(std::exception_ptr{},std::move(*res)); }
    }
    void interrupt_await()& { if (!self) return; self->exception=detached_exception(); if (self->awaited_from) self->awaited_from.release().resume(); }
  };
  task_promise<T>* promise;
  awaitable get_awaitable() { return {this}; }
  void interrupt_await()& { exception = detached_exception(); awaited_from.release().resume(); }
};

struct task_promise_result<Return> { task_receiver<Return>* receiver{nullptr};
  void return_value(Return {const&|&&} ret) { if (receiver) receiver->return_value(<std::move>(ret)); }
};
struct task_promise_result<void> { task_receiver<void>* receiver{nullptr};
  void return_void() { if (receiver) receiver->return_void(); }
};

struct task_promise<Return> : promise_memory_resource_base,
  promise_cancellation_base<asio::cancellation_slot, asio::enable_total_cancellation>, promise_throw_if_cancelled_base,
  enable_awaitables<self>, enable_await_allocator<self>, enable_await_executor<self>, enable_await_deferred, task_promise_result<Return> {
    mutable asio::cancellation_signal signal; bool started=false;
    using executor_type = executor;
    std::optional<asio::executor_work_guard<executor_type>> exec; std::optional<executor_type> exec_;
    ctor<...Args>(Args&...args): promise_cancellation_base(get_memory_resource_from_args(args...)) { reset_cancellation_source(signal.slot()); }
    ~dtor() { if(receiver) {
      if(!receiver->done && !receiver->exception) receiver->exception = completed_unexpected();
      receiver->set_done(); receiver->awaited_from.reset(nullptr);
    } }
    using base::await_transform;
    [[nodiscard]] task<Return> get_return_object() { return {this}; }
    const executor_type& get_executor() const { if (!exec) throw_bad_executor(); return *exec_; }
    struct initial_awaitable { task_promise* promise;
      bool await_ready() const noexcept { return false; }
      void await_suspend(std::coroutine_handle<>){}
      void await_resume() { promise->started=true; }
    };
    auto initial_suspend() noexcept {return initial_awaitable{};}
    struct final_awaitable { task_promise* promise;
      bool await_ready() const noexcept { return promise->receiver && promise->receiver->awaited_from.get()==nullptr; }
      auto await_suspend(std::coroutine_handle<task_promise> h) noexcept {
        std::coroutine_handle<void> res=std::noop_coroutine();
        if (promise->receiver && promise->receiver->awaited_from.get()) res=promise->receiver->awaited_from.release();
        if (auto& rec=h.promise().receiver; rec) {
          if (!rec->done&&!rec.exception) rec->ecxeption=completed_unexpected();
          rec->set_done(); rec->awaited_from.reset(nullptr); rec=nullptr;
        }
        self_destroy(h); return res;
      }
      void await_resume() {}
    };
    auto final_suspend() noexcept { return final_awaitable{this}; }
    void unhandled_exception() { if (receiver) receiver->unhandled_exception(); else throw;}
};

struct [[nodiscard]] task<Return> {// default move, delete copy
  auto operator co_await() { return receiver_.get_awaitable(); }
  using promise_type = task_promise<Return>;
  ctor(noop<Return> n): receiver_{std::move(n)}{}
private: task_receiver<Return> receiver_;
  ctor(task_promise<Return>* task) : receiver_{task}{}
};

struct use_task_t {
  struct executor_with_default<InnerExecutor>: InnerExecutor {
    using default_completion_token_type = use_task_t;
    ctor(const InnerExecutor& ex) noecxept; ctor<IE1>(const IE1& ex) noexcept requires(...);
  };
  using as_default_on_t<T> = T::rebind_executor<executor_with_default<T::executor_type>>::other;
  static as_default_on_t<std::decay_t<T>> as_default_on<T>(T&& object) { return {std::forward<T>(object)}; }
};
constexpr use_task_t use_task{};

struct asio::async_result<use_task_t,void(Args...)> {
  using return_type = task<decltype(interpret_as_result(std::declvalue<std::tuple<Args...>>()))::value_type>;
  static auto initiate<Initiation,...InitArgs>(Initiation initiation, use_task_t, InitArgs...args) -> return_type
  { co_return co_await async_initiate<const use_op_t&, void(Args...)>(std::move(initiation), use_op, std::move(args)...); }
};

struct detail::async_initiate_spawn {
  ctor(executor exec);
  using executor_type = executor;
  const executor_type& get_executor() const { return exec; }
  executor exec;
  void operator()<Handler,T>(Handler&& h, task<T> a);
  void operator()<Handler>(Handler&& h, task<void> a);
};

auto spawn<with_get_executor Context, T, Token>(Context& context, task<T>&& t, Token&& token)
{ return asio::async_initiate<Token,void(std::exception_ptr,T)>(async_initiate_spawn{context.get_executor()}, token, std::move(t)); }
auto spawn<convertible_to<executor> Executor, T, Token>(Executor& executor, task<T>&& t, Token&& token)
{ return asio::async_initiate<Token,void(std::exception_ptr,T)>(async_initiate_spawn{executor}, token, std::move(t)); }
auto spawn<with_get_executor Context, Token>(Context& context, task<void>&& t, Token&& token)
{ return asio::async_initiate<Token,void(std::exception_ptr)>(async_initiate_spawn{context.get_executor()}, token, std::move(t)); }
auto spawn<convertible_to<executor> Executor, Token>(Executor& executor, task<void>&& t, Token&& token)
{ return asio::async_initiate<Token,void(std::exception_ptr)>(async_initiate_spawn{executor}, token, std::move(t)); }

T run<T>(task<T> t) {
  pmr::unsynchronized_pool_resource root_resource{this_thread::get_default_resource()};
  struct reset_res{ void operator()(pmr::memory_resource* res){ this_thread::set_default_resource(res); } };
  std::unique_ptr<pmr::memory_resource, reset_res> pr{this_thread::set_default_resource(&root_resource)};
  std::future<T> f;
  { asio::io_context ctx{CONCURRENCY_HINT_1};
    struct reset_exec { std::optional<executor> exec;
      ctor() { if (this_thread::has_executor()) exec = this_thread::get_executor(); }
      ~dtor() { if (exec) this_thread::set_executor(*exec); }
    } re;
    this_thread::set_executor(ctx.get_executor());
    f = spawn(ctx, std::move(t), asio::bind_executor(ctx.get_executor(), asio::use_future));
    ctx.run(); }
  return f.get();
}
```

#### Generator

```c++
struct detail::generator_receiver_base<Yield,Push> {
  std::optional<Push> pushed_value;
  auto get_awaitable(Push {const&|&&} push);
};
struct detail::generator_receiver_base<Yield,void> { bool pushed_value{false}; auto get_awaitable(); };

struct detail::generator_receiver<Yield,Push> : generator_receiver_base<Yield,Push> {
  std::exception_ptr exception; std::optional<Yield> result, result_buffer;
  Yield get_result();
  bool done=false;
  unique_handle<void> awaited_from{nullptr};
  unique_handle<generator_promise<Yield, Push>> yield_from{nullptr};
  bool lazy=false;
  bool ready() { return exception||result||done; }
  ctor(noop<Yield> n); ctor()=default; ctor(self&& lhs); ~dtor();
  ctor(self*& reference, asio::cancellation_signal& cancel_signal);
  self& operator=(self&& lhs) noexcept;
  self **reference=nullptr; asio::cancellation_signal* cancel_signal=nullptr;
  using yield_awaitable = generator_yield_awaitable<Yield,Push>;
  yield_awaitable get_yield_awaitable(generator_promise<Yield,Push>* pro) {return {pro};}
  static yield_awaitable terminator() {return{nullptr};}
  void yield_value<T>(T&& t);
  struct awaitable {
    generator_receiver* self; std::exception_ptr ex; asio::cancellation_slot cl;
    variant2::variant<monostate, Push*, const Push*> to_push;
    ctor(generator_receiver* self, <const> Push* to_push);
    ctor(const self& aw) noexcept;
    bool await_ready() const { return self->ready(); }
    std::coroutine_handle<void> await_suspend<Promise>(std::coroutine_handle<Promise> h);
    Yield await_resume(const source_location& loc=CURRENT_LOCATION) { return await_resume(as_result_tag{}).value(loc); }
    std::tuple<std::exception_ptr, Yield> await_resume(const as_tuple_tag&);
    system::result<Yield,std::exception_ptr> await_resume(const as_result_tag&);
    void interrupt_await()& { if(!self)return; ex=detached_exception(); if(self->awaited_from) self->awaited_from.release().resume(); }
  };
  void interrupt_await()& { exception = detached_exception(); awaited_from.release().resume(); }
  void rethrow_if() {if(exception) std::rethrow_exception(exception);}
};

struct detail::generator_promise<Yield,Push> : promise_memory_resource_base,
  promise_cancellation_base<asio::cancellation_slot, asio::enable_total_cancellation>, promise_throw_if_cancelled_base,
  enable_awaitables<self>, enable_await_allocator<self>, enable_await_executor<self>, enable_await_deferred {
    using base::await_transform;
    [[nodiscard]] generator<Yield,Push> get_return_object(){return{this};}
    mutable asio::cancellation_signal signal;
    using executor_type = executor; executor_type exec;
    const executor_type& get_executor() const { return exec; }
    ctor<...Args>(Args&...args) : promise_memory_resource_base(get_memory_resource_from_args(args...)), exec(get_executor_from_args(args...)){reset_cancellation_source(signal.slot());}
    std::suspend_never initial_suspend() noexcept { return {}; }
    struct final_awaitable {
      generator_promise* generator;
      bool await_ready() const noexcept { return generator->receiver&& generator->receiver->awaited_from.get()==nullptr; }
      auto await_suspend(std::coroutine_handle<generator_promise> h) noexcept;
      void await_resume() noexcept { if(generator->receiver) generator->receiver->done=true; }
    };
    final_awaitable final_suspend() noexcept { return {this}; }
    void unhandled_exception() { if(receiver)receiver->exception=std::current_exception(); else throw; }
    void return_value(Yield {const&|&&} res) { if (receiver) receiver->yield_value(<std::move>(res)); }
    generator_receiver<Yield,Push>* receiver{nullptr};
    auto await_transform(this_coro::initial_t);
    auto yield_value<Yield_>(Yield_&& ret);
    void interrupt_await()&;
    ~dtor();
};

struct detail::generator_yield_awaitable<Yield,Push> {
  generator_promise<Yield,Push>* self;
  constexpr bool await_ready() const { return self&&self->receiver&&self->receiver->push_value&&!self->receiver->result; }
  std::coroutine_handle<void> await_suspend(std::coroutine_handle<generator_promise<Yield,Push>> h, const source_location& loc=CURRENT_LOCATION);
  Push await_resume() { return *std::exchange(self->receiver->pushed_value, std::nullopt); }
};
struct detail::generator_yield_awaitable<Yield,void> {
  generator_promise<Yield,void>* self;
  constexpr bool await_ready() const { return self&&self->receiver&&self->receiver->push_value; }
  std::coroutine_handle<> await_suspend(std::coroutine_handle<generator_promise<Yield,void>> h, const source_location& loc=CURRENT_LOCATION);
  void await_resume() { self->receiver->pushed_value = false; }
};
struct generator_base<Yield,Push>{ auto operator()(Push {const&|&&} push); };
struct generator_base<Yield,void>{ auto operator co_await(); };
struct generator_with_awaitable<T> {
  generator_base<T,void>& g; std::optional<generator_receiver<T,void>::awaitable> awaitable;
  void await_suspend<Promise>(std::coroutine_handle<Promise> h) { g.cancel(); awaitable.emplace(g.operator co_await()); return awaitable->await_suspend(h); }
  void await_resume(){}
};

struct [[nodiscard]] generator<Yield,Push=void> {
  ctor(self&&) noexcept=default; self& operator=(self&& lhs) noexcept{cancel(); receiver_=std::move(lhs.receiver_); return *this; } // no copy
  constexpr ctor(noop<Yield> n); ~dtor() {cancel();}
  explicit operator bool() const { return !receiver_.done||receiver_.result||receiver_.exception; }
  void cancel(asio::cancellation_type ct=all) { if (!receiver_.done&&*receiver_.reference=&receiver_) receiver_.cancel_signal->emit(ct); }
  bool ready() const { return receiver_.result||receiver_.exception; }
  Yield get() { receiver_.rethrow_if(); return receiver_.get_result(); }
  using promise_type = generator_promise<Yield,Push>;
private: ctor(generator_promise<Yield,Push>* generator);
  generator_receiver<Yield,Push> receiver_;
};
```

#### Detached Async Operation

```c++
struct detail::detached_promise : promise_memory_resource_base,
  promise_cancellation_base<asio::cancellation_slot, asio::enable_total_cancellation>, promise_throw_if_cancelled_base,
  enable_awaitables<self>, enable_await_allocator<self>, enable_await_executor<self>, enable_await_deferred, task_promise_result<Return> {
    [[nodiscard]] detached get_return_object(){return{};}
    std::suspend_never await_transform(this_core::reset_cancellation_source_t<asio::cancellation_slot> reset) noexcept
    { this->reset_cancellation_source(reset.source); return {}; }
    using executor_type = executor; executor_type exec;
    const executor_type& get_executor() const { return exec; }
    ctor<...Args>(Args&...args) : promise_memory_resource_base(get_memory_resource_from_args(args...)), exec(get_executor_from_args(args...)){}
    std::suspend_never initial_suspend() noexcept { return {}; }
    std::suspend_never final_suspend() noexcept { return {}; }
    void return_void(){}
    void unhandled_exception() { throw; }
};
struct detached { using promise_type = detached_promise; };
```

#### This Thread, This Coro

```c++
namespace this_thread {
pmr::memory_resource* get_default_resource() noexcept;
pmr::memory_resource* set_default_resource(pmr::memory_resource* r) noexcept;
pmr::polymorphic_allocator<void> get_allocator();
executor& get_executor(const source_location& loc=CURRENT_LOCATION);
bool has_executor();
void set_executor(executor exec) noexcept;
}
namespace this_coro {
struct allocator_t{}; constexpr allocator_t allocator;
struct cancelled_t{}; constexpr cancelled_t cancelled;
struct initial_t{}; constexpr initial_t initial;
struct reset_cancellation_source_t<Slot=asio::cancellation_slot> { Slot source; };
reset_cancellation_source_t<Slot=asio::cancellation_slot> reset_cancellation_source(Slot slot={}) { return{std::move(slot)}; }
}
struct promise_cancellation_base<Slot=asio::cancellation_slot,DefaultFilter=asio::enable_terminal_cancellation> {
  using cancellation_slot_type = asio::cancellation_slot;
  ctor<InitF=asio::enable_terminal_cancellation>(Slot slot={}, InitF filter={});
  auto await_transform(cancelled_t) noexcept { return cancelled_t_awaitable{state_.cancelled()}; }
  auto await_transform(cancellation_state_t) noexcept { return cancellation_state_t_awaitable{state_}; }
  auto await_transform<...F( resetreset_cancellation_state_n_t<F...> reset) noexcept; // 0~2
  <const> cancellation_state& cancellation_state() <const> { return state_; }
  cancellation_type cancelled() const { return state_.cancelled(); }
  cancellation_slot_type get_cancellation_slot() { return state_.slot(); }
  void reset_cancellation_source(Slot source={});
  <const> Slot& source() <const> { return source_; }
private: Slot source_; cancellation_state state_{source_, DefaultFilter{}};
  struct cancelled_t_awaitable;
  struct cancellation_state_t_awaitable;
  struct cancellation_state_n_t_awaitable; // 0~2
};
struct promise_throw_if_cancelled_base {
  ctor(bool throw_if_cancelled=true);
  auto await_transform(throw_if_cancelled_n_t) noexcept; // 0~1
protected: bool throw_if_cancelled_{true};
  struct throw_if_cancelled_n_awaitable_; // 0~1
};
struct promise_memory_resource_base {
  using allocator_type = pmr::polymorphic_allocator<void>;
  allocator_type get_allocator() const { return {resource}; }
  static void* operator new<...Args>(size_t, Args&...args);
  static void operator delete(void* raw, size_t size) noexcept;
  ctor(pmr::memory_resource* resource=this_thread::get_default_resource());
private: pmr::memory_resource* resource = this_thread::get_default_resource()
};
void* allocate_coroutine<Alloc>(size_t size, Alloc alloc_);
void deallocate_coroutine<Alloc>(void* raw_, size_t size);

struct enable_await_allocator<Promise> {
  auto await_transform(allocator_t);
private: struct allocator_awaitable_;
};
struct enable_await_executor<Promise> {
  auto await_transform(executor_t);
private: struct executor_awaitable_;
};
```

#### Channel

```c++
struct channel_reader<T>;
struct channel<T> { // delete move
  explicit ctor(size_t limit=0u, executor executor=this_thread::get_executor(), pmr::memory_resource* resource=this_thread::get_default_resource());
  ~dtor();
  using executor_type = executor;
  const executor_type& get_executor();
  bool is_open() const;
  void close();
private: circular_buffer<T,pmr::polymorphic_allocator<T>> buffer_;
  executor_type executor_; bool is_closed_{false};
  struct read_op : intrusive::list_base_hook<link_mode<auto_unlink>> {
    channel* chn; source_location loc; bool cancelled=false;
    std::optional<T> direct{}; asio::cancellation_slot cancel_slot{};
    unique_handle<void> awaited_from{nullptr}; void (*begin_transaction)(void*) =nullptr;
    void transactinoal_unlink() { if (begin_transaction) begin_transaction(awaited_from.get()); unlink(); }
    void interrupt_await() { if (!direct) { cancelled=true; if (awaited_from) awaited_from.release().resume(); } }
    struct cancel_impl;
    bool await_ready() const noexcept { return !chn->buffer_.empty() || chn->is_closed_; }
    std::coroutine_handle<void> await_suspend<Promise>(std::coroutine_handle<Promise> h);
    T await_resume();
    std::tuple<system::error_code, T> await_resume(const as_tuple_tag&);
    system::result<T> await_resume(const as_result_tag&);
    explicit operator bool() const { return chn && chn->is_open(); }
  };
  struct write_op : intrusive::list_base_hook<link_mode<auto_unlink>> {
    using ref_t = std::conditional_t<std::is_copy_constructible_v<T>, variant2::variant<T*,constT*>, T*>;
    channel* chn; ref_t ref; source_location loc;
    bool cancelled=false, direct=false, closed=!chn->is_open(); asio::cancellation_slot cancel_slot{};
    unique_handle<void> awaited_from{nullptr}; void (*begin_transaction)(void*) =nullptr;
    void transactinoal_unlink() { if (begin_transaction) begin_transaction(awaited_from.get()); unlink(); }
    void interrupt_await() { if (!direct) { cancelled=true; if (awaited_from) awaited_from.release().resume(); } }
    struct cancel_impl;
    bool await_ready() const noexcept { return !chn->buffer_.full() || chn->is_closed_; }
    std::coroutine_handle<void> await_suspend<Promise>(std::coroutine_handle<Promise> h);
    void await_resume();
    std::tuple<system::error_code> await_resume(const as_tuple_tag&);
    system::result<void> await_resume(const as_result_tag&);
    explicit operator bool() const { return chn && chn->is_open(); }
  };
  intrusive::list<read_op, constant_time_size<false>> read_queue_;
  intrusive::list<write_op, constant_time_size<false>> write_queue_;
public:
  read_op read(const source_location& loc=CURRENT_LOCATION) requires std::is_copy_constructible_v<T> { return {{}, this, &value, loc}; }
  write_op write(<const> T{&|&&} value, const source_location& loc=CURRENT_LOCATION) requires std::is_copy_constructible_v<T> { return {{}, this, &value, loc}; }
};

struct channel<void> { // delete move
  explicit ctor(size_t limit=0u, executor executor=this_thread::get_executor());
  ~dtor();
  using executor_type = executor;
  const executor_type& get_executor() { return executor_; }
  bool is_open() const { return !is_closed_;}
  void close();
private: size_t limit_, n_{0u}; executor_type executor_; bool is_closed_{false};
  struct read_op : intrusive::list_base_hook<link_mode<auto_unlink>> {
    channel* chn; source_location loc; bool cancelled=false, direct=false;
    asio::cancellation_slot cancel_slot{};
    unique_handle<void> awaited_from{nullptr}; void (*begin_transaction)(void*) =nullptr;
    void transactinoal_unlink() { if (begin_transaction) begin_transaction(awaited_from.get()); unlink(); }
    void interrupt_await() { if (!direct) { cancelled=true; if (awaited_from) awaited_from.release().resume(); } }
    struct cancel_impl;
    bool await_ready() const noexcept { return chn->n_>0 || chn->is_closed_; }
    std::coroutine_handle<void> await_suspend<Promise>(std::coroutine_handle<Promise> h);
    void await_resume();
    std::tuple<system::error_code> await_resume(const as_tuple_tag&);
    system::result<void> await_resume(const as_result_tag&);
    explicit operator bool() const { return chn && chn->is_open(); }
  };
  struct write_op : intrusive::list_base_hook<link_mode<auto_unlink>> {
    channel* chn; source_location loc;
    bool cancelled=false, direct=false, closed=!chn->is_open(); asio::cancellation_slot cancel_slot{};
    unique_handle<void> awaited_from{nullptr}; void (*begin_transaction)(void*) =nullptr;
    void transactinoal_unlink() { if (begin_transaction) begin_transaction(awaited_from.get()); unlink(); }
    void interrupt_await() { if (!direct) { cancelled=true; if (awaited_from) awaited_from.release().resume(); } }
    struct cancel_impl;
    bool await_ready() const noexcept { return chn->n_ < chn->limit_ || chn->is_closed_; }
    std::coroutine_handle<void> await_suspend<Promise>(std::coroutine_handle<Promise> h);
    void await_resume();
    std::tuple<system::error_code> await_resume(const as_tuple_tag&);
    system::result<void> await_resume(const as_result_tag&);
    explicit operator bool() const { return chn && chn->is_open(); }
  };
  intrusive::list<read_op, constant_time_size<false>> read_queue_;
  intrusive::list<write_op, constant_time_size<false>> write_queue_;
public:
  read_op read(const source_location& loc=CURRENT_LOCATION) requires std::is_copy_constructible_v<T> { return {{}, this, loc}; }
  write_op write(const source_location& loc=CURRENT_LOCATION) requires std::is_copy_constructible_v<T> { return {{}, this, loc}; }
};

struct channel_reader<T> {
  ctor(channel<T>& chan, const source_location& loc=CURRENT_LOCATION);
  auto operator co_await() { return chan_->read(loc_); }
  explicit operator bool() const { return chan_ && chan_->is_open(); }
private: channel<T>* chan_; source_location loc_;
};
```

#### With

```c++
using detail::co_awaitable_type<T> = ...;
using detail::co_await_result_t<T> = ...;
decltype(auto) detail::get_awaitable_type<T>(T&& t);
struct detail::awaitable_type_getter<T> {
  using type = co_awaitable_type<T&&>; std::decay_t<T>& ref;
  ctor<U>(U&& ref); operator type();
};
struct detail::awaitable_type_getter<awaitable_type T> {
  using type = T&&; std::decay_t<T>& ref;
  ctor<U>(U&& ref); operator type();
};

struct [[nodiscard]] detail::with_impl<T> {
  struct promise_type : with_promise_value<T>, enable_awaitables<promise_type>, enable_await_allocator<promise_type> {
    using base::await_transform;
    using executor_type = executor;
    const executor_type& get_executor() const { return *exec; }
    std::optional<executor_type> exec;
    self get_return_object() { return {*this}; }
    std::exception_ptr e;
    void unhandled_exception() { e = std::current_exception(); }
    std::suspend_always initial_suspend() noexcept { return{}; }
    struct final_awaitable { promise_type* promise;
      bool await_ready() const noexcept { return false; }
      auto await_suspend(std::coroutine_handle<promise_type> h) noexcept -> std::coroutine_handle<void>
      { return from_address(h.promise().awaited_from.address()); }
      void await_resume() noexcept{}
    };
    auto final_suspend() noexcept { return final_awaitable{this}; }
    using cancellation_slot_type = asio::cancellation_slot;
    cancellation_slot_type get_cancellation_slot() const { return slot_; }
    asio::cancellation_slot slot_;
    std::coroutine_handle<void> awaited_from{nullptr};
  };
  bool await_ready() const {return false;}
  auto await_suspend<Promise>(std::coroutine_handle<Promise> h) ->std::coroutine_handle<promise_type>;
  T await_resume();
};
struct detail::with_promise_value {
  std::optional<T> result;
  void return_value(std::optional<T>&& value) { if (value) result.emplace(std::move(*value)); }
  std::optional<T> get_result() { return std::move(result); }
};
struct detail::with_promise_value<void> { void return_void(){} void get_result(){} };
auto detail::invoke_await_exit<T>(T&& t, std::exception_ptr& e) { return std::forward<T>(t).await_exit(e); }

auto with<Arg, Func, Teardown>(Arg arg, Func func, Teardown teardown) -> with_impl<void> requires(...) {
  std::exception_ptr e;
  try { <co_await> std::move(func)(arg); }catch(...) { e = std::current_exception(); }
  try { co_await std::move(teardown)(std::move(arg), e); }catch(...) { if (!e) e = std::current_exception(); }
  if (e) std::rethrow_exception(e);
}
auto with<Arg, Func, Teardown>(Arg arg, Func func, Teardown teardown) -> with_impl<decltype(std::move(func)(arg))> requires(...) {
  std::exception_ptr e; std::optional<decltype(std::move(func)(arg))> res;
  try { res = <co_await> std::move(func)(arg); }catch(...) { e = std::current_exception(); }
  try { co_await std::move(teardown)(std::move(arg), e); }catch(...) { if (!e) e = std::current_exception(); }
  if (e) std::rethrow_exception(e); co_return std::move(res);
}
auto with<Arg,Func>(Arg&& arg, Func&& func) requires(...)
{ return with(std::forward<Arg>(arg), std::forward<Func>(func), &invoke_await_exit<Arg>); }
```

#### Race, Gather, Join, Wait Group

```c++
// fork
struct detail::fork {
  ctor()=default;
  struct shared_state {
    pmr::monotonic_buffer_resource resource{};
    ctor<...Args>(Args&&...args){}
    unique_handle<void> coro{}; size_t use_count=0u;
    friend void intrusive_ptr_add_erf(shared_state* st) { st->use_count++; }
    friend void intrusive_ptr_release(shared_state* st) { if (st->use_count--=1u) st->coro.reset(); }
    bool outstanding_work() {return use_count!=0u;}
    std::optional<executor> exec{};
    bool wired_up() {return exec.has_value();}
    using executor_type = executor;
    const executor_type& get_executor() const { return *exec; }
    source_location loc;
  };
  struct static_shared_state<bufSize> : private std::array<char,bufSize>, shared_state {};
  struct wired_up_t{};
  constexpr static wired_up_t wired_up{};
  struct set_transaction_function {
    void* begin_transaction_this = nullptr; void (*begin_transaction_func)(void*) = nullptr;
    ctor<BeginTx>(BeginTx& transaction);
  };
  struct promise_type {
    void* operator new<State,...Rest>(size_t size, State& st, Rest&&...) { return st.resource.allocate(size); }
    void operator delete(void*)noexcept{};
    ctor<...Rest>(shared_state& st, Rest&...);
    intrusive_ptr<shared_state> sate; asio::cancellation_slot cancel;
    using executor_type = executor;
    const executor_type& get_executor() const { return state->get_executor(); }
    using allocator_type = pmr::polymorphic_allocator<void>;
    const allocator_type get_allocator() const { return &state->resource; }
    using cancellation_slot_type = asio::cancellation_slot;
    cancellation_slot_type get_cancellation_slot() const { return cancel; }
    constexpr static std::suspend_never initial_suspend() noexcept {return{};}
    struct final_awaitable {
      promise_type* self;
      bool await_ready() const noexcept { return self->state->use_count!=1u; }
      std::coroutine_handle<void> await_suspend(std::coroutine_handle<promise_type> h) noexcept;
      constexpr static void await_resume() noexcept {}
    };
    final_awaitable final_suspend() noexcept { if (cancel.is_connected()) cancel.clear(); return {this}; }
    struct wrapped_awaitable<awaitable<promise_type> Aw> { Aw& aw;
      constexpr static bool await_ready() noexcept { return false; }
      auto await_suspend(std::coroutine_handle<promise_type> h) { return aw.await_suspend(h); }
      auto await_resume() { return aw.await_resume(); }
    };
    auto await_transform<awaitable<promise_type> Aw>(Aw& aw) { return wrapped_awaitable<Aw>{aw}; }
    struct wired_up_awaitable { promise_type* promise;
      bool await_ready() const noexcept { return promise->state->wired_up(); }
      void await_suspend(std::coroutine_handle<promise_type>){}
      constexpr static void await_resume() noexcept {}
    };
    auto await_transform(wired_up_t){ return wired_up_awaitable{this}; }
    auto await_transform(set_transaction_function sf) { begin_transaction_func = sf.begin_transaction_func; begin_transaction_this = sf.begin_transaction_this; return std::suspend_never{}; }
    auto await_transform(asio::cancellation_slot slot) { cancel = slot; return std::suspend_never{}; }
    [[noreturn]] void unhandled_exception() noexcept {std::terminate();}
    void* begin_transaction_this = nullptr; void (*begin_transaction_func)(void*) = nullptr;
    void begin_transaction() { if (begin_transaction_this) begin_transaction_func(begin_transaction_this); }
    fork get_return_object() { return this; }
  };
  [[nodiscard]] bool done() const { return !handle_||handle_.done(); }
  auto release() -> std::coroutine_handle<promise_type> { return handle_.release)(; )}
private: unique_handle<promise_type> handle_;
  ctor(promise_type* pt) : handle_{pt}{}
};

// race
struct detail::left_race_tag{};
struct detail::race_traits<Base,Awaitable=Base> {
  constexpr static bool is_lvalue = std::is_lvalue_reference_v<Base>, is_actual = awaitable_type<awaitable>;
  using awaitable = std::conditional_t<is_lvalue, std::decay_t<Awaitable>&, Awaitable&&>;
  using actual_awaitable = std::conditional_t<is_actual, awaitable, decltype(get_awaitable_type(declval<awaitable>()))>;
  using interruptible_type = std::conditional_t<is_lvalue, decay_t<actual_awaitable>&, decay_t<actual_awaitable>&&>;
  constexpr static bool interruptible = interruptible<interruptible_type>;
  static void do_interrupt(std::decay_t<actual_awaitable>& aw) { if constexpr(interruptible) ((interruptible_type)aw).interrupt_await(); }
};
struct detail::interruptible_base { virtual void interrupt_await() = 0; };
struct detail::race_variadic_impl<asio::cancellation_type ct, URBG,...Args> {
  ctor<URBG_>(URBG_&& g, Args&&...args);
  std::tuple<Args...> args; URBG g;
  constexpr static size_t tuple_size = sizeof...(Args);
  struct awaitable : fork::static_shared_state<256 * tuple_size> { source_location loc;
    ctor<...idx>(std::tuple<Args...>& args, URBG& g, std::index_sequence<idx...>);
    std::tuple<Args...>& aws; std::array<asio::cancellation_signal, tuple_size> cancel_;
    constexpr static auto make_null<_>() {return nullptr;}
    std::array<asio::cancellation_signal*, tuple_size> cancel = {make_null<Args>()...};
    std::array<interruptible_base*, tuple_size> working;
    size_t index{std::numeric_limits<size_t>::max()};
    constexpr static bool all_void = {std::is_void_v<co_await_result_t<Args>>&&...};
    std::optional<variant2::variant<void_as_monostate<co_await_result_t<Args>>...>> result;
    std::exception_ptr error;
    bool has_result() const { return index!=std::numeric_limits<size_t>::max(); }
    void cancel_all() { interrupt_await(); /* then emit and reset all cancel tokens*/ };
    void interrupt_await() { for (auto i:working) if(i) i->interrupt_await(); }
    void assign_error<T,Error>(system::result<T,Error>& res)
    try{ std::move(res).value(loc); }catch(...) { error = std::current_exception(); }
    void assign_error<T>(system::result<T,std::exception_ptr>& res) { error = std::move(res).error(); }
    static fork await_impl<idx>(awaitable& this_);
    std::array<fork(*)(awaitable&), tuple_size> impls {[]<...idx>(std::index_sequence<idx...>)
      { return {&await_impl<idx>...}; }(std::make_index_sequence<tuple_size>{})};
    fork last_forked;
    bool await_ready() { last_forked = impls[0](*this); return last_forked.done(); }
    auto await_suspend<H>(std::coroutine_handle<H> h, const source_location& loc=CURRENT_LOCATION);
    auto await_resume()
    { if(error) std::rethrow_exception(error); if constexpr (all_void) return index; else return std::move(*result); }
    auto await_resume(const as_tuple_tag&)
    { if constexpr (all_void) return std::make_tuple(error, index); else return std::make_tuple(error, std::move(*result)); }
    auto await_resume(const as_result_tag&) -> system::result<std::conditional_t<all_void,size_t,variant2::variant<void_as_monostate<co_await_result_t<Args>>...>>, std::exception_ptr>
    { if(error) return{in_place_error,error}; if constexpr (all_void) return {in_place_value,index}; else return {in_place_value,std::move(*result)}; }
  };
  awaitable operator co_await()&& { return {args, g, std::make_index_sequence<tuple_size>{}}; }
};
struct detail::race_ranged_impl<asio::cancellation_type ct, URBG,Range> {
  using result_type = co_await_result_t<std::decay_t<decltype(*std::begin(std::declval<Range>()))>>;
  ctor<URBG_>(URBG_&& g, Range&& rng);
  Range range; URBG g;
  struct awaitable : fork::shared_state { source_location loc;
    using type = std::decay_t<decltype(*std::begin(std::declval<Range>()))>; using traits = race_traits<Range,type>;
    size_t index{std::numeric_limits<size_t>::max()};
    std::conditional_t<std::is_void_v<result_type>, variant2::monostate, std::optional<result_type>> result;
    std::exception_ptr error;
    pmr::polymorphic_allocator<void> alloc{&resource}; Range &aws;
    struct dummy { ctor<...Args>(Args&&...){} };
    std::conditional_t<traits::interruptible, pmr::vector<std::decay_t<traits::actual_awaitable>*>, dummy> working{std::size(aws), alloc};
    pmr::vector<size_t> reorder{std::size(aws), alloc};
    pmr::vector<asio::cancellation_signal> cancel_{std::size(aws), alloc};
    pmr::vector<asio::cancellation_signal*> cancel{std::size(aws), alloc};
    bool has_result() const { return index!=std::numeric_limits<size_t>::max(); }
    ctor(Range& aws, URBG& g);
    void cancel_all() { interrupt_await(); /* then emit and reset all cancel tokens*/ };
    void interrupt_await() { if constexpr (traits::interruptible) for (auto i:working) if(i) traits::do_interrupt(*aw); }
    void assign_error<T,Error>(system::result<T,Error>& res)
    try{ std::move(res).value(loc); }catch(...) { error = std::current_exception(); }
    void assign_error<T>(system::result<T,std::exception_ptr>& res) { error = std::move(res).error(); }
    static fork await_impl(awaitable& this_, size_t idx);
    fork last_forked;
    bool await_ready() { last_forked = await_impl(*this, reorder.front()); return last_forked.done(); }
    auto await_suspend<H>(std::coroutine_handle<H> h, const source_location& loc=CURRENT_LOCATION);
    auto await_resume()
    { if(error) std::rethrow_exception(error); if constexpr (std::is_void_v<result_type>) return index; else return std::make_pair(index, *result); }
    auto await_resume(const as_tuple_tag&)
    { if constexpr (std::is_void_v<result_type>) return std::make_tuple(error, index); else return std::make_tuple(error, std::make_pair(index,std::move(*result))); }
    auto await_resume(const as_result_tag&) -> system::result<std::conditional_t<std::is_void_v<result>,size_t,std::pair<size_t,result_type>>, std::exception_ptr>
    { if(error) return{in_place_error,error}; if constexpr (std::is_void_v<result_type>) return {in_place_value,index}; else return {in_place_value,std::make_pair(index,std::move(*result))}; }
  };
  awaitable operator co_await()&& { return {range, g}; }
};

auto race<asio::cancellation_type ct=all, URBG, awaitable<fork::promise_type>,...Promise>(URBG&& g, Promise&&...p) ->race_variadic_impl<ct,URBG,Promise...>;
auto race<asio::cancellation_type ct=all, URBG, PromiseRange>(URBG&& g, PromiseRange&& p) -> race_ranged_impl<ct,URBG,PromiseRange>;
auto race<asio::cancellation_type ct=all, awaitable<fork::promise_type>,...Promise>(URBG&& g, Promise&&...p) ->race_variadic_impl<ct,std::default_random_engine&,Promise...>;
auto race<asio::cancellation_type ct=all, PromiseRange>(PromiseRange&& p) -> race_ranged_impl<ct,std::default_random_engine&,PromiseRange>;
auto left_race<asio::cancellation_type ct=all, awaitable<fork::promise_type>,...Promise>(Promise&&...p) ->race_variadic_impl<ct,left_race_tag,Promise...>;
auto left_race<asio::cancellation_type ct=all, PromiseRange>(PromiseRange&& p) -> race_ranged_impl<ct,left_race_tag,PromiseRange>;

// gather
struct detail::gather_variadic_impl<...Args> {
  using tuple_type = std::tuple<decltype(get_awaitable_type(std::declval<Args&&>()))...>;
  ctor(Args&&...args);
  std::tuple<Args...> args;
  constexpr static size_t tuple_size = sizeof...(Args);
  struct awaitable : fork::static_shared_state<256 * tuple_size> {
    ctor<...idx>(std::tuple<Args...>& args, std::index_sequence<idx...>);
    tuple_type aws; std::array<asio::cancellation_signal, tuple_size> cancel;
    using result_store_part<T> = variant2::variant<monostate, void_as_monostate<co_await_result_t<T>>, std::exception_ptr>;
    std::tuple<result_store_part<Args>...> result;
    void interrupt_await_step<idx>() { if constexpr(interruptible<...>) std::get<idx>(aws).interrupt_await(); }
    void interrupt_await() { mp11::mp_for_each<mp_iota_c<sizeof...Args>>([&](idx){interrupt_await_step<idx>();}); }
    static fork await_impl<idx>(awaitable& this_);
    std::array<fork(*)(awaitable&), tuple_size> impls {[]<...idx>(std::index_sequence<idx...>)
      { return std::array<fork(*)(awaitable&), tuple_size>{&await_impl<idx>...}; }(std::make_index_sequence<tuple_size>{})};
    fork last_forked; size_t last_index=0u;
    bool await_ready();
    auto await_suspend<H>(std::coroutine_handle<H> h, const source_location& loc=CURRENT_LOCATION);
    using result_part<T> = system::result<co_await_result_t<T>, std::exception_ptr>;
    std::tuple<result_part<Args>...> await_resume();
  };
  awaitable operator co_await()&& { return {args, g, std::make_index_sequence<tuple_size>{}}; }
};
struct detail::gather_ranged_impl<Range> {
  Range aws;
  using result_type = system::result<co_await_result_t<std::decay_t<decltype(*std::begin(std::declval<Range>()))>>,std::exception_ptr>;
  using result_storage_type = variant2::variant<monostate,void_as_monostate<co_await_result_t<std::decay_t<decltype(*std::begin(declval<Range>()))>>>,std::exception_ptr>;
  struct awaitable : fork::shared_state {
    using type = std::decay_t<decltype(*std::begin(std::declval<Range>()))>;
    pmr::polymorphic_allocator<void> alloc{&resource};
    std::conditional_t<awaitable_type<type>, Range&, pmr::vector<co_awaitable_type<type>>> aws;
    pmr::vector<bool> ready{std::size(aws), alloc};
    pmr::vector<asio::cancellation_signal> cancel{std::size(aws), alloc};
    pmr::vector<result_storage_type> result{cancel.size(), alloc};
    ctor(Range& aws);
    void interrupt_await() { if constexpr (interruptible<...>) for (auto aw:aws) aw.interrupt_await(); }
    static fork await_impl(awaitable& this_, size_t idx);
    fork last_forked; size_t last_index=0u;
    bool await_ready();
    auto await_suspend<H>(std::coroutine_handle<H> h, const source_location& loc=CURRENT_LOCATION);
    auto await_resume();
  };
  awaitable operator co_await()&& { return {aws}; }
};

auto gather<...Promise>(Promise&&...p) ->gather_variadic_impl<Promise...>;
auto gather<PromiseRange>(PromiseRange&& p) -> gather_ranged_impl<PromiseRange>;

// join
struct detail::join_variadic_impl<...Args> {
  using tuple_type = std::tuple<decltype(get_awaitable_type(std::declval<Args&&>()))...>;
  ctor(Args&&...args);
  std::tuple<Args...> args;
  constexpr static size_t tuple_size = sizeof...(Args);
  struct awaitable : fork::static_shared_state<256 * tuple_size> {
    ctor<...idx>(std::tuple<Args...>& args, std::index_sequence<idx...>);
    tuple_type aws;
    std::array<asio::cancellation_signal, tuple_size> cancel_;
    std::array<asio::cancellation_signal*, tuple_size> cancel={make_null<Args>()...};
    constexpr static bool all_void = (std::is_void_v<co_await_result_t<Args>> && ...);
    using result_store_part<T> = std::optional<void_as_monostate<co_await_result_t<T>>>;
    std::conditional_t<all_void, monostate, std::tuple<result_store_part<Args>...>> result;
    std::exception_ptr error;
    void cancel_step<idx>() { auto& r=cancel[idx]; if(r) std::exchange(r,nullptr)->emit(all); }
    void cancel_all() { mp_for_each<mp_iota_c<sizeof...(Args)>>([&](auto idx){cancel_step})}
    void interrupt_await_step<idx>() { if constexpr(interruptible<...>) std::get<idx>(aws).interrupt_await(); }
    void interrupt_await() { mp11::mp_for_each<mp_iota_c<sizeof...(Args)>>([&](idx){interrupt_await_step<idx>();}); }
    static fork await_impl<idx>(awaitable& this_);
    std::array<fork(*)(awaitable&), tuple_size> impls {[]<...idx>(std::index_sequence<idx...>)
      { return std::array<fork(*)(awaitable&), tuple_size>{&await_impl<idx>...}; }(std::make_index_sequence<tuple_size>{})};
    fork last_forked; size_t last_index=0u;
    bool await_ready();
    auto await_suspend<H>(std::coroutine_handle<H> h, const source_location& loc=CURRENT_LOCATION);
    auto await_resume();
    auto await_resume(const as_tuple_tag&);
    auto await_resume(const as_result_tag&);
  };
  awaitable operator co_await()&& { return {args, g, std::make_index_sequence<tuple_size>{}}; }
};
struct detail::join_ranged_impl<Range> {
  Range aws;
  using result_type = co_await_result_t<std::decay_t<decltype(*std::begin(std::declval<Range>()))>>;
  constexpr static size_t result_size = sizeof(std::conditional_t<std::is_void_v<result_type>, monostate, result_type>);
  struct awaitable : fork::shared_state {
    struct dumy {ctor<...Args>(Args&&...){}};
    using type = std::decay_t<decltype(*std::begin(std::declval<Range>()))>;
    pmr::polymorphic_allocator<void> alloc{&resource};
    std::conditional_t<awaitable_type<type>, Range&, pmr::vector<co_awaitable_type<type>>> aws;
    pmr::vector<bool> ready{std::size(aws), alloc};
    pmr::vector<asio::cancellation_signal> cancel_{std::size(aws), alloc};
    pmr::vector<asio::cancellation_signal*> cancel{std::size(aws), alloc};
    std::conditional_t<std::is_void_v<result_type>,dummy,pmr::vector<std::optional<void_as_monostate<result_type>>>>
      result{cancel.size(),alloc};
    std::exception_ptr error{};
    ctor(Range& aws);
    void cancel_all() { for(auto&r:cancel) if(r) std::excahnge(r,nullptr)->emit(all); }
    void interrupt_await() { if constexpr (interruptible<...>) for (auto aw:aws) if(cancel[0]) aw.interrupt_await(); }
    static fork await_impl(awaitable& this_, size_t idx);
    fork last_forked; size_t last_index=0u;
    bool await_ready();
    auto await_suspend<H>(std::coroutine_handle<H> h, const source_location& loc=CURRENT_LOCATION);
    auto await_resume(const as_tuple_tag&);
    auto await_resume(const as_result_tag&);
    auto await_resume();
  };
  awaitable operator co_await()&& { return {aws}; }
};

auto join<...Promise>(Promise&&...p) -> join_variadic_impl<Promise...>;
auto join<PromiseRange>(PromiseRange&& p) -> join_ranged_impl<PromiseRange>;

struct detail::race_wrapper {
  using impl_type = decltype(race(std::declval<std::list<promise<void>>&>()));
  std::list<promise<void>> &waitables_;
  ctor(std::list<promise<void>>& waitables);
  struct awaitable_type {
    bool await_ready() { if(waitables_.empty()) return true; else return impl_->await_ready(); }
    auto await_suspend<Promise>(std::coroutine_handle<Promise> h) { return impl_->await_suspend(h) }
    void await_resume();
    ctor(std::list<promise<void>>& waitables);
  private: std::optional<impl_type::awaitable> impl_;
    std::list<promise<void>>& waitables_; std::default_random_engine& random_{prng()};
  };
  awaitable_type operator co_await()&& { return {waitables_}; }
};
struct detail::gather_wrapper {
  using impl_type = decltype(gather(std::declval<std::list<promise<void>>&>()));
  std::list<promise<void>>& waitables_;
  ctor(std::list<promise<void>>& waitables);
  struct awaitable_type {
    bool await_ready() { if(waitables_.empty()) return true; else return impl_->await_ready(); }
    auto await_suspend<Promise>(std::coroutine_handle<Promise> h) { return impl_->await_suspend(h) }
    void await_resume();
    ctor(std::list<promise<void>>& waitables);
  private: std::optional<decltype(gather(waitables_).operator co_await())> impl_;
    std::list<promise<void>>& waitables_;
  };
  awaitable_type operator co_await()&& { return {waitables_}; }
};

struct wait_group {
  explicit ctor(asio::cancellation_type normal_cancel=none, asio:;cancellation_type exception_cancel=all);
  void push_back(promise<void> p) { waitables_.push_back(std::move(p)); }
  size_t size() const { return waitables_.size(); }
  size_t reap() { return erase_if(waitables_, [](promise<void>& p){return p.ready()&&p;}); }
  void cancel(asio::cancellation_type ct=all) { for (auto& w: waitables_) w.cancel(ct); }
  race_wrapper wait_one() { return {waitables_}; }
  gather_wrapper wait() { return {waitables_}; }
  gather_wrapper::awaitable_type operator co_await() { return wait().operator co_await(); }
  gather_wrapper await_exit(std::exception_ptr ep)
  { auto ct=ep? ct_except_:ct_normal_; if (ct!=none) for (auto& w:waitables_) w.cancel(ct); return wait(); }
private: std::list<promise<void>> waitables_; asio::cancellation_type ct_normal_, ct_except_;
};
```

-----
#### IO

```c++
```

io/: acceptor, buffer, datagram_socket, endpoint, file, ops, pipe, random_access_device, random_access_file, read, resolver,
  seq_packet_socket, serial_port, signal_set, sleep, socket, ssl, steady_timer, stream_file, stream_socket, stream, system_timer, write
src/io/: acceptor, datagram_socket, endpoint, file, pipe, random_access_file, read, resolver,
  seq_packet_socket, serial_port, signal_set, sleep, socket, ssl, steady_timer, stream_file, stream_socket, system_timer, write

-----
### Configuration

* `SBO_BUFFER_SIZE`: default 4096

`executor`: 
* `USE_IO_CONTEXT`: `executor` is `asio::io_context::executor_type`
* `CUSTOM_EXECUTOR`: `executor` is user-provided
* Otherwise: `executor` is `asio::any_io_context`

`pmr`:
* `USE_STD_PMR` (default): `pmr` is `std::pmr`
* `USE_BOOST_CONTAINER_PMR`: `pmr` is `container::pmr`
* `USE_CUSTOM_PMR`: user-provided
* `NO_PMR`: no allocator

-----
### Dependencies

#### Boost.ASIO

* `<boost/asio/**.hpp>`

#### Boost.CallableTraits

* `<boost/callable_traits/args.hpp>`

#### Boost.CircularBuffer

* `<boost/circular_buffer.hpp>`

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Container

* `<boost/container/pmr/*.hpp>`

#### Boost.Context

* `<boost/context/fiber.hpp>`
* `<boost/context/fixedsize_stack.hpp>`

#### Boost.Core

* `<boost/core/demangle.hpp>`
* `<boost/core/exchange.hpp>`
* `<boost/core/ignore_unused.hpp>`
* `<boost/core/no_exceptions_support.hpp>`
* `<boost/core/span.hpp>`

#### Boost.Endian

* `<boost/endian/conversion.hpp>`

#### Boost.Intrusive

* `<boost/intrusive/list.hpp>`

#### Boost.MP11

* `<boost/mp11/algorithm.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/cat.hpp>`

#### Boost.SmartPtr

* `<boost/intrusive_ptr.hpp>`
* `<boost/smart_ptr/allocate_unique.hpp>`

#### Boost.StaticString

* `<boost/static_string.hpp>`

#### Boost.System

* `<boost/system/result.hpp>`
* `<boost/system/system_error.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Variant2

* `<boost/variant2/variant.hpp>`

------
### Standard Facilities
