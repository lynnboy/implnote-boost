# Boost.Compat

* lib: `boost/libs/compat`
* repo: `boostorg/compat`
* commit: `a5a56ee`, 2025-09-05

------
#### `<functional>`

* Header `<boost/compat/bind_back.hpp>` and `<boost/compat/bind_front.hpp>`

```c++
#define BOOST_COMPAT_RETURNS(...) noexcept(noexcept(__VA_ARGS__)) ->decltype(__VA_ARGS__) { return __VA_ARGS__; }

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

------
#### Configuration

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`


