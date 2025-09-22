# Boost.Timer

* lib: `boost/libs/timer`
* repo: `boostorg/timer`
* commit: `b7f65f0`, 2024-12-14

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
using nanosecond_type = int_least64_t;
const short default_places = 6;

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

```c++
class cpu_timer {
  cpu_times m_times; bool m_is_stopped;
public:
  ctor() noexcept { start(); } ~dtor();

  bool is_stopped() const noexcept { return m_is_stopped; }
  cpu_times elapsed() const noexcept;
  std::string format(short places=default_places, const std::string& format) const;
  std::string format(short places=default_places) const;

  void start() noexcept; void stop() noexcept; void resume() noexcept;
};
```

#### Class `auto_cpu_timer`

```c++
const std::string default_fmt{" %ws wall, %us user + %ss system = %ts CPU (%p%)\n"};

class auto_cpu_timer : public cpu_timer {
  short m_places; std::string m_format; std::ostream* m_os;
public:
  explicit ctor(short places=default_places);
  ctor(short places, const std::string& format);
  explicit ctor(const std::string& format);
  ctor(std::ostream& os, short places, const std::string& format);
  explicit ctor(std::ostream& os, short places=default_places);
  ctor(std::ostream& os, const std::string& format);
  ~dtor();

  std::ostream& ostream() const { return *m_os; }
  short places() const { return m_places; }
  const std::string& format_string() const { return m_format; }
};
```

#### class `progress_display`

```c++
class progress_display { // non-copyable
  std::ostream& m_os;
  const std::string m_s1, m_s2, m_3;
  unsigned long _count=0, _expected_count, _nex_tic_count=0;
  unsigned int _tic=0;
public:
  explicit ctor(unsigned long expected_count, std::ostream& os =std::cout,
    const std::string& s1="\n", const std::string& s1="", const std::string& s1="");
  void restart(unsigned long expected_count);
  unsigned long operator+= (unsigned long increment);
};
```

prints:

```
{s1}0%   10   20   30   40   50   60   70   80   90   100%
{s2}|----|----|----|----|----|----|----|----|----|----|
{s3}********
```

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
* `<boost/config/abi_prefix.hpp>`, `<boost/config/abi_suffix.hpp>`

------
### Standard Facilities

