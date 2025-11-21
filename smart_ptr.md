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
### Shared Ownership (`shared_ptr`, `weak_ptr`, `local_shared_ptr`)

```c++
class bad_weak_ptr : public std::exception { char const* what() const noexcept override; };

struct owner_equal_to<T=void>{ using result_type = bool; using first_argument_type = T; using second_argument_type = T;
    bool operator()<U,V>(U const& u, V const& v) const noexcept { return u.owner_equals(v); }
};
struct owner_less<T=void>{ using result_type = bool; using first_argument_type = T; using second_argument_type = T;
    bool operator()<U,V>(U const& u, V const& v) const noexcept { return u.owner_before(v); }
};
struct owner_hash<T>{ using result_type = size_t; using argument_type = T;
    size_t operator()(T const& t) const noexcept { return t.owner_hash_value(); }
};

struct detail::sp_element<T> { using type = T; };
struct detail::sp_element<T[<n>]> { using type = T; };
struct detail::sp_dereference<T> { using type = T&; };
struct detail::sp_dereference<void [const][volatile]> { using type = void; };
struct detail::sp_dereference<T[<n>]> { using type = void; };
struct detail::sp_member_access<T> { using type = T*; };
struct detail::sp_member_access<T[<n>]> { using type = void; };
struct detail::sp_array_access<T> { using type = void; };
struct detail::sp_array_access<T[<n>]> { using type = T&; };
struct detail::sp_extent<T> { enum{value=0}; };
struct detail::sp_extent<T[n]> { enum{value=n}; };

void detail::sp_enable_shared_from_this<X,Y,T>(shared_ptr<X> const* ppx, Y const* py, enable_shared_from_this<T> const* pe)
{ if (pe) pe->_internal_accept_owner(ppx, const_cast<Y*>(py)); }
void detail::sp_enable_shared_from_this(...){}
void detail::sp_assert_convertible() noexcept;
void detail::sp_pointer_construct<T,Y>(shared_ptr<T>* ppx, Y* p, shared_count& pn)
{ shared_count{p}.swap(pn); sp_enable_shared_from_this(ppx, p, p); }
void detail::sp_pointer_construct<T,[n],Y>(shared_ptr<T[<n>]>*, Y* p, shared_count& pn)
{ sp_assert_convertible<Y[<n>], T[<n>]>(); shared_count{p, checked_array_deleter<T>{}}.swap(pn); }
void detail::sp_deleter_construct<T,Y>(shared_ptr<T>* ppx, Y* p) { sp_enable_shared_from_this(ppx,p,p); }
void detail::sp_deleter_construct<T,[n],Y>(shared_ptr<T[n]>*, Y* p) { sp_assert_convertible<Y[<n>], T[<n>]>(); }

struct detail::sp_internal_cosntructor_tag{};

class shared_ptr<T> {
    element_type* px{nullptr}; shared_count pn;
public: using element_type = sp_element<T>::type;
    constexpr ctor(<nullptr_t>) noexcept{}
    constexpr ctor(sp_internal_cosntructor_tag, element_type* px_, shared_count {const&|&&} pn_) noexcept: px{px_}, pn(<std::move>(pn_)) {}
    explicit ctor<Y>(Y* p): px{p} { sp_pointer_construct(this, p, pn); }
    ctor<Y,D,[A]>(Y* p, D d): px{p}, pn{p, (D&&)d, <a>} { sp_deleter_construct(this, p); }
    ctor<D,[A]>(nullptr_t p, D d): px{p}, pn{p, (D&&)d, <a>} {}
    ctor(self const&) noexcept: px{r.px}, pn{r.pn}{}
    explicit ctor<Y>(weak_ptr<Y> const& r) requires(...) : pn{r.pn} { sp_assert_convertible<Y,T>(); px = r.px; }
    ctor<Y>(weak_ptr<Y> const& r, sp_nothrow_tag) noexcept: pn{r.pn, sp_nothrow_tag{}} { if (!pn.empty()) px=r.px; }
    ctor<Y>(shared_ptr<Y> const& r) noexcept requires(...) : px{r.px}, pn{r.pn} { sp_assert_convertible<Y,T>(); }
    ctor<Y>(shared_ptr<Y> const& r, element_type* p) noexcept: px{p}, pn{r.pn} {}
    ctor<Y,D>(std::unique_ptr<Y,D>&& r) requires(...) : px{r.get()} { sp_assert_convertible<Y,T>(); // also movelib::unique_ptr
        if (auto tmp = r.get()) { pn = shared_count{r}; sp_deleter_construct(this, tmp); } }
    self& operator=<Y>(self<Y> const& r) noexcept { self{r}.swap(*this); return *this; }
    self& operator=<Y,D>(std::unique_ptr<Y,D>&& r) { self{std::move(r)}.swap(*this); return *this; }
    self& operator=<Y,D>(movelib::unique_ptr<Y,D> r) requires(...) { sp_assert_convertible<Y,T>();
        self tmp{}; if (auto p = r.get()) { tmp.px=p; tmp.pn=shared_count{r}; sp_deleter_construct(&tmp, p); }
        tmp.swap(*this); return *this;
    }
    ctor(self&& r) noexcept: px{r.px}, pn{std::move(r.pn)} { r.px = nullptr; }
    ctor<Y>(shared_ptr<Y>&& r) noexcept requires(...) : px{r.px}, pn{std::move(r.pn)} { sp_assert_convertible<Y,T>(); r.px = nullptr; }
    self& operator=<Y>(self<Y>&& r) noexcept { self{std::move(r)}.swap(*this); return *this; }
    ctor<Y>(shared_ptr<Y>&& r, element_type* p) noexcept: px{p} { pn.swap(r.pn); r.px = nullptr; }
    self& operator=(nullptr_t) noexcept { self{}.swap(*this); return *this; }
    void reset() noexcept { self{}.swap(*this); }
    void reset<Y,[D],[A]>(Y* p, <D d>, <A a>) { self{p, <(D&&)d>, <a>}.swap(*this); }
    void reset(shared_ptr<Y> {const&|&&} r, element_type* p) noexcept { self{<std::move>(r),p}.swap(*this); }
    sp_dereference<T>::type operator*() const noexcept { return *px; }
    sp_member_access<T>::type operator->() const noexcept { return px; }
    sp_array_access<T>::type operator[](ptrdiff_t i) const noexcept { return {px[i]}; }
    element_type* get() const noexcept { return px; }
    explicit operator bool() const noexcept { return px!=nullptr; }
    bool unique() const noexcept { return pn.unique(); }
    long use_count() const noexcept { return pn.use_count(); }
    void swap(self& other) noexcept { std::swap(px, other.px); pn.swap(other.pn); }
    bool owner_before<Y>({shared_ptr|weak_ptr}<Y> const& rhs) const noexcept { return pn < rhs.pn; }
    bool owner_equals<Y>({shared_ptr|weak_ptr}<Y> const& rhs) const noexcept { return pn == rhs.pn; }
    size_t owner_hash_value() const noexcept { return pn.hash_value(); }
    void* _internal_get_deleter(sp_typeinfo_ const& ti) const noexcept { return pn.get_deleter(ti); }
    void* _internal_get_local_deleter(sp_typeinfo_ const& ti) const noexcept { return pn.get_local_deleter(ti); }
    void* _internal_get_untyped_deleter() const noexcept { return pn.get_untyped_deleter(); }
    bool _internal_equiv(self const& r) const noexcept { return px==r.px && pn==r.pn; }
    shared_count _internal_count() const noexcept { return pn; }

    friend bool operator==(self const& a, self const& b) noexcept { return a.get() == b.get(); } // and !=, and comp with nullptr_t
    friend bool operator<(self const& a, self const& b) noexcept { return a.owner_before(b); }
    friend void swap(self& a, self& b) noexcept { a.swap(b); }
    friend self static_pointer_cast<U>(self<U> {const&|&&} r) noexcept // all 4 casts
    { auto p = (element_type*)r.get(); return {<std::move>(r),p}; }
    friend element_type* get_pointer(self const& p) noexcept { return p.get(); }
    friend std::basic_ostream<Ch,Tr>& operator<<(std::basic_ostream<Ch,Tr>& os, self const& p) { return os << p.get(); }
};
shared_ptr<T>(weak_ptr<T>) -> shared_ptr<T>;
shared_ptr<T,D>(std::unique_ptr<T,D>) -> shared_ptr<T>;

D* detail::basic_get_deleter<D,T>(shared_ptr<T> const& p) noexcept { return (D*)p._internal_get_deleter(typeid(D)); }
D<const>* detail::basic_get_local_deleter<D,T>(D<const>*, shared_ptr<T> const& p) noexcept { return (D*)p._internal_get_local_deleter(typeid(D)); }
class detail::esft2_deleter_wrapper {
    shared_ptr<void const volatile> deleter_;
public: ctor()noexcept{}
    void set_deleter<T>(shared_ptr<T>const& deleter) noexcept { deleter_ = deleter; }
    D* get_deleter<D>() const noexcept { return basic_get_deleter<D>(deleter_); }
    void operator()<T>(T*) noexcept { deleter_.reset(); }
};
D* get_deleter<D,T>(shared_ptr<T> const& p) noexcept {
    D* d = basic_get_deleter<D>(p); if (!d) d = basic_get_local_deleter(d, p);
    if (!d) if (auto del_wrapper = basic_get_deleter<esft2_deleter_wrapper>(p)) d = del_wrapper->get_deleter<D>();
    return d;
}
bool atomic_is_lock_free<T>(shared_ptr<T> const*) noexcept { return false; }
shared_ptr<T> atomic_load(shared_ptr<T> const* p) noexcept { spinlock_pool<2>::scoped_lock lock{p}; return *p; }
shared_ptr<T> atomic_load_explicit<T,M>(shared_ptr<T> const* p, M) noexcept { return atomic_load(p); }
void atomic_store(shared_ptr<T>* p, shared_ptr<T> r) noexcept { spinlock_pool<2>::scoped_lock lock{p}; p->swap(r); }
void atomic_store_explicit<T,M>(shared_ptr<T>* p, shared_ptr<T> r, M) noexcept { atomic_store(p, r); }
shared_ptr<T> atomic_exchange(shared_ptr<T>* p, shared_ptr<T> r) noexcept {
    auto& sp = spinlock_pool<2>::spinlock_for(p);
    sp.lock(); p->swap(r); sp.unlock(); return r;
}
shared_ptr<T> atomic_exchange_explicit<T,M>(shared_ptr<T>* p, shared_ptr<T> r, M) noexcept { return atomic_exchange(p, r); }
bool atomic_compare_exchange<T>(shared_ptr<T>* p, shared_ptr<T>* v, shared_ptr<T> w) noexcept {
    auto& sp = spinlock_pool<2>::spinlock_for(p);
    if (p->_internal_equiv(*v)) { p->swap(w); sp.unlock(); return true; }
    else { shared_ptr<T> tmp{*p}; sp.unlock(); tmp.swap(*v); return false; }
}
bool atomic_compare_exchange_explicit<T,M>(shared_ptr<T>* p, shared_ptr<T>* v, shared_ptr<T> w, M, M) noexcept { return atomic_compare_exchange(p,v,w); }
size_t hash_value<T>(shared_ptr<T> const& p) noexcept { return hash<shared_ptr<T>::element_type*>{}(p.get()); }
struct std::hash<shared_ptr<T>> { size_t operator()(shared_ptr<T> const& p) const noexcept { return std::hash<shared_ptr<T>::element_type*>{}(p.get()); } };

class shared_array<T> { using deleter = checked_array_deleter<T>;
    T* px{nullptr}; shared_count pn;
public: ctor(<nullptr_t>) noexcept{}
    explicit ctor<Y>(Y* p) requires(...) : px{p}, pn{p, checked_array_deleter<Y>{}} { sp_assert_convertible<Y[],T[]>(); }
    ctor<Y,D,[A]>(Y* p, D d, <A a>) requires(...) : px{p}, pn{p, d, <a>} { sp_assert_convertible<Y[],T[]>(); }
    ctor(self const& r) noexcept: px{r.px}, pn{r.pn}{}
    ctor(self && r) noexcept: px{r.px}, pn{r.pn}{ pn.swap(r.pn); r.px=nullptr; }
    ctor<Y>(self<Y> const& r) noexcept requires(...) : px{r.px}, pn{r.pn} { sp_assert_convertible<Y[],T[]>(); }
    ctor<Y>(self<Y> const& r, element_type* p) noexcept: px{p}, pn{r.pn} {}
    self& operator=<Y>(self<Y> {const&|&&} r) noexcept { self{<std::move>(r)}.swap(*this); return *this; }
    void reset() noexcept { self{}.swap(*this); }
    void reset<Y,[D],[A]>(Y* p, <D d>, <A a>) { self{p,<d>,<a>}.swap(*this); }
    void reset<Y>(self<Y> const& r, element_type* p) noexcept { self{r,p}.swap(*this); }
    T& operator[] (ptrdiff_t i) const noexcept { return px[i]; }
    T* get() const noexcept { return px; }
    explicit operator bool() const noexcept { return px!=nullptr; }
    bool unique() const noexcept { return pn.unique(); }
    long use_count() const noexcept { return pn.use_count(); }
    void swap(self& other) noexcept { std::swap(px, other.px); pn.swap(other.pn); }
    void* _internal_get_deleter(sp_typeinfo_ const& ti) const noexcept { return pn.get_deleter(ti); }

    friend bool operator==(self const& a, self const& b) noexcept { return a.get() == b.get(); } // and !=, and comp with nullptr_t
    friend bool operator<(self const& a, self const& b) noexcept { return a.owner_before(b); }
    friend void swap(self& a, self& b) noexcept { a.swap(b); }
    friend D* get_deleter<D,T>(self const& p) noexcept { (D*)p._internal_get_deleter(typeid(D)); }
};

class weak_ptr<T> {
    element_type* px{nullptr}; weak_count pn;
public: using element_type = sp_element<T>::type;
    constexpr ctor() noexcept{}
    ctor(self const&) noexcept: px{r.px}, pn{r.pn}{}
    ctor(self &&) noexcept: px{r.px}, pn{std::move(r.pn)}{ r.px = nullptr; }
    self& operator=(self const& r) noexcept { px=r.px; pn=r.pn; return *this; }
    self& operator=(self && r) noexcept { self{std::move(r)}.swap(*this); return *this; }
    ctor<Y>(self<Y> const& r) noexcept requires(...) : px{r.lock().get()}, pn{r.pn} { sp_assert_convertible<Y,T>(); }
    ctor<Y>(self<Y> && r) noexcept requires(...) : px{r.lock().get()}, pn{std::move(r.pn)} { sp_assert_convertible<Y,T>(); r.px=nullptr; }
    ctor<Y>(shared_ptr<Y> const& r) requires(...) : px{r.px}, pn{r.pn} { sp_assert_convertible<Y,T>(); }
    ctor<Y>({shared_ptr|weak_ptr}<Y> const& r, element_type* p) noexcept : px{p}, pn{r.pn} {}
    ctor<Y>(self<Y> && r, element_type* p) noexcept : px{p}, pn{std::move(r.pn)} {}
    self& operator=<Y>({shared_ptr|weak_ptr}<Y> const& r) noexcept requires(...) { sp_assert_convertible<Y,T>();
        px = r.lock().get(); pn = r.pn; return *this;
    }
    self& operator=<Y>(self<Y> && r) noexcept { self{std::move(r)}.swap(*this); return *this; }
    shared_ptr<T> lock() const noexcept { return {*this, sp_nothrow_tag{}}; }
    long use_count() const noexcept { return pn.use_count(); }
    bool expired() const noexcept { return pn.use_count() == 0; }
    bool <_>empty() const noexcept { return pn.empty(); }
    void reset() noexcept { self{}.swap(*this); }
    void swap(self& other) noexcept { std::swap(px, other.px); pn.swap(other.pn); }
    bool owner_before<Y>({shared_ptr|weak_ptr}<Y> const& rhs) const noexcept { return pn < rhs.pn; }
    bool owner_equals<Y>({shared_ptr|weak_ptr}<Y> const& rhs) const noexcept { return pn == rhs.pn; }
    size_t owner_hash_value() const noexcept { return pn.hash_value(); }

    friend bool operator<(self const& a, self const& b) noexcept { return a.owner_before(b); }
    friend void swap(self& a, self& b) noexcept { a.swap(b); }
};
weak_ptr<T>(shared_ptr<T>) -> weak_ptr<T>;
size_t hash_value<T>(weak_ptr<T> const& p) noexcept { return p.owner_hash_value(); }
struct std::hash<weak_ptr<T>> { size_t operator()(weak_ptr<T> const& p) const noexcept { return p.owner_hash_value(); } };
struct std::equal_to<weak_ptr<T>> { bool operator()(weak_ptr<T> const& a, weak_ptr<T> const& b) const noexcept { return a.owner_equals(b); } };

void detail::lsp_pointer_construct<E,Y>(local_shared_ptr<E>*, Y* p, local_counted_base*& pn) {
    sp_assert_convertible<Y,E>(); using D = local_sp_deleter<checked_deleter<E>>;
    shared_ptr<E> p2{p, D{}};  D* pd = (D*)p2._internal_get_untyped_deleter();
    pd->pn_ = p2._internal_count(); pn = pd;
}
void detail::lsp_pointer_construct<E,[n],Y>(shared_ptr<T[<n>]>*, Y* p, local_counted_base*& pn) {
    sp_assert_convertible<Y[<n>],E[<n>]>(); using D = local_sp_deleter<checked_array_deleter<E>>;
    shared_ptr<E[<n>]> p2{p, D{}};  D* pd = (D*)p2._internal_get_untyped_deleter();
    pd->pn_ = p2._internal_count(); pn = pd;
}
void detail::lsp_deleter_construct<E,P,D>(local_shared_ptr<E>*, P p, D const& d, local_counted_base*& pn) {
    using D2 = local_sp_deleter<D>;
    shared_ptr<E> p2{p, D2{d}};  D2* pd = (D2*)p2._internal_get_untyped_deleter();
    pd->pn_ = p2._internal_count(); pn = pd;
}
void detail::lsp_allocator_construct<E,P,D,A>(local_shared_ptr<E>*, P p, D const& d, A const& a, local_counted_base*& pn) {
    using D2 = local_sp_deleter<D>;
    shared_ptr<E> p2{p, D2{d}, a};  D2* pd = (D2*)p2._internal_get_untyped_deleter();
    pd->pn_ = p2._internal_count(); pn = pd;
}
struct detail::lsp_internal_constructor_tag{};

class local_shared_ptr<T> {
    element_type* px{nullptr}; local_counted_base* pn{nullptr};
public: using element_type = sp_element<T>::type;
    ~dtor() noexcept { if (pn) pn->release(); }
    constexpr ctor(<nullptr_t>) noexcept{}
    constexpr ctor(lsp_internal_cosntructor_tag, element_type* px_, local_counted_base* pn_) noexcept: px{px_}, pn{pn_} {}
    explicit ctor<Y>(Y* p): px{p} { lsp_pointer_construct(this, p, pn); }
    ctor<Y,D>(Y* p, D d): px{p} { lsp_deleter_construct(this, p, d, pn); }
    ctor<D>(nullptr_t p, D d): px{p} { lsp_deleter_construct(this, p, d, pn); }
    ctor<Y,D,A>(Y* p, D d, A a): px{p} { lsp_allocator_construct(this, p, d, a, pn); }
    ctor<D,A>(nullptr_t p, D d, A a): px{p} { lsp_allocator_construct(this, p, d, a, pn); }
    ctor<Y>(shared_ptr<Y> const& r) requires(...) : px{r.get()} { sp_assert_convertible<Y,T>();
        if (r.use_count()!=0) pn = new local_counted_impl{r._internal_count()}; }
    ctor<Y>(shared_ptr<Y> && r) requires(...) : px{r.get()} { sp_assert_convertible<Y,T>();
        if (r.use_count()!=0) { pn = new local_counted_impl{r._internal_count()}; r.reset(); } }
    ctor<Y,D>(std::unique_ptr<Y,D>&& r) requires(...) : px{r.get()} { sp_assert_convertible<Y,T>();
        if (px) px = new local_counted_impl{shared_ptr<T>{std::move(r)}._internal_count()}; }
    ctor(self const& r) noexcept: px{r.px}, pn{r.pn} { if (pn) pn->add_ref(); }
    ctor(self && r) noexcept: px{r.px}, pn{r.pn} { r.px = nullptr; r.pn = nullptr; }
    ctor<Y>(self<Y> const& r) noexcept requires(...) : px{r.px}, pn{r.pn} { sp_assert_convertible<Y,T>(); if (pn) pn->add_ref(); }
    ctor<Y>(self<Y> && r) noexcept requires(...) : px{r.px}, pn{r.pn} { sp_assert_convertible<Y,T>(); r.px=nullptr; r.pn=nullptr; }
    ctor<Y>(self<Y> const& r, element_type* p) noexcept: px{p}, pn{r.pn} { if (pn) pn->add_ref(); }
    ctor<Y>(self<Y> && r, element_type* p) noexcept: px{p}, pn{r.pn} { r.px = nullptr; r.pn = nullptr; }
    self& operator=<Y>(self<Y> const& r) noexcept { self{r}.swap(*this); return *this; }
    self& operator=<Y>(self<Y>&& r) noexcept { self{std::move(r)}.swap(*this); return *this; }
    self& operator=(nullptr_t) noexcept { self{}.swap(*this); return *this; }
    self& operator=<Y,D>(std::unique_ptr<Y,D>&& r) { self{std::move(r)}.swap(*this); return *this; }
    void reset() noexcept { self{}.swap(*this); }
    void reset<Y,[D],[A]>(Y* p, <D d>, <A a>) { self{p, <d>, <a>}.swap(*this); }
    void reset(self<Y> {const&|&&} r, element_type* p) noexcept { self{<std::move>(r),p}.swap(*this); }
    sp_dereference<T>::type operator*() const noexcept { return *px; }
    sp_member_access<T>::type operator->() const noexcept { return px; }
    sp_array_access<T>::type operator[](ptrdiff_t i) const noexcept { return {px[i]}; }
    element_type* get() const noexcept { return px; }
    explicit operator bool() const noexcept { return px!=nullptr; }
    long local_use_count() const noexcept { return pn ? pn->local_use_count() : 0; }
    operator {shared_ptr|weak_ptr}<Y>() const noexcept requires(...) { sp_assert_convertible<T,Y>();
        if (pn) return {sp_internal_cosntructor_tag{}, px, pn->local_cb_get_shared_count()}; else return {};
    }
    void swap(self& other) noexcept { std::swap(px, r.px); std::swap(pn, r.pn); }
    bool owner_before<Y>(self<Y> const& r) const noexcept { return std::less<local_counted_base*>{}(pn, r.pn); }
    bool owner_equals<Y>(self<Y> const& r) const noexcept { return pn == r.pn; }

    friend bool operator==(self const& a, self const& b) noexcept { return a.get() == b.get(); } // and !=, and comp with nullptr_t, shared_ptr
    friend bool operator<(self const& a, self const& b) noexcept { return a.owner_before(b); }
    friend void swap(self& a, self& b) noexcept { a.swap(b); }
    friend self static_pointer_cast<U>(self<U> {const&|&&} r) noexcept // all 4 casts
    { auto p = (element_type*)r.get(); return {<std::move>(r),p}; }
    friend element_type* get_pointer(self const& p) noexcept { return p.get(); }
    friend std::basic_ostream<Ch,Tr>& operator<<(std::basic_ostream<Ch,Tr>& os, self const& p) { return os << p.get(); }
    friend D* get_deleter<D>(self const& p) noexcept { return get_deleter<D>(shared_ptr<T>{p}); }
};
size_t hash_value<T>(local_shared_ptr<T> const& p) noexcept { return hash<local_shared_ptr<T>::element_type*>{}(p.get()); }
struct std::hash<local_shared_ptr<T>> { size_t operator()(local_shared_ptr<T> const& p) const noexcept { return std::hash<local_shared_ptr<T>::element_type*>{}(p.get()); } };
```

------
### Atomic `shared_ptr`

```c++
class atomic_shared_ptr { // no copy
    shared_ptr<T> p_; mutable spinlock l_;
    bool compare_exchange(shared_ptr<T>& v, shared_ptr<T> w) noexcept {
        l_.lock();
        if (p_._internal_equiv(v)) { p_.swap(w); l_.unlock(); return true; }
        else { shared_ptr<T> tmp{p_}; l_.unlock(); tmp.swap(v); return false; }
    }
public: constexpr ctor() noexcept : l_{ATOMIC_FLAG_INIT} {}
    ctor(shared_ptr<T> p) noexcept : p_{std::move(p)}, l_{ATOMIC_FLAG_INIT} {}
    self& operator=(shared_ptr<T> r) noexcept { spinlock::scoped_lock lock{l_}; p_.swap(r); return *this; }
    constexpr bool is_lock_free() const noexcept { return false; }
    shared_ptr<T> load<[M]>(<M>) const noexcept { spinlock::scoped_lock lock{l_}; return p_; }
    operator shared_ptr<T>() const noexcept { spinlock::scoped_lock lock{l_}; return p_; }
    void stored<[M]>(shared_ptr<T> r, <M>) noexcept { spinlock::scoped_lock lock{l_}; p_.swap(r); }
    shared_ptr<T> exchange<[M]>(shared_ptr<T> r, <M>) noexcept { store(r); return std::move(r); }
    bool compare_exchange_{weak|strong}<[M]>(shared_ptr<T>& v, shared_ptr<T> const& w, <M>, <M>) noexcept { return compare_exchange(v, w); }
    bool compare_exchange_{weak|strong}<[M]>(shared_ptr<T>& v, shared_ptr<T> && w, <M>, <M>) noexcept { return compare_exchange(v, std::move(w)); }
};
```

------
### Intrusive RefCounted Pointer

```c++
struct sp_adl_block::thread_unsafe_counter {
    using type = unsigned int;
    static unsigned load(unsigned const& counter) noexcept { return counter; }
    static void increment(unsigned& counter) noexcept { ++counter; }
    static unsigned decrement(unsigned& counter) noexcept {return --counter; }
};
struct sp_adl_block::thread_safe_counter {
    using type = atomic_count;
    static unsigned load(atomic_count const& counter) noexcept { return (unsigned)(long)counter; }
    static void increment(unsigned& counter) noexcept { ++counter; }
    static unsigned decrement(unsigned& counter) noexcept {return (unsigned)--counter; }
};

class sp_adl_block::intrusive_ref_counter<T,Pol> {
    mutable Pol::type m_ref_counter{0};
protected: ~dtor()=default;
public: ctor() noexcept{} ctor(self const&) noexcept{} self& operator=(self const&) noexcept { return *this; }
    unsigned use_count() const noexcept { return Pol::load(m_ref_counter); }

    friend void intrusive_ptr_add_ref(const self* p) noexcept { Pol::increment(p->m_ref_counter); }
    friend void intrusive_ptr_release(const self* p) noexcept { if (Pol::decrement(p->m_ref_counter) == 0) delete (const T*)p; }
};

class intrusive_ptr<T> { // all constexpr
    T* px{nullptr};
public: using element_type = T;
    ctor() noexcept{}
    ctor(T* p, bool add_ref=true): px{p} { if (px && add_ref) intrusive_ptr_add_ref(px); }
    ctor<U>(self<U> const& rhs) : px{rhs.get()} { if (px) intrusive_ptr_add_ref(px); }
    ~dtor() { if (px) intrusive_ptr_release(px); }
    self& operator=<U>(self<U> const& rhs) { self{rhs}.swap(*this); return *this; }
    ctor<U>(self<U> && rhs) noexcept : px{rhs.px} { rhs.px = nullptr; }
    self& operator=<U>(self<U> && rhs) { self{std::move(rhs)}.swap(*this); return *this; }
    self& operator=(T* rhs) { self{rhs}.swap(*this); return *this; }
    void reset() { self{}.swap(*this); }
    void reset(T* rhs, <bool add_ref>) { self{rhs, <add_ref>}.swap(*this); }
    T* get() const noexcept { return px; }
    T* detach() noexcept { T* ret = px; px=nullptr; return ret; }
    T& operator*() const noexcept { return *px; }
    T* operator->() const noexcept { return px; }
    explicit operator bool() const noexcept { return px != nullptr; }
    void swap(self& rhs) noexcept { T* tmp=px; px=rhs.px; rhs.px=tmp; }

    friend bool operator==<T,U>(self<T> const& a, self<U> const& b) noexcept { return a.get()==b.get(); } // and !=, and compare with U*, nullptr_t
    friend bool operator<(self const& a, self const& b) noexcept { return std::less<T*>{}(a.get(), b.get()); }
    friend void swap(self& a, self& b) { a.swap(b); }
    friend T* get_pointer(self const& p) noexcept { return p.get(); }
    friend self static_pointer_cast<U>(self<U> const& r) { return static_cast<T*>(p.get()); } // and const_cast, dynamic_cast
    friend self static_pointer_cast<U>(self<U> && r) noexcept { return {(T*)p.detach(), false}; } // and const_cast
    friend self dynamic_pointer_cast<U>(self<U> && r) noexcept
    { T* p2 = dynamic_cast<T*>(p.get()); self r{p2, false}; if (p2) p.detach(); return r; }
    friend std::basic_ostream<Ch,Tr>& operator<<(std::basic_ostream<Ch,Tr>& os, self const& p) { return os << p.get(); }
};
size_t hash_value<T>(intrusive_ptr<T> const& p) noexcept { return hash<T*>{}(p.get()); }
struct std::hash<intrusive_ptr<T>> { size_t operator()(intrusive_ptr<T> const& p) const noexcept { return std::hash<T*>{}(p.get()); } };
```

------
### `enable_shared_from_this`

```c++
class enable_shared_from_this<T> {
    mutable weak_ptr<T> weak_this_;
    void _internal_accept_owner<X,Y>(shared_ptr<X> const* ppx, Y* py) const noexcept
    { if (weak_this_.expired()) weak_this_ = shared_ptr<T>{*ppx, py}; }
protected: // defaulted ctor, copy ctor, copy assign, dtor
public: shared_ptr<T [const]> shared_from_this() <const> { return {weak_this_}; }
    weak_ptr<T[const]> weak_from_this() <const> noexcept { return weak_this_; }
};
class enable_shared_from: public enable_shared_from_this<self> {};
shared_ptr<T> shared_from<T>(T* p) { return {p->shared_from_this(), p}; }
weak_ptr<T> weak_from<T>(T* p) { return {p->weak_from_this(), p}; }

class enable_shared_from_raw {
    mutable weak_ptr<void const volatile> weak_this_; mutable shared_ptr<void const volatile> shared_this_;
    void _internal_accept_owner<X,Y>(shared_ptr<X> const* ppx, Y* py) const noexcept {
        if (weak_this_.expired()) weak_this_ = *ppx;
        else if (shared_this_.use_count() != 0) {
            auto pd = get_deleter<esft2_deleter_wrapper>(shared_this_);
            pd->set_deleter(*ppx); ppx->reset(shared_this_, ppx->get()); shared_this_.reset();
        }
    }
    void init_if_expired() const { if (weak_this_.expired()) { shared_this_.reset(nullptr, esft2_deleter_wrapper{}); weak_this_ = shared_this_; } }
    void init_if_empty() const { if (weak_this_.empty()) { shared_this_.reset(nullptr, esft2_deleter_wrapper{}); weak_this_ = shared_this_; } }
    shared_ptr<void const volatile> shared_from_this() const <volatile> { init_if_expired(); return {weak_this_}; }
    weak_ptr<void const volatile> weak_from_this() const <volatile> { init_if_empty(); return weak_this_; }
protected: // defaulted ctor, copy ctor, copy assign, dtor
};
shared_ptr<T> shared_from_raw<T>(T* p) { return {p->shared_from_this(), p}; }
weak_ptr<T> weak_from_raw<T>(T* p) { return {p->weak_from_this(), p}; }
```

------
### `make_unique`, `make_shared`, `make_local_shared`

```c++
std::unique_ptr<T> make_unique<T,...Args>(Args&&...args) requires(!std::is_array_v<T>) { return {new T{std::forward<Args>(args)...}}; }
std::unique_ptr<T> make_unique<T>(std::remove_reference_t<T>&& value) requires(!std::is_array_v<T>) { return {new T{std::move(value)}}; }
std::unique_ptr<T> make_unique_noinit<T>() requires(!std::is_array_v<T>) { return {new T}; }
std::unique_ptr<T> make_unique<T>(size_t size) requires(sp_is_unbounded_array<T>::value) { return {new std::remove_extent_t<T>[size]{}}; }
std::unique_ptr<T> make_unique_noinit<T>(size_t size) requires(sp_is_unbounded_array<T>::value) { return {new std::remove_extent_t<T>[size]}; }

struct detail::sp_alloc_size<T> { static constexpr size_t value = 1; };
struct detail::sp_alloc_size<T[]> { static constexpr size_t value = sp_alloc_size<T>::value; };
struct detail::sp_alloc_size<T[n]> { static constexpr size_t value = n*sp_alloc_size<T>::value; };
struct detail::sp_alloc_result<T> { using type = T; };
struct detail::sp_alloc_result<T[n]> { using type = T[]; };
struct detail::sp_alloc_value<T> { using type = std::remove_cv_t<std::remove_extent_t<T>>; };
class detail::sp_alloc_ptr<T,P> {
    P p_{};
public: using element_type = T;
    ctor(<nullptr_t>) noexcept{}    ctor(size_t, P p) noexcept :p_{p}{}
    T& operator*() const{ return *p_; } T* operator->() const noexcept{ return to_address(p_); }
    explicit operator bool() const noexcept { return !!p_; } bool operator!() const noexcept { return !p_; }
    P ptr() const noexcept { return p_; }
    static constexpr size_t size() noexcept { return 1; }
};
class detail::sp_alloc_ptr<T[],P> {
    P p_{}; size_t n_;
public: using element_type = T;
    ctor(<nullptr_t>) noexcept{}    ctor(size_t n, P p) noexcept :p_{p}, n_{n}{}
    T& operator[](size_t i) const { return p_[i]; }
    explicit operator bool() const noexcept { return !!p_; } bool operator!() const noexcept { return !p_; }
    P ptr() const noexcept { return p_; }
    size_t size() noexcept { return n_; }
};
class detail::sp_alloc_ptr<T[n],P> {
    P p_{};
public: using element_type = T;
    ctor(<nullptr_t>) noexcept{}    ctor(size_t, P p) noexcept :p_{p}{}
    T& operator[](size_t i) const { return p_[i]; }
    explicit operator bool() const noexcept { return !!p_; } bool operator!() const noexcept { return !p_; }
    P ptr() const noexcept { return p_; }
    static constexpr size_t size() noexcept { return n; }
};
bool detail::operator==<T,P>(const sp_alloc_ptr<T,P>& lhs, const sp_alloc_ptr<T,P>& rhs) { return lhs.ptr() == rhs.ptr(); } // and !=, and compare with nullptr_t

class alloc_deleter<T,A> : empty_value<allocator_rebind_t<A,sp_alloc_value<T>::type>> {
    using allocator = base::type;
public: using pointer = sp_alloc_ptr<allocator_pointer<allocator>::type>;
    explicit ctor(const allocator& a) noexcept : base{empty_init_t{}, a} {}
    void operator()(pointer p) {
        if constexpr (std::is_array_v<T>) alloc_destroy(a, boost::to_address(p));
        else alloc_destroy_n(a, boost::first_scalar(boost::to_address(p)), n*sp_alloc_size<A::value_type>::value);
    }
};
using alloc_noinit_deleter<T,A> = alloc_deleter<T,noinit_adaptor<A>>;

class detail::sp_alloc_make<T,A> {
    using pointer = allocator_pointer<allocator>::type;
    allocator a_; size_t n_; pointer p_;
public: using allocator = allocator_rebind<A,sp_alloc_value<T>:::type>::type;
    using type = unique_ptr<sp_alloc_result<T>::type, deleter>;
    ctor(const A& a, size_t n) : a_{a}, n_{n}, p_{a_.allocate(n)} {}
    ~dtor() { if (p_) a_.deallocate(p_, n_); }
    allocator::value_type* get() const noexcept { return boost::to_address(p_); }
    allocator& state() noexcept { return a_; }
    type release() noexcept { pointer p = p_; p = {}; return type{deleter::pointer{n_,p}, deleter{a_}}; }
};

std::unique_ptr<T,alloc_deleter<T,A>> allocate_unique<T,A,...Args>(const A& alloc, Args&&...args) requires(!std::is_array_v<T>)
{ sp_alloc_make<T,A> c{alloc,1}; alloc_construct(c.state(), c.get(), std::forward<Args>(args)...); return c.release(); }
std::unique_ptr<T,alloc_deleter<T,A>> allocate_unique<T,A>(const A& alloc, sp_type_identity<T>::type&& value) requires(!std::is_array_v<T>)
{ sp_alloc_make<T,A> c{alloc,1}; alloc_construct(c.state(), c.get(), std::move(value)); return c.release(); }
std::unique_ptr<T,alloc_deleter<T,A>> allocate_unique_noinit<T>(const A& alloc) requires(!std::is_array_v<T>)
{ return allocate_unique<T,noinit_adaptor<A>>(alloc); }
std::unique_ptr<T,alloc_deleter<T,A>> allocate_unique<T,A>(const A& alloc, size_t size) requires(sp_is_unbounded_array<T>::value)
{ sp_alloc_make<T,A> c{alloc,size}; alloc_construct_n(c.state(), first_scalar(c.get()), size*sp_alloc_size<T>::value); return c.release(); }
std::unique_ptr<T,alloc_deleter<T,A>> allocate_unique<T,A>(const A& alloc) requires(sp_is_bounded_array<T>::value)
{ sp_alloc_make<T,A> c{alloc,std::extent_v<T>}; alloc_construct_n(c.state(), first_scalar(c.get()), std::extent_v<T>*sp_alloc_size<T>::value); return c.release(); }
std::unique_ptr<T,alloc_deleter<T,A>> allocate_unique_noinit<T,A>(const A& alloc, size_t size) requires(sp_is_unbounded_array<T>::value)
{ return allocate_unique<T,noinit_adaptor<A>>(alloc, size); }
std::unique_ptr<T,alloc_deleter<T,A>> allocate_unique_noinit<T,A>(const A& alloc) requires(sp_is_bounded_array<T>::value)
{ return allocate_unique<T,noinit_adaptor<A>>(alloc); }
std::unique_ptr<T,alloc_deleter<T,A>> allocate_unique<T,A>(const A& alloc, size_t size, std::remove_extent_t<T> const& value) requires(sp_is_unbounded_array<T>::value)
{ sp_alloc_make<T,A> c{alloc,size}; alloc_construct_n(c.state(), first_scalar(c.get()), size*sp_alloc_size<T>::value, first_scalar(&value), sp_alloc_size<std::remove_extent_t<T>>::value); return c.release(); }
std::unique_ptr<T,alloc_deleter<T,A>> allocate_unique<T,A>(const A& alloc, std::remove_extent_t<T> const& value) requires(sp_is_bounded_array<T>::value)
{ sp_alloc_make<T,A> c{alloc,std::extent_v<T>}; alloc_construct_n(c.state(), first_scalar(c.get()), std::extent_v<T>*sp_alloc_size<T>::value, first_scalar(&value), sp_alloc_size<std::remove_extent_t<T>>::value); return c.release(); }
auto get_allocator_pointer<T,U,A>(const std::unique_ptr<T,alloc_deleter<U,A>>& p) noexcept -> allocator_pointer<allocator_rebind<A,sp_alloc_value<T>::type>::type>::type { return p.get().ptr(); }

struct detail::sp_aligned_storage<n,a> { union type { char data_[n]; sp_type_with_alignment<a>::type align_; }; };
class detail::sp_ms_deleter {
    bool initialized_{false}; sp_aligned_storage<sizeof(T), alignof(T)>::type storage_;
    void destroy() noexcept { if (initialized) ((T*)storage_.data_)->~T(); initialized = false; }
public: ctor() noexcept{}  explicit ctor<A>(A {const&|&&}) noexcept{}
    ~dtor() noexcept { destroy(); }     void operator()(T*) noexcept { destroy(); }
    static void operator_fn(T*) noexcept {}
    void* address() noexcept { return storage_.data_; }
    void set_initialized() noexcept { initialized_ = true; }
};
class detail::sp_as_deleter<T,A> {
    bool initialized_{false}; sp_aligned_storage<sizeof(T), alignof(T)>::type storage_; A a_;
    void destroy() noexcept { if (initialized) ((T*)storage_.data_)->~T(); std::allocator_traits<A>::destroy(a_,p); initialized = false; }
public: ctor(A const& a) noexcept : a_{a}{}     ctor(self const& r) noexcept : a_{r.a_}{}
    ~dtor() noexcept { destroy(); }     void operator()(T*) noexcept { destroy(); }
    static void operator_fn(T*) noexcept {}
    void* address() noexcept { return storage_.data_; }
    void set_initialized() noexcept { initialized_ = true; }
};
struct detail::sp_if_not_array<T> { using type = shared_ptr<T>; };
struct detail::sp_if_not_array<T[<n>]>{};

constexpr detail::SP_MSD = sp_inplace_tag<sp_ms_deleter<T>>{};
sp_if_not_array<T>::type make_shared_noinit<T>() {
    shared_ptr<T> pt{nullptr, SP_MSD};
    auto pd = (sp_ms_deleter<T>*)pt._internal_get_untyped_deleter();
    auto pv = pd->address(); ::new(pv) T; pd->set_initialized(); T* pt2 = (T*)pv;
    sp_enable_shared_from_this(&pt, pt2, pt2); return {pt, pt2};
}
sp_if_not_array<T>::type allocate_shared_noinit<T,A>(A const& a) {
    shared_ptr<T> pt{nullptr, SP_MSD, a};
    auto pd = (sp_ms_deleter<T>*)pt._internal_get_untyped_deleter();
    auto pv = pd->address(); ::new(pv) T; pd->set_initialized(); T* pt2 = (T*)pv;
    sp_enable_shared_from_this(&pt, pt2, pt2); return {pt, pt2};
}
sp_if_not_array<T>::type make_shared<T,...Args>(Args&&...args) {
    shared_ptr<T> pt{nullptr, SP_MSD};
    auto pd = (sp_ms_deleter<T>*)pt._internal_get_untyped_deleter();
    auto pv = pd->address(); ::new(pv) T{std::forward<Args>(args)...}; pd->set_initialized(); T* pt2 = (T*)pv;
    sp_enable_shared_from_this(&pt, pt2, pt2); return {pt, pt2};
}
sp_if_not_array<T>::type allocate_shared<T,A,...Args>(A const&a, Args&&...args) {
    using A2 = std::allocator_traits<A>::rebind_alloc<T>;  A2 a2{a};
    using D = sp_as_deleter<T,A2>; shared_ptr<T> pt{nullptr, sp_inplace_tag<D>{}, a2};
    auto pd = (sp_ms_deleter<T>*)pt._internal_get_untyped_deleter();
    auto pv = pd->address(); std::allocator_traits<A2>::construct(a2, (T*)pv, std::forward<Args>(args)...); pd->set_initialized(); T* pt2 = (T*)pv;
    sp_enable_shared_from_this(&pt, pt2, pt2); return {pt, pt2};
}
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
