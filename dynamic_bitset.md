# Boost.DynamicBitset

* lib: `boost/libs/dynamic_bitset`
* repo: `boostorg/dynamic_bitset`
* commit: `11be69a`, 2025-09-19

------
### Dynamic bitset

#### Header

* `<boost/dynamic_bitset_fwd.hpp>`
* `<boost/dynamic_bitset.hpp>`

```c++
class detail::is_container<C,B>::value = is_same<C::value_type, B> && requires(C c) { {c.resize(size_t{})}; {u[0]}; };
using detail::allocator_type_extractor<AorC,B>::type = 
T::size_type detail::vector_max_size_workaround<T> (const T& v) noexcept
{ return min(allocator_traits<T::allocator_type>::max_size(v.get_allocator()), v.max_size()); }
struct detail::allowed_block_type<T>::value = is_unsigned_v<T> && !is_same_v<T,bool>;
struct detail::is_numeric<T>::value = is_character_v<T> || is_integral_v<T>;

class bit_iterator_base<It> {
protected:
  It m_block_iterator; int m_bit_index=0;
  void increment(); void decrement(); void add(It::difference_type n);
public:
    using iterator_category = It::iterator_category;
    using value_type = bool; using difference_type = ptrdiff_t; using pointer = bool*; using reference = bool&;
    static constexpr int bits_per_block = std::numeric_limits< Block >::digits;
    ctor(It, int idx);
};
struct bit_iterator<DBS> : bit_iterator_base<DBS::buffer_type::iterator> {
    using reference = DBS::reference; using pointer = reference*;
    ctor(); ctor(DBS::buffer_type::iterator block_iterator, int bit_index);
    // *, ++, --, +=, -=, []
};
struct const_bit_iterator<DBS> : bit_iterator_base<DBS::buffer_type::const_iterator> {
    using reference = bool; using const_reference = bool; using pointer = bool*;
    ctor(DBS::buffer_type::const_iterator block_iterator, int bit_index);
    ctor(const bit_iterator<DBS>&); // nonconst -> const
    // *, ++, --, +=, -=, []
};
bool operator== <It> (const bit_iterator_base<It>&, const bit_iterator_base<It>&); // also <

class dynamic_bitset<Block,AllocOrCont> { // requires allowed_block_type<Block>, Block=AllocOrCont::value_type
    buffer_type m_bits; size_type m_num_bits;
public:
    using block_type = Block; using size_type = size_t;
    using allocator_type = is_container<AllocOrCont,Block> ? AllocOrCont::allocator_type : AllocOrCont;
    using buffer_type = is_container<AllocOrCont,Block> ? AllocOrCont : std::vector<Block,allocator_type>;
    static constexpr int bits_per_block = std::numeric_limits< Block >::digits;
    static constexpr size_type npos = -1;

    class reference { // proxy ref class
        block_type& m_block; const block_type m_mask;
        void operator&() = delete;
    public: // copy-ctor, copy-assign
        operator bool() const; bool operator~() const;
        reference& flip();
        reference& operator=(bool); // |=, &=, ^=, -=
    };
    using const_reference = bool;
    using iterator = bit_iterator<dynamic_bitset>; using const_iterator = const_bit_iterator<dynamic_bitset>;
    using reverse_iterator = std::reverse_iterator<iterator>; using const_reverse_iterator = std::reverse_iterator<const_iterator>;

    ctor(); explicit ctor(const allocator_type&); // and copy-ctor, move-ctor, copy-assign, move-assign, dtor
    explicit ctor (size_type nbits, unsigned long v=0, const allocator_type& alloc={});
    explicit ctor<Ch,Tr,A> (const std::basic_string<Ch,Tr,A>& s, size_type nbits=npos, const allocator_type& alloc={});
    explicit ctor<Ch,Tr> (const std::basic_string_view<Ch,Tr>& sv, size_type nbits=npos, const allocator_type& alloc={});
    explicit ctor<Ch> (const Ch* s, size_t n=-1, size_type nbits=npos, const allocator_type& alloc={});
    ctor<BIt> (BIt f, BIt l, const allocator_type& alloc={});

    [const_]iterator begin()[const]; [const_]iterator end()[const];
    [const_]reverse_iterator rbegin()[const]; [const_]reverse_iterator rend()[const];
    const_iterator cbegin() const; const_iterator end() const;
    const_reverse_iterator crbegin() const; const_reverse_iterator crend() const;

    void swap(dynamic_bitset&) noexcept;

    allocator_type get_allocator() const;
    size_type size() const noexcept; bool empty() const noexcept;
    void resize(size_type nbits, bool v=false);
    void clear();

    size_type capacity() const noexcept; size_type num_blocks() const noexcept; // current capacity
    size_type max_size() const noexcept; // allocation limit
    void reserve(size_type nbits); void shrink_to_fit();

    unsigned long to_ulong() const; // throws overflow_error if size is too large
    void push_{back|front}(bool bit); void pop_{back|front}();
    void append(Block b); void append<BIt>(BIt f, BIt l);

    // set operations
    bool all() const; bool any() const; bool none() const; size_t count() const noexcept;
    dynamic_bitset& operator&= (const dynamic_bitset& b); // also |=, ^=, -=
    dynamic_bitset& operator<<= (size_type n); // and >>
    dynamic_bitset operator<< (size_type n) const; // and >>
    dynamic_bitset operator~() const; // flip all
    bool is_subset_of(const dynamic_bitset& b) const;
    bool is_proper_subset_of(const dynamic_bitset& b) const;
    bool intersects(const dynamic_bitset& b) const;

    // element accessing
    bool test(size_type pos) const; // getter
    reference operator[] (size_type pos); bool operator[] (size_type pos) const;
    reference at(size_type pos); bool at(size_type pos) const; // throws out_of_range for invalid pos
    bool test_set(size_type pos, bool val=true); // exchange
    // set n
    dynamic_bitset& set(size_type pos, size_type len, bool val);
    dynamic_bitset& reset(size_type pos, size_type len);
    dynamic_bitset& flip(size_type pos, size_type len);
    // set 1
    dynamic_bitset& set(size_type pos, bool val=true);
    dynamic_bitset& reset(size_type pos);
    dynamic_bitset& flip(size_type pos);
    // set all
    dynamic_bitset& set();
    dynamic_bitset& reset();
    dynamic_bitset& flip();

    // find
    size_type find_first(size_type pos=0) const; size_type find_first_off(size_type pos=0) const;
    size_type find_next(size_type pos) const; size_type find_next_off(size_type pos) const;
};
bool operator== <B,A> (const dynamic_bitset<B,A>&, const dynamic_bitset<B,A>&); // also !=, <, >, <=, >=
void swap<B,A>(dynamic_bitset<B,A>&, dynamic_bitset<B,A>&) noexcept;

dynamic_bitset<B,A> operator& <B,A> (const dynamic_bitset<B,A>&, const dynamic_bitset<B,A>&); // and |, ^, -

void to_string<B,A,Str> (const dynamic_bitset<B,A>&, Str&);
void to_block_range<B,A,BIt> (const dynamic_bitset<B,A>&, BIt out);
void from_block_range<BIt,B,A> (BIt f, BIt l, dynamic_bitset<B,A>& out);

std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,B,A> (std::basic_ostream<Ch,Tr>&, const dynamic_bitset<B,A>&);
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr,B,A> (std::basic_istream<Ch,Tr>&, dynamic_bitset<B,A>&);
size_t hash_value<B,A> (const dynamic_bitset<B,A>&);
```

#### API difference with `std::bitset`

* Argumented by `Block` type and `Allocator`, by default are `ulong` and `std::allocator<Block>`
* Also can provide underlying container.
* Wraps a `std::vector<Block>` for internal storage.
* Member types `block_type`, `allocator_type`, `size_type`
* Member constants `bits_per_block` and `npos`
* Member `reference` provides `|=`, `&=`, `^=`, and `-=` (in addition to `std::bitset::reference`)
* Constructors have additional allocator parameter
* Provides Block-range oriented constructor and `assign`, `append`, and `to_block_range`, `from_block_range` functions.
* Additionally provides move-constructor and move-assignment, and `swap`
* Provides `vector`-like members `resize`, `clear`, `push_back`, `pop_back`, `empty`, `size`, `at`
* Provides capacity members `num_blocks`, `max_size`, `capacity`, `reserve`, `shrink_to_fit`
* Provides set operations `-=`, `-`, `is_subset_of`, `is_proper_subset_of`, `intersects`
* Provides lookup: `find_first(bs, offset=0)`, `find_pos(bs, pos)`, `find_first_off(pos=0)`
* Provides ordering `<` etc.
* Additional member overloads `set(n, len, val=true)`, `reset(n, len)`, `flip(n, len)`

### Serialization of `dynamic_bitset`

Header `<boost/dynamic_bitset/serialization.hpp>`

Provide overload for `serialize`.

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/limits.hpp>`

#### Boost.ContainerHash

* `<boost/functional/hash/hash.hpp>`

#### Boost.Core

* `<boost/core/bit.hpp>`
* `<boost/core/no_exceptions_support.hpp>`
* `<boost/core/nvp.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

------
### Standard Facilities
