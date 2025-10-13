# Boost.Scope

* lib: `boost/libs/scope`
* repo: `boostorg/scope`
* commit: `5a0b5ea`, 2025-09-24

------
### Scope Guards

------
#### `defer_guard`

* Header `<boost/scope/defer.hpp>`

```c++
struct detail::is_not_like<T, Template<...>> : std::true_type{}; // primary, T is not spec of Template
struct detail::is_not_like<T& Template<...>> : is_not_like<T,Template> {}; // T& => T
struct detail::is_not_like<[const][volatile]Template<Ts...>, Template> : std::false_type{}; // disable temp specs
struct detail::move_or_copy_construct_ref<From,To=From> { using type = is_nothrow_constructible_v<To,From> ? From&& : From const& };
struct detail::move_or_copy_construct_ref<From&,To> { using type = From&; }; // fixed if From is a ref
struct detail::move_or_copy_assign_ref<From,To=From> { using type = is_nothrow_assignable_v<To,From> ? From&& : From const& };
struct detail::move_or_copy_assign_ref<From&,To> { using type = From&; }; // fixed if From is a ref

class defer_guard<Func> {
    struct data { Func m_func;
        explicit ctor <F> (F&& func, true_type) noexcept requires is_constructible_v<Func,F>; // init for noexcept Func
        explicit ctor <F> (F&& func, false_type) requires is_constructible_v<Func,F> // init for throwing Func
        try : m_func((F&&)func) {} catch(...) { func(); }
    } m_data;
public: // disable copy/move
    ctor <F>(F&& func) noexcept(...) requires is_not_like<F, defer_guard> && (...)
        : m_data(move_or_copy_construct_ref<F,Func>::type(func),
                 is_nothrow_constructible_t<Func,move_or_copy_construct_ref<F,Func>::type>{}) {}
    ~dtor() noexcept(is_nothrow_invocable_v<Func&>) { m_data.m_func(); } // invoke on dtor
};
defer_guard(Func) -> defer_guard<Func>;

#define BOOST_SCOPE_DEFER boost::scope::defer_guard _boost_defer_guard_##__LINE__ =
```

`defer_guard` is a lightweight scope guard, no condition, no activation, no moveability.
Usage:

```c++
BOOST_SCOPE_DEFER []{/* cleanup code here */};

defer_guard g1 = []{}; // init with lambda
defer_guard g2{[]{}};

function f = []{}; // lateron can change f at runtime
defer_guard g3 = ref(f); // init with reference to f
```

------
#### `scope_exit`, `scope_success`, and `scope_fail`

* Header `<boost/scope/scope_exit.hpp>`
* Header `<boost/scope/scope_success.hpp>`
* Header `<boost/scope/scope_fail.hpp>`

```c++
struct detail::is_nonnull_default_constructible<T> : is_default_constructible<T>{}; // primary
struct detail::is_nonnull_default_constructible<T*> : false_type {}; // pointers can be null
struct detail::is_nothrow_nonnull_default_constructible<T> : is_nothrow_default_constructible<T>{}; // primary
struct detail::is_nothrow_nonnull_default_constructible<T*> : false_type {}; // pointers can be null

class detail::compact_storage<T,Tag=void,b=is_class_v<T> && !is_final_v<T>> : private T { // primary, inherit so that EBO works
public: // copy/move all =default, based on T's
    constexpr ctor<...Args> (Args&&...args) noexcept(...) : T((Args&>(args)...)) {} // init
    T<const>& get() <const> noexcept { return *(T<const>*)(this); }
};
class detail::compact_storage<T,Tag,false> { // cannot inherit T
    T m_data;
public: // copy/move all =default, based on T's
    constexpr ctor<...Args> (Args&&...args) noexcept(...) : m_data((Args&)args...) {} // init
    T<const>& get() <const> noexcept { return m_data; }
};

class detail::init_guard<Func,Cond> {
    Func& m_func; Cond& m_cond; bool m_active;
public: ctor(Func& f, Cond& c, bool activate) noexcept; // init
    ~dtor() noexcept(is_nothrow_invocable_v<Func> && is_nothrow_invocable_v<Cond&>) { if (m_active && m_cond()) m_func(); }
    Func&& get_func() noexcept; Cond&& get_cond() noexcept;
    void deactivate() noexcept { m_active = false; }
};

class always_true { using result_type = bool; result_type operator()() const noexcept { return true; }};

class scope_exit<Func,Cond=always_true> {
    struct func_holder : compact_storage<Func>; // use init_guard if ctor of Func with F may throw
    struct cond_holder : compact_storage<Cond>; // use init_guard if ctor of Cond with C may throw
    struct data: func_holder, cond_holder {
        bool m_active;
        ctor <F,C> (F&& f, C&& c, bool active) noexcept(is_nothrow_constructible_v<Func,F> && is_nothrow_constructible_v<Cond, C>); // init
        Func <const>& get_func() <const>noexcept; COnd <const>& get_cond() <const>noexcept;
        bool deactivate() noexcept { bool a = m_active; m_active = false; return a; } // transfer activation state
    } m_data;
public:
    explicit ctor <F> (F&& func, bool active=true) noexcept(...); // init
    explicit ctor <F,C> (F&& func, C&& cond, bool active=true) noexcept(...); // init
    // move-ctor, disable copy-ctor, copy/move-assign
    ctor(self&&) noexcept(...) requires is_constructible_v<data, move_or_copy_construct_ref<Func>, move_or_copy_construct_ref<Cond>, bool>; // move
    ~dtor() noexcept(...) { if (m_data.m_active && m_data.get_cond()()) m_data.get_func(); } // invoke on dtor
    bool active() const noexcept; void set active(bool active) noexcept; // m_data.m_active
};
explicit scope_exit<F> (F,<bool>) -> scope_exit<F>;
explicit scope_exit<F,C> (F,C,<bool>) -> scope_exit<F,C>;
scope_exit<decay_t<F>> make_scope_exit <F> (F&& f, bool active=true) noexcept(...);
scope_exit<decay_t<F>,decay_t<C>> make_scope_exit <F,C> (F&& f, C&& c, bool active=true) noexcept(...) requires(...);

class scope_success<Func,Cond=exception_checker> : public scope_exit<Func,logical_not<Cond>> {
public: // ctors & deducing-guides similar to scope_exit
};
scope_success<decay_t<F>> make_scope_success <F> (F&& f, bool active=true) noexcept(...);
scope_success<decay_t<F>,decay_t<C>> make_scope_success <F,C> (F&& f, C&& c, bool active=true) noexcept(...) requires(...);

class scope_fail<Func,Cond=exception_checker> : public scope_exit<Func,Cond> {
public: // ctors & deducing-guides similar to scope_exit
};
scope_fail<decay_t<F>> make_scope_fail <F> (F&& f, bool active=true) noexcept(...);
scope_fail<decay_t<F>,decay_t<C>> make_scope_fail <F,C> (F&& f, C&& c, bool active=true) noexcept(...) requires(...);
```

* Header `<boost/scope/exception_checker.hpp>` and `<boost/scope/error_code_checker.hpp>`

```c++
class exception_checker {
    unsigned int m_uncaught_count;
public: using result_type = bool;
    ctor() noexcept : m_uncaught_count(boost::core::uncaught_exceptions()) {} // remember ex count on ctor
    result_type operator()() const noexcept { return core::uncaught_exceptions() > m_uncaught_count; } // check ex count growth
};
exception_checker check_exception() noexcept; // factory

class error_code_checker<ErrorCode> {
    ErrorCode* m_error_code; // ptr referring to ec
public: using result_type = bool;
    explicit ctor(ErrorCode& ec) noexcept : m_error_code(addressof(ec)) {} // init ref
    result_type operator()() const noexcept { return !!(*m_error_code); } // check ec
};
error_code_checker<ErrorCode> check_error_code<ErrorCode> (ErrorCode& ec) noexcept; // factory
```

-----
### RAII Resource Wrappers

------
#### `unique_resource`

* Header `<boost/scope/unique_resource_fwd.hpp>`
* Header `<boost/scope/unique_resource.hpp>`

```c++
// traits
template<auto defVal, auto... vals>
struct unallocated_resource {
    static decltype(defVal) make_default() noexcept { return defVal; }
    static bool is_allocated<Res> (Res const& res) noexcept { res != defVal && (...&& res!=vals); }
};
struct default_resource_t {};
using detail::is_default_resource<T> = is_same<remove_cv_ref_t<T>,default_resource_t>;

class detail::ref_wrapper<T> {
    T* m_value;
public: ctor(T& val) noexcept{} self& operator=(T& val) noexcept; // init/assign
    // disable move
    operator T&() const noexcept; // impl conv
    void operator() <...Args> (Args&&...args) const noexcept(...); // invoke
};
struct detail::wrap_reference<T> { using type = is_reference_v<T> ? ref_wrapper<T> : T; };

class detail::resource_holder<Res,useCompSt> : public compact_storage<wrap_reference<Res>::type> {
    explicit ctor<R,D> (R&& res, D&& del, bool allocated, true_type) noexcept : base((R&&)(res)) {}
    explicit ctor<R,D> (R&& res, D&& del, bool allocated, false_type) // ctor for throwing init
        try : base(res) {} catch(...) { if (allocated) del(res); }
public: using resource_type = Res; using internal_resource_type = wrap_reference<Res>::type;
    constexpr ctor() noexcept(...){} // init
    explicit ctor <R> (R&& res) noexcept(...) : base((R&&)(res)) {} // move-init
    explicit ctor <R,D> (R&& res, D&& del, bool allocated) noexcept(...)
        : base((R&&)(res), (D&&)(del), allocated, is_nothrow_constructible_t<Res,R>) {}
    resource_type <const>& get() <const> noexcept;
    internal_resource_type <const>& get_internal() <const> noexcept;
    void move_from(internal_resource_type&& that) noexcept(...); // move-in
};
class detail::resource_holder<Res,false> {
    internal_resource_type m_resource; // can't inherit, use data member
public: // same types and members like above.
};
using detail::use_resource_compact_storage<Res,Del> = ...; // Res& can nothrow move, Del& can nothrow construct from Del, ..

class detail::deleter_holder<Res,Deleter> : compact_storage<wrap_reference<Deleter>::type> {
    explicit ctor<R,D> (R&& res, D&& del, bool allocated, true_type) noexcept : base((R&&)(res)) {}
    explicit ctor<R,D> (R&& res, D&& del, bool allocated, false_type) // ctor for throwing init
        try : base(res) {} catch(...) { if (allocated) del(res); }
public: using resource_type = Res; using deleter_type = Deleter; using internal_deleter_type = wrap_reference<Deleter>::type;
    constexpr ctor() noexcept(...){} // init
    explicit ctor <D> (D&& del) noexcept(...) : base((D&&)(res)) {} // move-init
    explicit ctor <R,D> (D&& del, Res& res, bool allocated) noexcept(...)
        : base((D&&)(del), res, allocated, is_nothrow_constructible_t<internal_deleter_type,D>) {}
    deleter_type <const>& get() <const> noexcept;
    internal_deleter_type <const>& get_internal() <const> noexcept;
};

class detail::unique_resource_data<Res,Del,Tr>
    : public resource_holder<Res, use_resource_compact_storage<Res,Del>>, public deleter_holder<Res,Del>
{ // disable copy-ctor and copy-assign
public: using result_of_make_default = decltype(Tr::make_default()); // and resource_type, deleter_type, traits_type
    constexpr ctor() noexcept(...) : resource_holder(Tr::make_default()) {}
    ctor(self&& that) noexcept(...); // move-ctor
    explicit ctor<D> (default_resource_t, D&& del) noexcept(...) : resource_holder(Tr::make_default()), deleter_holder((D&&)(del)) {}
    explicit ctor<R,D> (R&& res, D&& del) noexcept(...) requires(...); // move-in init
    self& operator=(self&& that) noexcept(...) requires(...); // move-assign
    <internal_>resource_type <const>& get_<internal_>resource()<const> noexcept;
    <internal_>deleter_type <const>& get_<internal_>deleter()<const> noexcept;
    bool is_allocated() const noexcept { return Tr::is_allocated(get_resource()); }
    void set_unallocated() noexcept { get_internal_resource() = Tr::make_default(); } // set to default
    void assign_resource<R>(R&& res) noexcept(...) { get_internal_resource() = (R&&)(res); }
    void swap(self& that) requires(...);
};
class detail::unique_resource_data<Res,Del,void> // specialize for no traits
    : public resource_holder<Res, use_resource_compact_storage<Res,Del>>, public deleter_holder<Res,Del>
{   bool m_allocated {false}; //set to true for ctors and assigns that accept resource
public: // types and other members same as above,
    bool is_allocated() const noexcept { return m_allocated; }
    void set_unallocated() noexcept { m_allocated = false; }
    void assign_resource<R>(R&& res) noexcept(...) { get_internal_resource() = (R&&)(res); m_allocated = true; }
};

using detail::is_dereferenceable<T> = !is_same_v<{cv}void*, remove_cv_ref_t<T>> && requires requires(T const& t){*t;}
struct deail::dereference_traits<T,deref=is_dereferenceable<T>::value> {}; // primary
struct deail::dereference_traits<T,true> {
    using result_type = decltype(*std::declval<T const&>());
    static constexpr bool is_noexcept = noexcept(*std::declval<T const&>());
};

class unique_resource<Res,Del,Tr=void> {
    unique_resource_data<Res,Del,Tr> m_data;
public: // resource_type, deleter_type, traits_type
    // same ctor, op=, and swap members as m_data
    ~dtor() noexcept(...) { if (m_data.is_allocated()) m_data.get_deleter()(m_data.get_resource()); }
    explicit operator bool() const noexcept; bool allocated() const noexcept;
    void release() noexcept { m_data.set_unallocated(); }
    void reset() noexcept(...) { if (is_allocated()) { get_deleter()(get()); release(); } }
    void reset <R> (R&& res) noexcept(...);
    resource_type const& get() const noexcept; deleter_type const& get_deleter() const noexcept;
    resource_type const& operator-> () const noexcept;
    auto operator* () const noexcept(...) requires is_dereferenceable<resource_type>::value { return *get(); }
    friend void swap(self& l, self& r) noexcept(...) requires is_swappable_v<decltype(m_data)>;
};
unique_resource<Res,Del>(Res,Del) -> unique_resource<Res,Del>;
unique_resource<decay_t<Res>,decay_t<Del>> make_unique_resource_checked<Res,Del,Invalid> (Res&& res, Invalid const& invalid, Del&& del) noexcept(...)
{ if (!(res == invalid)) return {(Res&&)(res), (Del&&)(del)}; else return {default_resource_t(), (Del&&)(del)}; }
```

-----
#### File Descriptor Support

* Header `<boost/scope/fd_deleter.hpp>`
* Header `<boost/scope/fd_resource_traits.hpp>`

```c++
struct fd_deleter { using result_type = void;
    result_type operator() (int fd) const noexcept { ::close(fd); } // on HPUX, repeat until errno != EINTR, on Windows, call ::_close(fd)
};
struct fd_resource_traits {
    static int make_default() noexcept { return -1; }
    static bool is_allocated(int fd) noexcept { return fd >= 0; }
};
```

------
#### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Core

* `<boost/core/addressof.hpp>`
* `<boost/core/invoke_swap.hpp>`
* `<boost/core/uncaught_exceptions.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>` -- For old C++ stdlibs

------
### Standard Facilities

Proposal: Library Fundamentals V3 `scope_exit`, `scope_success`, `scope_fail`, `unique_resource`.
