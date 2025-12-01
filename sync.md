# Boost.Sync

* lib: `boost/libs/sync`
* repo: `boostorg/sync`
* commit: `15b4079`, 2025-08-27

------
### Common Part

```c++
void detail::throw_exception<Ex,...T>(source_location const& loc, T&&...args) { boost::throw_exception(Ex{(T&&)args...}, loc); }
void detail::pause() noexcept; // _mm_pause() on MSVC, __asm__ pause for x86/x64 on GCC

struct detail::time_point_tag{};
struct detail::time_duration_tag{};
struct detail::time_traits<T,Void=void> { static constexpr bool is_specialized = false; };
struct detail::enable_if_tag<T,Tag,R>;
struct detail::is_time_tag_of<T,TagT>;

class detail::system_duration {
    native_type m_value{0};
public: using native_type = int64_t;
    static constexpr uint64_t subsecond_fraction = 1000u; // 1000u for Win, otherwise 1000000000u
    explicit ctor(native_type value) noexcept :m_value{value}{}
    native_type get() const noexcept { return m_value; }
    self& operator{+|-}=(self const& that) noexcept { m_value {+|-}= that.m_value; return *this; }
    self operator-() const noexcept { return self{-m_value}; }
    friend self operator{+|-}(self left, self const& right) noexcept { left {+|-}= right; return left; }
};
class detail::system_time_point {
    native_type m_value{0};
public: using native_type = int64_t;
    static constexpr uint64_t subsecond_fraction = ...;
    explicit ctor(time_t t, unsigned subsecond=0) noexcept;
    explicit ctor(system_duration dur) noexcept;
    native_type const& get() const noexcept;
    static self now() noexcept; // win: GetSystemTimeAsFileTime, other: clock_gettime(CLOCK_REALTIME) or gettimeofday
    self& operator{+|-}=(self const& that) noexcept;
    friend self operator{+|-}(self left, self const& right) noexcept;
    friend system_duration operator-(self const& left, self const& right) noexcept;
};
class detail::chrono_time_point<TimePoint> : public TimePoint {
    time_point m_value{};
public: using time_point = TimePoint; using clock = time_point::clock; using duration = time_point::duration;
    // defaulted def-ctor, copy-ctor
    explicit ctor(time_point const& that);
    explicit ctor<T>(T const& arg);
    time_point const& get() const;
    static self now() noexcept { return self{clock::now()}; }
    self& operator{+|-}=(duration const& dur) noexcept;
    friend self operator{+|-}(self left, self const& right);
    friend duration operator-(self const& left, self const& right) noexcept;
};

struct time_traits<chrono::duration<Rep,Period>> { // boost:: and std::
    using tag = time_duration_tag; using unit_type = system_duration;
    static constexpr bool is_specialized = true;
    static system_duration to_sync_unit(duration<Rep,Period> const& dur) noexcept;
};
struct time_traits<chrono::time_point<Clock,Duration>> { // boost:: and std::
    using tag = time_point_tag; using unit_type = chrono_time_point<time_point<Clock,Duration>>;
    static constexpr bool is_specialized = true;
    static unit_time to_sync_unit(time_point<Clock,Duration> const& point) noexcept;
};
struct time_traits<chrono::time_point<system_clock,Duration>> { // boost:: and std::
    using tag = time_point_tag; using unit_type = system_time_point;
    static constexpr bool is_specialized = true;
    static system_time_point to_sync_unit(time_point<system_clock,Duration> const& point) noexcept;
};
struct time_traits<T,T::_is_boost_date_time_duration> {
    using tag = time_duration_tag; using unit_type = system_duration;
    static constexpr bool is_specialized = true;
    static system_duration to_sync_unit(T const& dur);
};
struct time_traits<T,T::_is_boost_date_time_time_point> {
    using tag = time_point_tag; using unit_type = system_time_point;
    static constexpr bool is_specialized = true;
    static system_time_point to_sync_unit(T const& point);
};
struct time_traits<struct ::timespec> {
    using tag = time_point_tag; using unit_type = system_time_point;
    static constexpr bool is_specialized = true;
    static system_time_point to_sync_unit(struct ::timespec const& point);
};
```

#### Exceptions

```c++
using errc_t = std::errc; // system::errc::errc_t if no STD <system>
struct runtime_exception : system_error {
    explicit ctor(int sys_err = 0); ctor(int sys_err, {const char*|std::string const&} what); ~dtor() noexcept;
    int native_error() const noexcept { return code().value(); }
};
struct lock_error : runtime_exception {};
struct overflow_error : runtime_exception {};
struct resource_error : runtime_exception {};
struct wait_error : runtime_exception {};
```

#### Windows Commons

```c++
struct detail::weak_linkage<T,Tag=void> { static T value={}; };
struct detail::windows::waitable_timer_state {
    enum init_state { st_uninitialized, st_in_progress, st_initialized };
    long initialized; DWORD_ tls_key; HANDLE_ tls_key_holder;
    struct thread_local_context {  // no copy
        const uint32_t boost_version{BOOST_VERSION};
        HANDLE_ waitable_timer{NULL}, current_thread{NULL}, wait_handle{NULL};
        ~dtor() { if (wait_handle) UnregisterWait(wait_handle); if (current_thread) CloseHandle(current_thread); if (waitable_timer) CloseHandle(waitable_timer); }
        static void STDCALL destroy(PVOID_ p, BOOLEAN_) { delete (self*)p; }
    };
    struct semaphore_basic_information { ULONG_ current_count, maximum_count; };
    void init() {
        // 1. atomically set `initialized` from `st_uninitialized` to `st_in_progress`
        tls_key_holder = OpenSemaphoreW(semaphore_all_access, false, "boost_sync_tls_<PID>"); // 2. create SEM
        if (!tls_key_holder) { // 3. not created - this is first time
            DWORD_ key = TlsAlloc(); // 3.1 alloc TLS slot.  check error and throw
            tls_key_holder = CreateSemaphoreW(NULL, key, LONG_MAX, "boost_sync_tls_<PID>"); // 3.2 create it. And handle errors
            tls_key = key; std::atexit(&destroy); // and atomically set to `st_initialized`
        } else { // 4. already created by other call
            std::atexit(&destroy); // 4.1
            if (...) { // loaded 'ntdll.dll!NtQuerySemaphore`
                semaphore_basic_information info; NtQuerySemaphore(tls_key_holder, 0, &info, sizeof(info), NULL); // 4.2 fetch the SEM
                tls_key = (DWORD_)info.current_count; // And set to `st_initialized`
                return;
            }
            tls_key = TlsAlloc(); // 5. fallback, just alloc TLS. And set to `st_initialized`
        }
    }
    thread_local_context* create_thread_local_context() {
        auto ctx = new thread_local_context{}; const HANDLE_ current_process = GetCurrentProcess();
        if (DuplicateHandle(...)) { // for GetCurrentThread() into ctx->current_thread
            if (RegisterWaitForSingleObject(...)) { // Call `destroy` on ctx->current_thread, set to ctx->wait_handle
                if (ctx->waitable_timer = create_anonymous_waitable_timer(NULL, false) && TlsSetValue(tls_key, ctx)) return ctx;
            }
        }
        delete ctx; // handle error and throw
    }
    static void destroy() {
        auto& state = weak_linkage<waitable_timer_state>::value;
        if (state.tls_key_holder) { CloseHandle(state.tls_key_holder); state.tls_key_holder=NULL; }
    }
};
HANDLE_ detail::windows::get_waitable_timer() {
    auto& state = weak_linkage<waitable_timer_state>::value;
    if (state.initialized != st_initialized) state.init();
    auto p = static_cast<thread_local_context*>(TlsGetValue(state.tls_key))
    if (!p) p = state.create_thread_local_context();
    if (p->boost_version != BOOST_VERSION) throw_exception<std::logic_error>(...);
    return p->waitable_timer;
}
```

------
### Mutexes

```c++
struct is_condition_variable_compatible<Mutex>; // Mutex::_is_condition_variable_compatibile

class spin_mutex { // no copy
    atomic<bool> state; // when atomic<bool> always lockfree, otherwise use atomic_flag
public: void lock() noexcept; bool try_lock() noexcept; void unlock() noexcept;
};
class shared_spin_mutex { // no copy
    enum {unlocked_state, reader_mask=0x7fffffff, locked_state=0x80000000};
    atomic<bool> state{unlocked_state};
public: void lock() noexcept; bool try_lock() noexcept; void unlock() noexcept;
    void lock_shared() noexcept; bool try_lock_shared() noexcept; void unlock_shared() noexcept;
};
```

##### Posix

```c++
class detail::posix::pthread_mutex_lock_guard { pthread_mutex_t* const m_mutex; // no copy
public: explicit ctor(pthread_mutex_t& m) noexcept : m_mutex{&m}{pthread_mutex_lock(m_mutex);}
    ~dtor() { pthread_mutex_unlock(m_mutex); } };
class detail::posix::pthread_mutex_unlock_guard { pthread_mutex_t* const m_mutex;  // no copy
public: explicit ctor(pthread_mutex_t& m) noexcept : m_mutex{&m}{pthread_mutex_unlock(m_mutex);}
    ~dtor() { pthread_mutex_lock(m_mutex); } };

class mutex { // no copy
    pthread_mutex_t m_mutex{PTHREAD_MUTEX_INITIALIZER};
public: using _is_condition_variable_compatible = void; using native_handle_type = pthread_mutex_t*;
    ~dtor(){ pthread_mutex_destroy(&m_mutex); }
    void lock(); void unlock(); bool try_lock(); // pthread_mutex_XXX
    native_handle_type native_handle() noexcept { return &m_mutex; }
};

class timed_mutex { // no copy
    pthread_mutex_t m_mutex; // when POSIX has timedlock, impl by single mutex
    pthread_mutex_t m_mutex; pthread_cond_t m_cond; bool m_is_locked{false}; // when old POSIX
    bool priv_timed_lock(system_duratino const& dur);
    bool priv_timed_lock(system_time_point const& t); // pthread_mutex_timedlock on m_mutex or pthread_cond_timedwait on m_cond
    bool priv_timed_lock<TimePoint>(chrono_time_point<TimePoint> const& t);
public: using _is_condition_variable_compatible = void; using native_handle_type = pthread_mutex_t*; // when single mutex
    ctor(); ~dtor(); // init/destroy m_mutex, and m_cond
    void lock(); void unlock(); bool try_lock(); // either just lock m_mutex, or work on m_is_locked
    bool timed_lock<Time>(Time const& t) requires(...);
    bool try_lock_for<Duration>(Duration const& rel_time) requires(...);
    bool try_lock_for<TimePoint>(TimePoint const& abs_time) requires(...);
    native_handle_type native_handle() noexcept { return &m_mutex; } // only when single mutex
};
```

##### Windows

```c++
class detail::windows::basic_mutex { // no copy
    void priv_lock();
public: enum { lock_flag_bit=31u, event_set_flag_bit=30u, lock_flag_value = 1<<31, event_set_flag_value = 1<<30};
    atomic<uint32_t> m_active_count{0u}; atomic<uintptr_t> m_event{0u};
    constexpr ctor() noexcept{}  ~dtor() { auto event = (HANDLE_)(m_event.load(memory_order_relaxed)); }
    void lock(); void unlock(); bool try_lock();
    HANDLE_ get_event();
    void mark_waiting_and_try_lock(uint32_t& old_count);
    void clear_waiting_and_try_lock(uint32_t& old_count);
};

class mutex { // no copy
    windows::basic_mutex m_mutex;
public: using _is_condition_variable_compatible = void;
    void lock(); void unlock(); bool try_lock(); // pthread_mutex_XXX
};

class timed_mutex { // no copy
    windows::basic_mutex m_mutex;
    bool priv_timed_lock(system_duratino const& dur);
    bool priv_timed_lock(system_time_point const& t);
    bool priv_timed_lock<TimePoint>(chrono_time_point<TimePoint> const& t);
public: using _is_condition_variable_compatible = void;
    void lock(); void unlock(); bool try_lock();
    bool timed_lock<Time>(Time const& t) requires(...);
    bool try_lock_for<Duration>(Duration const& rel_time) requires(...);
    bool try_lock_for<TimePoint>(TimePoint const& abs_time) requires(...);
    native_handle_type native_handle() noexcept { return &m_mutex; } // only when single mutex
};
```

------
### Locks

```c++
constexpr struct defer_lock_t{} defer_lock{};
constexpr struct try_to_lock_t{} try_to_lock{};
constexpr struct adopt_lock_t{} adopt_lock{};

class lock_guard<Mutex> { // no copy
    mutex_type & m_mutex;
public: using mutex_type = Mutex;
    explicit ctor(mutex_type& m) :m_mutex{m}{m.lock();}
    ctor(mutex_type& m, adopt_lock_t) :m_mutex{m}{}
    ~dtor(){m_mutex.unlock();}
};

class unlock_guard<Lockable> { // no copy
    mutex_type & m_lockable;
public: using mutex_type = Lockable;
    explicit ctor(mutex_type& m) noexcept :m_lockable{m}{m.unlock();}
    ~dtor(){m_lockable.lock();}
};

class shared_lock_guard<Mutex> { // no copy
    mutex_type & m_mutex;
public: using mutex_type = Mutex;
    explicit ctor(mutex_type& m) :m_mutex{m}{m.lock_shared();}
    ctor(mutex_type& m, adopt_lock_t) :m_mutex{m}{}
    ~dtor(){m_mutex.unlock_shared();}
};

class unique_lock<Mutex> { // allow move, no copy
    mutex_type* m_mutex{nullptr}; bool m_is_locked{false};
public: using mutex_type = Mutex;
    ctor()noexcept{}; explicit ctor(mutex_type& m) :m_mutex{&m} { lock(); }
    ctor(mutex_type& m, adopt_lock_t) noexcept : m_mutex{&m}, m_is_locked{true} {}
    ctor(mutex_type& m, defer_lock_t) noexcept : m_mutex{&m} {}
    ctor(mutex_type& m, try_to_lock_t) noexcept : m_mutex{&m} {try_lock();}
    ctor<Time>(mutex_type& m, Time const& t) requires(...) { timed_lock(t); }
    ctor(self&& that) noexcept : m_mutex{that.m_mutex}, m_is_locked{that.m_is_locked}{ that.m_mutex=nullptr; that.m_is_locked=false; }
    explicit ctor(upgrade_lock<mutex_type>&& that) : m_mutex{that.m_mutex}, m_is_locked{that.m_is_locked}
    { if (m_is_locked) m_mutex->unlock_upgrade_and_lock(); that.release(); }
    ctor(upgrade_lock<mutex_type>&& ul, try_to_lock_t)
    { if (ul.owns_lock()) { if (ul.mutex()->try_unlock_upgrade_and_lock()) { m_mutex=ul.release(); m_is_locked=true; }} else m_mutex = ul.release(); }
    ctor(shared_lock<mutex_type>&& sl, try_to_lock_t)
    { if (sl.owns_lock()) { if (sl.mutex()->try_unlock_shared_and_lock()) { m_mutex=sl.release(); m_is_locked=true; }} else m_mutex = sl.release(); }
    ctor<Time>(shared_lock<mutex_type>&& sl, Time const& t) requires(...)
    { if (sl.owns_lock()) { if (sl.mutex()->timed_unlock_shared_and_lock(t)) { m_mutex=sl.release(); m_is_locked=true; }} else m_mutex = sl.release(); }
    ~dtor() { if (m_is_locked) m_mutex->unlock(); }
    self& operator=(self&& that) noexcept { swap(that); return *this; }
    self& operator=(upgrade_lock<mutex_type>&& that) noexcept { self temp{move(that)}; swap(temp); return *this; }
    void lock(); bool try_lock(); void unlock(); // check state, and forward to m_mutex with m_is_locked
    bool timed_lock<Time>(Time const& t) requires(...);
    bool try_lock_for<Duration>(Duration const& rel_time) requires(...);
    bool try_lock_for<TimePoint>(TimePoint const& abs_time) requires(...);
    explicit operator bool() const; bool operator! () const noexcept; // m_is_locked
    bool owns_lock() const noexcept; mutex_type* mutex() const noexcept();
    mutex_type* release() noexcept { auto res = m_mutex; mutex=nullptr; m_is_locked=false; return res; }
    void swap(self& that) noexcept; friend void swap(self& l, self& r) noexcept;
};

class shared_lock<Mutex> { // allow move, no copy
    mutex_type* m_mutex{nullptr}; bool m_is_locked{false};
public: using mutex_type = Mutex;
    ctor()noexcept{}; explicit ctor(mutex_type& m) :m_mutex{&m} { lock(); }
    ctor(mutex_type& m, adopt_lock_t) noexcept : m_mutex{&m}, m_is_locked{true} {}
    ctor(mutex_type& m, defer_lock_t) noexcept : m_mutex{&m} {}
    ctor(mutex_type& m, try_to_lock_t) noexcept : m_mutex{&m} {try_lock();}
    ctor<Time>(mutex_type& m, Time const& t) requires(...) { timed_lock(t); }
    ctor(self&& that) noexcept : m_mutex{that.m_mutex}, m_is_locked{that.m_is_locked}{ that.m_mutex=nullptr; that.m_is_locked=false; }
    explicit ctor(unique_lock<mutex_type>&& that) : m_mutex{that.m_mutex}, m_is_locked{that.m_is_locked}
    { if (m_is_locked) m_mutex->unlock_and_lock_shared(); that.release(); }
    explicit ctor(upgrade_lock<mutex_type>&& that) : m_mutex{that.m_mutex}, m_is_locked{that.m_is_locked}
    { if (m_is_locked) m_mutex->unlock_upgrade_and_lock_shared(); that.release(); }
    ~dtor() { if (m_is_locked) m_mutex->unlock_shared(); }
    self& operator=(self&& that) noexcept { swap(that); return *this; }
    self& operator=({unique|upgrade}_lock<mutex_type>&& that) noexcept { self temp{move(that)}; swap(temp); return *this; }
    void lock(); bool try_lock(); void unlock(); // check state, call lock_shared, unlock_shared, try_lock_shared
    bool timed_lock<Time>(Time const& t) requires(...); // check state, call timed_lock_shared
    bool try_lock_for<Duration>(Duration const& rel_time) requires(...); // check state, call timed_lock_shared_for
    bool try_lock_for<TimePoint>(TimePoint const& abs_time) requires(...); // check state, call timed_lock_shared_until
    explicit operator bool() const; bool operator! () const noexcept; // m_is_locked
    bool owns_lock() const noexcept; mutex_type* mutex() const noexcept();
    mutex_type* release() noexcept { auto res = m_mutex; mutex=nullptr; m_is_locked=false; return res; }
    void swap(self& that) noexcept; friend void swap(self& l, self& r) noexcept;
};

class upgrade_lock<Mutex> { // allow move, no copy
    mutex_type* m_mutex{nullptr}; bool m_is_locked{false};
public: using mutex_type = Mutex;
    ctor()noexcept{}; explicit ctor(mutex_type& m) :m_mutex{&m} { lock(); }
    ctor(mutex_type& m, adopt_lock_t) noexcept : m_mutex{&m}, m_is_locked{true} {}
    ctor(mutex_type& m, defer_lock_t) noexcept : m_mutex{&m} {}
    ctor(mutex_type& m, try_to_lock_t) noexcept : m_mutex{&m} {try_lock();}
    ctor<Time>(mutex_type& m, Time const& t) requires(...) { timed_lock(t); }
    ctor(self&& that) noexcept : m_mutex{that.m_mutex}, m_is_locked{that.m_is_locked}{ that.m_mutex=nullptr; that.m_is_locked=false; }
    explicit ctor(unique_lock<mutex_type>&& that) : m_mutex{that.m_mutex}, m_is_locked{that.m_is_locked}
    { if (m_is_locked) m_mutex->unlock_and_lock_upgrade(); that.release(); }
    ctor(shared_lock<mutex_type>&& sl, try_to_lock_t)
    { if (sl.owns_lock()) if (sl.mutex()->try_unlock_shared_and_lock_upgrade()) { m_mutex=sl.release(); m_is_locked=true; } else m_mutex = sl.release(); }
    ctor<Time>(shared_lock<mutex_type>&& sl, Time const& t) requires(...)
    { if (sl.owns_lock()) if (sl.mutex()->timed_unlock_shared_and_lock_upgrade(t)) { m_mutex=sl.release(); m_is_locked=true; } else m_mutex = sl.release(); }
    ~dtor() { if (m_is_locked) m_mutex->unlock_upgrade(); }
    self& operator=(self&& that) noexcept { swap(that); return *this; }
    self& operator=({unique|upgrade}_lock<mutex_type>&& that) noexcept { self temp{move(that)}; swap(temp); return *this; }
    void lock(); bool try_lock(); void unlock(); // check state, call lock_upgrade, unlock_upgrade, try_lock_upgrade
    bool timed_lock<Time>(Time const& t) requires(...); // check state, call timed_lock_upgrade
    bool try_lock_for<Duration>(Duration const& rel_time) requires(...); // check state, call timed_lock_upgrade_for
    bool try_lock_for<TimePoint>(TimePoint const& abs_time) requires(...); // check state, call timed_lock_upgrade_until
    explicit operator bool() const; bool operator! () const noexcept; // m_is_locked
    bool owns_lock() const noexcept; mutex_type* mutex() const noexcept();
    mutex_type* release() noexcept { auto res = m_mutex; mutex=nullptr; m_is_locked=false; return res; }
    void swap(self& that) noexcept; friend void swap(self& l, self& r) noexcept;
};
```

------
### Condition Variables

```c++
enum class cv_status { no_timeout, timeout };
void add_thread_exit_notify_entry(mutex& mtx, condition_variable& cond);
void notify_all_at_thread_exit(condition_variable& cond, unique_lock<mutex> lock)
{ add_thread_exit_notify_entry(*lock.mutex(), cond); lock.release(); }
```

##### POSIX & Other

```c++
class condition_variable { // no copy
    pthread_cond_t m_cond{PTHREAD_COND_INITIALIZER};
public: using native_handle_type = pthread_cond_t*;
    ctor(); ~dtor(){pthread-cond_destroy(&m_cond);}
    void notify_one() noexcept { pthread_cond_signal(&m_cond); }
    void notify_all() noexcept { pthread_cond_broadcast(&m_cond); }
    void wait<Mutex>(unique_lock<Mutex>& lock) { priv_wait(lock.mutex()->native_handle()); }
    void wait<Mutex,Predicate>(unique_lock<Mutex>& lock, Predicate pred) requires(...)
    { while (!pred()) priv_wait(lock.mutex()->native_handle()); }
    bool timed_wait<Mutex,Time>(unique_lock<Mutex>& lock, Time const& t)
    { return priv_timed_wait(lock,time_traits::to_sync_unit(t)) == no_timeout; }
    bool timed_wait<Mutex,{TimePoint|Duration},Predicate>(unique_lock<Mutex>& lock, {TimePoint|Duration} const& t, Predicate pred);
    cv_status wait_until<Mutex,TimePoint>(unique_lock<Mutex>& lock, TimePoint const& abs_time);
    bool wait_until<Mutex,TimePoint,Predicate>(unique_lock<Mutex>& lock, TimePoint const& abs_time, Predicate pred);
    cv_status wait_for<Mutex,Duration>(unique_lock<Mutex>& lock, TimePoint const& rel_time);
    bool wait_for<Mutex,Duration,Predicate>(unique_lock<Mutex>& lock, TimePoint const& rel_time, Predicate pred);
    native_handle_type native_handle() noexcept { return &m_cond; }
};
class condition_variable_any { // no copy
    class relocker<Lock>{ Lock* m_lock{nullptr};
    public: ~dtor() { if (m_lock) m_lock->lock(); }
        void unlock(Lock& lock) { lock.unlock(); m_lock = &lock; } };
    mutex m_mutex; condition_variable m_cond;
public:
    void notify_one() { lock_guard internal_lock{m_mutex}; m_cond.notify_one(); }
    void notify_all() { lock_guard internal_lock{m_mutex}; m_cond.notify_all(); }
    void wait<Lock>(Lock& lock) { relocker<Lock> relock_guard; unique_lock internal_lock(m_mutex); relock_guard.unlock(lock); m_cond.wait(internal_lock); }
    void wait<Lock,Predicate>(Lock& lock, Predicate pred) { while (!pred()) wait(lock); }
    bool timed_wait<Lock,Time,[Predicate]>(Lock& lock, Time const& t, <Predicate pred>);
    cv_status wait_until<Lock,TimePoint>(Lock& lock, TimePoint const& abs_time);
    bool wait_until<Lock,TimePoint,Predicate>(Lock& lock, TimePoint const& abs_time, Predicate pred);
    cv_status wait_for<Lock,Duration>(Lock& lock, Duration const& rel_time);
    bool wait_for<Lock,Duration,Predicate>(Lock& lock, Duration const& rel_time, Predicate pred);
};
```

##### Windows

```c++
class detail::windows::basic_condition_variable {
    using mutex_type = mutex;
    struct waiter_state { // no copy
        self *m_prev, *m_next;  HANDLE_ m_semaphore; unsigned long m_waiter_count{0}, m_notify_count{0};
        ctor(): m_prev{this}, m_next{this} { m_semaphore = create_anonymous_semaphore(NUL,0,LONG_MAX); } // check for error
        ~dtor() { CloseHandle(m_semaphore); }
    };
    class relocker<Lockable> { // no copy
        Lockable& m_lock; bool m_unlocked{false};
    public: explicit ctor(Lockable& lock) noexcept: m_lock{lock}{}
        ~dtor() { if (m_unlocked) m_lock.lock(); }
        void unlock() noexcept { m_lock.unlock(); m_unlocked = true; }
    };
    mutex_type m_internal_mutex; waiter_state *m_notify_state{NULL}, *m_wait_state{NULL}; atomic<size_t> m_total_waiter_count{0};
    void wake_waiters(size_t count_to_wake) noexcept; // ReleaseSemaphore on waiter_state.m_semaphore
    waiter_state* initiate_wait();
    void priv_wait(waiter_state* state);
    cv_status priv_timed_wait(waiter_state* state, system_duration t); // WaitForSingleObject on m_semaphore at t
    cv_status priv_timed_wait(waiter_state* state, HANDLE_ waitable_timer); // WaitForMultipleObjects on m_semaphore and waitable_timer
public: ~dtor(); // delete chain of waiter_state of m_notify_state
    void notify_one() noexcept; // wake_wakers(1)
    void notify_all() noexcept; // wake_wakers(count)
    void wait<Lockable>(Lockable& lock) { relocker unlocker(lock); auto state = initiate_wait(); unlocker.unlock() priv_wait(state); }
    cv_status timed_wait<Lockable>(Lockable& lock, {system_duration|system_time_point const&} t);
    cv_status timed_wait<Lockable,TimePoint>(Lockable& lock, chrono_time_point<TimePoint> const& t);
};

class condition_variable { // no copy
    windows::basic_condition_variable m_cond;
public: using native_handle_type = pthread_cond_t*;
    void notify_one() noexcept { m_cond.notify_one(); }
    void notify_all() noexcept { m_cond.notify_all(); }
    void wait<Mutex>(unique_lock<Mutex>& lock) { m_cond.wait(lock); }
    void wait<Mutex,Predicate>(unique_lock<Mutex>& lock, Predicate pred) requires(...) { while (!pred()) m_cond.wait(lock); }
    bool timed_wait<Mutex,Time>(unique_lock<Mutex>& lock, Time const& t);
    bool timed_wait<Mutex,{TimePoint|Duration},Predicate>(unique_lock<Mutex>& lock, {TimePoint|Duration} const& t, Predicate pred);
    cv_status wait_until<Mutex,TimePoint>(unique_lock<Mutex>& lock, TimePoint const& abs_time);
    bool wait_until<Mutex,TimePoint,Predicate>(unique_lock<Mutex>& lock, TimePoint const& abs_time, Predicate pred);
    cv_status wait_for<Mutex,Duration>(unique_lock<Mutex>& lock, TimePoint const& rel_time);
    bool wait_for<Mutex,Duration,Predicate>(unique_lock<Mutex>& lock, TimePoint const& rel_time, Predicate pred);
};
class condition_variable_any { // no copy
    windows::basic_condition_variable m_cond;
public:
    void notify_one() noexcept { m_cond.notify_one(); }
    void notify_all() noexcept { m_cond.notify_all(); }
    void wait<Lock>(Lock& lock) { m_cond.wait(lock); }
    void wait<Lock,Predicate>(Lock& lock, Predicate pred) { while (!pred()) m_cond.wait(lock); }
    bool timed_wait<Lock,Time>(Lock& lock, Time const& t);
    bool timed_wait<Lock,{TimePoint|Duration},Predicate>(Lock& lock, {TimePoint|Duration} const& t, Predicate pred);
    cv_status wait_until<Lock,TimePoint>(Lock& lock, TimePoint const& abs_time);
    bool wait_until<Lock,TimePoint,Predicate>(Lock& lock, TimePoint const& abs_time, Predicate pred);
    cv_status wait_for<Lock,Duration>(Lock& lock, TimePoint const& rel_time);
    bool wait_for<Lock,Duration,Predicate>(Lock& lock, TimePoint const& rel_time, Predicate pred);
};
```

------
### Semaphores

##### Windows

```c++
class semaphore { // no copy
    HANDLE_ m_sem;
    bool priv_timed_wait(system_duration t); // WaitForSingleObject
    bool priv_timed_wait(system_time_point const& t);
    bool priv_timed_wait<TimePoint(chrono_time_point<TimePoint> const& t);
public: explicit ctor(unsigned i=0); // create_anonymous_semaphore
    ~dtor() noexcept; // CloseHandle
    void post(); // ReleaseSemaphore
    void wait(); // WaitForSingleObject for infinite
    bool try_wait(); // WaitForSingleObject for 0
    bool timed_wait<{Time|Duration|TimePoint}>({Time|Duration|TimePoint} const& t);
};
```

##### MACH with libdispatch

```c++
class semaphore { // no copy
    dispatch_semaphore_t m_sem;
    bool priv_timed_wait(system_duration t); // dispatch_semaphore_wait
    bool priv_timed_wait(system_time_point const& t);
    bool priv_timed_wait<TimePoint(chrono_time_point<TimePoint> const& t);
public: explicit ctor(unsigned i=0); // dispatch_semaphore_create, dispatch_semaphore_signal by i times
    ~dtor() noexcept; // dispatch_release
    void post(); // dispatch_semaphore_signal
    void wait(); // dispatch_semaphore_wait for DISPATCH_TIME_FOREVER
    bool try_wait(); // dispatch_semaphore_wait for DISPATCH_TIME_NOW
    bool timed_wait<{Time|Duration|TimePoint}>({Time|Duration|TimePoint} const& t);
};
```

##### POSIX

```c++
class semaphore { // no copy
    sem_t m_sem;
    bool priv_timed_wait(system_duration t); // sem_timedwait
    bool priv_timed_wait(system_time_point const& t);
    bool priv_timed_wait<TimePoint(chrono_time_point<TimePoint> const& t);
public: explicit ctor(unsigned i=0); // sem_init
    ~dtor() noexcept; // sem_destroy
    void post(); // sem_post
    void wait(); // sem_wait
    bool try_wait(); // sem_trywait
    bool timed_wait<{Time|Duration|TimePoint}>({Time|Duration|TimePoint} const& t);
};
```

##### Emulation

```c++
class semaphore { // no copy
    mutex m_mutex; condition_variable m_cond; unsigned m_count{0};
public: explicit ctor(unsigned i=0) : m_count{i}{}
    void post() { lock_guard lock{m_mutex}; ++m_count; m_cond.notify_one(); }
    void wait() { lock_guard lock{m_mutex}; while (m_count==0) m_cond.wait(lock); --m_count; }
    bool try_wait() { lock_guard lock{m_mutex}; if (m_count==0) return false; --m_count; return true; }
    bool timed_wait<{Time|Duration|TimePoint}>({Time|Duration|TimePoint} const& t)
    { unique_lock lock{m_mutex}; while (m_count==0) if (!m_cond.timed_wait(lock, t)) if (m_count==0) return false; else break; --m_count; return true; }
};
```

------
### Events

##### Windows

```c++
class detail::windows::basic_event { // no copy
    HANDLE_ m_handle;
public: explicit ctor(bool manual_reset); // create_anonymous_event
    ~dtor(); // CloseHandle
    void set(); void reset(); // SetEvent/ResetEvent
    void wait(); // WaitForSingleObject for infinite
    void try_wait(); // WaitForSingleObject for 0
    bool timed_wait(system_duration t); // WaitForSingleObject for t
    bool timed_wait(system_time_point const& t); // WaitForMultipleObjects on m_handle and get_waitable_timer
    bool timed_wait<TimePoint>(chrono_time_point<TimePoint> const& t);
};

class auto_reset_event { // no copy
    basic_event m_event{false};
public: ctor();
    void post() { m_event.set(); }
    void wait() { m_event.wait(); }
    bool try_wait() { return m_event.try_wait(); }
    bool timed_wait<Time>(Time timeout);
    bool timed_wait<Duration>(Duration const& duration);
    bool timed_wait<TimePoint>(TimePoint const& abs_time);
};

class manual_reset_event { // no copy
    basic_event m_event{true};
public: ctor();
    void set() { m_event.set(); }
    void reset() { m_event.reset(); }
    void wait() { m_event.wait(); }
    bool try_wait() { return m_event.try_wait(); }
    bool timed_wait<Time>(Time timeout);
    bool timed_wait<Duration>(Duration const& duration);
    bool timed_wait<TimePoint>(TimePoint const& abs_time);
};
```

##### Linux

```c++
int detail::linux_::futex_invoke(int* addr1, int op, int val1, const struct ::timespec* timeout=NULL, int* addr2=NULL, int val3=0) noexcept
{ return ::syscall(SYS_futex, addr1, op, val1, timeout, addr2, val3); }
int detail::linux_::futex_wait(int* pval, int expected) noexcept { return futex_invoke(pval, FUTEX_WAIT, expected); }
int detail::linux_::futex_timedwait(int* pval, int expected, uint64_t timeout_nsec) noexcept
{ struct ::timespec timeout{timeout_nsec}; return futex_invoke(pval, FUTEX_WAIT, expected, &timeout); }
int detail::linux_::futex_broadcast(int* pval) noexcept { return futex_invoke(pval, FUTEX_WAKE, INT_MAX); }
int detail::linux_::futex_signal(int* pval, int count=1) noexcept { return futex_invoke(pval, FUTEX_WAKE, count); }

class auto_reset_event { // no copy
    enum { wait_count_bits = 22u, post_count_one = 1<<22, post_count_mask = -post_count_one, wait_count_mask = post_count_one - 1u };
    atomic<unsigned int> m_state{0};
    bool priv_timed_wait(system_time_point const& abs_timeout); // futex_timedwait
    bool priv_timed_wait(system_duration const& dur); // futex_timedwait
    bool priv_timed_wait<TimePoint>(chrono_time_point<TimePoint> const& t);
    bool on_wait_timed_out();
public: ctor() noexcept;
    void post() noexcept; // cas-loop add `post_count_one` on m_state, call futex_signal to notify
    void wait() noexcept; // mark waiter count in m_state, wait by futex_wait
    bool try_wait() noexcept;
    bool timed_wait<Time>(Time timeout);
    bool timed_wait<Duration>(Duration const& duration);
    bool timed_wait<TimePoint>(TimePoint const& abs_time);
};

class manual_reset_event { // no copy
    enum { post_bit = 1u, wait_count_one = 2u, wait_count_mask = ~(unsigned)post_bit };
    atomic<unsigned int> m_state{0};
    bool priv_timed_wait(system_time_point const& abs_timeout); // futex_timedwait
    bool priv_timed_wait(system_duration const& dur); // futex_timedwait
    bool priv_timed_wait<TimePoint>(chrono_time_point<TimePoint> const& t);
public: ctor() noexcept;
    void set() noexcept; void reset() noexcept;
    void wait() noexcept; // mark waiter count in m_state, wait by futex_wait
    bool try_wait();
    bool timed_wait<Time>(Time timeout);
    bool timed_wait<Duration>(Duration const& duration);
    bool timed_wait<TimePoint>(TimePoint const& abs_time);
};
```

##### Semaphore based

```c++
class auto_reset_event { // no copy
    semaphore m_sem;
    atomic<int32_t> m_state{0};
public: ctor();
    void post() noexcept; // m_sem.post();
    void wait(); // m_sem.wait()
    bool try_wait(); // m_sem.try_wait();
    bool timed_wait<Time>(Time timeout); // m_sem.timed_wait(t);
    bool timed_wait<Duration>(Duration const& duration);
    bool timed_wait<TimePoint>(TimePoint const& abs_time);
};
```

##### Emulation

```c++
class auto_reset_event { // no copy
    mutex m_mutex; condition_variable m_cond, bool m_is_set{false};
public: ctor();
    void post();
    void wait();
    bool try_wait();
    bool timed_wait<Time>(Time timeout);
    bool timed_wait<Duration>(Duration const& duration);
    bool timed_wait<TimePoint>(TimePoint const& abs_time);
};

class manual_reset_event { // no copy
    mutex m_mutex; condition_variable m_cond, bool m_is_set{false};
public: ctor();
    void set();
    void reset();
    void wait();
    bool try_wait();
    bool timed_wait<Time>(Time timeout);
    bool timed_wait<Duration>(Duration const& duration);
    bool timed_wait<TimePoint>(TimePoint const& abs_time);
};
```

------
### Thread-Specific Storage

```c++
using detail::thread_specific_key = int;
using detail::at_thread_exit_callback = void(*)(void*);
void detail::add_thread_exit_callback(at_thread_exit_callback callback, void* context);
thread_specific_key detail::new_thread_specific_key(at_thread_exit_callback callback, bool cleanup_at_delete);
void detail::delete_thread_specific_key(thread_specific_key key) noexcept;
void* detail::get_thread_specific(thread_specific_key key) noexcept;
void detail::set_thread_specific(thread_specific_key key, void* p);

void detail::at_thread_exit_trampoline<Fun>(void* p) { auto fun = (Fun*)p; (*fun)(); delete fun; }
void detail::at_thread_exit<Fun>(Fun const& fun)
{ auto p = new Fun(fun); try { add_thread_exit_callback(&at_thread_exit_trampoline<Fun>, p); } catch(...) { delete p; throw; } }

class thread_specific_ptr<T> { // no copy
    const thread_specific_key m_key; const at_thread_exit_callback m_cleanup;
    static void default_cleanup(void* p) { delete (T*)p; }
public: using element_type T;
    explicit ctor() : m_key{new_thread_specific_key(&default_cleanup, true)}, m_cleanup{&default_cleanup}{}
    explicit ctor() : m_key{new_thread_specific_key((at_thread_exit_callback)cleanup, true)}, m_cleanup{cleanup}{}
    ~dtor() { delete_thread_specific_key(m_key); }
    T* get() const noexcept { return (T*)get_thread_specific(m_key); }
    T* operator->() const noexcept { return get(); }
    T& operator*() const noexcept { return *get(); }
    bool operator!() const noexcept { return !get(); }
    explicit operator bool() const noexcept { return !!(*this); }
    T* release() { T* const p = get(); if (p) set_thread_specific(m_key 0); return p; }
    void reset(T* new_value=nullptr) { T* const old = get();
        if (old != new_value) { set_thread_specific(m_key, new_value); if (old&&m_cleanup) m_cleanup(old); } }
};

class detail::tss_manager {
public:
    using thread_context_list_hook = intrusive::list_base_hook<tag<struct for_thread_context_list>, link_mode<safe_link>>;
    class thread_context : public thread_context_list_hook {
        struct at_exit_entry { at_thread_exit_callback callback; void* context; };
        struct notify_at_exit_entry { mutex* mtx; condition_variable* cond; };
        std::vector<void*> m_storage; std::vector<at_exit_entry> m_at_exit_functions; std::vector<notify_at_exit_entry> m_notify_at_exit;
    public:
        void* get_value(thread_specific_key key) const noexcept { if (key < m_storage.size()) return m_storage[key] return NULL; }
        void set_value(thread_specific_key key, void* value) { if (key >= m_storage.size()) m_storage.resize(key+1, nullptr); m_storage[key]=value; }
        void add_at_exit_entry(at_thread_exit_callback callback, void* context) { m_at_exit_functions.push_back({callback, context}); }
        void add_notify_at_exit_entry(mutex* mtx, condition_variable* cond) { m_notify_at_exit.push_back({mtx, cond}); }
    };
    using mutex_type = mutex;
    using thread_context_list = intrusive::list<thread_context, base_hook<thread_context_list_hook>, constant_time_size<false>>;
    struct cleanup_info { at_thread_exit_callback cleanup; bool cleanup_at_delete; };
    mutex_type mutex; thread_context_list m_thread_contexts;
    std::vector<cleanup_info> m_storage_cleanup; std::stack<thread_specific_key> m_freed_keys;
public: ctor();
    ~dtor() { while (!m_thread_contexts.empty()) destroy_thread_context(&m_thread_contexts.front()); }
    thread_context* create_thread_context() { auto p = new thread_context{}; lock_guard lock{m_mutex}; m_thread_contexts.push_back(*p); return p; }
    void destroy_thread_context(thread_context* p) noexcept {
        while (!p->m_at_exit_functions.empty()) {...} // 1. eval each cleanup
        while (!p->m_storage.empty()) {...} // 2. eval m_storage_cleanup[key].cleanup for each key
        for (auto it : p->m_notify_at_exit) {...} // 3. notify
        { lock_guard lock{m_mutex}; m_thread_contexts.erase(m_thread_contexts.iterator_to(*p)); }
        delete p;
    }
    thread_specific_key new_key(at_thread_exit_callback cleanup, bool cleanup_at_delete) {
        lock_guard lock{m_mutex};
        thread_specific_key key; // recycle key in m_freed_keys or allocate in storage_clenaup
        return key;
    }
    void delete_key(thread_specific_key key) {
        lock_guard lock{m_mutex};
        cleanup_info& info = m_storage_cleanup[key];
        if (info.cleanup_at_delete) {
            std::vector<void*> storage; auto cleanup = info.cleanup; info.cleanup = NULL;
            if (cleanup) // collect all m_thread_contexts[n].m_storage[key] into storage;
            m_freed_keys.push(key); // recycle
            lock.unlock(); for (auto it: storage) cleanup(it); // cleanup without holding lock
        }
    }
};
```

------
### Configuration

* `USE_PTHREAD` - force pthread

------
### Dependencies

#### Boost.Assert

* `<boost/assert.hpp>`
* `<boost/assert/source_location.hpp>`

#### Boost.Atomic - when no STD `<atomic>`

* `<boost/atomic/atomic.hpp>`
* `<boost/memory_order.hpp>`

#### Boost.Chrono - for `boost::chrono` support

* `<boost/chrono/duration.hpp>`, `<boost/chrono/time_point.hpp>`, `<boost/chrono/system_clocks.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/abi_prefix>`, `<boost/config/abi_suffix>`
* `<boost/cstdint.hpp>`
* `<boost/version.hpp>`

#### Boost.Core

* `<boost/core/enable_if.hpp>`
* `<boost/core/explicit_operator_bool.hpp>`
* `<boost/core/scoped_enum.hpp>`

#### Boost.DateTime - for `boost::date_time` support

* `<boost/date_time/posix_time/posix_time_types.hpp>`

#### Boost.Intrusive

* `<boost/intrusive/options.hpp>`
* `<boost/intrusive/list.hpp>`, `<boost/intrusive/list_hook.hpp>`

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
