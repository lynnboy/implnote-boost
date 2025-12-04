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
#### Main API

```c++
struct detail::signal_helper { asio::cancellation_signal signal; };
struct detail::main_promise : signal_helper, promise_cancellation_base<cancellation_slot,enable_total_cancellation>,
  promise_throw_if_cancelled_base, enable_awaitables<main_promise>, enable_await_allocator<main_promise>,
  enable_await_executor<main_promise>, enable_await_deferred {
    ctor(int, char**);
    static pmr::memory_resource* my_resource = pmr::get_default_resource();
    void* operator new(size_t size); void operator delete(void* raw, size_t size);
    std::suspend_always initial_suspend() noexcept { return {}; }
    auto final_suspend() noexcept -> std::suspend_never;
    void unhandled_exception() { throw; }
    void return_value(int res=0) { if (result) *result=res; }
    static int run_main(main mn);
    friend int main(int argc, char* argv[]){
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
auto co_main(int argc, char* argv[]) -> main;
```

async_for, channel, composition, config, detached, error, gather, generator, io, join, main, noop,
op, promise, race, result, run, spawn, task, this_coro, this_thread, thread, unique_handle, wait_group, with

detail/: await_result_helper, detached, exception, fork, forward_cancellation, gather, generator, handler, join,
main, monotonic_resource, promise, race, sbo_resource, spawn, task, this_thread, thread, util, wait_group, with, wrapper

experimental/: context, frame, yield_context

impl/channel

io/: acceptor, buffer, datagram_socket, endpoint, file, ops, pipe, random_access_device, random_access_file, read, resolver,
seq_packet_socket, serial_port,signal_set, sleep, socket, ssl, steady_timer, stream_file, stream_socket, stream, system_timer, write

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
