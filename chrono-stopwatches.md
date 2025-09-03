# Boost.Chrono/StopWatches

* lib: `boost/libs/chrono`
* repo: `boostorg/chrono`
* commit: `bff204f1`, 2017-02-18
* removed officially.

------
### Stop Watches

Header `<boost/chrono/stopwatches.hpp>` for all 

Supersedes **Boost.Timer**.

#### Simple Stopwatch

Header `<boost/chrono/stopwatches/simple_stopwatch.hpp>`

```c++
template <typename Clock=high_resolution_clock>
class simple_stopwatch {
public:
  typedef Clock clock, duration, time_point, rep, period;
  constexpr static bool is_steady = clock::is_steady;
  simple_stopwatch(); // remember start = clock::now()
  duration elapsed(); // clock::now() - start
};
typedef simple_stopwatch<system_clock> system_simple_stopwatch;
// also typedef for steady, high_resolution clocks
// typedef for process_user_cpu, process_real_cpu, process_system_cpu, process_cpu clocks
// typedef for thread_clock
```

#### Stopwatch

Header `<boost/chrono/stopwatches/stopwatch.hpp>`

```c++
static const struct dont_start_t {} dont_start;

template <typename Clock=high_resolution_clock, typename LapsCollector=no_memory<Clock::duration>>
class stopwatch {
public:
  using laps_collector=LapsCollector, clock=Clock, duration, time_point, rep, period;
  constexpr static bool is_steady = clock::is_steady;

  stopwatch([laps_collector const&]); // start, optionally copy-in collector
  stopwatch([laps_collector const&, ]const dont_start_t&); // dont start, optionally copy-in collector
  start();
  restart(); // stop 'now() - start' as new lap
  stop(); // stop 'now() - start' as new lap
  reset(); // stop, clear laps_collector

  bool is_running() const;
  duration elapsed_current_lap() const; // clock::now - start
  duration elapsed() const; // return laps_collector's accumulated elapsed + current lap
  duration last() const; // return laps_collector's last collected lap
  laps_collector const & get_laps_collector() noexcept;

  using scoped_run = stopwatch_runner<stopwatch>>;
  using scoped_stop = stopwatch_stopper<stopwatch>>;
};

template <class StopWatch> class stopwatch_runner;    // call start on ctor, call stop on dtor
template <class StopWatch> class stopwatch_stopper;   // call stop/start
template <class StopWatch> class stopwatch_suspender; // call suspend/resume
template <class StopWatch> class stopwatch_resumer;   // call resume/suspend
```

------
### Laps Collectors

#### No Memory (Dummy Collector)

Header `<boost/chrono/stopwatches/collector/no_memory.hpp>`

```c++
template <typename Duration> struct no_memory;
```

Remembers nothing, always return `duration::zero()`.

#### Last Lap Collector

Header `<boost/chrono/stopwatches/collector/no_memory.hpp>`

```c++
template <typename Duration> struct last_lap;
```

Remembers last lap and return by `last`, always return `duration::zero()` for `elapsed`.

#### Last Lap Collector With Accumulator

Header `<boost/chrono/stopwatches/collector/laps_accumulator_set.hpp>`

```c++
template <typename Duration,
          typename Features = ..., // count, sum, min, max, mean
          typename Weight = void>
struct laps_accumulator_set : last_lap<Duration>;
```

* Additionally add laps to `accumulator<rep, Features, Weight>`, return its `sum` for `elapsed`.
* Member accessor `accumulator_set` gets the accumulator instance.

#### Laps Remembered In List

Header `<boost/chrono/stopwatches/collector/laps_sequence_container.hpp>`

```c++
template <typename Duration,
          typename SequenceContainer = std::list<Duration>>
struct laps_accumulator_set : last_lap<Duration>;
```

* Member accessor `container` gets the sequence container.

------
### Formatters For Stopwatches

Header `<boost/chrono/stopwatches/formatter/base_formatter.hpp>`
Header `<boost/chrono/stopwatches/formatter/elapsed_formatter.hpp>`
Header `<boost/chrono/stopwatches/formatter/times_formatter.hpp>`
Header `<boost/chrono/stopwatches/formatter/accumulator_set_formatter.hpp>`

```c++
template <typename CharT=char, typename Traits=std::char_traits<CharT>>
struct base_formatter;

template <typename Ratio, typename CharT,
  typename Traits = std::char_traits<CharT>, typename Alloc = std::allocator<CharT>>
class basic_elapsed_formatter             // default format: "%1%\n"
  : public base_formatter<CharT,Traits>, public basic_format<CharT, Traits> {
public:
  template <class Stopwatch> void operator()(Stopwatch &); // format 'elapsed()' to stream
};

typedef basic_elapsed_formatter<milli, char>    elapsed_formatter;
typedef basic_elapsed_formatter<milli, wchar_t> welapsed_formatter;

template<typename Ratio = milli, typename CharT = char,
    typename Traits = std::char_traits<CharT>, class Alloc = std::allocator<CharT> >
class basic_times_formatter               // default format: "real %1%, cpu %4% (%5%%%), user %2%, system %3%\n"
  : public base_formatter<CharT, Traits>, public basic_format<CharT, Traits> {
public:
  template <class Stopwatch> void operator()(Stopwatch &); // format 'elapsed()' as process times to stream
};

typedef basic_times_formatter<milli, char>    times_formatter;
typedef basic_times_formatter<milli, wchar_t> wtimes_formatter;

template<typename Ratio = milli, typename CharT = char,
    typename Traits = std::char_traits<CharT>, class Alloc = std::allocator<CharT> >
class basic_accumulator_set_formatter     // default format: "count=%1%, sum=%2%, min=%3%, max=%4%, mean=%5%\n"
  : public base_formatter<CharT, Traits>, public basic_format<CharT, Traits> {
public:
  template <class Stopwatch> void operator()(Stopwatch &); // query `get_laps_collector().accumulator_set()`
};

typedef basic_accumulator_set_formatter<milli, char>    accumulator_set_formatter;
typedef basic_accumulator_set_formatter<milli, wchar_t> waccumulator_set_formatter;
```

* `base_formatter` stores a `basic_ostream<CharT,Traits>`, a precision value (default 3),
  and a `duration_style` value (default `symbol`).
* Provide setters for attributes, constructible from a stream.

* Each formatter type can be constructed from a format string (either `const charT*` or `std::basic_string`)
  and an ostream (by default is `cout`)
* Each formatter outputs to stream in `operator()`, with specified `Ratio`, respecting the specified
  format string, precision, and duration style.

------
### Stopwatch Reporter (Stopwatch With Embedded Formatter)

Header `<boost/chrono/stopwatches/reporter/stopwatch_reporter.hpp>`
Header `<boost/chrono/stopwatches/reporter/*_default_formatter.hpp>`

```c++
using basic_clock_default_formatter<CharT, Clock>::type = basic_elapsed_formatter<milli, CharT>;
using basic_clock_default_formatter<CharT, process_cpu_clock>::type = basic_times_formatter<milli, CharT>;

using basic_stopwatch_reporter_default_formatter<CharT, Stopwatch> = basic_clock_default_formatter<CharT, Stopwatch::clock>;
using basic_stopwatch_reporter_default_formatter<CharT, stopwatch<Clock, laps_accumulator_set<Clock::duration, Features, Weight>>>::type
  = basic_accumulator_set_formatter<milli, CharT>;

template <typename CharT, typename Stopwatch,
  typename Formatter = basic_stopwatch_reporter_default_formatter<CharT, Stopwatch>::type>
class basic_stopwatch_reporter : public Stopwatch {       // Just a stopwatch bundled with a formatter
public:
  typedef StopWatch stopwatch_type; typedef Formatter formatter_type;
  // default ctor, ctor for format string or formatter, ctor for 'dont_start'
  ~basic_stopwatch_reporter() noexcept;   // call 'report()' if not reported

  void report() noexcept;     // invoke the formatter
  bool reported() const;
  formatter_type& format();   // get bundled formatter
};

using stopwatch_reporter<Stopwatch, Formatter=...> = basic_stopwatch_reporter<char, Stopwatch, Formatter>;
using wstopwatch_reporter<Stopwatch, Formatter=...> = basic_stopwatch_reporter<wchar_t, Stopwatch, Formatter>;
```

------
### Dependency

#### Boost.Chrono

#### Boost.Accumulators

* `<boost/accumulators/*.hpp>` - for accumulator-based laps collecting/statistics.

#### Boost.Format

* `<boost/format.hpp>`
* `<boost/format/group.hpp>`

#### Boost.System

* `<boost/system/error_code.hpp>` - used when hybrid error handling is enabled.

------
### Standard Facilities
