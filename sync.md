# Boost.Sync

* lib: `boost/libs/sync`
* repo: `boostorg/sync`
* commit: `15b4079`, 2025-08-27

------
### Common Part

detail/: atomic, auto_handle, config, footer, futex, header, link_config, lockable_wrapper,
    pause, pthread.hpp, pthread_mutex_locks, system_error, throw_exception,
    time_traits, time_units, tss, waitable_timer, weak_linkage

#### Exceptions

exceptions/: lock_error, overflow_error, resource_error, runtime_exception, wait_error

------
### Mutexes

mutexes/: mutex, shared_spin_mutex, spin_mutex, timed_mutex
detail/mutexes/: <timed>_mutex_{posix|windows}, <shared>_spin_mutex, basic_mutex_windows

------
### Locks

locks/: lock_options, <un>lock_guard_<fwd>, shared_lock_guard_<fwd>, {unique|shared|upgrade}_lock_<fwd>

------
### Condition Variables

condition_variables/: condition_variable, condition_variable_any, cv_status, notify_all_at_thread_exit
detail/condition_variables/: basic_condition_variable_windows, condition_variable_any_{generic|windows}, condition_variable_{posix|windows}
thread_specific/: at_thread_exit, thread_specific_ptr
traits/is_condition_variable_compatible

------
### Semaphores & Events

semaphore
detail/semaphore_config
detail/semaphore/: semaphore_{windows|dispatch|posix|emulation|mach}, basic_semaphore_mach

events/: {auto|manual}_reset_event
detail/events/: auto_reset_event{wnidows|futex|semaphore|emulation}, manual_reset_event{wnidows|futex|emulation}, basic_event_windows

------
### Time Types Support

support/: boost_chrono, boost_date_time, posix_time, std::chrono

------
### Dependencies

#### Boost.Assert

* `<boost/assert.hpp>`
* `<boost/assert/source_location.hpp>`

#### Boost.Atomic

* `<boost/atomic/atomic.hpp>`
* `<boost/memory_order.hpp>`

#### Boost.Chrono

* `<boost/chrono/duration.hpp>`, `<boost/chrono/time_point.hpp>`, `<boost/chrono/system_clocks.hpp>` - for `boost::chrono` support

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/abi_prefix>`, `<boost/config/abi_suffix>`
* `<boost/cstdint.hpp>`
* `<boost/version.hpp>`

#### Boost.Core

* `<boost/core/enable_if.hpp>`
* `<boost/core/explicit_operator_bool.hpp>`
* `<boost/core/scoped_enum.hpp>`

#### Boost.DateTime

* `<boost/date_time/posix_time/posix_time_types.hpp>` - for `boost::date_time` support

#### Boost.Move

* `<boost/move/core.hpp>`, `<boost/move/utility.hpp>`

#### Boost.MPL

* `<boost/mpl/and.hpp>`
* `<boost/mpl/bool.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/seq/enum.hpp>`

#### Boost.Ratio

* `<boost/ratio/ratio.hpp>` - for `boost::chrono` support

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.System - when !USE_STD_SYSTEM_ERROR

* `<boost/system/error_code.hpp>`
* `<boost/system/system_error.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.WinAPI - on Windows

* `<boost/winapi/**.hpp>`

------
### Deprecated

Deprecated by **Boost.Thread**.

------
### Standard Facilities

Library: `<condition_variable>` (C++11), `<mutex>` (C++11), `<shared_mutex>` (C++14), `<semaphore>` (C++20)
