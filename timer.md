# Boost.Timer

* lib: `boost/libs/timer`
* repo: `boostorg/timer`
* commit: `f4ec9ccd`, 2014-9-1

------
### Timer Version 1 (Deprecated)

#### Header

* `<boost/timer.hpp>`
* `<boost/progress.hpp>`

#### Class `timer`

##### Members

* `timer()`
* `restart()` - reset `elapsed` value
* `elapsed() const -> double`
* `elapsed_max() const -> double`, `elapsed_min() const -> double`

#### Class `progress_timer`

```c++
class progress_timer : public timer, noncopyable;
```

##### Members

* `progress_timer(std::ostream& = std::cout)`
* `~progress_timer()` - print "$elapsed s" on output stream.

#### Class `progress_display`

```c++
class progress_display : noncopyable; // not a timer, just a progress ruler
```

##### Members

* `explicit progress_display(unsigned long expected_count, os=std::cout, s1="\n", s2="", s3="")`
* `restart(expected_count)` - display 3-line progress percentage ruler on ostream.
* `+=`, `++` - add progress ruler on demand.
* `count() const -> unsigned long`
* `expected_count() const -> unsigned long`

------
### Timer Version 2

Superseded by **Boost.Chrono/Stopwatches**

#### Header

* `<boost/timer/timer.hpp>`

#### Struct `cpu_times`

```c++
struct cpu_times {
  nanosecond_type wall, user, system;
  void clear() { wall = user = system = 0LL; }
};
std::string format(const cpu_times& times, short places, const std::string& format);
std::string format(const cpu_times& times, short places = default_places);
```

#### Format specifiers

* `w` - wall time.
* `u` - user time.
* `s` - system time.
* `t` - total time.
* `p` - percentage.
* default format: `" %ws wall, %us user + %ss system = %ts CPU (%p%)\n"`.
* default decimal places: 6.

#### Class `cpu_timer`

* `cpu_timer() noexcept`
* `is_stopped() const noexcept`
* `elapsed() const noexcept -> cpu_times`
* `format(...) const -> std::string`
* `start() noexcept`
* `stop() noexcept`
* `resume() noexcept`

#### Class `auto_cpu_timer`

```c++
class auto_cpu_timer : public cpu_timer;
```

##### Members

* `auto_cpu_timer([std::ostream &], [short places], [const std::string& format])`
* `~auto_cpu_timer()` - print result on output stream.
* `ostream() const -> std::ostream&`
* `places() const -> short`
* `format_string() const -> const std::string&`
* `report()` - Write current elapsed string to output stream

------
### Rationale

* On Windows, use `GetProcessTimes()`
* On POSIX, use `times()`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/auto_link.hpp>` - For timer library linkage.
* `<boost/cstdint.hpp>`
* `<boost/config/warning_disable.hpp>`
* `<boost/config/abi_prefix.hpp>`, `<boost/config/abi_suffix.hpp>`

#### Boost.System

* `<boost/system/api_config.hpp>` - Detect POSIX/Windows.
* `<boost/cerrno.hpp>`.

#### Boost.Chrono

* `<boost/chrono/chrono.hpp>`

#### Boost.IO

* `<boost/io/ios_state.hpp>` - IOS state saver

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

------
### Standard Facilities

