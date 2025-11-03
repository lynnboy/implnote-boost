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
    std::iterator_traits<BaesIt>::reference operator[] <Diff> (Diff n) const;
};

class stride_iterator<Value> : public boost::iterator_facade<self, Value, random_access_traversal_tag> {
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
    using segment_allocator_type = std::allocator_traits<Alloc>::rebind_alloc<store_value_type>;
    using store = std::vector<store_value_type, segment_allocator_type>;
    using <const>_store_iterator = store::<const>_iterator;
    using const_iterator = base::const_iterator<Concrete>;
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

------
#### `base_collection`

* Header `<boost/poly_collection/base_collection.hpp>`

```c++
struct base_model<Base> {
    using value_type = Base; using type_index = std::type_info;
    using is_implementation<Derived> = std::is_base_of<Base,Derived>;
    using is_terminal<T> = is_final<T>;
    static const std::type_info& index();
    static const std::type_info& subindex(const T&) { if constexpr(is_terminal<T>) return typeid(T); else return typeid(x); }
    static <const> void* subaddress(<const>T& x) { if constexpr(is_terminal<T>) return addressof(x); else return dynamic_cast<void*>(addressof(x)); }
    using <const>_base_iterator = stride_iterator<[const]Base>;
    using <const>_base_sentinel = <const> Base*;
    using <const>_iterator<Derived> = <const> Derived*;
    using segment_backend<Alloc>=segment_backend<self,Alloc>;
    using segment_backend_implementation<Derived,Alloc>=packed_segment<self,Derived,Alloc>;
    static base_iterator nonconst_iterator(const_base_iterator it);
    static iterator<T> nonconst_iterator<T>(const_iterator<T> it);
};
```

------
#### XXH3-128

* Header `<boost/hash2/xxh3.hpp>`

```c++
```

------
#### SipHash and HalfSipHash

* Header `<boost/hash2/siphash.hpp>`

```c++
```

------
#### HMAC

* Header `<boost/hash2/hmac.hpp>`

```c++
```

------
#### MD5

* Header `<boost/hash2/md5.hpp>`

```c++
```

------
#### SHA-1

* Header `<boost/hash2/sha1.hpp>`

```c++
```

------
#### SHA-2 Family

* Header `<boost/hash2/sha2.hpp>`

```c++
```

------
#### SHA-3 Family

* Header `<boost/hash2/sha3.hpp>`

```c++
```

------
#### RIPEMD Family

* Header `<boost/hash2/ripemd.hpp>`

```c++
```

------
#### BLAKE2 Faimly

* Header `<boost/hash2/blake2.hpp>`

```c++
```

------
### Common Bits

```c++
```

------
### Object Hashing

* Header `<boost/hash2/hash_append.hpp>`, `<boost/hash2/hash_append_fwd.hpp>`

```c++
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
