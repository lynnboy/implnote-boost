# Boost.PolyCollection

* lib: `boost/libs/poly_collection`
* repo: `boostorg/poly_collection`
* commit: `3175ae3`, 2025-09-19

------
### Common Bits

#### Exception

* Header `<boost/poly_collection/exception.hpp>`

```c++
struct unregistered_type : std::logic_error { ctor(const std::type_info& info); const std::type_info* pinfo; };
struct not_copy_constructible : std::logic_error { ctor(const std::type_info& info); const std::type_info* pinfo; };
struct not_equality_comparable : std::logic_error { ctor(const std::type_info& info); const std::type_info* pinfo; };
```

#### Value Holder

```c++
class detail::value_holder_base<T> { protected: alignas(T) unsigned char s[sizeof(T)]; };
class detail::value_holder<T,U=T> : public value_holder_base<T> {
    <const> T* data()<const>noexcept; <const>T& value()<const>noexcept;
public: ctor<Alloc>(Alloc& a, U {const&|&&} x) noexcept(...);
    ctor<Alloc,...Args>(Alloc& a, value_holder_emplacing_ctor_t, Args&&...args);
    ctor<Alloc>(Alloc& a, self {const&|&&} x) noexcept(...);
    ctor(U{const&|&&} x) noexcept(...);
    ctor<...Args>(value_holder_emplacing_ctor_t, Args&&...args);
    ctor(self {const&|&&} x) noexcept(...);
    self& operator=(self const& x) = delete; self& operator=(self&& x) noexcept(...);
    ~dtor()noexcept(...);
    friend bool operator==(self const& x, self const& y);
};
```

#### Allocator Adaptor

```c++
struct detail::uses_alloc_ctor<T,Alloc,...Args>; // 0, 1, 2
struct detail::allocator_is_always_equal<Alloc>; // empty type or allocator_traits::is_always_equal
struct detail::allocator_adaptor<Alloc> {
    using traits = std::allocator_traits<Alloc>;
    using value_type = traits::value_type; // and size_type, difference_type, <const>_pointer, <const>_void_pointer, propagate_on_container_xxx
    using is_always_equal = allocator_is_always_equal<Alloc>::type;
    struct rebind<U>{ using other = self<traits::rebind_alloc<U>>; };
    ctor()=default; ctor(const self&)=default;
    ctor<Alloc2>(const Alloc2& x) noexcept requires is_constructible_v<Alloc,const Alloc2&>;
    ctor<Alloc2>(const self<Alloc2>& x) noexcept requires is_constructible_v<Alloc,const Alloc2&>;
    self& operator=(const self&)=default;

    <const> Alloc& allocator() <const> noexcept;
    void construct<T,...Args>(T* p, Args&&...args);
    void construct<T,U,...Args>(value_holder<T,U>* p, Args&&...args);
    void construct<T1,T2,...Args1,...Args2>(std::pair<T1,T2>* p, std::piecewise_construct_t, std::tuple<Args1...> x, std::tuple<Args2...> y);
    void construct<T1,T2>(std::pair<T1,T2>* p);
    void construct<T1,T2,U,V>(std::pair<T1,T2>* p, U&& x, V&& y);
    void construct<T1,T2,U,V>(std::pair<T1,T2>* p, std::pair<U,V>{const&|&&} x);
    void destroy<T>(T* p);
    void destroy<T,U>(value_holder<T,U>* p);
    self select_on_container_copy_construction() const noexcept;
};
bool operator== <Alloc1,Alloc2> (const allocator_adaptor<Alloc1>& x, const allocator_adaptor<Alloc2>& y) noexcept; // and !=
```

#### Iterators

```c++
struct detail::poly_collection_of<It> { using type = ...; };
struct detail::model_of<PolyCollection> { using type = ...; };
struct detail::iterator_traits<It> {
    using container_type = poly_collection_of<It>::type;
    using model_type = model_of<container_type>::type;
    using type_index = container_type::type_index;
    using is_const_iterator = std::is_const_t<remove_reference_t<std::iterator_traits<It>::reference>>;
    using iterator = std::conditional_t<is_const_iterator::value, container_type::const_iterator, container_type::iterator>;
    using base_segment_info_iterator = std::conditional_t<is_const_iterator::value, container_type::const_base_segment_info_iterator, container_type::base_segment_info_iterator>;
    using local_base_iterator = std::conditional_t<is_const_iterator::value, container_type::const_local_base_iterator, container_type::local_base_iterator>;
    using local_iterator = std::conditional_t<is_const_iterator::value, container_type::const_local_iterator, container_type::local_iterator>;
    static base_segment_info_iterator base_segment_info_iterator_from(<local_base>_iterator it) noexcept;
    static base_segment_info_iterator end_base_segment_info_iterator_from(iterator it) noexcept;
    static local_base_iterator local_base_iterator_from(iterator it) noexcept;
    static iterator iterator_from(local_base_iterator lbit, base_segment_info_iterator mapend) noexcept;
    static auto index<T>() -> decltype(model_type::index<T>());
};

using detail::iterator_impl_value_type<PolyColl,isConst> = std::conditional_t<isConst, const PolyColl::value_type, PolyColl::value_type>;
class detail::iterator_impl<PolyColl,isConst>: public boost::iterator_facade<self, iterator_impl_value_type<PolyColl,isCOnst>, forward_traversal_tag> {
    using segment_type = PolyColl::segment_type;
    using const_segment_base_{iterator|sentinel} = PolyColl::const_segment_base_{iterator|sentinel};
    using const_segment_map_iterator = PolyColl::const_segment_map_iterator;
    const_segment_map_iterator mapit, mapend; const_segment_base_iterator segpos;

    ctor(const_segment_map_iterator mapit, const_segment_map_iterator mapend,<const_segment_base_iterator segpos>) noexcept;
    value_type& dereference() const noexcept;
    bool equal(const self& x) const noexcept;
    void increment() noexcept;
    void next_segment_position() noexcept;
    <const> segment_type& segment() <const> noexcept;
    const_segment_base_sentinel sentinel() const noexcept;
public: using value_type = iterator_impl_value_type<PolyColl,isCOnst>;
    ctor()=default; ctor(const self&)=default; self& operator=(const self&)=default;
};

class detail::local_iterator_impl<PolyColl,BaseIt>: public boost::iterator_adaptor<self, BaseIt> {
    using segment_type = PolyColl::segment_type;
    using segment_base_iterator = PolyColl::segment_base_iterator;
    using const_segment_map_iterator = PolyColl::const_segment_map_iterator;

    ctor<It>(const_segment_map_iterator mapit, It it);
    static BaseIt base_iterator_from<BaseIt2>(const segment_type& s, BaseIt2 it);
    base_iterator base() const noexcept;
    const PolyColl::type_index& type_info() const;
    <const> segment_type& segment() <const> noexcept;
public: using base_iterator = BaseIt;
    ctor()=default; ctor(const self&)=default; self& operator=(const self&)=default;
    ctor<BaseIt2>(const self<PolyColl,BaseIt2>& x);
    std::iterator_traits<BaseIt>::reference operator[] <Diff> (Diff n) const;
};
```

#### Map Wrappers

```c++
class detail::size_t_map<T,Alloc> {
    using vector_type = std::vector<std::pair<size_t,T>, allocator_traits<Alloc>::rebind_alloc<std::pair<size_t,T>>>;
    vector_type v;
public: using key_type = size_t; using mapped_type = T; using value_type = vector_type::value_type;
    using allocator_type = vector_type::allocator_type; using <const>_iterator = vector_type::<const>_iterator;
    ctor(<allocator_type const& a>); ctor(self {const&|&&} x, <allocator_type const& a>);
    self& operator=(self {const&|&&} x);
    allocator_type get_allocator() const noexcept;
    <const>_iterator {begin|end}() <const> noexcept; const_iterator {cbegin|cend}() const noexcept;
    const_iterator find(const key_type& key) const;
    std::pair<iterator,bool> insert<P>(const key_type& key, P&& x);
    void swap(self& x); friend void swap(self& x, self& y);
};

struct detail::type_info_hash { size_t operator()(const std::type_info&) const noexcept; };
struct detail::type_info_equal_to { bool operator()(const std::type_info&, const std::type_info&) const noexcept; };
class detail::type_info_map<T,Alloc> {
    using map_type = std::unordered_map<std::reference_wrapper<const std::type_info>, T, type_info_hash, type_info_euqal_to,
        allocator_traits<Alloc>::rebind_alloc<std::pair<const std::reference_wrapper<const std::type_info>,T>>>;
    using cache_type = std::unordered_map<const std::type_info*, iterator, std::hash<const std::type_info*>, std::equal_to<const std::type_info*>,
        allocator_traits<Alloc>::rebind_alloc<std::pair<const std::type_info* const,iterator>>>;
    using cache_allocator_type = cache_type::allocator_type;
    map_type map; cache_type cache;

    static UnordMap make<UnordMap> (const UnordMap::allocator_type& a);
    static std::decay_t<UnordMap> make<UnordMap>(UnordMap && x, const std::decay_t<UnordMap>::allocator_type& a);
    void build_cache(const cache_type& x); void rebuild_cache();
public: using key_type = std::type_info; using mapped_type = T; using value_type = map_type::value_type;
    using allocator_type = map_type::allocator_type; using <const>_iterator = map_type::<const>_iterator;
    ctor(<allocator_type const& a>); ctor(self {const&|&&} x, <allocator_type const& a>);
    self& operator=(self {const&|&&} x);
    allocator_type get_allocator() const noexcept;
    <const>_iterator {begin|end}() <const> noexcept; const_iterator {cbegin|cend}() const noexcept;
    <const>_iterator find(const key_type& key) <const>;
    std::pair<iterator,bool> insert<P>(const key_type& key, P&& x);
    void swap(self& x); friend void swap(self& x, self& y);
};

struct detail::segment_map_helper<std::type_info> { using fn<...Args> = type_info_map<Args...>; };
struct detail::segment_map_helper<size_t> { using fn<...Args> = size_t_map<Args...>; };
using detail::segment_map<Key,...Args> = segment_map_helper<Key>::fn<Args...>;
```

#### Segment

```c++
struct detail::segment_backend<Model,Alloc> { // base interface
    using segment_backend_unique_ptr = std::unique_ptr<self,void(*)(self*)>;
    using <const>_value_pointer = void*; using <const>_base_iterator = Model::base_iterator;
    using const_iterator<T> = Model::const_iterator<T>; using base_sentinel=Model::base_sentinel;
    using range=std::pair<base_iterator, base_sentinel>;
    ctor()=default; ctor(const self&)=delete; self& operator=(const self&)=delete; // no copy/move

    virtual ~dtor()=default;
    virtual segment_backend_unique_ptr copy() const=0;
    virtual segment_backend_unique_ptr <empty>_copy(const Alloc&) const=0;
    virtual segment_backend_unique_ptr move(const Alloc&)=0;
    virtual bool equal(const self&) const=0;

    virtual Alloc get_allocator() const noexcept=0;
    virtual base_iterator {begin|end} const noexcept=0;
    virtual bool empty() const noexcept=0;
    virtual size_t {size|max_size|capacity}() const noexcept=0;
    virtual base_sentinel reserve(size_t)=0;
    virtual base_sentinel shrink_to_fit()=0;
    virtual range push_back(const_value_pointer)=0;
    virtual range push_back_move(value_pointer)=0;
    virtual range insert(const_base_iterator, const_value_pointer)=0;
    virtual range insert_move(const_base_iterator, value_pointer)=0;
    virtual range erase(const_base_iterator,<const_value_pointer>)=0;
    virtual range erase_{till_end|from_begin}(const_base_iterator)=0;
    virtual base_sentinel clear() noexcept=0;
};

class detail::packed_segment<Model,Concrete,Alloc> : public segment_backend<Model,Alloc> {
    using value_type = Model::value_type; using final_type = {SFINAE Model::final_type<Concrete> ?? Concrete};
    using store_value_type = value_holder<final_type,Concrete>;
    using store = std::vector<store_value_type, std::allocator_traits<Alloc>::rebind_alloc<store_value_type>>;
    using <const>_store_iterator = store::<const>_iterator;
    using const_iterator = base::const_iterator<Concrete>;
    using segment_allocator_type = std::allocator_traits<Alloc>::rebind_alloc<self>;
    store s;
public: virtual ~dtor()=default; // and all virtual overrides
    static segment_backend_unique_ptr make(const segment_allocator_type& a);
    base_iterator nv_{begin|end}() const noexcept;
    bool nv_empty() const noexcept; size_t nv_{size|max_size|capacity} const noexcept;
    base_sentinel nv_reserve(size_t n); base_sentinel nv_shrink_to_fit();
    range nv_emplace<It,...Args>(It p, Args&&...args);
    range nv_emplace_back<..Args>(Args&&...args);
    range nv_push_back(Concrete {const&|&&} x);
    range nv_insert(const_iterator p, Concrete {const&|&&} x);
    range nv_insert<It>(<const_iterator p>, It first, It last);
    range nv_erase(const_iterator p, <const_iterator last>);
    base_sentinel nv_clear() noexcept;
};

class detail::split_segment<Model,Concrete,Alloc> : public segment_backend<Model,Alloc> {
    using value_type = Model::value_type;
    using store_value_type = value_holder<Concrete>;
    using store = std::vector<store_value_type, std::allocator_traits<Alloc>::rebind_alloc<store_value_type>>;
    using <const>_store_iterator = store::<const>_iterator;
    using index = std::vector<value_type, std::allocator_traits<Alloc>::rebind_alloc<value_type>>;
    using const_index_iterator = index::const_iterator;
    using const_iterator = base::const_iterator<Concrete>;
    using segment_allocator_type = std::allocator_traits<Alloc>::rebind_alloc<self>;
    store s; index i;
public: virtual ~dtor()=default; // and all virtual overrides
    static segment_backend_unique_ptr make(const segment_allocator_type& a);
    base_iterator nv_{begin|end}() const noexcept;
    bool nv_empty() const noexcept; size_t nv_{size|max_size|capacity} const noexcept;
    base_sentinel nv_reserve(size_t n); base_sentinel nv_shrink_to_fit();
    range nv_emplace<It,...Args>(It p, Args&&...args);
    range nv_emplace_back<..Args>(Args&&...args);
    range nv_push_back(Concrete {const&|&&} x);
    range nv_insert(const_iterator p, Concrete {const&|&&} x);
    range nv_insert<It>(<const_iterator p>, It first, It last);
    range nv_erase(const_iterator p, <const_iterator last>);
    base_sentinel nv_clear() noexcept;
};

class detail::segment<Model,Alloc> { // PImpl wrapper
    using segment_backend = Model::segment_backend<Alloc>;
    using segment_backend_implementation<Concrete> = Model::segment_backend_implementation<Concrete,Alloc>;
    using segment_backend_unique_ptr = segment_backend::segment_backend_unique_ptr;
    using range = segment_backend::range;
    struct from_prototype{};
    segment_backend_unique_ptr pimpl; base_sentinel snt;

    ctor(segment_backend_unique_ptr&& pimpl);
    ctor(from_prototype, const self& x, const allocator_type& a);
    <const> segment_backend& impl() <const> noexcept;
    <const> segment_backend_implementation<Concrete>& impl<Concrete>() <const> noexcept;
    static <const> void* subaddress<T> (<const> T& x);
    void set_sentinel();
    void filter(base_sentinel x); base_iterator filter(const range& x);

public: using value_type = Model::value_type; using allocator_type = Alloc;
    using <const>_base_iterator = Mode::<const>_base_iterator;
    using <const>_base_sentinel = Model::<const>_base_sentinel;
    using <const>_iterator<T> = Model::<const>_iterator<T>;
    static self make<T>(const allocator_type& a);
    static self make_from_prototype(const self& x, const allocator_type& a);

    ctor(self {const&|&&} x, <const allocator_type& a>);
    self& operator=(self {const&|&&} x);
    friend bool operator==(const self& x, const self& y); // and !=
    base_iterator {begin|end}[<U>] () const noexcept;
    base_sentinel sentinel() const noexcept;
    bool empty[<U>] () const noexcept;
    size_t size[<U>] () const noexcept; size_t max_size[<U>] () const noexcept;
    size_t capacity[<U>] () const noexcept;
    void reserve [<U>] (size_t n); void shrink_to_fit [<U>] ();
    base_iterator emplace <U,It,...Args> (It it, Args&&...args);
    base_iterator emplace_back <U,...Args> (Args&&...args);
    base_iterator push_back<T> (T {const&|&&} x);
    base_iterator push_back_terminal<U> (U&& x);
    base_iterator insert<T> (const_base_iterator it, T {const&|&&} x);
    base_iterator insert<U,T> (<const>_iterator<U> it, T {const&|&&} x);
    base_iterator insert<It>(<const_base_iteartor it>, It first, It last);
    base_iterator insert<U,It>(const_iteartor<U> it, It first, It last);
    base_iterator erase(const_base_iterator it, <const_base_iteartor l>);
    base_iterator erase<U> (const_iterator<U> it, <const_iterator<U> l>);
    base_iterator erase_till_end<It>(It f);
    base_iterator erase_from_begin<It>(It l);
    void clear [<U>] () noexcept;
};
```

#### Model traits

```c++
struct detail::is_moveable<T>; // move_constructible && (move_assignable || nothrow_move_constructible)
struct detail::is_closed_collection<Model>; // exists Model::acceptable_type_list
struct detail::is_acceptable<T,Model>; // Model::is_implementation<T> Model::acceptable_type_list
```

#### Basic Collection Implementation

```c++
class common_impl::poly_collection<Model,Alloc> {
    static constexpr bool is_closed_collection = is_closed_collection<Model>::value;
    using segment_allocator_type = allocator_adaptor<Alloc>;
    using segment_type = segment<Model,segment_allocator_type>;
    using <const>_segment_base_iterator = segment_type::<const>_base_iterator;
    using <const>_segment_base_sentinel = segment_type::<const>_base_sentinel;
    using <const>_segment_iterator<T> = segment_type::<const>_iterator<T>;
    using segment_map = segment_map<type_index, std::allocator_traits<segment_allocator_type>::rebind_alloc<segment_type>>;
    using segment_map_allocator_type = segment_map::allocator_type;
    using <const>_segment_map_iterator = segment_map::<const>_iterator;
    using iterator_impl<isConst> = iterator_impl<self, isConst>;
    using local_iterator_impl<BaseIt> = local_iterator_impl<self,BaseIt>;
    class segment_info_iterator_impl<SegmentInfo> : public boost::iterator_adaptor<self, const_segment_map_iterator, SegmentInfo, std::input_iterator_tag, SegmentInfo> {
        ctor(const_segment_map_iterator it);
        SegmentInfo dereference() const noexcept;
    public: ctor()=default; ctor(const self&)=default; self& operator=(const self&)=default;
        ctor<SegmentInfo2>(const self<SegmentInfo2>& x) requires is_base_of_v<SegmentInfo, SegmentInfo2>;
        self& operator=<SegmentInfo2>(const self<SegmentInfo2>& x) requires is_base_of_v<SegmentInfo, SegmentInfo2>;
    };
    using nonconst_version<It> = ...; // const_xx_iterator -> xx_iterator

    segment_map map;

    static auto index <T> () -> decltype(...) { return Model::index<T>(); }
    static auto subindex <T> (const T& x) -> decltype(...) { return Model::subindex(x); }
    void initialize_map(); void initialize_map(segment_map&);
    static const std::type_info& type_info(const std::type_info&); static const std::type_info& type_info<TI>(const TI&);
    static const std::type_info& subtype_info<T>(const T&);
    const_segment_map_iterator get_map_iterator_for<T>(const T&) requires is_acceptable<decay_t<T>,Model>;
    const_segment_map_iterator get_map_iterator_for<T>(const T&) const requires !is_acceptable<decay_t<T>,Model>;
    const_segment_map_iterator get_map_iterator_for<T>(const T&, const segment_type&);
    const_segment_map_iterator get_map_iterator_for<T>();
    const_segment_map_iterator get_map_iterator_for(const type_index&)<const>;
    static segment_type& segment(const_segment_map_iterator pos);
    segment_base_iterator push_back<T>(segment_type&, T&&);
    static segment_base_iterator local_insert<T,BaseIt,U>(segment_type&, BaseIt, U&&);

public: using value_type = segment_type::value_type; using allocator_type = Alloc;
    using size_type = size_t; using difference_type = ptrdiff_t;
    using <const>_reference = <const> value_type&; using <const>_pointer = std::allocator_traits<Alloc>::<const>_pointer;
    using type_index = Model::type_index;
    using <const>_iterator = iterator_impl<{false|true}>;
    using <const>_local_base_iterator = local_iterator_impl<<const>_segment_base_iterator>;
    using <const>_local_iterator<T> = local_iterator_impl<<const>_segment_iterator<T>>;
    class const_base_segment_info {
    protected: const_segment_map_iterator it; ctor(const_segment_map_iterator it) noexcept;
    public: ctor(const self&)=default; self& operator=(const self&)=default;
        const_local_base_iterator <c>{begin|end}() const noexcept;
        const_local_iterator<T> <c>{begin|end}<T>() const noexcept;
        const type_index& type_info() const;
    };
    class base_segment_info : public const_base_segment_info {
    public: // using base::{ctor|begin|end}
        local_base_iterator {begin|end}() noexcept;
        local_iterator<T> <c>{begin|end}<T>() noexcept;
    };
    class const_segment_info<T> {
    protected: const_segment_map_iterator it; ctor(const_segment_map_iterator it) noexcept;
    public: ctor(const self&)=default; self& operator=(const self&)=default;
        const_local_iterator<T> <c>{begin|end}() const noexcept;
    };
    class segment_info<T> : public const_segment_info<T> {
    public: // using base::{ctor|begin|end}
        local_iterator<T> <c>{begin|end}() noexcept;
    };
    using <const>_base_segment_info_iterator = segment_info_iterator_impl<const_base_segment_info>;
    class const_segment_traversal_info {
    protected: segment_map* pmap; ctor(const segment_map& map) noexcept;
    public: ctor(const self&)=default; self& operator=(const self&)=default;
        const_base_segment_info_iterator <c>{begin|end}() const noexcept;
    };
    class segment_traversal_info : public const_segment_traversal_info {
    public: // using base::{ctor|begin|end}
        base_segment_info_iterator<T> <c>{begin|end}() noexcept;
    };

    ctor(<const allocator_type& a>); ctor(self {const&|&&} x, <const allocator_type& a>);
    ctor<It>(It first, It last, const allocator_type& a={});
    self& operator=(self {const&|&&} x);
    allocator_type get_allocator() const noexcept;
    void register_types<...T>() requires is_acceptable<decay_t<T>,Model>; && ... && !Model::is_closed_collection;
    bool is_registered(const type_index& info) const requires !Model::is_closed_collection;
    bool is_registered<T>() const requires is_acceptable<decay_t<T>,Model>; && !Model::is_closed_collection;

    iterator {begin|end}() noexcept; const_iterator <c>{begin|end}() const noexcept;
    local_base_iterator {begin|end}(const type_index&); const_local_base_iterator <c>{begin|end}(const type_index&) const;
    local_iterator<T> {begin|end}<T>() noexcept requires is_acceptable<decay_t<T>,Model>; const_local_iterator<T> <c>{begin|end}<T>() const noexcept requires is_acceptable<decay_t<T>,Model>;
    <const>_base_segment_info segment(const type_index& info) <const>;
    <const>_segment_info<T> segment<T>() <const> requires is_acceptable<decay_t<T>,Model>;
    <const>_segment_traversal_info segment_traversal() <const> noexcept;

    bool empty() const noexcept; bool empty(const type_index& info) const; bool empty<T>() const requires is_acceptable<decay_t<T>,Model>;
    size_type size() const noexcept; size_type size(const type_index& info) const; size_type size<T>() const requires is_acceptable<decay_t<T>,Model>;
    size_type max_size(const type_index& info) const; size_type max_size<T>() const requires is_acceptable<decay_t<T>,Model>;
    size_type capacity(const type_index& info) const; size_type capacity<T>() const requires is_acceptable<decay_t<T>,Model>;
    void reserve(size_type n); void reserve(const type_info&, size_type); void reserve<T>(size_type n) requires is_acceptable<decay_t<T>,Model>;
    void shrink_to_fit(); void shrink_to_fit(const type_info&); void shrink_to_fit<T>() requires is_acceptable<decay_t<T>,Model>;

    iterator emplace<T,...Args>(Args&&...args) requires is_acceptable<decay_t<T>,Model>;
    iterator emplace_hint<T,...Args>(const_iterator hint, Args&&...args) requires is_acceptable<decay_t<T>,Model>;
    local_base_iterator emplace_pos<T,...Args>(<const>_local_base_iterator pos, Args&&...args);
    local_iterator<T> emplace_pos<T,...Args>(<const>_local_iterator<T> pos, Args&&...args);
    iterator insert<T>(<const_iterator hint>, T&& x) requires Model::is_implementation<decay_t<T>>;
    nonconst_version<local_iterator_impl<BaseIt>> insert<BaseIt,T>(local_iterator_impl<BaseIt> pos, T&& x) requires Model::is_implementation<decay_t<T>>;
    void insert<It>(<const_iterator hint>, It first, It last) requires Model::is_implementation<decay_t<It::value_type>>;
    void insert<isConst>(<const_iterator hint>, iterator_impl<isConst> first, iterator_impl<isConst> last);
    void insert<BaseIt>(<const_iterator hint>, local_iterator_impl<BaseIt> first, local_iterator_impl<BaseIt> last);
    local_base_iterator insert<It>(const_local_base_iterator pos, It first, It last) requires Model::is_implementation<decay_t<It::value_type>>;
    local_iterator<T> insert<T,It>(<const>_local_iterator<T> pos, It first, It last);
    iterator erase(const_iterator pos, <const_iterator last>);
    nonconst_version<local_iterator_impl<BaseIt>> erase<BaseIt>(local_iterator_impl<BaseIt> pos, <local_iterator_impl<BaseIt> last>);
    void clear() noexcept; void clear(const type_index&); void clear<T>() requires is_acceptable<decay_t<T>,Model>;
    void swap(self&);
};
bool operator== <M,A> (const poly_collection<M,A>& x, const poly_collection<M,A>& y); // and !=
void swap<M,A> (poly_collection<M,A>& x, poly_collection<M,A>& y);
```

------
#### `base_collection`

* Header `<boost/poly_collection/base_collection.hpp>`

```c++
class detail::stride_iterator<Value> : public boost::iterator_facade<self, Value, random_access_traversal_tag> {
    using char_pointer=conditional_t<is_const_v<Value>, const char*, char*>;
    Value* p; size_t stride_;

    static char_pointer char_ptr(Value* p) noexcept;
    static Value*   value_ptr(char_pointer p) noexcept;
    Value& dereference() const noexcept;
    bool equal(const self& x) const noexcept;
    void {increment|decrement} () noexcept;
    void advance<Int>(Int n) noexcept;
    ptrdiff_t distance_to(const self& x) const noexcept;
public:
    ctor()=default; ctor(Value* p, size_t stride);
    ctor(const self&)=default; self& operator=(const self&)=default;
    ctor<NonConstValue> (const self<NonConstValue>& x) noexcept requires std::is_same_v<Value, const NonConstValue>;
    self& operator= <NonConstValue> (const self<NonConstValue>& x) noexcept requires std::is_same_v<Value, const NonConstValue>;
    self& operator=(Value* p_) noexcept;
    operator Value*() const noexcept;
    explicit operator DerivedValue* <DerivedValue> () const noexcept requires is_base_of_v<Value,DerivedValue> && (!is_const_v<Value> || is_const_v<DerivedValue>);
    size_t stride() const noexcept;
};

struct base_model<Base> {
    using value_type = Base; using type_index = std::type_info;
    using is_implementation<Derived> = std::is_base_of<Base,Derived>;
    using is_terminal<T> = is_final<T>;
    static const std::type_info& index<T>();
    static const std::type_info& subindex<T>(const T&) { if constexpr(is_terminal<T>) return typeid(T); else return typeid(x); }
    static <const> void* subaddress(<const>T& x) { if constexpr(is_terminal<T>) return addressof(x); else return dynamic_cast<void*>(addressof(x)); }
    using <const>_base_iterator = stride_iterator<[const]value_type>;
    using <const>_base_sentinel = <const> value_type*;
    using <const>_iterator<Derived> = <const> Derived*;
    using segment_backend<Alloc> = segment_backend<self,Alloc>;
    using segment_backend_implementation<Derived,Alloc> = packed_segment<self,Derived,Alloc>;
    static base_iterator nonconst_iterator(const_base_iterator it);
    static iterator<T> nonconst_iterator<T>(const_iterator<T> it);
};

class base_collection<Base,Alloc> : public common_impl::poly_collection<base_model<Base>,Alloc> {
    using base_type = base; <const> base_type& base() <const> noexcept;
public: using base::ctor; // all 5 ctor/op= =default
};
bool operator== <B,A> (const base_collection<B,A>& x, const base_collection<B,A>& y); // and !=
void swap<B,A> (base_collection<B,A>& x, base_collection<B,A>& y);
```

------
#### `function_collection`

* Header `<boost/poly_collection/function_collection.hpp>`

```c++
class detail::callable_wrapper<SiR(Args...)> {
    struct table { R(*call)(void*, Args...); const std::type_info& info; std::function<R(Args...)>(*convert)(void*); };
    table* pt; void* px;
public: explicit ctor<Callable> (Callable& x) noexcept requires !std::is_same_v<Callable,self> && is_invocable_r_v<R,Calalble,Args...>;
    ctor(const self&)=default; self& operator=(const self&)=default;
    explicit operator bool() const noexcept { return true; }
    R operator() (Args...args) const { return pt->call(px, std::forward<Args>(args)...); }
    const std::type_info& target_type() const noexcept { return pt->info; }
    <const> T* target<T> <const> noexcept { return typeid(T) == pt->info? (<const>T*)(px) : nullptr; }
    operator std::function<R(Args...)>() const noexcept { return pt->convert(px); }
    <const> void* data() <const> noexcept { return px; }
};

struct detail::callable_wrapper_iterator<CW> : public boost::iterator_adaptor<self, CW*> {
    ctor()=default; ctor(const self&)=default; self& operator=(const self&)=default;
    explicit ctor(CW* p) noexcept; ctor<NonConstCW>(const self<NonConstCW>& x) noexcept requires is_same_v<CW, const NonConstCW>;
    self& operator=(CW* p) noexcept; self& operator= <NonConstCW> (const self<NonConstCW>& x) noexcept requires is_same_v<CW, const NonConstCW>;
    explicit operator Callable* <Callable> () const noexcept requires is_constructible_v<CW, Callable&> && (!is_const_v<CW> || is_const_v<Callable>);
};

struct function_model<R(Args...)> {
    using value_type = callable_wrapper<R(Args...)>; using type_index = std::type_info;
    using is_implementation<Callable> = std::is_invocable_r<R,Callable&,Args...>;
    using is_terminal<T> = ...; // T is not spec of callable_wrapper
    static const std::type_info& index<T>();
    static const std::type_info& subindex<T>(const T&) { return typeid(T); }
    static const std::type_info& subindex<Sig>(callable_wrapper<Sig> const& f) { return f.target_type(); }
    static <const> void* subaddress(<const>T& x) { return addressof(x); }
    static <const> void* subaddress<Sig>(<const> callable_wrapper<Sig>& f) { return f.data(); }
    using <const>_base_iterator = callable_wrapper_iterator<<const> value_type>;
    using <const>_base_sentinel = <const> value_type*;
    using <const>_iterator<Callable> = <const> Callable*;
    using segment_backend<Alloc> = segment_backend<self,Alloc>;
    using segment_backend_implementation<Callable,Alloc> = split_segment<self,Callable,Alloc>;
    static base_iterator nonconst_iterator(const_base_iterator it);
    static iterator<T> nonconst_iterator<T>(const_iterator<T> it);
};

class function_collection<Sig,Alloc> : public common_impl::poly_collection<function_model<Sig>,Alloc> {
    using base_type = base; <const> base_type& base() <const> noexcept;
public: using base::ctor; // all 5 ctor/op= =default
};
bool operator== <S,A> (const function_collection<S,A>& x, const function_collection<S,A>& y); // and !=
void swap<S,A> (function_collection<S,A>& x, function_collection<S,A>& y);
```

------
#### `any_collection`

* Header `<boost/poly_collection/any_collection.hpp>`

```c++
struct detail::any_iterator<Any> : public boost::iterator_adaptor<self, Any*> {
    ctor()=default; ctor(const self&)=default; self& operator=(const self&)=default;
    explicit ctor(Any* p) noexcept; ctor<NonConstAny>(const self<NonConstAny>& x) noexcept requires is_same_v<Any, const NonConstAny>;
    self& operator=(Any* p) noexcept; self& operator= <NonConstAny> (const self<NonConstAny>& x) noexcept requires is_same_v<Any, const NonConstAny>;
    explicit operator Concrete* <Concrete> () const noexcept requires !is_const_v<Any> || is_const_v<Concrete>;
};

struct detail::any_model<Concept> {
    using value_type = type_erasure::any<std::conditional_t<type_erasure::is_subconcept<typeid_<>,Concept>::value, Concept, mpl::vector2<Concept,typeid_>>, type_erasure::_self&>;
    using type_index = std::type_info;
    using is_implementation<Concrete> = std::true_type; using is_terminal<T> = ...;
    static const std::type_info& index<T>() { return typeid(T); }
    static const std::type_info& subindex<T>(const T&) { return typeid(T); }
    static const std::type_info& subindex<Concept2,T>(const type_erasure::any<Concept2,T>& a) { return type_erasure::typeid_of(a); }
    static <const> void* subaddress(<const>T& x) { return addressof(x); }
    static <const> void* subaddress<Concept2,T>(<const> type_erasure::any<Concept2,T>& a) { return type_erasure::any_cast<<const> void*>(&a); }
    using <const>_base_iterator = callable_wrapper_iterator<<const> value_type>;
    using <const>_base_sentinel = <const> value_type*;
    using <const>_iterator<Concrete> = <const> Concrete*;
    using segment_backend<Alloc> = segment_backend<self,Alloc>;
    using segment_backend_implementation<Concrete,Alloc> = split_segment<self,Concrete,Alloc>;
    static base_iterator nonconst_iterator(const_base_iterator it);
    static iterator<T> nonconst_iterator<T>(const_iterator<T> it);
};

class any_collection<Concept,Alloc> : public common_impl::poly_collection<any_model<Concept>,Alloc> {
    using base_type = base; <const> base_type& base() <const> noexcept;
public: using base::ctor; // all 5 ctor/op= =default
};
bool operator== <C,A> (const any_collection<C,A>& x, const any_collection<C,A>& y); // and !=
void swap<C,A> (any_collection<C,A>& x, any_collection<C,A>& y);
```

------
#### `variant_collection`

* Header `<boost/poly_collection/variant_collection.hpp>`

```c++
class fixed_variant<...Ts> {
    static constexpr size_t N = sizeof...(Ts); using index_type = ...; // choose unsigned char, unsigned short, size_t by N
    index_type index_;
protected: ctor(const self&)=default; self& operator=(const self&)=default;
public: explicit ctor<T, i=mp_find<self,T>::value> (const T&) requires i < mp_size<self>::value;
    size_t index() const noexcept; bool valueless_by_exception() const noexcept; // always false
};

struct fixed_variant_store<T> { ctor<...Args> (Args&&...args); T value; };
struct fixed_variant_closure<T,Base> : fixed_variant_store<T>, Base {
    ctor<...Args>(Args&&...args) noexcept(...) requires std::is_constructible_v<T,Args&&...>;
    ctor(self{const&|&&})=default; self& operator=(self&&)=default;
    bool operator== <Q=T>(const self& x) const noexcept(...) requires is_equality_comparable_v<Q>;
};

struct bad_variant_access : std::exception { ctor()noexcept; const whar* what() const noexcept; };

struct variant_size<V>;
size_t variant_size_v<V> = variant_size<V>::value;
struct is_fixed_variant<T>;
struct variant_alternative<i,V>;
using variant_alternative_t<i,V> = variant_alternative<i,V>::type;
bool holds_alternative<T,...Ts>(const fixed_variant<Ts...>& x) noexcept;

<const> T{&|&&} <unsafe>_get<T,...Ts>(<const> fixed_variant<Ts...>{&|&&} x);
variant_alternative_t<i,V&&> <unsafe>_get<i,V>(V&& x);
<const> variant_alternative_t<i,fixed_variant<Ts...>> get_if<i,...Ts>(<const> fixed_variant<Ts...>* px) noexcept;
<const> T* get_if<T,...Ts>(<const> fixed_variant<Ts...>* px) noexcept;
struct deduced;
auto visit<R=deduced,F> (F&& f) -> conditional_t<is_same_v<R,deduced>, invoke_result_t<F&&>, R>;
auto visit<R=deduced,F,V> (F&& f, V&& v) -> conditional_t<is_same_v<R,deduced>, invoke_result_t<F&&, V&&>, R>;
auto visit<R=deduced,F,V1,V2,...Vs> (F&& f, V1&& v1, V2&& v2, Vs&&...xs) -> conditional_t<is_same_v<R,deduced>, invoke_result_t<F&&, V1&&, V2&&, Vs&&...>, R>;
auto visit_by_index<R=deduced,V,...Fs> (V&&x, Fs&&...fs) -> conditional_t<is_same_v<R,deduced>, invoke_result_t<Fs...[0]&&,decltype(get<0>(x))>, R>;
bool operator== <...Ts> (const fixed_variant<Ts...>& x, const fixed_variant<Ts...> y); // and !=, <, >, <=, >=

class detail::fixed_variant_alternative_iterator<Variant,T> : boost::iterator_facade<self, T, random_access_traversal_tag> {
    T* p;
    using char_pointer = conditional_t<is_const_v<V>, const char*, char*>;
    static char_pointer char_ptr(T* p) noexcept; static T* value_ptr(char_pointer p) noexcept;
    T& dereference() const noexcept; bool equal(const self& x) const noexcept;
    void {increment|decrement}() noexcept;
    void advance<Int>(Int n) noexcept; ptrdiff_t distance_to(const self& x) const noexcept;
public:
    ctor()=default; ctor(const self&)=default; self& operator=(const self&)=default;
    ctor(T* p) noexcept; ctor<NonConstT>(const self<NonConstT>& x) noexcept requires is_same_v<T, const NonConstT>;
    self& operator=(T* p) noexcept; self& operator= <NonConstT> (const self<NonConstT>& x) noexcept requires is_same_v<T, const NonConstT>;
    operator T* () const noexcept;
    size_t stride() const noexcept { return sizeof(fixed_variant_closure<remove_const_t<T>, Variant>); }
};

class detail::fixed_variant_iterator<V> : boost::iterator_facade<self, V, random_access_traversal_tag> {
    V* p; size_t stride_;
    using char_pointer = conditional_t<is_const_v<V>, const char*, char*>;
    static char_pointer char_ptr(V* p) noexcept; static V* value_ptr(char_pointer p) noexcept;
    V& dereference() const noexcept; bool equal(const self& x) const noexcept;
    void {increment|decrement}() noexcept;
    void advance<Int>(Int n) noexcept; ptrdiff_t distance_to(const self& x) const noexcept;
public:
    ctor()=default; ctor(const self&)=default; self& operator=(const self&)=default;
    ctor(V* p) noexcept; ctor<NonConstV>(const self<NonConstV>& x) noexcept requires is_same_v<V, const NonConstV>;
    self& operator=(V* p) noexcept; self& operator= <NonConstV> (const self<NonConstV>& x) noexcept requires is_same_v<V, const NonConstV>;
    operator V* () const noexcept;
    explicit operator fixed_variant_alternative_iterator<NonConstV,T> <T,NonConstT=remove_const_t<T>,NonConstV=remove_const_t<V>> () const noexcept
        requires mp_contains<NonConstV,NonConstT>::value && (!is_const_v<V> || is_const_v<T>);
    size_t stride() const noexcept
};

struct detail::variant_model_is_subvariant<V,...Ts>;
struct detail::variant_model_is_subset<TL1,TL2>;
auto detail::invoke_visit<F,...Ts>(F&& f, const fixed_variant<Ts...>& x) -> decltype(...) { return visit(std::forward<F>(f),x); }
auto detail::invoke_visit<F,...Ts>(F&& f, const std::variant<Ts...>& x) -> decltype(...) { return std::visit(std::forward<F>(f),x); }
auto detail::invoke_visit<F,...Ts>(F&& f, const boost::variant2::variant<Ts...>& x) -> decltype(...) { return visit(std::forward<F>(f),x); }

struct detail::variant_model<...Ts> {
    using value_type = fixed_variant<Ts...>; using type_index = size_t; using acceptable_type_list = mp_list<Ts...>;
    using is_terminal<T> = mp_contains<acceptable_type_list,T>;
    using is_implementation<Concrete> = std::bool_constant<is_terminal<T>::value || variant_model_is_subvariant<T,Ts...>::value>;
    static const type_index index<T>() { return mp_find<acceptable_type_list,T>::value; }
    static const type_index subindex<T>(const T&) requires is_terminal<T>::value { return index<T>(); }
    static const type_index subindex<V<...>,...Qs>(const V<Qs...>& x) requires !is_terminal<T>::value
    { return mp_with_index<sizeof...(Qs)>(i,[](auto i){return mp_find<acceptable_type_list,mp_at<mp_list<Qs...>,i>>::value;}); }
    static <const> void* subaddress(<const>T& x)
    { if constexpr (is_terminal<T>::value) return addressof(x); else return invoke_visit([](auto const& x){return (const void*)addressof(x);}); }
    static const std::type_info& subtype_info<T> (const T& x)
    { if constexpr (is_terminal<T>::value) return typeid(x); else return invoke_visit([](auto const& x){return typeid(T);}); }
    using <const>_base_iterator = fixed_variant_iterator<<const> value_type>;
    using <const>_base_sentinel = <const> value_type*;
    using <const>_iterator<Concrete> = fixed_variant_alternative_iterator<value_type, <const> T>;
    using segment_backend<Alloc> = segment_backend<self,Alloc>;
    using segment_backend_implementation<T,Alloc> = split_segment<self,T,Alloc>;
    static base_iterator nonconst_iterator(const_base_iterator it);
    static iterator<T> nonconst_iterator<T>(const_iterator<T> it);
};

class variant_collection<TypeList,Alloc> : public common_impl::poly_collection<mp_rename<TypeList,variant_model>,Alloc> {
    using base_type = base; <const> base_type& base() <const> noexcept;
public: using base::ctor; // all 5 ctor/op= =default
};
bool operator== <TL,A> (const variant_collection<TL,A>& x, const variant_collection<TL,A>& y); // and !=
void swap<TL,A> (variant_collection<TL,A>& x, variant_collection<TL,A>& y);
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`

#### Boost.Core

* `<boost/core/addressof.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_adaptor.hpp>`
* `<boost/iterator/iterator_facade.hpp>`

#### Boost.MP11

* `<boost/mp11/**.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.TypeErasure

* `<boost/type_erasure/**.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

------
### Standard Facilities
