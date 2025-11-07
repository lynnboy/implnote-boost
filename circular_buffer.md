# Boost.CircularBuffer

* lib: `boost/libs/circular_buffer`
* repo: `boostorg/circular_buffer`
* commit: `0320ba3`, 2025-05-03

------
### Circular Buffer

#### Common Parts

* Header `<boost/circular_buffer.hpp>`, `<boost/circular_buffer_fwd.hpp>`

```c++
void detail::uninitialized_fill_n_with_alloc<FwdIt,Diff,T,Alloc>(FwdIt first, Diff n, const T& item, Alloc& a) {
    FwdIt next = first;
    try { for(; n>0; ++first, --n) allocator_construct(a, to_address(dest), item); }
    catch(...) { for(; next!=first; ++next) allocator_destroy(a, to_address(next)); throw; }
}
FwdIt detail::uninitialized_copy<InIt,FwdIt,Alloc>(InIt first, InIt last, FwdIt dest, Alloc& a) {
    FwdIt next = dest;
    try { for(; first!=last; ++first, ++dest) allocator_construct(a, to_address(dest), *first); }
    catch(...) { for(; next!=dest; ++next) allocator_destroy(a, to_address(next)); throw }
    return dest;
}
FwdIt detail::uninitialized_move_if_noexcept<InIt,FwdIt,Alloc>(InIt first, InIt last, FwdIt dest, Alloc& a) {
    if constexpr (is_nothrow_move_constructible_v<Alloc::value_type>)
    { for (; first!=last; ++first, ++dest) allocator_construct(a, to_address(dest), move(*first)); return dest; }
    else return uninitialized_copy(first, last, dest, a);
}

struct detail::<non>const_traits<Alloc> {
    using value_type = Alloc::value_type; using reference = <const> value_type&; using pointer = allocator_<const>_pointer<Alloc>;
    using difference_type = allocator_difference_type<Alloc>::type; using size_type = allocator_size_type<Alloc>::type;
    using nonconst_self = nonconst_traits<Alloc>;
};
class detail::capacity_control<Size> { Size m_capacity, m_min_capacity;
public: ctor(Size buffer_capacity, Size min_buffer_capacity=0);
    Size capacity() cosnt; Size min_capacity() const; operator Size() const;
};
struct detail::iterator<Buf,Tr> { // : debug_iterator_base (when BOOST_CB_ENABLE_DEBUG)
    using nonconst_self = iterator<Buf,Tr::nonconst_self>; // using Tr::value_type, pointer, reference, size_type, difference_type
    using iterator_category = std::random_access_iterator_tag;
    const Buf* m_buff{nullptr}; pointer m_it{nullptr};
    ctor(); ctor(const nonconst_self& it); ctor(const Buf* cb, pointer p); self& operator=(const self&);
    reference operator*() const{return *m_it;} pointer operator->() const{return &*m_it;}
    difference_type operator- <Tr0> (const self<Buf,Tr0>& it) const {return linearize_pointer(*this) - linearize_pointer(it);}
    self& operator++() { m_buff->increment(m_it); if (m_it==m_buff->m_last) m_it=0; return *this;}
    self& operator--() { if (m_it==0) m_it=m_buff->m_last; m_buff->decrement(m_it); return *this;}
    self operator{++|--}(int) {self tmp{*this}; {++|--}*this; return tmp;}
    self& operator+=(difference_type n) { if (n>0) {m_it=m_buff->add(m_it,n); if(m_it==m_buff->m_last)m_it=0;} else if (n<0) *this -= -n; return *this; }
    self& operator-=(difference_type n) { if (n>0) m_it=m_buff->sub(m_it==0?m_buff->m_last:m_it,n); else if (n<0) *this += -n; return *this; }
    self operator{+|-}(difference_type n) const { return self{*this} {+|-}= n; }
    reference operator[](difference_type n) const { return *(*this+n); }
    bool operator== <Tr0>(const self<Buf,Tr0>& it) const { return m_it == it.m_it; } // and !=
    bool operator< <Tr0>(const self<Buf,Tr0>& it) const { return linearize_pointer(*this) < linearize_pointer(it); } // derive: >, <=, >=
    Tr0::pointer linearize_pointer<Tr0>(const self<Buf,Tr0>& it) const
    { return it.m_it==0 ? m_buff->m_buff + m_buff->size() :
            it.m_it<m_buff->m_first ? it.m_it + (m_buff->m_end - m_buff->m_first)
                : m_buff->m_buff + (it.m_it - m_buff->m_first); }
    friend operator+(difference_type n, const self& it) { return it + n; }
};
```

#### Main Class

```c++
class circular_buffer<T,Alloc=std::allocator<T>> : private empty_value<Alloc> { // Alloc as EBO
    pointer m_buff, m_end, m_first, m_last; size_type m_size;
public: using value_type = Alloc::value_type; using allocator_type = Alloc;
    using <const>_pointer = allocator_<const>_pointer<Alloc>::type; using <const>_reference = <const> value_type&;
    using difference_type = allocator_difference_type<Alloc>::type; using size_type = allocator_size_type<Alloc>::type;
    using <const>_iterator = iterator<self,<non>const_traits<Alloc>>;
    using <const>_reverse_iterator = std::reverse_iterator<<const>_iterator>;
    using <const>_array_range = std::pair<<const>_pointer,size_type>;
    using capacity_type = size_type; using param_value_type = const value_type&; using rvalue_type = value_type&&;

    explicit ctor(<capacity_type cap>, const allocator_type& alloc={});
    ctor(<capacity_type cap>, size_type n, param_value_type item, const allocator_type& alloc={});
    ctor(const self& r); ctor(self&& r) noexcept;
    ctor<InIt>(<capacity_type cap>, InIt first, InIt last, const allocator_type& alloc={});
    ~dtor() noexcept;
    self& operator=(const self& r); self& operator=(self&& r) noexcept;
    void assign(<capacity_type cap>, size_type n, param_value_type item);
    void assign<InIt>(<capacity_type cap>, InIt first, InIt last);
    void swap(self& r) noexcept;

    allocator_type get_allocator() const noexcept { return alloc(); } allocator_type& get_allocator() noexcept { return alloc(); }
    <const>_iterator begin() noexcept { return {this, empty()?0:m_first}; } <const>_iterator end() noexcept { return {this, 0}; }
    const_iterator c{begin|end}() const noexcept { return {begin|end}(); }
    <const>_reverse_iterator {begin|end} <const> noexcept { return {{begin|end}()}; }
    const_reverse_iterator c{begin|end} const noexcept { return r{begin|end}(); }
    <const>_reference operator[] (size_type index) <const> { return *add(m_first,index); }
    <const>_reference at(size_type index) <const> { return (*this)[index]; }
    <const>_reference front() <const> { return *m_first; }
    <const>_reference back() <const> { return *((m_last==m_buff ? m_end : m_last) - 1); }

    <const>_array_range array_one() const { return {m_first, (m_last<=m_first&&!empty() ? m_end : m_last) - m_first}; }
    <const>_array_range array_two() const { return {m_buff, m_last<=m_first&&!empty() ? m_last - m_first : 0}; }
    pointer linearize();
    bool is_linearized() const noexcept { return m_first<m_last || m_last==m_buff; }
    void rotate(const_iterator new_begin);

    size_type size() const noexcept { return m_size; }
    size_type max_size() const noexcept { return std::min(allocator_max_size(alloc()), std::numeric_limits<difference_type>::max()); }
    bool empty() const noexcept { return size() == 0; }
    bool full() const noexcept { return capacity() == size(); }
    size_type reserve() const noexcept { return capacity() - size(); }
    capacity_type capacity() const noexcept { return m_end - m_buff; }
    void <r>set_capacity(capacity_type new_capacity);
    void <r>resize(size_type new_size, param_value_type item={});

    void push_{back|front}(param_value_type item); void push_{back|front}(rvalue_type item={});
    void pop_{back|front}();
    iterator <r>insert(iterator pos, param_value_type item); iterator <r>insert(iterator pos, rvalue_type item={});
    void <r>insert(iterator pos, size_type n, param_value_type item);
    void <r>insert<InIt>(iterator pos, InIt first, InIt last);
    iterator <r>erase(iterator pos); iterator <r>erase(iterator first, iterator last);
    void erase_{begin|end}(size_type n);
    void clear() noexcept;

    friend bool operator==(const self& lhs, cosnt self& rhs) { return lhs.size() == rhs.size() && std::equal(lhs.begin(), lhs.end(), rhs.begin()); }
    friend bool operator<(const self& lhs, cosnt self& rhs) { return std::lexicographical_compare(lhs.begin(), lhs.end(), rhs.begin(), rhs.end()); }
    // others: !=, >, <=, >=
    friend void swap(self& lhs, self& rhs) noexcept { lhs.swap(rhs); }
};
```

#### Space Optimized Implementation

```c++
class circular_buffer_space_optimized<T,Alloc> : circular_buffer<T,Alloc> {
    capacity_type m_capacity_ctrl {0};
public: using base::value_type; // all types from base
    using capacity_type = capacity_control<size_type>;
    using base::get_allocator; // members from base: <r>begin, <r>end, at, operator[], front, back, array_one/two, linearize, is_linearized, rotate, size, max_size, empty
    // reimplement upon m_capacity_ctrl: capacity, full, reserve, <r>set_capacity, <r>resize
    // all ctor, op=, and assign, swap from base with additional init for m_capacity_ctrl
    // push_xx, pop_xx, <r>insert, <r>erase from base with additional capacity check
};
```

#### Debugging support

```c++
const int detail::UNINITIALIZED = 0xcc;
void detail::do_fill_uninitialized_memory<T>(T* p, size_t n) noexcept { std::memset(p, UNINITIALIZED, n); }
void detail::do_fill_uninitialized_memory<T>(T&, size_t) noexcept {}

class detail::debug_iterator_base { using reg = debug_iterator_registry;
    mutable const reg* m_registry{nullptr}; mutable const self* m_next{nullptr}; // s-list root/next
    void register_self() { if(m_registry) m_registry->register_iterator(this); }
    void unregister_self() { if(m_registry) m_registry->unregister_iterator(this); }
public: ctor(){} ctor(const reg* registry):m_registry{reg} {register_self();}
    ctor(const self& rhs) :m_registry{rhs.m_registry}{register_self();} ~dtor(){ unregister_self(); }
    self& operator=(const self& rhs) {
        if (m_registry==rhs.m_registry) return *this;
        unregister_self(); m_registry = rhs.m_registry(); register_self(); return *this;
    }
    bool is_valid(const reg* registry) const { return m_registry == registry; } void invalidate() const { m_registry = 0; }
    const self* next() const { return m_next; } void set_next(const self* it) const { m_next = it; }
};
class detail::debug_iterator_registry { using iter = debug_iterator_base;
    mutable const iter* m_iterators{nullptr}; // s-list head
    void remove(const iter* cur, const iter* prev) const { if (prev==0) m_iterators=m_iterators->next(); else prev->set_next(cur->next); }
public: ctor(){};
    void register_iterator(const iter* it) const { it->set_next(m_iterators); m_iterators = it; }
    void unregister_iterator(const iter* it) const { const iter* prev=0; for(auto p=m_iterators; p!=it; prev=p, p=p->next()); remove(it, prev); }
    void invalidate_iterators<It>(const It& it) {
        const iter* prev=0;
        for (auto p=m_iterators; p!=0; p=p->next())
        { if (((It*)p)->m_it == it.m_it) { p->invalidate(); remove(p, prev); continue; } prev=p; }
    }
    void invalidate_iterators_except<It>(const It& it) {
        const iter* prev=0;
        for (auto p=m_iterators; p!=0; p=p->next())
        { if (((It*)p)->m_it != it.m_it) { p->invalidate(); remove(p, prev); continue; } prev=p; }
    }
    void invalidate_all_iterators() { for (auto p=m_iterators; p!=0; p=p->next()) p->invalidate(); m_iterators = 0; }
}；
```

------
### Configuration

* `BOOST_CB_ENABLE_DEBUG` enable debugging.

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>` - used for debugging

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`
* `<boost/limits.hpp>`

#### Boost.Core

* `<boost/core/allocator_access.hpp>`
* `<boost/core/empty_value.hpp>`
* `<boost/core/no_exceptions_support.hpp>`
* `<boost/core/pointer_traits.hpp>`

#### Boost.Move

* `<boost/move/adl_move_swap.hpp>`
* `<boost/move/move.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

------
### Standard Facilities
