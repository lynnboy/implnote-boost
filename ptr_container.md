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

struct detail::ref_pair<F,S> { using first_type = F; using second_type = S;
    const F& first; S second;
    ctor<F2,S2>(const std::pair<F2,S2>& p); ctor<RP>(const RP* rp);
    const self* operator->() const;
    friend bool operator==(self l, self r); // and !=, <, >, <=, >=
};
struct detail::ptr_map_iterator<It,F,S> : boost::iterator_adaptor<self, It, ref_pair<F,S>, use_default, ref_pair<F,S>> {
    ctor(); explicit ctor(const It& it); ctor<It2,F2,S2>(const self<It2,F2,S2>& r);
};

class detail::associative_ptr_container<Config,CA> : public reversible_ptr_container<Config,CA> {
    using container_type = Config::container_type;
public: using key_type = Config::key_type;
    using key_compare = Config::key_compare; using value_compare = Config::value_compare;
    using hasher = Config::hasher; using key_equal = Config::key_equal;
    using <const>_iterator = Config::<const>_iterator; using <const>_local_iterator = Config::<const>_local_iterator;
    ctor(); ctor<Size>(Size n, unordered_associative_container_tag);
    ctor<Comp,A>(const Comp& comp, const A& a); ctor<InIt,Comp,A>(InIt f, InIt l, const Comp& comp, const A& a);
    ctor<Hash,Pred,A>(const Hash& hash, const Pred& pred, const A& a); ctor<InIt,Hash,Pred,A>(InIt f, InIt l, const Hash& hash, const Pred& pred, const A& a);
    explicit ctor<PtrCont>(std::unique_ptr<PtrCont> r); ctor( const self& r); ctor<C,V>(const self<C,V>& r);
    self& operator=<PtrCont>(std::unique_ptr<PtrCont> r); self& operator=<PtrCont>(self r);

    key_compare key_comp() const; value_compare value_comp() const;
    size_type erase(const key_type& x);
    iterator erase(iterator f, iterator l); iterator erase<Range>(const Range& r) requires !is_convertible_v<Range&,key_type&>;
    using base::begin; using base::end; using base::cbegin; using base::cend;
protected:
    void {single|multi}_transfer<AssocPtrCont>(AssocPtrCont::iterator p, <AssocPtrCont::iterator last>, AssocPtrCont& from);
    <const>_reference front() <const>; <const>_reference back() <const>;
    hasher hash_function() const; key_equal key_eq() const;
    size_type bucket_count() const; size_type max_bucket_count() const; size_type bucket_size(size_type n) const;
    float load_factor() const; float max_load_factor() const; void max_load_factor(float factor);
    void rehash(size_type n);
    <const>_local_iterator {begin|end}(size_type n) <const>; const_local_iterator c{begin|end}(size_type n) const;
};

struct detail::select_value_compare<T> { using type = T::value_compare; };
struct detail::select_key_compare<T> { using type = T::key_compare; };
struct detail::select_hasher<T> { using type = T::hasher; };
struct detail::select_key_equal<T> { using type = T::key_equal; };
struct detail::select_iterator<T> { using type = T::iterator; };
struct detail::select_<const>_local_iterator<T> { using type = T::<const>_local_iterator; };

struct detail::map_config<T,VoidPtrMap,ordered> {
    using U =remove_nullable<T>::type; using value_type = U; using key_type = VoidPtrMap::key_type;
    using void_container_type = VoidPtrMap; using allocator_type = VoidPtrMap::allocator_type;
    using value_compare = mpl::eval_if_c<ordered, select_value_compare<VoidPtrMap>, identity<void>>::type;
    using key_compare = mpl::eval_if_c<ordered, select_key_compare<VoidPtrMap>, identity<void>>::type;
    using hasher = mpl::eval_if_c<ordered, identity<void>, select_hasher<VoidPtrMap>>::type;
    using key_equal = mpl::eval_if_c<ordered, identity<void>, select_key_equal<VoidPtrMap>>::type;
    using container_type = mpl::if_c<ordered, ordered_associative_container_tag, unordered_associative_container_tag>::type;
    using <const>_iterator = ptr_map_iterator<VoidPtrMap::<const>_iterator, key_type, <const> U* const>;
    using <const>_local_iterator = ptr_map_iterator<mpl::eval_if_c<ordered, select_iterator<VoidPtrMap>, select_<const>_local_iterator<VoidPtrMap>>::type, key_type, U* const>;
    static <const> U* get_<const>_pointer<It>(It i);
    static constexpr bool allow_null = is_nullable<T>::value;
};

class detail::ptr_map_adapter_base<T,VoidPtrMap,CA,ordered> : public associative_ptr_container<map_config<T,VoidPtrMap,ordered>,CA> {
    const_mapped_reference lookup(const key_type& key) const;
    mapped_reference insert_lookup(const key_type& key);
protected: size_type bucket(const key_type& k) const;
public: // all other types from base
    using mapped_type = base::value_type; using value_type = iterator_value<iterator>;
    using <const>_mapped_reference = base::<const>_reference;
    using <const>_reference = iterator_value<const_iterator>; using <const>_pointer = <const>_reference;
    using base::ctor; using base::operator=; // all except tagged other than `unordered_associative_container_tag`
    <const>_iterator find(const key_type& x) <const>;
    size_type count(const key_type& x) const;
    <const>_iterator {lower|upper}_bound(const key_type& x) <const>;
    iterator_range<<const>_iterator> equal_range(const key_type& x) <const>;
    mapped_reference operator[](const key_type& k); <const>_mapped_reference at(const key_type& k)<const>;
    auto_type replace(iterator p, mapped_type x); auto_type replace<U>(iterator p, std::unique_ptr<U> x);
};

class ptr_map_adapter<T,VoidPtrMap,CA=heap_clone_allocator,ordered=true> : public ptr_map_adapter_base<T,VoidPtrMap,CA,ordered> {
    void safe_insert(const key_type& k, auto_type ptr);
    void map_basic_clone_and_insert<It>(It f, It l);
public: // types from base: <const>_iterator, size_type, key_type, const_reference, auto_type, mapped_type
    using allocator_type = VoidPtrMap::allocator_type;
    ctor(); ctor(const self& r); ctor<K,U,CA2,b>(const self<K,U,CA2,b>& r); ctor<U>(std::unique_ptr<U> r);
    ctor<Size>(Size n, unordered_associative_container_tag);
    explicit ctor<Comp>(const Comp& comp, const allocator_type& a);
    ctor<Hash,Pred,A>(const Hash&, const Pred&, const A& a);
    ctor<InIt>(InIt f, InIt l);
    ctor<InIt,Comp>(InIt f, InIt l, const Comp, const allocator_type&={});
    ctor<InIt,Hash,Pred,A>(InIt f, InIt l, const Hash&, const Pred&, const A& a);
    self& operator=(self r); self& operator=<U>(std::unique_ptr<U> r);

    void insert<InIt>(InIt f, InIt l); void insert<Range>(const Range& r);
    std::pair<iterator,bool> insert(key_type& k, mapped_type x); std::pair<iterator,bool> insert<U>(const key_type& k, std::unique_ptr<U> x);
    iterator insert<F,S>(iterator p, ref_pair<F,S> p);
    iterator insert(iterator p, key_type& k, mapped_type x); iterator insert<U>(iterator p, const key_type& k, std::unique_ptr<U> x);
    bool transfer<PtrMapAdapter>(PtrMapAdapter::iterator p, PtrMapAdapter& from);
    size_type transfer<PtrMapAdapter>(PtrMapAdapter::iterator f, PtrMapAdapter::iterator l, PtrMapAdapter& from);
    size_type transfer<PtrMapAdapter,Range>(const Range& r, PtrMapAdapter& from) requires !is_same_v<Range,PtrMapAdapter::iterator>;
    size_type transfer<PtrMapAdapter>(PtrMapAdapter& from);
};

class ptr_multimap_adapter<T,VoidPtrMultiMap,CA=heap_clone_allocator,ordered=true> : public ptr_map_adapter_base<T,VoidPtrMultiMap,CA,ordered> {
    void safe_insert(const key_type& k, auto_type ptr);
    void map_basic_clone_and_insert<It>(It f, It l);
public: // types from base: <const>_iterator, size_type, key_type, const_reference, auto_type, mapped_type
    using allocator_type = VoidPtrMultiMap::allocator_type;
    ctor(); ctor(const self& r); ctor<K,U,CA2,b>(const self<K,U,CA2,b>& r); ctor<U>(std::unique_ptr<U> r);
    ctor<Size>(Size n, unordered_associative_container_tag);
    explicit ctor<Comp>(const Comp& comp, const allocator_type& a);
    ctor<Hash,Pred,A>(const Hash&, const Pred&, const A& a);
    ctor<InIt>(InIt f, InIt l);
    ctor<InIt,Comp>(InIt f, InIt l, const Comp, const allocator_type&={});
    ctor<InIt,Hash,Pred,A>(InIt f, InIt l, const Hash&, const Pred&, const A& a);
    self& operator=(self r); self& operator=<U>(std::unique_ptr<U> r);

    void insert<InIt>(InIt f, InIt l); void insert<Range>(const Range& r);
    iterator insert(key_type& k, mapped_type x); iterator insert<U>(const key_type& k, std::unique_ptr<U> x);
    iterator insert<F,S>(iterator p, ref_pair<F,S> p);
    iterator insert(iterator p, key_type& k, mapped_type x); iterator insert<U>(iterator p, const key_type& k, std::unique_ptr<U> x);
    void transfer<PtrMapAdapter>(PtrMapAdapter::iterator p, PtrMapAdapter& from);
    size_type transfer<PtrMapAdapter>(PtrMapAdapter::iterator f, PtrMapAdapter::iterator l, PtrMapAdapter& from);
    size_type transfer<PtrMapAdapter,Range>(const Range& r, PtrMapAdapter& from) requires !is_same_v<Range,PtrMapAdapter::iterator>;
    void transfer<PtrMapAdapter>(PtrMapAdapter& from);

    bool is_null<It,F,S>(const self<It,F,S>& it);
};

struct detail::set_config<Key,VoidPtrSet,ordered> {
    using value_type = Key; using key_type = Key;
    using void_container_type = VoidPtrSet; using allocator_type = VoidPtrSet::allocator_type;
    using value_compare = mpl::eval_if_c<ordered, select_value_compare<VoidPtrSet>, identity<void>>::type;
    using key_compare = value_compare;
    using hasher = mpl::eval_if_c<ordered, identity<void>, select_hasher<VoidPtrSet>>::type;
    using key_equal = mpl::eval_if_c<ordered, identity<void>, select_key_equal<VoidPtrSet>>::type;
    using container_type = mpl::if_c<ordered, ordered_associative_container_tag, unordered_associative_container_tag>::type;
    using <const>_iterator = void_ptr_iterator<VoidPtrSet::<const>_iterator, <const> Key>;
    using <const>_local_iterator = void_ptr_iterator<mpl::eval_if_c<ordered, select_iterator<VoidPtrSet>, select_<const>_local_iterator<VoidPtrSet>>::type, <const> Key>;
    static <const> Key* get_<const>_pointer<It>(It i);
    static constexpr bool allow_null = false;
};

class detail::ptr_set_adapter_base<Key,VoidPtrSet,CA=heap_clone_allocator,ordered=true> : public associative_ptr_container<set_config<Key,VoidPtrSet,ordered>,CA> {
protected: size_type bucket(const key_type& k) const;
public: using key_type = Key;
    using base::ctor; using base::operator=; // all except tagged other than `unordered_associative_container_tag`
    using base::erase; size_type erase(const key_type& x);
    <const>_iterator find(const key_type& x) <const>;
    size_type count(const key_type& x) const;
    <const>_iterator {lower|upper}_bound(const key_type& x) <const>;
    iterator_range<<const>_iterator> equal_range(const key_type& x) <const>;
};

class ptr_set_adapter<Key,VoidPtrSet,CA=heap_clone_allocator,ordered=true> : public ptr_set_adapter_base<Key,VoidPtrSet,CA,ordered> {
    void set_basic_clone_and_insert<It>(It f, It l);
public: // types from base: <const>_iterator, size_type, auto_type
    using allocator_type = VoidPtrSet::allocator_type;
    ctor(); ctor(const self& r); ctor<U,Set,CA2,b>(const self<U,Set,CA2,b>& r); explicit ctor<U>(std::unique_ptr<U> r);
    ctor<Size>(Size n, unordered_associative_container_tag);
    explicit ctor<Comp>(const Comp& comp, const allocator_type& a);
    ctor<Size,Hash,Pred,A>(Size n, const Hash&, const Pred&, const A& a);
    ctor<Hash,Pred,A>(const Hash&, const Pred&, const A& a);
    ctor<InIt>(InIt f, InIt l);
    ctor<InIt,Comp,A>(InIt f, InIt l, const Comp, const A&={});
    ctor<InIt,Hash,Pred,A>(InIt f, InIt l, const Hash&, const Pred&, const A& a);
    self& operator=<U,Set,CA2,b>(const self<U,Set,CA2,b>& r); self& operator=<T>(std::unique_ptr<T> r);

    std::pair<iterator,bool> insert(key_type* k); std::pair<iterator,bool> insert<U>(std::unique_ptr<U> x);
    iterator insert(iterator p, key_type* x); iterator insert<U>(iterator p, std::unique_ptr<U> x);
    void insert<InIt>(InIt f, InIt l); void insert<Range>(const Range& r) requires !is_pointer_or_integral<Range>;
    bool transfer<PtrSetAdapter>(PtrSetAdapter::iterator p, PtrSetAdapter& from);
    size_type transfer<PtrSetAdapter>(PtrMapAdapter::iterator f, PtrSetAdapter::iterator l, PtrSetAdapter& from);
    size_type transfer<PtrSetAdapter,Range>(const Range& r, PtrSetAdapter& from) requires !is_same_v<Range,PtrSetAdapter::iterator>;
    size_type transfer<PtrSetAdapter>(PtrSetAdapter& from);
};

class ptr_multiset_adapter<Key,VoidPtrMultiSet,CA=heap_clone_allocator,ordered=true> : public ptr_set_adapter_base<Key,VoidPtrMultiSet,CA,ordered> {
    void set_basic_clone_and_insert<It>(It f, It l);
public: // types from base: <const>_iterator, size_type, auto_type
    using allocator_type = VoidPtrMultiSet::allocator_type;
    ctor(); ctor(const self& r); explicit ctor<U,Set,CA2,b>(const self<U,Set,CA2,b>& r); explicit ctor<U>(std::unique_ptr<U> r);
    ctor<Size>(Size n, unordered_associative_container_tag);
    explicit ctor<Comp>(const Comp& comp, const allocator_type& a);
    ctor<Hash,Pred,A>(const Hash&, const Pred&, const A& a);
    ctor<InIt>(InIt f, InIt l);
    ctor<InIt,Comp>(InIt f, InIt l, const Comp, const allocator_type&={});
    ctor<InIt,Hash,Pred,A>(InIt f, InIt l, const Hash&, const Pred&, const A& a);
    self& operator=<U,Set,CA2,b>(const self<U,Set,CA2,b>& r); self& operator=<T>(std::unique_ptr<T> r);

    iterator insert(key_type* k); iterator insert<U>(std::unique_ptr<U> x);
    iterator insert(iterator p, key_type* x); iterator insert<U>(iterator p, std::unique_ptr<U> x);
    void insert<InIt>(InIt f, InIt l); void insert<Range>(const Range& r) requires !is_pointer_or_integral<Range>;
    bool transfer<PtrSetAdapter>(PtrSetAdapter::iterator p, PtrSetAdapter& from);
    size_type transfer<PtrSetAdapter>(PtrMapAdapter::iterator f, PtrSetAdapter::iterator l, PtrSetAdapter& from);
    size_type transfer<PtrSetAdapter,Range>(const Range& r, PtrSetAdapter& from) requires !is_same_v<Range,PtrSetAdapter::iterator>;
    size_type transfer<PtrSetAdapter>(PtrSetAdapter& from);
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

class ptr_map<Key,T,Comp=std::less<Key>,CA=heap_clone_allocator,A=std::allocator<std::pair<const Key,void_ptr<T>::type>>>
    : ptr_map_adapter<T,std::map<Key,void_ptr<T>::type,Comp,A>,CA> {
public:
    ctor(); explicit ctor(const Comp& comp, const A& a={}); ctor<U>(const self<Key,U>& r); explicit ctor(std::unique_ptr<self> r );
    ctor<InIt>(InIt f, InIt l); ctor<InIt>(InIt f, InIt l, const Comp& comp, const A& a={});
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    using base::release;

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};

class ptr_multimap<Key,T,Comp=std::less<Key>,CA=heap_clone_allocator,A=std::allocator<std::pair<const Key,void*>>>
    : ptr_multimap_adapter<T,std::multimap<Key,void*,Comp,A>,CA> {
public:
    ctor(); explicit ctor(const Comp& comp, const A& a={}); ctor<U>(const self<Key,U>& r); explicit ctor(std::unique_ptr<self> r );
    ctor<InIt>(InIt f, InIt l); ctor<InIt>(InIt f, InIt l, const Comp& comp, const A& a={});
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    using base::release;

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};

class ptr_set<Key,Comp=std::less<Key>,CA=heap_clone_allocator,A=std::allocator<void_ptr<Key>::type>>
    : ptr_set_adapter<Key,std::set<void_ptr<Key>::type,void_ptr_indirect_fun<Comp,Key>,A>,CA, true> {
public:
    ctor(); explicit ctor(const Comp& comp, const A& a={}); ctor<U>(const self<U>& r); explicit ctor(std::unique_ptr<self> r );
    ctor<InIt>(InIt f, InIt l); ctor<InIt>(InIt f, InIt l, const Comp& comp, const A& a={});
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    using base::release;

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};

class ptr_multiset<Key,Comp=std::less<Key>,CA=heap_clone_allocator,A=std::allocator<void*>>
    : ptr_multiset_adapter<Key,std::multiset<void*,void_ptr_indirect_fun<Comp,Key>,A>,CA, true> {
public:
    ctor(); explicit ctor(const Comp& comp, const A& a={}); ctor<U>(const self<U>& r); explicit ctor(std::unique_ptr<self> r );
    ctor<InIt>(InIt f, InIt l); ctor<InIt>(InIt f, InIt l, const Comp& comp, const A& a={});
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    using base::release;

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};

class ptr_unordered_map<Key,T,Hash=boost::hash<Key>,Pred=std::equal_to<Key>,CA=heap_clone_allocator,A=std::allocator<std::pair<const Key,void_ptr<T>::type>>>
    : ptr_map_adapter<T,boost::unordered_map<Key,void_ptr<T>::type,Hash,Pred,A>,CA,false> {
    using base::{lower|upper}_bound; using base::<c>r{begin|end}; using base::{key|value}_comp; using base::{front|back}; // disable
public:
    ctor(); explicit ctor(size_type n); explicit ctor(size_type n, const Hash&, const Pred&={}, const A& a={});
    ctor<U>(const self<Key,U>& r); explicit ctor(std::unique_ptr<self> r);
    ctor<InIt>(InIt f, InIt l); ctor<InIt>(InIt f, InIt l, const Hash&, const Pred&={}, const A& a={});
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    using base::release;

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};

class ptr_unordered_multimap<Key,T,Hash=boost::hash<Key>,Pred=std::equal_to<Key>,CA=heap_clone_allocator,A=std::allocator<std::pair<const Key,void*>>>
    : ptr_multimap_adapter<T,boost::unordered_multimap<Key,void*,Hash,Pred,A>,CA,false> {
    using base::{lower|upper}_bound; using base::<c>r{begin|end}; using base::{key|value}_comp; using base::{front|back}; // disable
public:
    ctor(); explicit ctor(size_type n); explicit ctor(size_type n, const Hash&, const Pred&={}, const A& a={});
    ctor<U>(const self<Key,U>& r); explicit ctor(std::unique_ptr<self> r);
    ctor<InIt>(InIt f, InIt l); ctor<InIt>(InIt f, InIt l, const Hash&, const Pred&={}, const A& a={});
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    using base::release;

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};

class ptr_unordered_set<Key,Hash=boost::hash<Key>,Pred=std::equal_to<Key>,CA=heap_clone_allocator,A=std::allocator<void_ptr<Key>::type>>
    : ptr_set_adapter<Key,boost::unordered_set<void_ptr<Key>::type,void_ptr_indirect_fun<Hash,Key>,void_ptr_indirect_fun<Pred,Key>,A>,CA,false> {
    using base::{lower|upper}_bound; using base::<c>r{begin|end}; using base::{key|value}_comp; using base::{front|back}; // disable
public:
    ctor(); explicit ctor(size_type n); explicit ctor(size_type n, const Hash&, const Pred&={}, const A& a={});
    ctor<U>(const self<U>& r); explicit ctor(std::unique_ptr<self> r);
    ctor<InIt>(InIt f, InIt l); ctor<InIt>(InIt f, InIt l, const Hash&, const Pred&={}, const A& a={});
    self& operator=(self r); self& operator=(std::unique_ptr<self> r);
    std::unique_ptr<self> release(); std::unique_ptr<self> clone() const;
    using base::release;

    friend self* new_clone(const self& r);
    friend void swap(self&, self&);
};

class ptr_unordered_multiset<Key,Hash=boost::hash<Key>,Pred=std::equal_to<Key>,CA=heap_clone_allocator,A=std::allocator<void*>>
    : ptr_multiset_adapter<Key,boost::unordered_multiset<void*,void_ptr_indirect_fun<Hash,Key>,void_ptr_indirect_fun<Pred,Key>,A>,CA,false> {
    using base::{lower|upper}_bound; using base::<c>r{begin|end}; using base::{key|value}_comp; using base::{front|back}; // disable
public:
    ctor(); explicit ctor(size_type n); explicit ctor(size_type n, const Hash&, const Pred&={}, const A& a={});
    ctor<U>(const self<U>& r); explicit ctor(std::unique_ptr<self> r);
    ctor<InIt>(InIt f, InIt l); ctor<InIt>(InIt f, InIt l, const Hash&, const Pred&={}, const A& a={});
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

class ptr_back_insert_iterator<PtrCont> {
protected: PtrCont* container;
public: using iterator_category = std::output_iterator_tag; using container_type = PtrCont;
    using value_type = void; using difference_type = void; using pointer = void; using reference = void;
    explicit ctor(PtrCont& cont);
    self& operator=(PtrCont::value_type r) { container->push_back(container->null_policy_allocate_clone(r)); return *this; }
    self& operator=(PtrCont::const_reference r) { container->push_back(container->null_policy_allocate_clone(&r)); return *this; }
    self& operator=<T>(std::unique_ptr<T> r) { container->push_back(std::move(r)); return *this; }
    self& operator*(); self& operator++(); self operator++(int) { return *this; }
};

class ptr_front_insert_iterator<PtrCont> {
protected: PtrCont* container;
public: using iterator_category = std::output_iterator_tag; using container_type = PtrCont;
    using value_type = void; using difference_type = void; using pointer = void; using reference = void;
    explicit ctor(PtrCont& cont);
    self& operator=(PtrCont::value_type r) { container->push_front(container->null_policy_allocate_clone(r)); return *this; }
    self& operator=(PtrCont::const_reference r) { container->push_front(container->null_policy_allocate_clone(&r)); return *this; }
    self& operator=<T>(std::unique_ptr<T> r) { container->push_front(std::move(r)); return *this; }
    self& operator*(); self& operator++(); self operator++(int) { return *this; }
};

class ptr_insert_iterator<PtrCont> {
protected: PtrCont* container; PtrCont::iterator iter;
public: using iterator_category = std::output_iterator_tag; using container_type = PtrCont;
    using value_type = void; using difference_type = void; using pointer = void; using reference = void;
    explicit ctor(PtrCont& cont, PtrCont::iterator it);
    self& operator=(PtrCont::value_type r) { container->insert(iter, container->null_policy_allocate_clone(r)); return *this; }
    self& operator=(PtrCont::const_reference r) { container->insert(iter, container->null_policy_allocate_clone(&r)); return *this; }
    self& operator=<T>(std::unique_ptr<T> r) { container->insert(iter, std::move(r)); return *this; }
    self& operator*(); self& operator++(); self operator++(int) { return *this; }
};

ptr_back_insert_iterator<PtrCont> ptr_back_inserter<PtrCont>(PtrCont& cont);
ptr_front_insert_iterator<PtrCont> ptr_front_inserter<PtrCont>(PtrCont& cont);
ptr_insert_iterator<PtrCont> ptr_inserter<PtrCont>(PtrCont& cont, PtrCont::iterator it);
```

------
#### Serialization Support

```c++
const char* detail::{count|item|first|second}() { return ""; } // corresponding func name
T const& detail::serialize_as_const<T>(T const& r) { return r; }

void detail::save_helper<Ar,Config,CA>(Ar& ar, const reversible_ptr_container<Config,CA>& c);
void detail::load_helper<Ar,Config,CA>(Ar& ar, reversible_ptr_container<Config,CA>& c, reversible_ptr_container<Config,CA>::size_type n);

void serialization::save<Ar,Config,CA>(Ar& ar, const reversible_ptr_container<Config,CA>& c, unsigned ver);
void serialization::load<Ar,Config,CA>(Ar& ar, reversible_ptr_container<Config,CA>& c, unsigned ver);

void serialization::save<Ar,T,VoidPtrMap,CA,ordered>(Ar& ar, const ptr_map_adapter_base<T,VoidPtrMap,CA,ordered>& c, unsigned ver);
void serialization::load<Ar,T,VoidPtrMap,CA,ordered>(Ar& ar, ptr_map_adapter<T,VoidPtrMap,CA,ordered>& c, unsigned ver);
void serialization::load<Ar,T,VoidPtrMap,CA,ordered>(Ar& ar, ptr_multimap_adapter<T,VoidPtrMap,CA,ordered>& c, unsigned ver);

void serialization::save<Ar,T,n,CA>(Ar& ar, const ptr_array<T,n,CA>& c, unsigned ver) { save_helper(ar, c); }
void serialization::load<Ar,T,n,CA>(Ar& ar, ptr_array<T,n,CA>& c, unsigned ver)
{ for (size_type i=0; i!=n; ++i) { T* p; ar >> make_nvp(item(), p); c.replace(i,p); } }
void serialization::serialize<Ar,T,n,CA>(Ar& ar, ptr_array<T,n,CA>& c, unsigned ver) { core::split_free(ar,c,ver); }
void serialization::serialize<Ar,T,CA,A>(Ar& ar, ptr_deque<T,CA,A>& c, unsigned ver) { core::split_free(ar,c,ver); }
void serialization::serialize<Ar,T,CA,A>(Ar& ar, ptr_list<T,CA,A>& c, unsigned ver) { core::split_free(ar,c,ver); }
void serialization::load<Ar,T,n,CA>(Ar& ar, ptr_vector<T,n,CA>& c, unsigned ver)
{ size_type n; ar >> make_nvp(count(), n); c.reserve(n); load_helper(ar,c,n); }
void serialization::serialize<Ar,T,CA,A>(Ar& ar, ptr_vector<T,CA,A>& c, unsigned ver) { core::split_free(ar,c,ver); }
void serialization::serialize<Ar,Key,T,Comp,CA,A>(Ar& ar, ptr_map<Key,T,Comp,CA,A>& c, unsigned ver) { core::split_free(ar,c,ver); }
void serialization::serialize<Ar,Key,T,Comp,CA,A>(Ar& ar, ptr_multimap<Key,T,Comp,CA,A>& c, unsigned ver) { core::split_free(ar,c,ver); }
void serialization::serialize<Ar,T,CA,A>(Ar& ar, ptr_set<T,CA,A>& c, unsigned ver) { core::split_free(ar,c,ver); }
void serialization::serialize<Ar,T,CA,A>(Ar& ar, ptr_multiset<T,CA,A>& c, unsigned ver) { core::split_free(ar,c,ver); }
void serialization::serialize<Ar,Key,T,Hash,Pred,CA,A>(Ar& ar, ptr_unordered_map<Key,T,Hash,Pred,CA,A>& c, unsigned ver) { core::split_free(ar,c,ver); }
void serialization::serialize<Ar,Key,T,Hash,Pred,CA,A>(Ar& ar, ptr_unordered_multimap<Key,T,Hash,Pred,CA,A>& c, unsigned ver) { core::split_free(ar,c,ver); }
void serialization::serialize<Ar,T,Hash,Pred,CA,A>(Ar& ar, ptr_unordered_set<T,Hash,Pred,CA,A>& c, unsigned ver) { core::split_free(ar,c,ver); }
void serialization::serialize<Ar,T,Hash,Pred,CA,A>(Ar& ar, ptr_unordered_multiset<T,Hash,Pred,CA,A>& c, unsigned ver) { core::split_free(ar,c,ver); }
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
* `<boost/iterator/iterator_adaptor.hpp>`
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
