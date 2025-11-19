# Boost.SmartPtr

* lib: `boost/libs/smart_ptr`
* repo: `boostorg/smart_ptr`
* commit: `5a0b5ea`, 2025-09-24

------
### Scoped Pointer/Array

```c++
class scoped_ptr<T> { // no copy, no self equality
    T* px;
public: using element_type = T;
    explicit ctor(T* p=0) noexcept :px{p}{}
    ~dtor() noexcept { checked_delete(px); }
    void reset(T* p=0) noexcept { self(p).swap(*this); }
    T& operator*() const noexcept { return *px; }
    T* operator->() const noexcept { return px; }
    T* get() const noexcept { return px; }
    explicit operator bool() const noexcept { return px!=0; }
    void swap(self& b) noexcept { T* tmp=b.px; b.px=px; px=tmp; }
};
bool operator==(scoped_ptr<T> const& p, nullptr_t) { return p.get() == 0; } // and !=, and inversed
void swap<T>(scoped_ptr<T>& a, scoped_ptr<T>& b) noexcept { a.swap(b); }
T* get_pointer<T>(scoped_ptr<T> const& p) noexcept { return p.get(); }

class scoped_array<T> { // no copy, no self equality
    T* px;
public: using element_type = T;
    explicit ctor(T* p=0) noexcept :px{p}{}
    ~dtor() noexcept { checked_array_delete(px); }
    void reset(T* p=0) noexcept { self(p).swap(*this); }
    T& operator[](ptrdiff_t i) const noexcept { return px[i]; }
    T* get() const noexcept { return px; }
    explicit operator bool() const noexcept { return px!=0; }
    void swap(self& b) noexcept { T* tmp=b.px; b.px=px; px=tmp; }
};
bool operator==(scoped_array<T> const& p, nullptr_t) { return p.get() == 0; } // and !=, and inversed
void swap<T>(scoped_array<T>& a, scoped_array<T>& b) noexcept { a.swap(b); }
```

------
### Shared Ownership

```c++
class bad_weak_ptr : public std::exception { char const* what() const noexcept override; };


```

------
### Pointer Cast & Traits

```c++
// all 4 casts
T* static_pointer_cast<T,U>(U* ptr) noexcept { return static_cast<T*>(ptr); }
using std::static_pointer_cast; // for std::shared_ptr
std::unique_ptr<T> static_pointer_cast<T,U>(std::unique_ptr<U>&& r) noexcept
{ return { static_cast<std::unique_ptr<T>::element_type*>(r.release()) }; }

struct pointer_to_other<T,U>;
struct pointer_to_other<Sp<T>,U> { using type = Sp<U>; };
struct pointer_to_other<Sp<T,T2>,U> { using type = Sp<U,T2>; };
struct pointer_to_other<Sp<T,T2,T3>,U> { using type = Sp<U,T2,T3>; };
struct pointer_to_other<T*,U> { using type = U*; };
```

------
### Implementation Details

```c++
using detail::sp_typeinfo_ == std::type_info; // or boost::core::typeinfo when no `typeid`

class detail::sp_nothrow_tag{};
class detail::sp_inplace_tag<D>{};
class detail::sp_reference_wrapper<T>
{ T* t_; explicit ctor(T& t): t_{addressof(t)}{} void operator()<Y>(Y*p) noexcept { (*t_)(p); } };
struct detail::sp_convert_reference<D> { using type = D; };
struct detail::sp_convert_reference<D&> { using type = sp_reference_wrapper<D>; };
size_t detail::sp_hash_pointer<T>(T* p) noexcept { uintptr_t v = uintptr_t(p); return size_t(v+(v>>3)); }
struct detail::sp_convertible<Y,T> { enum {value = ...}; };
struct detail::sp_empty{};
struct detail::sp_enable_if_convertible<Y,T>;
struct detail::sp_is_bounded_array<T>;
struct detail::sp_is_unbounded_array<T>;
struct detail::sp_type_identity<T> { using type = T; };
struct detail::sp_type_with_alignment<a> { struct alignas(a) type { unsigned char padding[a]; }; };

union detail::freeblock<size,align> { using aligner_type = sp_type_with_alignment<align>;
    aligner_type aligner; char bytes[size]; freeblock* next;
};
struct detail::allocator_impl<size,align> { using block = freeblock<size,align>;
    enum { items_per_page = 512/size };
    static lightweight_mutex* mutex_init = mutex();
    static lightweight_mutex& mutex() { static lightweight_mutex m; return m; }
    static block *free{nullptr}, *page{nullptr}; static unsigned last = items_per_page;
    static void* alloc(); static void* alloc(size_t n);
    static void dealloc(void* pv); static void dealloc(void* pv, size_t n);
};
struct detail::quick_allocator<T> : allocator_impl<sizeof(T), alignof(T)> {};

class detail::atomic_count { // no-threading version
    long value_; // no copy
public: explicit ctor(long v) : value_{v}{}
    long operator++() { return ++value_; }
    long operator--() { return --value_; }
    operator long() const { return value_; }
};
class detail::atomic_count { // gcc intrinsic version
    mutable _Atomic_word value_; // no copy
public: explicit ctor(long v) : value_{v}{}
    long operator++() { return __exchange_and_add(&value_, +1) + 1; }
    long operator--() { return __exchange_and_add(&value_, -1) - 1; }
    operator long() const { return __exchange_and_add(&value_, 0); }
};
class detail::atomic_count { // gcc atomic intrinsic version
    int_least32_t value_; // no copy
public: explicit ctor(long v) : value_{v}{}
    long operator++() { return __atomic_add_fetch(&value_, +1, __ATOMIC_ACQ_REL); }
    long operator--() { return __atomic_add_fetch(&value_, -1, __ATOMIC_ACQ_REL); }
    operator long() const { return __atomic_load_n( &value_, __ATOMIC_ACQUIRE); }
};
class detail::atomic_count { // sync intrinsic version
    mutable int_least32_t value_; // no copy
public: explicit ctor(long v) : value_{v}{}
    long operator++() { return __sync_add_and_fetch(&value_, +1); }
    long operator--() { return __sync_add_and_fetch(&value_, -1); }
    operator long() const { return __sync_fetch_and_add( &value_, 0); }
};
class detail::atomic_count { // std version
    std::atomic_int_least32_t value_; // no copy
public: explicit ctor(long v) : value_{v}{}
    long operator++() { return value_.fetch_add(1, memory_order_acq_rel) + 1; }
    long operator--() { return value_.fetch_sub(1, memory_order_acq_rel) -1 1; }
    operator long() const { return value_.load(memory_order_acquire); }
};
class detail::atomic_count { // win32 version
    long value_; // no copy
public: explicit ctor(long v) : value_{v}{}
    long operator++() { return InterlockedIncrement(&value_); }
    long operator--() { return InterlockedDecrement(&value_); }
    operator long() const { return (long const volatile&)value_; }
};
class detail::atomic_count { // win32 version
    long value_; mutable pthread_mutex_t mutex_; // no copy
    class scoped_lock { pthread_mutex_t& m_;
    public: ctor(pthread_mutex_t& m): m_{m} { pthread_mutex_lock(&m_); } ~dtor() { pthread_mutex_unlock(&m_); }
    };
public: explicit ctor(long v) : value_{v}{ pthread_mutex_init(&mutex_, 0); } ~dtor(){ pthread_mutex_destroy(&mutex_); }
    long operator++() { scoped_lock lock{mutex_}; return ++value_; }
    long operator--() { scoped_lock lock{mutex_}; return --value_; }
    operator long() const { scoped_lock lock{mutex_}; return value_; }
};
class detail::atomic_count { // spinlock version
    long value_; // no copy
public: explicit ctor(long v) : value_{v}{}
    long operator++() { spinlock_pool<0>::scoped_lock lock{&value_}; return ++value_; }
    long operator--() { spinlock_pool<0>::scoped_lock lock{&value_}; return --value_; }
    operator long() const { spinlock_pool<0>::scoped_lock lock{&value_}; return value_; }
};

struct detail::spinlock { // no-threading version
    bool locked_;
    bool try_lock() { if (locked_) return false; locked_=true; return true; }
    void lock() { locked_ = true; } // 
    void unlock() { locked_ = false; }
    class scoped_lock { spinlock& sp_; // no copy
    public: explicit ctor(spinlock& sp): sp_{sp} {sp_.lock();}  ~dtor(){sp_.unlock();}
    };
};
void detail::yield(unsigned k) { if (k%1) sp_thread_sleep() else sp_thread_pause(); }
struct detail::spinlock { // atomic version (similar: sync, gcc atomic intrinsic)
    std::atomic_flag v_;
    bool try_lock() { return !v_.test_and_set(std::memory_order_acquire); }
    void lock() { for (unsigned k=0; !try_lock(); ++k) yield(k); }
    void unlock() { v_.clear(std::memory_order_release); }
    class scoped_lock; // same
};
struct detail::spinlock { // PThread version
    pthread_mutex_t v_;
    bool try_lock() { return pthread_mutex_trylock(&v_) == 0; }
    void lock() { pthread_mutex_lock(&v_); }
    void unlock() { pthread_mutex_unlock(&v_); }
    class scoped_lock; // same
};

class detail::spinlock_pool<m> {
    static spinlock pool_[41];
public:
    static spinlock& spinlock_for(void const* pv) { return pool_[(size_t)pv % 41]; }
    class scoped_lock { spinlock& sp_; // no copy
    public: explicit ctor(spinlock& sp): sp_{sp} {sp_.lock();}  ~dtor(){sp_.unlock();}
    };
};

class detail::lightweight_mutex { std::mutex m_; // std::mutex version
public: ctor(){} // nocopy
    class scoped_lock { std::mutex& m_; // no copy
    public: explicit ctor(lightweight_mutex& m): m_{m.m_} {m_.lock();}  ~dtor(){m_.unlock();}
    };
};
class detail::lightweight_mutex { pthread_mutex_t m_; // pthread version
public: ctor(){ pthread_mutex_init(&m_, 0); } ~dtor(){ pthread_mutex_destroy(&m_); }
    class scoped_lock { pthread_mutex_t& m_; // no copy
    public: explicit ctor(lightweight_mutex& m): m_{m.m_} {pthread_mutex_lock(&m_);}  ~dtor(){pthread_mutex_unlock(&m_);}
    };
};
class detail::lightweight_mutex { RTL_CRITICAL_SECTION cs_; // win32 version
public: ctor(){ InitializeCriticalSection(&cs_); } ~dtor(){ DeleteCriticalSection(&cs_); }
    class scoped_lock { lightweight_mutex& m_; // no copy
    public: explicit ctor(lightweight_mutex& m): m_{m} {EnterCriticalSection(&m_.cs_);}  ~dtor(){LeaveCriticalSection(&m_.cs_);}
    };
};

using lw_thread_t = HANDLE; // pthread_t for pthread
int detail::lw_thread_create_(lw_thread_t* th, pthread_attr_t const*, void* (*)(void*), void* arg); // pthread
int detail::lw_thread_create_(lw_thread_t* th, void const*, unsigned (__stdcall *)(void*), void* arg); // win32
void detail::lw_thread_join(lw_thread_t th);
struct detail::lw_abstract_thread { virtual ~dtor(){}  virtual void run()=0; };
void* detail::lw_thread_routine(void* pv) { std::unique_ptr<lw_abstract_thread> pt{(lw_abstract_thread*)pv}; pt->run(); return 0; }
class detail::lw_thread_impl<F> : public lw_abstract_thread { F f_;
public: explicit ctor(F f): f_{f}{}  void run() { f_(); }
};
int detail::lw_thread_create<F>(lw_thread_t& th, F f){
    std::unique_ptr<lw_abstract_thread> p{new lw_thread_impl<F>{f}};
    int r = lw_thread_create_(&th, 0, lw_thread_routine, p.get());
    if (r==0) p.release();
    return r;
}

void detail::atomic_increment(int* pw);
int detail::atomic_exchange_and_add(int* pw, int dv);
int detail::atomic_decrement(int* pw);
int detail::atomic_conditional_increment(int* pw);
class detail::sp_counted_base { // no copy
    int_least32_t use_count_{1}, weak_count_{1};
public: ctor() noexcept {} virtual ~dtor(){}
    virtual void dispose() noexcept=0;
    virtual void destroy() noexcept { delete this; }
    virtual void* get_deleter(sp_typeinfo_ const& ti) noexcept=0;
    virtual void* get_local_deleter(sp_typeinfo_ const& ti) noexcept=0;
    virtual void* get_untyped_deleter() noexcept=0;
    void add_ref_copy() noexcept { ++use_count_; } // atomic_increment, or InterLocked intrinsics on Win32
    bool add_ref_lock() noexcept { if (use_count_==0) return false; ++use_count_; return true; } // atomic_conditional_increment, or InterLocked intrinsics on Win32
    void release() noexcept { if (--use_count_==0) { dispose(); weak_release(); } } // atomic_decrement or atomic_exchange_and_add, or InterLocked intrinsics on Win32
    void weak_add_ref() noexcept { ++weak_count_; } // atomic_increment, or InterLocked intrinsics on Win32
    void weak_release() noexcept { if (--weak_count_==0) destroy(); } // atomic_decrement or atomic_exchange_and_add, or InterLocked intrinsics on Win32
    long use_count() const noexcept { return use_count_; } // volatile access for threading cases
};

class detail::sp_counted_impl_p<X> : public sp_counted_base { // ref-counter and data ptr, no copy
    X* px_;
public: explicit ctor(X* px): px_{px}{}
    void dispose() noexcept override { checked_delete(px_); }
    // get_deleter, get_local_deleter, get_untyped_deleter all return nullptr
    void* operator new(size_t) { return quick_allocator<self>::alloc(); } // when `USE_QUICK_ALLOCATOR`
    void operator delete(void* p) { return quick_allocator<self>::dealloc(p); } // when `USE_QUICK_ALLOCATOR`
};
class detail::sp_counted_impl_pd<P,D> : public sp_counted_base { // ref-counter, deleter and data ptr, no copy
    P ptr; D del{};
public: ctor(P p, D& d): ptr{p}, del{(D&&)d}{} ctor(P p): ptr{p}{}
    void dispose() noexcept override { del(ptr); }
    void* get_deleter(sp_typeinfo_ const& ti) noexcept override { return ti==typeid(D) ? addressof(del) : nullptr; }
    void* get_local_deleter(sp_typeinfo_ const& ti) noexcept override { return ti==typeid(D) ? get_local_deleter(addressof(del)) : nullptr; }
    void* get_untyped_deleter() noexcept override { return addressof(del); }
    void* operator new(size_t) { return quick_allocator<self>::alloc(); } // when `USE_QUICK_ALLOCATOR`
    void operator delete(void* p) { return quick_allocator<self>::dealloc(p); } // when `USE_QUICK_ALLOCATOR`
};
class detail::sp_counted_impl_pda<P,D,A> : public sp_counted_base { // ref-counter, deleter and data ptr, no copy
    P p_; D d_; A a_;
public: ctor(P p, D& d, A a): p_{p}, d_{(D&&)d}, a_{a}{} ctor(P p, A a): p_{p}, a_{a}{}
    void dispose() noexcept override { d_(p_); }
    void destroy() noexcept override { using A2 = std::allocator_traits<A>::rebind_alloc<self>; A2 a2{a_}; this->~dtor(); a2.deallocate(this, 1); }
    void* get_deleter(sp_typeinfo_ const& ti) noexcept override { return ti==typeid(D) ? addressof(del) : nullptr; }
    void* get_local_deleter(sp_typeinfo_ const& ti) noexcept override { return ti==typeid(D) ? get_local_deleter(addressof(del)) : nullptr; }
    void* get_untyped_deleter() noexcept override { return addressof(del); }
};

class detail::shared_count {
    sp_counted_base* pi_{nullptr};
public: constexpr ctor() noexcept{} ~dtor() { if (pi_) pi_->release(); }
    ctor(self const& r) noexcept: pi_{r.pi_} { if (pi_) pi_->add_ref_copy(); }
    ctor(self && r) noexcept: pi_{r.pi_} { r.pi_ = nullptr; }
    explicit ctor(weak_count const& r) :pi_{r.pi}{ if (pi_==nullptr||!pi_->add_ref_lock()) boost::throw_exception(bad_weak_ptr{}); }
    ctor(weak_count const& r, sp_nothrow_tag) noexcept :pi_{r.pi}{ if (pi_&&!pi_->add_ref_lock()) pi_=nullptr; }
    self& operator=(self const& r) noexcept {
        if (auto tmp = r.pi_; tmp!=pi_) { if (tmp) tmp->add_ref_copy(); if (pi_) pi_->release(); pi_=tmp; }
        return *this;
    }
    void swap(self& r) noexcept { auto tmp=r.pi_; r.pi_=pi_; pi_ = tmp; }

    constexpr explicit ctor(sp_counted_base* pi) noexcept: pi_{pi}{}
    explicit ctor<Y>(Y* p) { try { pi_=new sp_counted_impl_p{p}; } catch(...) { checked_delete(p); throw; } }
    ctor<P,D>(P p, D d) { try { pi_=new sp_counted_impl_pd{p,d}; } catch(...) { d(p); throw; } }
    ctor<P,D>(P p, sp_inplace_tag<D>) { try { pi_=new sp_counted_impl_pd<P,D>{p}; } catch(...) { D::operator_fn(p); throw; } }
    ctor<P,D,A>(P p, D d, A a) {
        using impl_type = sp_counted_impl_pda<P,D,A>;
        using A2 = std::allocator_traits<A>::rebind_alloc<impl_type>; A2 a2{a};
        try { pi_= a2.allocate(1); ::new(pi_) impl_type{p,d,a}; }
        catch(...) { d(p); if(pi_) a2.deallocate((impl_type*)pi_, 1); throw; }
    }
    ctor<P,D,A>(P p, sp_inplace_tag<D>, A a) {
        using impl_type = sp_counted_impl_pda<P,D,A>;
        using A2 = std::allocator_traits<A>::rebind_alloc<impl_type>; A2 a2{a};
        try { pi_= a2.allocate(1); ::new(pi_) impl_type{p,a}; }
        catch(...) { D::operator_fn(p); if(pi_) a2.deallocate((impl_type*)pi_, 1); throw; }
    }
    explicit ctor<Y,D>(std::unique_ptr<Y,D>& r) { // and support `movelib::unique_ptr` from Boost.Move
        using D2 = sp_convert_reference<D>::type; D2 d2{(D&&)r.get_deleter()};
        pi_ = new sp_counted_impl_pd{r.get(),d2}; r.release();
    }

    long use_count() const noexcept { return pi_ ? pi_->use_count() : 0; }
    bool unique() const noexcept { return use_count() == 1; }
    bool empty() const noexcept { return pi_ == nullptr; }
    bool operator==(shared_count const& r) const noexcept { return pi_ == r.pi_; }
    bool operator==(weak_count const& r) const noexcept { return pi_ == r.pi_; }
    bool operator<(shared_count const& r) const noexcept { return std::less<>{}(pi_, r.pi_); }
    bool operator<(weak_count const& r) const noexcept { return std::less<>{}(pi_, r.pi_); }
    void* get_deleter(sp_typeinfo_ const& ti) const noexcept { return pi_ ? pi_->get_deleter(ti) : nullptr; }
    void* get_local_deleter(sp_typeinfo_ const& ti) const noexcept { return pi_ ? pi_->get_local_deleter(ti) : nullptr; }
    void* get_untyped_deleter() const noexcept { return pi_ ? pi_->get_untyped_deleter() : nullptr; }
    size_t hash_value() const noexcept { return sp_hash_pointer(pi_); }
};

class weak_count {
    sp_counted_base* pi_{nullptr};
public: constexpr ctor() noexcept{} ~dtor() { if (pi_) pi_->weak_release(); }
    ctor({weak|shared}_count const& r) noexcept: pi_{r.pi_} { if (pi_) pi_->weak_add_ref(); }
    ctor(self && r) noexcept: pi_{r.pi_} { r.pi_ = nullptr; }
    self& operator=({weak|shared}_count const& r) noexcept {
        if (auto tmp = r.pi_; tmp!=pi_) { if (tmp) tmp->weak_add_ref(); if (pi_) pi_->weak_release(); pi_=tmp; }
        return *this;
    }
    void swap(self& r) noexcept { auto tmp=r.pi_; r.pi_=pi_; pi_ = tmp; }

    long use_count() const noexcept { return pi_ ? pi_->use_count() : 0; }
    bool empty() const noexcept { return pi_ == nullptr; }
    bool operator==({weak|shared}_count const& r) const noexcept { return pi_ == r.pi_; }
    bool operator<({weak|shared}_count const& r) const noexcept { return std::less<>{}(pi_, r.pi_); }
    size_t hash_value() const noexcept { return sp_hash_pointer(pi_); }
};

class detail::local_counted_base { // no copy-assign
    enum count_type { min_=0, initial_=1, max_=2147483647 } local_use_count_={initial_};
public: constexpr ctor()noexcept=default; constexpr ctor(self const&) noexcept{} virtual ~dtor()=default;
    virtual void local_cb_destroy() noexcept=0;
    virtual shared_count local_cb_get_shared_count() const noexcept=0;
    void add_ref() noexcept { local_use_count_=static_cast<count_type>(local_use_count_+1); }
    void release() noexcept { local_use_count_=static_cast<count_type>(local_use_count_-1); if (local_use_count_==0) local_cb_destroy(); }
    long local_use_count() const noexcept { return local_use_count_; }
};
class detail::local_counted_impl : local_counted_base { // both local and shared counter
    shared_count pn_;
public: explicit ctor(shared_count {const&|&&} pn) noexcept :pn_{<std::move>(pn)} {}
    void local_cb_destroy() noexcept override { delete this; }
    shared_count local_cb_get_shared_count() const noexcept override { return pn_; }
};
struct detail::local_counted_impl_em : public local_counted_base { // both local and shared counter
    shared_count pn_;
    void local_cb_destroy() noexcept override { shared_count{}.swap(pn_); } // just clear shared counter
    shared_count local_cb_get_shared_count() const noexcept override { return pn_; }
};

class detail::local_sp_deleter<D> : public local_counted_impl_em { // ref-counter and deleter
    D d_{};
public: ctor()=default;
    explicit ctor(D const& d) noexcept: d_{d}{}  explicit ctor(D&& d) noexcept: d_{std::move(d)}{}
    D& deleter() noexcept { return d_; }
    void operator()<Y> (Y* p) noexcept { d_(p); } void operator()(std::nullptr_t p) noexcept { d_(p); }
};
class detail::local_sp_deleter<void>{};
D* detail::get_local_deleter<D>(D*) noexcept { return nullptr; }
D* detail::get_local_deleter<D>(local_sp_deleter<D>* p) noexcept { return &p->deleter(); }
void* detail::get_local_deleter(local_sp_deleter<void>*) noexcept { return nullptr; }
```

------
### Configurations

* `BOOST_SP_NO_SYNC_INTRINSICS` & `BOOST_SP_NO_SYNC`
* `BOOST_SP_NO_ATOMIC_ACCESS`
* Atomic counter config:
    * `BOOST_AC_DISABLE_THREADS` & `BOOST_SP_DISALBE_THREADS` : force no-threading
    * `BOOST_AC_USE_STD_ATOMIC` & `BOOST_SP_USE_STD_ATOMIC` : force use STD lib
    * Otherwise: gcc intrinsics -> STD lib -> sync intrinsics -> gcc-x86 and other specific arch -> win32 -> glibc -> no-threading/spinlock
* `BOOST_SP_USE_QUICK_ALLOCATOR`: use quick allocator instead of global `new/delete`
* `BOOST_SP_REPORT_IMPLEMENTATION`, `BOOST_SP_NO_OBSOLETE_MESSAGE`

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>` - used for debugging

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`
* `<boost/config/header_deprecated.hpp>`
* `<boost/config/pragma_message.hpp>`
* `<boost/cstdint.hpp>`

#### Boost.Core

* `<boost/core/addressof.hpp>`
* `<boost/core/allocator_access.hpp>`
* `<boost/core/alloc_construct.hpp>`
* `<boost/core/checked_delete.hpp>`
* `<boost/core/default_allocator.hpp>`
* `<boost/core/empty_value.hpp>`
* `<boost/core/first_scalar.hpp>`
* `<boost/core/noinit_adaptor.hpp>`
* `<boost/core/pointer_traits.hpp>`
* `<boost/core/typeinfo.hpp>`
* `<boost/core/yield_primitives.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

------
### Standard Facilities

Library: `shared_ptr`, `weak_ptr`, `static_pointer_cast` family (C++11)
