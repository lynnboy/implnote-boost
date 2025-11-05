# Boost.PtrContainer

* lib: `boost/libs/ptr_container`
* repo: `boostorg/ptr_container`
* commit: `ea15c57`, 2025-06-28

------
### Pointer Based Polimorphic Containers

* Main header: `<boost/ptr_container/ptr_container.hpp>`
* Serialization header: `<boost/ptr_container/ptr_container.hpp>`

------
#### Common Bits

```c++
class detail::scoped_deleter<Cont> {
    using size_type = Cont::size_type; using T = Cont::object_type;
    Cont& cont_; boost::scoped_array<T*> ptrs_; size_type stored_; bool released_;
public: ctor(Cont& cont, T** a, size_type size); ctor(Cont& cont, size_type size);
    ctor(Cont& cont, size_type n, const T& x); // null_policy_allocate_clone
    ctor<It>(Cont& cont, It first, It last); // null_policy_allocate_clone_from_iterator
    ~dtor(); // null_policy_deallocate_clone
    void add(T* t); void release();
    T** begin(); T** end();
};

struct detail::deleter_base<T> { using deleter = void(*)(T*); static deleter delete_; ctor(deleter d); void operator() (T* t); };
struct detail::scalar_deleter<T> : deleter_base<T>
{ ctor():base(do_delete){}; static void do_delete(T* t) { checked_delete(t); } };
struct detail::array_deleter<T> : deleter_base<remove_bounds_t<T>>
{ ctor():base(do_delete){}; static void do_delete(remove_bounds_t<T>* t) { checked_array_delete(t); } };
using detail::default_deleter<T> = if_<is_array<T>, array_deleter<T>, scalar_deleter<T>>::type;

struct detail::is_array_convertible<T,U>; // both array of same type, ignoring cv
using detail::is_smart_ptr_convertible<T,U> = if_<is_array<T>, is_array_convertible<T,U>, is_convertible<T*,U*>>;

class detail::move_source<Ptr> { Ptr& ptr_; ctor(const self&)=delete; public: ctor(self& ptr):ptr_{ptr}{} Ptr& ptr() const {return ptr_;} };
move_source<T> detail::move(T& x) { return {x}; }

class detail::static_move_ptr<T,D=default_deleter<T>> {
    using impl_type = compressed_pair<element_type*,D>;
public: using element_type = remove_bounds_t<T>; using deleter_type = D;
    using deleter_<const>_reference = impl_type::second_<const>_reference;
    ctor(); ctor(<const> self&); ctor(const move_source<self>& src);
    ctor<TT>(TT* tt, D d); ~dtor(); self& operator=(self rhs);
    element_type* get() const;
    <const> element_type& operator*() <const>; <const> element_type* operator->() <const>;
    element_type* release();
    void reset(); void reset<TT>(TT* tt, D d);
    explicit operator bool() const;
    void swap(self& p);
    deleter_<const>_reference get_deleter() <const>;
    impl_type::first_<const>_reference ptr() <const>;
};

class bad_ptr_container_operation : public std::exception { const char* what_; ctor(const char*); const char* what() const noexcept override; };
class bad_index : public bad_ptr_container_operation {};
class bad_pointer : public bad_ptr_container_operation {};

T* new_clone<T>(const T& r) { return new T{r}; }
void delete_clone<T>(const T* r) { checked_delete(r); }
struct heap_clone_allocator {
    static U* allocate_clone<U>(const U& r) { return new_clone(r); }
    static void deallocate_clone<U>(const U* r) { delete_clone(r); }
};
struct view_clone_allocator {
    static U* allocate_clone<U>(const U& r) { return (U*)&r; }
    static void deallocate_clone<U>(const U* r) {}
};

struct nullable<T> { using type = T; };
using is_nullable<T> = is_convertible<T*, const nullable<T>*>;
struct remove_nullable<T> { using type = eval_if<is_nullable<T>, T, identity<T>>::type; };
struct detail::void_ptr<T> { using type = if_<is_const<remove_nullable<T>::type>, const void*, void*>; };

using detail::is_compatible<T,U> = is_same<remove_const_t<T>, remove_const_t<U>>;
struct void_ptr_iterator<VoidIt,T> { VoidIt iter_;
public: using value_type = remove_const_t<T>; using reference = T&; using pointer = T*;
    using difference_type = iterator_difference<VoidIt>::type; using iterator_category = iterator_category<VoidIt>::type;
    ctor(); ctor(VoidIt r); ctor<MutIt,MutT>(const self<MutIt,MutT>& r);
    T& operator*() const; T* operator->() const;
    self& operator++(); self operator++(int); self& operator--(); self operator--(int);
    self& operator+=(difference_type n); self& operator-=(difference_type n);
    T& operator[](difference_type n) const;
    VoidIt base() const;
    friend self operator+(self, difference_type n); friend self operator+(difference_type n, self); // and -
    friend difference_type operator-(self, self<VoidIt,U>) requires is_compatible<T,U>;
    friend bool operator==(const self&, const self<VoidIt,U>&) requires is_compatible<T,U>; // and !=, <, >, <=, >=
};

class indirect_fun<Fun> { Fun fun;
public: ctor(); ctor(Fun f);
    auto operator() <T> (const T& r) const -> decltype(...) { return fun(*r); }
    auto operator() <T,U> (const T& r, const U& r2) const -> decltype(...) { return fun(*r, *r2); }
};
indirect_fun<Fun> make_indirect_fun<Fun>(Fun f) { return {f}; }
```

------
#### Container Base Types

```c++
struct detail::dynamic_clone_deleter<Cont> { Cont* cont; void operator()<T> (const T* p) const { cont->get_clone_allocator().deallocate_clone(p); } };
struct detail::static_clone_deleter<CA> { void operator()<T> (const T* p) const { CA::deallocate_clone(p); } };

using detail::is_pointer_or_integral<T> = or_<is_pointer<T>, is_integral<T>>;
struct is_pointer_or_integral_tag{};
struct is_range_tag{};
struct sequence_tag{}; struct fixed_length_sequence_tag :sequence_tag{};
struct associative_container_tag{}; struct {ordered|unordered}_associative_container_tag :associative_container_tag{};

class reversible_ptr_container<Config,CA> {
    static constexpr bool allow_null = Config::allow_null, is_clone_allocator_empty = sizeof(CA)<sizeof(void*);
    using Ty_ = Config::value_type; using container_type = Config::void_container_type;
    container_type c_;

    void copy<It>(It f, It l); void copy(const self&);
    void copy_clones_and_release(scoped_deleter&);
    void clone_assign<FwdIt>(FwdIt f, FwdIt l); void clone_back_insert<FwdIt>(FwdIt f, FwdIt l);
    void remove_all();
    FwdIt advance<FwdIt>(FwdIt begin, size_type n);
protected: using scoped_deleter = scoped_deleter<self>;
    using ptr_<const>_iterator = container_type::<const>_iterator;
    <const> container_type& base() <const>;
    void insert_clones_and_release(scoped_deleter&, <iterator where>);
    void remove<U>(U* ptr); void remove<It>(It it, <It last>);
    static void enforce_null_policy(const Ty_* x, const char* msg);
public: using object_type = Ty_; using value_type = Ty_*; using pointer = Ty_*; using <const>_reference = <const> Ty&;
    using <const>_iterator = Config::<const>_iterator;
    using <const>_reverse_iterator = reverse_iterator<<const>_iterator>;
    using difference_type = container_type::difference_type; using size_type = container_type::size_type;
    using allocator_type = Config::allocator_type; using clone_allocator_type = CA;
    using auto_type = static_move_ptr<Ty_, if_c<is_clone_allocator_empty, static_clone_deleter<CA>, dynamic_clone_deleter<self>>>

    Ty_* null_policy_allocate_clone(const Ty_* x);
    Ty_* null_policy_allocate_clone_from_iterator<It>(It it);
    void null_policy_deallocate_clone(const Ty_* x);

    ctor(); explicit ctor(const allocator_type& a);
    ctor(const self&); ctor<C,V>(const self<C,V>&);
    explicit ctor<PtrCont>(std::unique_ptr<PtrCont> clone);
    self& operator=<PtrCont>(std::unique_ptr<PtrCont> clone); self& operator=(self r);
    ~dtor();

    explicit ctor(size_type n);
    ctor<Size>(Size n, unordered_associative_container_tag);
    ctor<Size>(Size n, <const allocator_type& a>, fixed_length_sequence_tag);
    ctor<InIt>(InIt f, InIt l, cosnt allocator_type& a={});
    ctor<Comp>(const Comp& comp, const allocator_type& a);
    ctor<FwdIt>(FwdIt f, FwdIt l, fixed_length_sequence_tag);
    ctor<Size,InIt>(Size n, InIt f, InIt l, fixed_length_sequence_tag);
    ctor<Comp>(const Comp& comp, const allocator_type& a, associative_container_tag);
    ctor<InIt>(InIt f, InIt l, associative_container_tag);
    ctor<InIt,Comp>(InIt f, InIt l, const Comp& comp, const allocator_type& a, associative_container_tag);
    ctor<Hash,Pred>(const Hash& h, const Pred& p, const allocator_type& a);
    ctor<InIt,Hash,Pred>(InIt f, InIt l, const Hash& h, const Pred& p, const allocator_type& a);

    allocator_type get_allocator() const; <const> clone_allocator_type& get_clone_allocator() <const>;
    <const>_iterator <c>{begin|end} () <const>; <const>_reverse_iterator <c>r{begin|end} () <const>
    void swap(self&)
    size_type size() const; size_type max_size() const; bool empty() const;
    bool operator==(const self&) const; // also !=, <, >, <=, >=

    iterator insert(iterator p, Ty_* x); iterator insert<U>(iterator p, std::unique_ptr<U> x);
    iterator erase(iterator p, <iterator last>); iterator erase<Range>(const Range&);
    void clear();
    auto_type release(iterator p);
    auto_type replace(iterator p, Ty_* x); auto_type replace<U>(iterator p, std::unique_ptr<U> x);
    auto_type replace(size_type idx, Ty_* x); auto_type replace<U>(size_type idx, std::unique_ptr<U> x);
};

struct detail::sequence_config<T,VoidPtrSeq> {
    using U =remove_nullable<T>::type; using value_type = U;
    using void_container_type = VoidPtrSeq; using allocator_type = VoidPtrSeq::allocator_type;
    using <const>_iterator = void_ptr_iterator<VoidPtrSeq::<const>_iterator, <const> U>;
    static <const> U* get_<const>_pointer<It>(void_ptr_iterator<It,U> i);
    static <const> U* get_<const>_pointer<It>(It i);
    static constexpr bool allow_null = is_nullable<T>::value;
};
bool is_null<It,T>(void_ptr_iterator<It,T> i);

class ptr_sequence_adapter<T,VoidPtrSeq,CA=heap_clone_allocator> : public reversible_ptr_container<sequence_config<T,VoidPtrSeq>,CA> {
    void range_check(iterator f, iterator l);
protected: class void_ptr_delete_if<Fun,Arg1> { Fun fun; bool operator()(void* r) const; };
public: // types from base
    using base::ctor; using base::operator=; // all except tagged other than `fixed_length_sequence_tag`
    void push_{back|front}(value_type x); void push_{back|front}<U>(std::unique_ptr<U> x);
    auto_type pop_{back|front}(); <const>_reference {front|back}()<const>;
    <const>_reference operator[](size_type n)<const>; <const>_reference at(size_type n)<const>;
    size_type capacity() const; void reserve(size_type n);
    void reverse();
    void assign<InIt>(InIt f, InIt l); void assign<Range>(const Range& r);
    void insert<InIt>(iterator p, InIt f, InIt l); void insert<Range>(iterator p, const Range& r) requires !is_pointer_or_integral<Range>;
    void transfer<PtrSeqAdapter>(iterator p, PtrSeqAdapter::iterator f, <PtrSeqAdapter::iterator l>, PtrSeqAdapter& from);
    void transfer<PtrSeqAdapter,Range>(iterator p, const Range& r, PtrSeqAdapter& from) requires !is_same_v<Range,PtrSeqAdapter::iterator>;
    void transfer<PtrSeqAdapter>(iterator p, PtrSeqAdapter& from);
    void transfer(iterator p, value_type* from, size_type n, bool delete_from=true);
    value_type* c_array();
    bool is_null(size_type idx) const;
    void <r>resize(size_type size, <value_type to_clone>);
    void sort(); void sort(iterator f, iterator l);
    void sort<Comp>(Comp comp); void sort<Comp>(iterator f, iterator l, Comp comp);
    void unique(); void unique(iterator f, iterator l);
    void unique<Comp>(Comp comp); void unique<Comp>(iterator f, iterator l, Comp comp);
    void erase_if<Pred>(Pred pred); void erase_if<Pred>(iterator f, iterator l, Pred pred);
    void merge(self& r); void merge<BinPred>(self& r, BinPred pred);
    void merge(iterator f, iterator l, self& from); void merge<BinPred>(iterator f, iterator l, self& from, BinPred pred);
};
```

------
#### Containers

```c++
class detail::ptr_array_impl<T,n,A=int> : array<T,n> { using allocator_type = A; ctor(A={}); ctor(size_t, T*, A={}); };
class ptr_array<T,n,CA=heap_clone_allocator> : public ptr_sequence_adapter<T,ptr_array_impl<void_ptr<T>::type,n>,CA> {
    using U = remove_nullable<T>::type;
    using base::xxx; // hide insert, erase, push, pop, transfer, get_allocator
public: using size_type = size_t; using value_type = U*; using pointer = U*; using <const>_reference = <const> U&; using auto_type = base::auto_type;
    ctor(); ctor(const self&); ctor<U>(const self<U,n>& r); explicit ctor(std::unique_ptr<self> r);
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    auto_type replace<idx>(U* r); auto_type replace<idx,V>(std::unique_ptr<V> r);
    auto_type replace(size_t idx, U* r); auto_type replace<V>(size_t idx, std::unique_ptr<V> r);
    using base::at; <const> T& at<idx>() <const>; 
    bool is_null(size_t idx) const; bool is_null<idx>() const;

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};

class ptr_deque<T,CA=heap_clone_allocator,A=std::allocator<void_ptr<T>::type>> : ptr_sequence_adapter<T,std::deque<void_ptr<T>::type,A>,CA> {
public: using size_type = base::size_type; using iterator = base::iterator; using const_reference = base::const_reference; using allocator_type = base::allocator_type;
    ctor(); ctor<U>(const self<U>& r); explicit ctor(const allocator_type& a); explicit ctor(std::unique_ptr<self> r);
    ctor<InIt>(InIt f, InIt l); ctor<InIt>(InIt f, InIt l, const allocator_type& a );
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    using base::release;

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};

class ptr_list<T,CA=heap_clone_allocator,A=std::allocator<void_ptr<T>::type>> : ptr_sequence_adapter<T,std::list<void_ptr<T>::type,A>,CA> {
    using U = remove_nullable<T>::type;
public: using size_type = base::size_type; using iterator = base::iterator; using const_reference = base::const_reference; using allocator_type = base::allocator_type;
    ctor(); ctor<U>(const self<U>& r); explicit ctor(const allocator_type& a); explicit ctor(std::unique_ptr<self> r);
    ctor<InIt>(InIt f, InIt l); ctor<InIt>(InIt f, InIt l, const allocator_type& a );
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    using base::release;
    using base::merge; void merge(self& x); void merge<Comp>(self& x, Comp comp);
    void sort(); void sort<Comp>(Comp comp);
    void erase_if<Pred>(Pred pred); void erase_if<Pred>(iterator f, iterator l, Pred pred);

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};

class ptr_vector<T,CA=heap_clone_allocator,A=void> : ptr_sequence_adapter<T,std::vector<void_ptr<T>::type, if_<is_same<A,void>,std::allocator<void_ptr<T>::type>,A>::type>,CA> {
public: using size_type = base::size_type; using iterator = base::iterator; using const_reference = base::const_reference; using allocator_type = base::allocator_type;
    ctor(); ctor<U>(const self<U>& r); explicit ctor(const allocator_type& a); explicit ctor(size_type n, const allocator_type& a={});
    explicit ctor(std::unique_ptr<self> r);
    ctor<InIt>(InIt f, InIt l); ctor<InIt>(InIt f, InIt l, const allocator_type& a );
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    using base::release;

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};

class ptr_circular_buffer<T,CA=heap_clone_allocator,A=std::allocator<void>> : ptr_sequence_adapter<T,boost::circular_buffer<void_ptr<T>::type,A>,CA> {
    using circular_buffer_type = boost::circular_buffer<void_ptr<T>::type,A>;
public: using value_type = base::value_type; using <const>_pointer = <const> value_type*;
    using size_type = base::size_type; using <const>_iterator = base::<const>_iterator; using allocator_type = base::allocator_type; using auto_type = base::auto_type;
    using <const>_array_range = std::pair<<const>_pointer,size_type>; using capacity_type = circular_buffer_type::capacity_type;
    ctor(); ctor(const self& r); ctor<U>(const self<U>& r); explicit ctor(capacity_type n, <const allocator_type& a>);
    explicit ctor(std::unique_ptr<self> r);
    ctor<FwdIt>(FwdIt f, FwdIt l); ctor<InIt>(capacity_type n, InIt f, InIt l);
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    using base::release;
    allocator_type& get_allocator() <const>;
    <const>_array_range array_{one|two}() <const>;
    pointer linearize();
    bool full() const;
    size_type reserve() const; void reserve(size_type n);
    capacity_type capacity() const; void <r>set_capacity(capacity_type new_cap);
    void <r>resize(size_type size, <value_type to_clone>);
    void assign<InIt>(<capacity_type cap>, InIt f, InIt l); void assign<Range>(const Range& r);
    void assign(<capacity_type cap>, size_type n, value_type to_clone);
    void push_{front|back}(value_type ptr); void push_{front|back}<U>(std::unique_ptr<U> ptr);
    iterator <r>insert(iterator pos, value_type ptr); iterator <r>insert<U>(iterator pos, std::unique_ptr<U> ptr);
    void <r>insert<InIt>(iterator pos, InIt f, InIt l); void <r>insert<Range>(iterator pos, const Range& r) requires !is_pointer_or_integral<Range>;
    iterator rerase(iterator pos, <iterator last>); iterator rerase<Range>(const Range& r);
    void rotate(const_iterator new_begin);
    void transfer<PtrSeqAdapter>(iterator pos, PtrSeqAdapter::iterator f, <PtrSeqAdapter::iterator l>, PtrSeqAdapter& from);
    void transfer<PtrSeqAdapter,Range>(iterator pos, const Range& r, PtrSeqAdapter& from) requires !is_same_v<Range,PtrSeqAdapter::iterator>;
    void transfer<PtrSeqAdapter>(iterator pos, PtrSeqAdapter& from);
    void transfer(iterator pos, value_type* from, size_type size, bool delete_from=true);
    value_type* c_array();

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};
```

------
### Dependency

#### Boost.Array

* `<boost/array.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.CircularBuffer

* `<boost/circular_buffer.hpp>`

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Core

* `<boost/checked_delete.hpp>`
* `<boost/core/invoke_swap.hpp>`
* `<boost/core/serialization.hpp>`
* `<boost/utility/enable_if.hpp>`

#### Boost.Iterator

* `<boost/pointee.hpp>`
* `<boost/next_prior.hpp>`
* `<boost/iterator/iterator_adapter.hpp>`
* `<boost/iterator/iterator_traits.hpp>`
* `<boost/iterator/reverse_iterator.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.Range

* `<boost/range/functions.hpp>`
* `<boost/range/iterator.hpp>`
* `<boost/range/iterator_range.hpp>`

#### Boost.SmartPtr

* `<boost/scoped_array.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

#### Boost.Unordered

* `<boost/unordered_map.hpp>`
* `<boost/unordered_set.hpp>`

#### Boost.Utility

* `<boost/compressed_pair.hpp>`
* `<boost/utility/compare_pointees.hpp>`
* `<boost/utility/result_of.hpp>`

------
### Standard Facilities
