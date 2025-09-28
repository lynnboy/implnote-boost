# Boost.Compat

* lib: `boost/libs/compat`
* repo: `boostorg/compat`
* commit: `a5a56ee`, 2025-09-05

------
#### `<type_traits>`

* Header `<boost/compat/type_traits.hpp>`

```c++
using add_lvalue_reference_t<T> = std::add_lvalue_reference<T>::type;
using add_rvalue_reference_t<T> = std::add_rvalue_reference<T>::type;
using add_const_t<T> = std::add_const<T>::type;
using remove_const_t<T> = std::remove_const<T>::type;
using remove_cv_t<T> = std::remove_cv<T>::type;
using remove_reference_t<T> = std::remove_reference<T>::type;
using remove_cvref_t<T> = remove_cv_t<remove_reference_t<T>>;
using remove_pointer_t<T> = std::remove_pointer<T>::type;
using decay_t<T> = std::decay<T>::type;
using enable_if_t<b,T=void> = std::enable_if<b,T>::type;
using conditional_t<b,T,F> = std::conditional<b,T,F>::type;
using void_t<...T> = void;
struct type_identity<T> { using type = T; };
```

* `add_lvalue_reference_t`, and similar, alias templates (C++14)
* `void_t` (C++17)
* `type_identity` (C++20)

------
#### `<functional>`

* Header `<boost/compat/mem_fn.hpp>`

```c++
#define BOOST_COMPAT_RETURNS(...) noexcept(noexcept(__VA_ARGS__)) ->decltype(__VA_ARGS__) { return __VA_ARGS__; }

using detail::is_same_or_base<T,U> = is_base_of<remove_cvref_t<T>, remove_cvref_t<U>>;
struct detail::is_reference_wrapper<T> : std::false_type {};
struct detail::is_reference_wrapper<std::reference_wrapper<remove_cvref_t<T>>> : std::true_type {};
struct detail::_mfn<M,T> { // pointer to member function
    M T::* pm_;
    constexpr auto operator() <U,...A> (U&& u, A&&... a) const requires is_same_or_base<T,U>::value
        BOOST_COMPAT_RETURNS( (std::forward<U>(u).*pm_)(std::forward<A>(a)...) )
    constexpr auto operator() <U,...A> (U&& u, A&&... a) const requires !is_same_or_base<T,U>::value && is_reference_wrapper<U>::value
        BOOST_COMPAT_RETURNS( (u.get().*pm_)(std::forward<A>(a)...) )
    constexpr auto operator() <U,...A> (U&& u, A&&... a) const requires !is_same_or_base<T,U>::value && !is_reference_wrapper<U>::value
        BOOST_COMPAT_RETURNS( ((*std::forward<U>(u)).*pm_)(std::forward<A>(a)...) )
};
struct detail::_md<M,T> { // pointer to data member
    M T::* pm_;
    constexpr auto operator() <U> (U&& u) const requires is_same_or_base<T,U>::value
        BOOST_COMPAT_RETURNS( std::forward<U>(u).*pm_ )
    constexpr auto operator() <U> (U&& u) const requires !is_same_or_base<T,U>::value && is_reference_wrapper<U>::value
        BOOST_COMPAT_RETURNS( u.get().*pm_ )
    constexpr auto operator() <U> (U&& u) const requires !is_same_or_base<T,U>::value && !is_reference_wrapper<U>::value
        BOOST_COMPAT_RETURNS( (*std::forward<U>(u)).*pm_ )
};
constexpr auto mem_fn <M,T> (M T::* pm) noexcept -> _mfn<M,T> requires is_function_v<M> { return {pm}; }
constexpr auto mem_fn <M,T> (M T::* pm) noexcept -> _dm<M,T> requires !is_function_v<M> { return {pm}; }
```

* `mem_fn` (C++11)

------
* Header `<boost/compat/bind_back.hpp>` and `<boost/compat/bind_front.hpp>`

```c++
static constexpr auto detail::invoke_bind_back_<F,A,...B,...i> (F&& f, A&& a, index_sequence<i...>, B&&...b)
    BOOST_COMPAT_RETURNS( invoke(std::forward<F>(f), std::forward<B>(b)..., std::get<i>(std::forward<A>(a))...) )
static constexpr auto detail::invoke_bind_front_<F,A,...B,...i> (F&& f, A&& a, index_sequence<i...>, B&&...b)
    BOOST_COMPAT_RETURNS( invoke(std::forward<F>(f), std::get<i>(std::forward<A>(a))..., std::forward<B>(b)...) )

class detail::bind_{back|front}_<F,...A> {
    F f_; std::tuple<A...> a_; // bound args
public: constexpr ctor<F2,...A2> (F2&& f2, A2&&... a2); // init ctor
    constexpr auto operator() <...B> (B&&...b) [const]{&|&&}
        BOOST_COMPAT_RETURNS( invoke_bind_{back|front}_(<std::move>(f_), <std::move>(a_), make_index_sequence<sizeof...(A)>(), std::forward<B>(b)...) )
};

constexpr auto bind_{back|front} <F,...A> (F&& f, A&&... a) -> bind_back_<decay_t<F>,decay_t<A>...>
{ return { std::forward<F>(f), std::forward<A>(a)... }; }
```

* `bind_back(f,...args)` (C++23), `bind_front(f, ...args)` (C++20).

------
* Header `<boost/compat/function_ref.hpp>`

```c++
struct nontype_t<auto v> { explicit nontype_t() = default; };
union detail::thunk_storage<noEx> { void* pobj_; void(*pfn_)() noexcept(noEx); };

// invokes
struct detail::invoke_function_holder<noEx,Fp,R,...Args> { // s.pfn_ (args...)
    static R invoke_function(thunk_storage<noEx> s, Args&&...args) noexcept(noEx)
    { return invoke_r<R>((Fp)(s.pfn_), std::forward<Args>(args)...); }
};
struct detail::invoke_object_holder<const_,noEx,F,R,...Args> { // (*s.pobj_) (args...)
    static R invoke_object(thunk_storage<noEx> s, Args&&...args) noexcept(noEx)
    {   using T = remove_reference_t<F>; using cv_T = conditional_t<const_, add_const_t<T>, T>;
        return invoke_r<R>(*(cv_T*)(s.pobj_), std::forward<Args>(args)...); }
};
struct detail::invoke_mem_fn_holder<auto f,const_,noEx,R,...Args> { // f(args...)
    static R invoke_mem_fn(thunk_storage<noEx> s, Args&&...args) noexcept(noEx)
    { return invoke_r<R>(f, std::forward<Args>(args)...); }
};
struct detail::invoke_target_mem_fn_holder<auto f,U,const_,noEx,R,...Args> { // (*(s.pobj_).*f)(args...)
    static R invoke_mem_fn(thunk_storage<noEx> s, Args&&...args) noexcept(noEx)
    {   using T = remove_reference_t<U>; using cv_T = conditional_t<const_, add_const_t<T>, T>;
        return invoke_r<R>(f, *(cv_T*)(s.pobj_), std::forward<Args>(args)...); }
};
struct detail::invoke_ptr_mem_fn_holder<auto f,T,const_,noEx,R,...Args> { // ((s.pobj_)->*f)(args...)
    static R invoke_mem_fn(thunk_storage<noEx> s, Args&&...args) noexcept(noEx)
    {   using cv_T = conditional_t<const_, add_const_t<T>, T>;
        return invoke_r<R>(f, (cv_T*)(s.pobj_), std::forward<Args>(args)...); }
};
struct detail::function_ref_base<const_,noEx,R,...Args> {
private: thunk_storage<noEx> thunk_={nullptr}; // bound arguments
    R (*invoke_)(thunk_storage<noEx>, Args&&...) noexcept(noEx) = {nullptr}; // invoker
public: struct fp_tag{}; struct obj_tag{}; struct mem_fn_tag{};
    ctor<F> (fp_tag, F* fn) noexcept    : thunk_{.pfn_=(void(*)())(fn)},
        invoke_(&invoke_function_holder<noEx,F*,R,Args...>::invoke_function) {}
    ctor<F> (obj_tag, F&& fn) noexcept  : thunk_{.pobj_=(void*)std::addressof(fn)},
        invoke_(&invoke_object_holder<const_,noEx,F,R,Args...>::invoke_object) {}
    ctor<auto f,F=decltype(f)> (mem_fn_tag, nontype_t<f>)            : thunk_{.pobj_=nullptr},
        invoke_(&invoke_mem_fn_holder<F{f},const_,noEx,R,Args...>::invoke_mem_fn) {}
    ctor<auto f,U,F=decltype(f)> (mem_fn_tag, nontype_t<f>, U&& obj) : thunk_{.pobj_=(void*)std::addressof(obj)},
        invoke_(&invoke_target_mem_fn_holder<F{f},U,const_,noEx,R,Args...>::invoke_mem_fn) {}
    ctor<auto f,T,F=decltype(f)> (mem_fn_tag, nontype_t<f>, T* obj)  : thunk_{.pobj_=(void*)obj},
        invoke_(&invoke_ptr_mem_fn_holder<F{f},T,const_,noEx,R,Args...>::invoke_mem_fn) {}
    // copy-ctor, copy-assign =default
    R operator()(Args...args) [const] noexcept(noEx) { return invoke_(thunk_, std::forward<Args>(args)...); } // dispatch
};

template<class R, class...Args> struct function_ref; // primary
// specializations for function signatures
struct function_ref<R(Args...) [const][noexcept]>
    : public function_ref_base<{false|true},{false|true},R,Args...> { //
    ctor <F>                         (F* fn) noexcept requires (is_function_v<F> && is_invocable_r_v<R,F,Args...>)
        : base(fp_tag{}, fn) {}
    ctor <F,T=remove_reference_t<F>> (F&& fn) noexcept
        requires (!is_same_v<remove_cvref_t<F>, function_ref> && !is_member_pointer_v<T> && is_invocable_r_v<R,T&,Args...>)
        : base(obj_tag{}, fn) {}
    ctor <auto f,F=decltype(f)>     (nontype_t<f> x) noexcept requires (is_invocable_r_v<R,F,Args...>)
        : base(mem_fn_tag{}, x) {}
    ctor <auto f,U,T=remove_reference_t<U>,F=decltype(f)> (nontype_t<f> x, U&& obj) noexcept
        requires (!is_rvalue_reference_v<U&&> && is_invocable_r_v<R,F,T&,Args...>)
        : base(mem_fn_tag{}, x, std::forward<U>(obj)) {}
    ctor <auto f,T,F=decltype(f)>   (nontype_t<f> x, T* obj) noexcept
        requires (!is_rvalue_reference_v<U&&> && is_invocable_r_v<R,F,T&,Args...>)
        : base(mem_fn_tag{}, x, obj) {}

    // copy-ctor, copy-assign = default; asign to obj/mptr =delete
};
```

* `function_ref` (C++26). Trivally copyable reference to a callable.

-----
* Header `<boost/compat/move_only_function.hpp>`

```c++
struct in_place_type_t<T> { explicit in_place_type_t() = default; };

union detail::pointers { void* pobj_; void (*pfn_)(); };
struct detail::storage {
    struct delegate { void(storage::*pmfn_)(); storage* pobj_; };
    union {
        void* pobj_; void (*pfn_);
        alignas(delegate) unsigned char buf_[sizeof(delegate)]; // raw bytes
    };
    constexpr static bool use_sbo <T> () noexcept // T can put into a `delegate`, and can nothrow-move around
    { return sizeof(T) <= sizeof(storage) && alignof(T) <= alignof(storage) && is_nothrow_move_constructible_v<T>; }
    void* addr() noexcept { return buf_; }
};

// traits
struct detail::is_polymorphic_function<T> : false_type;
struct detail::is_polymorphic_function<move_only_function<R(Args...)[const][&|&&][noexcept]>> : true_type {};
using detail::is_move_only_function<T> = is_polymorphic_function<T>;
struct detail::is_in_place_type_t<T> : false_type;
struct detail::is_in_place_type_t<in_place_type_t<T>> : true_type {};
struct detail::nothrow_init<T,...Args> {value = storage::use_sbo<T>() && is_nothrow_constructible_v<T,Args...>; }
enum class detail::ref_quals {none,lvalue,rvalue};
struct detail::is_callable_from<refq,const_,noex,VT,R,...Args> {
    using cv_VT = const_ ? const VT : VT;
    using cv_ref_VT     = refq == ref_quals::none ? cv_VT   : refq == ref_quals::rvalue ? cv_VT && : cv_VT &;
    using inv_quals_VT  = refq == ref_quals::none ? cv_VT & : refq == ref_quals::rvalue ? cv_VT && : cv_VT &;
    constexpr static bool value = noex ? // R(cv_ref_VT)(Args...) and R(inv_quals_VT)(Args...)
        is_nothrow_invocable_r_v<R,cv_ref_VT,Args...> && is_nothrow_invocable_r_v<R,inv_quals_VT,Args...> :
        is_invocable_r_v<R,cv_ref_VT,Args...> && is_invocable_r_v<R,inv_quals_VT,Args...>;
};
nullptr_t detail::get_first_arg();
T&& detail::get_first_arg<T,...CArgs> (T&& t, CArgs&&...) { return std::forward<T>(t); }
bool detail::is_nullary_arg<...Ts> (Ts&&...) { return false; }
bool detail::is_nullary_arg<F, VT=decay_t<F>> (F&& f)
    requires (is_member_pointer_v<VT> || is_move_only_function<VT>::value) { return f == nullptr; }
bool detail::is_nullary_arg<F, VT=decay_t<F>> (F f)
    requires (is_function_v<remove_pointer_t<VT>>) { return f == nullptr; }

// invokes
struct detail::mo_invoke_function_holder<noEx,R,...Args> { // s.pfn_ (args...)
    static R invoke_function(storage s, Args&&...args) noexcept(noEx)
    {   using Fp = R(*)(Args...);
        return invoke_r<R>((Fp)(s.pfn_), std::forward<Args>(args)...); }
};
struct detail::mo_invoke_object_holder<refq,const_,noEx,F,R,...Args> { // (*s.pobj_) (args...)
    static R invoke_object(storage s, Args&&...args) noexcept(noEx)
    {   using T = remove_reference_t<F>; using cv_T = conditional_t<const_, add_const_t<T>, T>; using cv_ref_T = ...;
        return invoke_r<R>((cv_ref_T)*(cv_T*)(s.pobj_), std::forward<Args>(args)...); }
};
struct detail::mo_invoke_local_holder<refq,const_,noEx,F,R,...Args> { // f(args...)
    static R invoke_local(storage s, Args&&...args) noexcept(noEx)
    {   using T = remove_reference_t<F>; using cv_T = conditional_t<const_, add_const_t<T>, T>; using cv_ref_T = ...;
        return invoke_r<R>((cv_ref_T)*(cv_T*)(s.addr()), std::forward<Args>(args)...); }
};
enum class detail::op_type {move,destroy};
struct detail::move_only_function_base<refq,const_,noEx,R,...Args> {
    storage s_;
    R (*invoke_)(thunk_storage<noEx>, Args&&...) noexcept(noEx) = {nullptr}; // invoker
    void (*manager_)(op_type, storage&, storage*) = &manage_empty; // lifetime state-machine manager

    ctor() = default;
    ctor(self&& rhs) noexcept { // move
        manager_ = rhs.manager_; manager_(op_type::move, s_, &rhs.s_);
        invoke_ = rhs.invoke_; rhs.invoke = nullptr; rhs.manager_ = &manage_empty;
    }
    ~dtor() { destroy(); }
    void swap(self& rhs) noexcept {
        storage s; rhs.manager_(move, s, &rhs.s_); // move rhs to s
        manager_(move, rhs.s_, &s_); // move this to rhs
        rhs.manager(move, s_, &s); // move s to this
        std::swap(manager_, rhs.manager_); std::swap(invoke_, rhs.invoke_);
    }
    self& operator=(self&& rhs); // destroy() then like move-ctor
    self& operator=(nullptr_t) noexcept; // destroy() then set to empty;

    static void manage_empty (op_type, storage&, storage*) {}
    static void manage_function (op_type op, storage& s, storage* src) {
        switch (op) { case op_type::move: s.pfn_ = src->pfn_; src->pfn_ = nullptr; }
    }
    static void manage_object<VT> (op_type op, storage& s, storage* src) {
        switch (op) {
            case op_type::destroy: delete (VT*)(s.pobj_); break;
            case op_type::move: s.pobj_ = src->pobj_; src->pobj_ = nullptr; break; }
    }
    static void manage_local<VT> (op_type op, storage&, storage* src) { // sbo embedded
        switch (op) {
            case op_type::destroy: (VT*)(s.pobj_)->~VT(); break;
            case op_type::move: VT* p = (VT*)src->addr(); // relocate by move+dtor
                new(s.addr()) VT(std::move(*p)); p->~VT(); break; }
    }

    void move_from_compatibile_base <refq2, const2, noex2, R2,...Args2> (move_only_function_base<refq2,const2,noex2,R2,Args2...>& base) {
        manager_ = base.manager_; manager_(op_type::move, s_, &base.s_);
        invoke_ = base.invoke_; base.invoke = nullptr; base.manager_ = &base.manage_empty;
    }
    void init<VT,...CArgs> (type_identity<VT>, CArgs&&... args) {
        if constexpr (std::is_function_v<remove_pointer_t<VT>>) {
            s_.pfn_ = (void(*)())get_first_arg(args...);   manager_ = &manage_function;
            invoke_ = &mo_invoke_function_holder<noex,R,Args...>::invoke_function;
        } else if constexpr (is_polymorphic_function<VT>::value) {
            move_from_compatibile_base(get_first_arg(args...)); // just move in another move_only_function
        } else if constexpr (!storage::use_sbo<VT>()) { // allocate for object
            s_.pobj_ = new VT(std::forward<CArgs>(args)...);    manager_ = &manage_object<VT>;
            invoke_ = &mo_invoke_object_holder<refq,const_,noex,VT,R,Args...>::invoke_object;
        } else { // SBO, embed object into storage
            new (s_.addr()) VT(std::forward<CArgs>(args)...);   manager_ = &manage_local<VT>;
            invoke_ = &mo_invoke_local_holder<refq,const_,noex,VT,R,Args...>::invoke_local;
        }
    }

    void destroy() { manager_(op_type::destroy, s_, nullptr); }

    explicit operator bool() const noexcept { return invoke_ != nullptr; }
};

template<class R, class...Args> struct move_only_function; // primary
// specializations for function signatures
struct move_only_function<R(Args...) [const][&|&&][noexcept]>
    : public move_only_function_base<{none|lvalue|rvalue},{false|true},{false|true},R,Args...> { // refq,const_,noex
    ctor() noexcept {} ctor(nullptr_t) noexcept {};
    ctor(self const&) = delete; ctor(self&&) = default; // move only
    ~dtor() = default;
    self& operator=(self&& rhs) { if (this != rhs) { base::operator(rhs)} return *this; }// move
    self& operator=(nullptr_t) noexcept { base::operator=(nullptr); return *this; }

    ctor<F,VT=decay_t<F>> (F&& f) noexcept(nothrow_init<VT,F>::value) // move-in a callable
        requires !is_same_v<remove_cvref_t<F>,move_only_function> && !is_in_place_type_t<VT>::value &&
                is_callable_from<refq,const_,noex,T,R,Args...>::value
    { if (!is_nullary_arg(std::forward<F>(f))) init(type_identity<VT>{}, std::forward<F>(f)); }

    explicit ctor <T,[U],...CArgs> (in_place_type_t<T>, <std::initializer_list<U>& il>, CArgs&&...args)
        noexcept(nothrow_init<T,<std::initializer_list<U>&>,CArgs...>::value)
        requires is_constructible_v<T,<std::initializer_list<U>&>,CArgs...> &&
        is_callable_from<refq,const_,noex,T,R,Args...>::value
    { init(type_identity<T>{}, <il>, std::forward<CArgs>(args)...); }

    self& operator= <F> (F&& f); // move-in a callable
    { self(std::forward<F>(f)).swap(*this); return *this; }

    void swap(self& rhs) noexcept { if (this != &rhs) base::swap(rhs); }
    
    R operator() (Args...args) [const][&|&&][noexcept] { return invoke_(s_, std::forward<Args>(args)...); }
};
bool operator== (move_only_function const& fn, nullptr_t) noexcept { return fn.invoke_ == nullptr; }
bool operator!= (move_only_function const& fn, nullptr_t) noexcept { return !(fn==nullptr); }
void swap (move_only_function& lhs, move_only_function& rhs) noexcept { lhs.swap(rhs); }
```

* `move_only_function` (C++23).

----
* Header `<boost/compat/invoke.hpp>`

```c++
constexpr auto invoke <F,...A> (F&& f, A&&... a) BOOST_COMPAT_RETURNS( std::forward<F>(f)(std::forward<A>(a)...) )
constexpr auto invoke <M,T,...A> (M T::* pm, A&&... a) BOOST_COMPAT_RETURNS( mem_fn(f)(std::forward<A>(a)...) )

using invoke_result_t <F,...A> = decltype()
using is_invocable<F,...A> = {SFINAE invok_result_t<F,A...>} {};
using is_nothrow_invocable<F,...A> = is_invocable<F,A...> && noexcept(invoke(declval<F>(),declval<A>()...)) {};

constexpr R invoke_r <R,F,...A> (F&& f, A&&... a) requires is_void_v<R> && is_invocable<F,A...>::value;
constexpr R invoke_r <R,F,...A> (F&& f, A&&... a) requires !is_void_v<R> && is_converible_v<invoke_result_t<F,A...>,R>;

using is_invocable_r <R,F,...A> = is_invocable<F,...A>::value && is_void_v<V> || is_converible_v<invoke_result_t<F,A...>,R>;
using is_nothrow_invocable_r <R,F,...A> = is_invocable<F,A...> && noexcept(invoke_r<R>(declval<F>(),declval<A>()...)) {};
```

* `invoke` (C++17), `invoke_r` (C++23)
* `invoke_result_t` (C++17)
* `is_invocable`, `is_nothrow_invocable`, `is_invocable_r`, `is_nothrow_invocable_r` (C++17)

-----
#### `<utility>`

* Header `<boost/compat/integer_sequence.hpp>`

```c++
template <class T, T...I> struct integer_sequence {};
template <class T, T N> using make_integer_sequence = ...;
template <size_t...i> using index_sequence = integer_sequence<size_t,i...>;
template <size_t N> using make_index_sequence = make_integer_sequence<size_t,N>;
template <class ...T> using index_sequence_for = make_index_sequence<sizeof...(T)>;
```

* `integer_sequence` (C++14).

-----
* Header `<boost/compat/to_underlying.hpp>`

```c++
constexpr std::unerlying_type<E>::type to_underlying <E> (E e) noexcept { return {e}; }
```

* `to_underlying` (C++23)

-----
#### `<latch>`

* Header `<boost/compat/latch.hpp>`

```c++
class latch {
    ptrdiff_t n_;
    mutable std::mutex m_{}; mutable std::condition_variable cv_{};

    bool is_ready() const { return n_ == 0; }
    bool count_down_and_notify(std::unique_lock<std::mutex>& lk, ptrdiff_t n)
    { if ((n_ -= n) == 0) { lk.unlock(); cv_.notify_all(); return false; } else return true; }
    void wait_impl(std::unique_lock<std::mutex>& lk) const
    { cv_.wait(lk, [this]{ return this->is_ready(); }); }

public: explicit latch(ptrdiff_t expected) : n{expected} {};    ~dtor() = default;
    ctor(self const&) = delete; self& operator=(self const&) = delete; // non-copy/move

    void count_down(ptrdiff_t n=1) { std::unique_lock lk{m_}; count_down_and_notify(lk, n); }
    bool try_wait() const noexcept { std::unique_lock lk{m_}; return is_ready(); }
    void wait() const { std::unique_lock lk{m_}; wait_impl(lk); }
    void arrive_and_wait(ptrdiff_t n=1)
    { std::unique_lock lk{m_}; if (count_down_and_notify(lk, n)) wait_impl(lk); }

    static constexpr ptrdiff_t max() noexcept { return PTRDIFF_MAX; }
};
```

* `latch` (C++20).

-----
#### `<shared_mutex>`

* Header `<boost/compat/shared_lock.hpp>`

```c++
void detail::throw_system_error(std::errc e, boost::source_location const& loc=BOOST_SOURCE_LOCATION)
{ boost::throw_exception(std::system_error(std::make_error_code(e)), loc); }

class shared_lock<Mutex> {
    Mutex* pm_{nullptr}; bool owns_{false}; // owns locking-state
public:
    using mutex_type = Mutex;
    ctor() noexcept = default;
    ctor(const self&) = delete; self& operator=(const self&) = delete; // no copy
    ctor(self&& u) noexcept { pm_ = u.pm_; owns_ = u.owns_; u.pm_ = nullptr; u.owns_ = false; } // move
    self& operator=(self&& u) noexcept { shared_lock(std::move(u)).swap(*this); return *this; } // move
    void swap(self& u) noexcept { std::swap(pm_, u.pm_); std::swap(owns_, u.owns_); }

    explicit ctor(mutex_type& m) : pm_(std::addressof(m)) { lock(); }
    ctor(mutex_type& m, std::defer_lock_t) noexcept : pm_(std::addressof(m)) {}
    ctor(mutex_type& m, std::try_to_lock_t) noexcept : pm_(std::addressof(m)) { try_lock(); }
    ctor(mutex_type& m, std::adopt_lock_t) noexcept : pm_(std::addressof(m)), owns{true} {}
    ~dtor() { if (owns_) unlock(); }
    mutex_type* release() noexcept { mutex_type* pm = pm_; pm_=nullptr; owns_ = false; return pm; }

    mutex_type* mutex() const noexcept { returm pm_; }
    bool owns_lock() const noexcept { return owns_; }
    explicit operator bool() const noexcept { return owns_; }

    void lock() {
        if (!pm_) throw_system_error(std::errc::operation_not_permitted);
        if (owns_) throw_system_error(std::errc::resource_deadlock_would_occur);
        pm_->lock_shared(); owns_ = true;
    }
    void try_lock() {
        if (!pm_) throw_system_error(std::errc::operation_not_permitted);
        if (owns_) throw_system_error(std::errc::resource_deadlock_would_occur);
        bool b = pm_->try_lock_shared(); owns_ = b; return b;
    }
    void unlock() {
        if (!pm_ || !owns_) throw_system_error(std::errc::operation_not_permitted);
        pm_->unlock_shared(); owns_ = false;
    }
};
void swap<Mutex> (shared_lock<Mutex>& x, shared_lock<Mutex>& y) noexcept { x.swap(y); }
```

* `shared_lock` (C++14).

------
#### `<array>`

* Header `<boost/compat/to_array.hpp>`

```c++
constexpr std::array<remove_cv_t<T>,n> detail::to_array_lvalue <T,n,...i> (T (&a)[n], index_sequence<i...>) { return {{a[i]...}}; }
constexpr std::array<remove_cv_t<T>,n> detail::to_array_rvalue <T,n,...i> (T (&&a)[n], index_sequence<i...>) { return {{std::move(a[i])...}}; }
constexpr std::array<remove_cv_t<T>,n> to_array <T,n> (T (&a)[n]) requires is_constructible_v<remove_cv_t<T>,T&> && !is_array_v<T>
    { return to_array_lvalue(a, make_index_sequence<n>{}); }
constexpr std::array<remove_cv_t<T>,n> to_array <T,n> (T (&&a)[n]) requires is_constructible_v<remove_cv_t<T>,T&&> && !is_array_v<T>
    { return to_array_rvalue(std::move(a), make_index_sequence<n>{}); }
```

* `to_array` (C++20)

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`


