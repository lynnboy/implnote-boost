# Boost.Cobalt

* lib: `boost/libs/cobalt`
* repo: `boostorg/cobalt`
* commit: `694d9c4`, 2025-09-11

------
#### Concepts

```c++
concept awaitable_type<Awaitable,Promise=void> = requires (Awaitable aw, std::coroutine_handler<Promise> h)
{ {aw.await_ready()}->std::convertible_to<bool>; {aw.await_suspend(h)}; {aw.await_resume()}; };
concept awaitable<Awaitable,Promise=void> = awaitable_type<Awaitable,Promise>
  || requires (Awaitable&& aw) { {std::forward<Awaitable>(aw).operator co_await()}-> awaitable_type<Promise>;}
  || requires (Awaitable&& aw) { {operator co_await(std::forward<Awaitable>(aw))}-> awaitable_type<Promise>;};
struct promise_throw_if_cancelled_base;
struct enable_awaitables<Promise=void> { Aw&& await_transform<awaitable<Promise> Aw>(Aw&& aw, const source_location& loc=CURRENT_LOCATION); };
concept with_get_executor<T> = requires (T& t) { {t.get_executor()}->asio::execution::executor; };
```

------
#### Main Entrance

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
      auto p = std::coroutine_handler::from_promise(*mn.promise); asio::basic_signal_set<executor_type> ss{ctx,SIGNINT,SIGTERM};
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

async_for, channel, composition, detached, error, gather, generator, io, join, noop, op,
  promise, race, result, run, spawn, task, thread, unique_handle, wait_group, with
src/: channel, error, main, thread

detail/: await_result_helper, detached, exception, fork, forward_cancellation, gather, generator, handler, join,
  monotonic_resource, promise, race, sbo_resource, spawn, task, thread, util, wait_group, with, wrapper
src/detail/: exception, util

experimental/: context, frame, yield_context

impl/channel

io/: acceptor, buffer, datagram_socket, endpoint, file, ops, pipe, random_access_device, random_access_file, read, resolver,
  seq_packet_socket, serial_port, signal_set, sleep, socket, ssl, steady_timer, stream_file, stream_socket, stream, system_timer, write
src/io/: acceptor, datagram_socket, endpoint, file, pipe, random_access_file, read, resolver,
  seq_packet_socket, serial_port, signal_set, sleep, socket, ssl, steady_timer, stream_file, stream_socket, system_timer, write

-----
### Configuration

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
