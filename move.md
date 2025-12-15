# Boost.Move

* lib: `boost/libs/move`
* repo: `boostorg/move`
* commit: `a541a8d`, 2024-12-13

------
#### Move Semantic

* `struct has_move_emulation_enabled<T>`, `struct has_move_emulation_disabled<T>`
* `RV_REF(T)`: `T&&`
* `RV_REF_BEG`: nothing, `RV_REF_END`: `&&`
* `COPY_ASSIGN_REF(T)`: `const T&`
* `FWD_REF(T)`: `T&&`
* `MOVE_REF(RetType,Ref)`: `Ref`
* `MOVE_BASE(BaseType,arg)`: `::boost::move((BaseType&)arg)`
* `MOVE_TO_LV(arg)`: `arg`
* `move`, `forward`: using `std::`
* `move_if_noexcept`

#### Iterators

* `class move_iterator<It>`, `make_move_iterator<It>(const It& it)`: adaptor to deref as rvalue reference.
* `class back_move_insert_iterator<C>`, `back_move_inserter<C>(C& x)`: move constructs element at back
* `class front_move_insert_iterator<C>`, `front_move_inserter<C>(C& x)`: move constructs element at front
* `class move_insert_iterator<C>`, `move_inserter<C>(C& x, C::iterator it)`: move constructs element

#### Traits

```c++
struct has_trival_destructor_after_move<T> : is_trivally_destructible<T>{};
struct has_nothrow_move<T> { static const bool value = is_nothrow_move_constructible<T>::value && is_nothrow_move_assignable<T>::value; };
```

#### Algorithms

```c++
OutIt move<InIt,OutIt>(InIt f, InIt l, OutIt result) { while (f!=l) { *result=boost::move(*f); ++f; ++result;} return result; }
OutIt move_backward<InIt,OutIt>(InIt f, InIt l, OutIt result) { while (f!=l) { --l; --result; *result=boost::move(*f); } return result; }
FwdIt move<InIt,FwdIt>(InIt f, InIt l, FwdIt r) {
    FwdIt back=r;
    try { while (f!=l) { void* addr= (void*)addressof(*r); new(addr) input_value_type{boost::move(*f)}; ++f; ++r; } }
    catch(...) { for (; back!=r; ++back) iterator_to_raw_pointer(back)->~dtor(); throw; }
    return r;
}
FwdIt uninitialized_copy_or_move<InIt,FwdIt>(InIt f, InIt l, FwdIt r) requires (is_move_iterator<I>) { boost::uninitialized_move(f,l,r); }
FwdIt copy_or_move<InIt,FwdIt>(InIt f, InIt l, FwdIt f) requires (is_move_iterator<InIt>) { boost::move(f,l,r); }
FwdIt uninitialized_copy_or_move<InIt,FwdIt>(InIt f, InIt l, FwdIt r) requires (!is_move_iterator<InIt>) { return std::uninitialized_copy(f,l,r); }
FwdIt copy_or_move<InIt,FwdIt>(InIt f, InIt l, FwdIt f) requires (!is_move_iterator<InIt>) { return std::copy(f,l,r); }

void adaptive_merge<RandIt,Compare>(RandIt f, RandIt m, RandIt l, Compare comp, iterator_traits<RandIt>::value_type* uninitialized=0, iter_size<RandIt>::type uninitialized_len=0);
void adaptive_sort<RandIt,RandRawIt,Compare>(RandIt f, RandIt l, Compare comp, RandRawIt uninitialized, iter_size<RandIt>::type uninitialized_len);
void adaptive_sort<RandIt,Compare>(RandIt f, RandIt l, Compare comp);

struct antistable<Comp> {
    explicit ctor(Comp& comp); ctor(const self& other);
    bool operator()<U,V>(const U& u, const V& v) { return !m_comp(v,u); }
    const Comp& get() const { return m_comp; }
private: Comp& m_comp;
};
Comp unantistable<Comp>(Comp comp) { return comp; }
Comp unantistable<Comp>(antistable<Comp> comp) { return comp.get(); }
struct negate<Comp> {
    ctor(){}; explicit ctor(Comp comp): m_comp{comp}{}
    bool operator()<T1,T2>(const T1& l, const T2& r) { return !m_comp(l,r); }
private: Comp m_comp;
};
struct inverse<Comp> {
    ctor(){}; explicit ctor(Comp comp): m_comp{comp}{}
    bool operator()<T1,T2>(const T1& l, const T2& r) { return m_comp(r,l); }
private: Comp m_comp;
};

FwdIt unique<FwdIt,BinPred>(FwdIt f, FwdIt l, BinPred pred);
```

#### ADL Swap Invoke

```c++
void adlswap::swap_proxy<T>(T& x, T& y) { using std::swap; swap(x,y); }
void adlswap::swap_proxy<T,n>(T(&x)[n], T(&y)[n]) { for (size_t i=0; i<n; ++i) adlswap::swap_proxy(x[i],y[i]); }
void adl_move_swap<T>(T& x, T& y) { adlswap::swap_proxy(x,y); }
FwdIt2 adl_move_swap_ranges<FwdIt1,FwdIt2>(FwdIt1 f1, FwdIt1 l1, FwdIt2 f2)
{ while (f1!=l1) { adl_move_swap(*f1,*f2); ++f1; ++f2; } return f2; }
BiIt2 adl_move_swap_ranges_backward<BiIt1,BiIt2>(BiIt1 f1, BiIt1 l1, BiIt2 l2)
{ while (f1!=l1) adl_move_swap(*(--l1),*(--l2)); return l2; }
void adl_move_iter_swap<FwdIt1,FwdIt2>(FwdIt1 a, FwdIt2 b) { adl_move_swap(*a, *b); }
```

#### Unique Ownership Pointer

```c++
struct default_delete<T> {
    constexpr ctor()=default;
    using element_type = remove_extent<T>::type;
    ctor<U>(const self<U>&) noexcept{}
    self& operator=<U>(const self<U>&) noexcept { return *this; }
    void operator()<U>(U* ptr) const noexcept { auto p=(element_type*)ptr; call_delete(p, is_array_del<is_array<T>::value>()); }
    void operator()(nullptr_t) const noexcept {}
};
struct deleter_types<D> {
    using del_<c>ref = add_<const>_lvalue_reference<D>::type;
    using deleter_arg_type1 = if_c<is_lvalue_reference<D>::value, D, del_cref>::type;
    using deleter_arg_type2 = remove_reference<D>::type&&;
};
struct unique_ptr_data<P,D,b=is_unary_function<D>||is_reference<D>> { // no copy
    using deleter_arg_type1 = deleter_types<D>::deleter_arg_type1;
    using del_<c>ref = deleter_types<D>::del_<c>ref;
    ctor() noexcept; ctor(P p) noexcept; ctor(P p, deleter_arg_type1 d1) noexcept;
    ctor<U>(P p, FWD_REF(U) d1) noexcept;
    del_<c>ref deleter() <const> { return d; }
    P m_p; D d;
};
struct unique_ptr_data<P,D,false> : private D { // no copy
    using deleter_arg_type1 = deleter_types<D>::deleter_arg_type1;
    using del_<c>ref = deleter_types<D>::del_<c>ref;
    ctor() noexcept; ctor(P p) noexcept; ctor(P p, deleter_arg_type1 d1) noexcept;
    ctor<U>(P p, FWD_REF(U) d1) noexcept;
    del_<c>ref deleter() <const> noexcept { return (del_<c>ref)*this; }
    P m_p;
};
struct get_element_type<T>;
struct get_cvelement<T>;
struct is_same_cvelement_and_convertible<P1,P2>;
struct is_unique_ptr_convertible<isArray,FromPtr,ThisPtr>;
struct enable_up_ptr<T,FromPtr,ThisPtr,Type=nat>;
struct unique_moveconvert_assignable<T,D,U,E>;
struct enable_up_moveconv_assign<T,D,U,E,Type=nat>;
struct unique_deleter_is_initializable<D,E,isRef=is_reference<D>::value>;
class is_rvalue_convertible<T,U>;
struct enable_up_moveconv_constr<T,D,U,E,Type=nat>;

class unique_ptr<T,D=default_delete<T>> { // no copy
    unique_ptr_data<pointer,D> m_data{};
public:
    using pointer = pointer_type<T,D>::type;
    using element_type = remove_extent<T>::type;
    using deleter_type = D;
    constexpr ctor() noexcept; constexpr ctor(nullptr_t) noexcept;
    explicit ctor<Ptr>(Ptr p) noexcept;
    ctor<Ptr>(Ptr p, deleter_types::deleter_arg_type1 d1) noexcept;
    ctor(nullptr_t, deleter_types::deleter_arg_type1 d1) noexcept;
    ctor<Ptr>(Ptr p, deleter_types::deleter_arg_type2 d2) noexcept;
    ctor(nullptr_t, deleter_types::deleter_arg_type2 d2) noexcept;
    ctor(self&& u) noexcept; ctor<U,E>(self<U,E>&& u) noexcept;
    ~dtor() { if (m_data.m_p) m_data.deleter()(m_data.m_p); }
    self& operator=(self&& u) noexcept { reset(u.release()); m_data.deleter()=move_if_not_lvalue_reference<D>(u.get_deleter()); return *this; }
    self& operator=<U,E>(self<U,E>&& u) noexcept  { reset(u.release()); m_data.deleter()=move_if_not_lvalue_reference<E>(u.get_deleter()); return *this; }
    self& operator=(nullptr_t) noexcept { reset(); return *this; }
    element_type& operator*() const noexcept { return *m_data.m_p; }
    element_type& operator[](size_t i) const noexcept { return m_data.m_p[i]; }
    pointer operator->() const noexcept { return m_data.m_p; }
    pointer get() const noexcept { return m_data.m_p; }
    const D& get_deleter() const noexcept { m_data.deleter(); }
    explicit operator bool() const noexcept { return m_data.m_p; }
    pointer release() noexcept { pointer tmp = m_data.m_p; m_data.m_p = pointer{}; return tmp; }
    void reset<Ptr>(Ptr p) noexcept { pointer tmp = m_data.m_p; m_data.m_p = p; if (tmp) m_data.deleter()(tmp); }
    void reset() noexcept { reset(pointer{}); } void reset(nullptr_t) noexcept { reset(); }
    void swap(self& u) noexcept { adl_move_swap(m_data.m_p, u.m_data.m_p); adl_move_swap(m_data.deleter(), u.m_data.deleter()); }
};
void swap<T,D>(unique_ptr<T,D>& x, unique_ptr<T,D>& y) noexcept { x.swap(y); }
bool operator{==|!=|<|<=|>|>=}<T1,D1,T2,D2>(const unique_ptr<T1,D1>& x, const unique_ptr<T2,D2>& y);
bool operator{==|!=|<|<=|>|>=}<T,D>(const unique_ptr<T,D>& x, nullptr_t);
bool operator{==|!=|<|<=|>|>=}<T,D>(nullptr_t, const unique_ptr<T,D>& x);

unique_ptr<T> make_unique_<nothrow> <T,...Args>(Args&&...args);
unique_ptr<T> make_unique_<nothrow>_definit <T>();
unique_ptr<T> make_unique_<nothrow>_<definit> <T>(size_t n);
```

-------
#### Configuration

* `DISABLE_FORCEINLINE`, `FORCEINLINE_IS_BOOST_FORCELINE`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`, `<boost/config/pragma_message.hpp>`

------
### Standard Facilities

Language: Rvalue-reference, move-semantic (C++11)
Library: `unique_ptr` (C++11)
