# Boost.Bloom

* lib: `boost/libs/bloom`
* repo: `boostorg/bloom`
* commit: `d4b5eb6`, 2025-09-14

------

* Header `<boost/bloom.hpp>`

#### Subfilter types: `block`, `multiblock`, `fast_multiblock32`, `fast_multiblock64`

```c++
// type traits
using detail::is_nothrow_swappable<T> = ...; // requires (T& x) { {swap(x,x)}; } && noexcept(swap(x,x))
using detail::is_cv_unqualified_object<T> = ...; // remove_cv_t<T> == T && !is_function && !is_reference && !is_void
using detail::remove_cvref<T>::type = detail::remove_cvref_t<T> = std::remove_cv_t<std::remove_reference_t<T>>;
using detail::is_transparent<T> = {SFINAE T::is_transparent};
using detail::enable_if_transparent_t<T,Q=void> = std::enable_if<is_transparent<T>::value,Q>::type;
using detail::is_unsigned_integral_or_extended_unsigned_integral<T> = std::is_integral<T> && std::is_unsigned<T>; // supports __int128
using detail::is_array_of<T, Tr<...>> = false_type; // primary, default
using detail::is_array_of<T[N],Tr> = Tr<T>;
using detail::array_size<T> = std::integral_constant<size_t,0>; // primary, default
using detail::array_size<T[N]> = std::integral_constant<size_t,N>;
using detail::is_power_of_two<N> = std::bool_constant<N!=0 && (N & N-1) == 0>;

constexpr size_t detail::constexpr_bit_width(size_t x);
uint64_t detail::mulx64(uint64_t x) noexcept { uint128_t x = x*0x9E3779B97F4A7C15ull; return x.hi^x.lo; }

struct detail::block_base<Block,K> { // requires Block is unsigned integral or array of it, array size is power of 2
    static constexpr size_t k=K, hash_width=sizeof(uint64_t)*CHAR_BIT, block_width=sizeof(Block)*CHAR_BIT,
        mask=block_width-1, shift=constexpr_bit_width(mask), rehash_k=(hash_width-shift)/shift;
    static void loop<F>(uint64_t hash, F f); // call f on each hash value
    static bool loop_while<F>(uint64_t hash, F f); // loop, returns when f returns false
};

struct detail::block_ops<Block> {
    using is_extended_block = std::false_type; using value_type = Block;
    static void zero(value_type& x) { x = 0; }
    static void set(value_type& x, uint64_t n) { x |= 1 << n; }
    static int get_at_lsb(value_type const& x, uint64_t n) { return static_cast<int>(x >> n); }
    static void reduce(int& res, value_type const& x, uint64_t n) { res &= get_at_lsb(x, n); }
    static bool testc(value_type const& x, value_type const& y) { return (x & y) == y; }
};
struct detail::block_ops<Block[N]> { // specialize for array
    using is_extended_block = std::true_type; using value_type = Block[N];
    static void zero(value_type& x); // fill array with 0
    static void set(value_type& x, uint64_t n) { x[n%N] |= 1 << (n/N); }
    static int get_at_lsb(value_type const& x, uint64_t n) { return static_cast<int>(x[n%N] >> (n/N)); }
    static void reduce(int& res, value_type const& x, uint64_t n) { res &= get_at_lsb(x, n); }
};

struct detail::block_fpr_base<K> { static double fpr(size_t i, size_t w) }; // ( 1 - (1-1/w)^{K*i} )^K
struct detail::multiblock_fpr_base<K> { static double fpr(size_t i, size_t w) }; // ( 1 - (1-K/w)^i )^K

struct block<Block,K> : public block_fpr_base<K>, private block_base<Block,K> { // fpr, loop, loop_while, mask
    static constexpr size_t k = K; using value_type = Block; using block_ops = block_ops<Block>;
    static void mark(value_type& x, uint64_t hash) { loop(hash, [&](uint64_t h){block_ops::set(x, h&mask);}); }
    static bool check(const value_type& x, uint64_t hash) { 
        if constexpr (block_ops::is_extended_block)
        { int res=1; loop(hash, [&](uint64_t h){res &= block_ops::get_at_lsb(x, h&mask);}); return res; }
        else { Block fp{0}; mark(fp, hash); return block_ops::testc(x, fp); }
    }
};

struct multiblock<Block,K> : public multiblock_fpr_base<K>, private block_base<Block,K> { // fpr, loop, loop_while, mask
    static constexpr size_t k = K; using value_type = Block[k]; using block_ops = block_ops<Block>;
    static void mark(value_type& x, uint64_t hash) { size_t i{0}; loop(hash, [&](uint64_t h){block_ops::set(x[i++], h&mask);}); }
    static bool check(const value_type& x, uint64_t hash) { 
        int res=1; size_t i{0} loop(hash, [&](uint64_t h){block_ops::reduce(res, x[i++], h&mask);}); return res;
    }
};

struct detail::m128ix2 { __m128i lo,hi; }; // for SSE2
struct detail::m256ix2 { __m256i lo,hi; }; // for AVX2
struct fast_multiblock32<K> : public multiblock_fpr_base<K> { // optimized for AVX2, SSE2, NEON
    static constexpr size_t k = K, used_value_size = sizeof(uint32_t) * k;
    using value_type = __m256i[(k+7)/8]; // __m256i for AVX2, m128ix2 for SSE2, uint32x4x2_t for NEON
    static void mark(value_type& x, uint64_t hash);
    static bool check(const value_type& x, uint64_t hash);
};
struct fast_multiblock64<K> : public multiblock_fpr_base<K> { // optimized for AVX2
    static constexpr size_t k = K, used_value_size = sizeof(uint64_t) * k;
    using value_type = m256ix2[(k+7)/8]; // m256ix2 for AVX2
    static void mark(value_type& x, uint64_t hash);
    static bool check(const value_type& x, uint64_t hash);
};
using fast_multiblock32<K> = multiblock<uint32_t,K>; // fallback
using fast_multiblock64<K> = multiblock<uint64_t,K>; // fallback
```

#### Filter API

```c++
struct detail::fastrange_and_mcg {
    uint64_t rng; constexpr ctor(size_t m) noexcept; // seed and init
    void prepare_hash(uint64_t& hash) const noexcept { hash |= 1u; }
    size_t next_position(uint64_t& hash) const noexcept;
};
using detail::used_value_size<Subfilter> = {SFINAE Subfilter::used_value_size} ? Subfilter::used_value_size : sizeof(Subfilter::value_type);
struct detail::filter_array { unsigned char* data, array; };

struct detail::no_mix_policy { static uint64_t mix <Hash,T>(const Hash& h, const T& x) { return h(x); } };
struct detail::mulx64_mix_policy { static uint64_t mix <Hash,T>(const Hash& h, const T& x) { return mulx64(h(x)); } };

template<size_t K, class Subfilter, size_t Stride, class Alloc>
class detail::filter_core : empty_value<Alloc, 0> { // requires K > 0, allocator_value_type_t<Alloc> == unsigned char
    using block_type = subfilter::value_type;
    static constexpr size_t kp=subfilter::k, k_total=k*kp, used_value_size=used_value_size<subfilter>::value;
    static constexpr size_t block_size=sizeof(block_type), tail_size = block_size-stride;
    using hash_strategy = fastrange_and_mcg;

    hash_strategy hs; filter_array ar;

    [const] Alloc& al() [const]; // get allocator
    static size_t requested_range(size_t m);
    static filter_array new_array(allocator_type& al, size_t rng); // factory
    void delete_array() noexcept;
    void clear_bytes() noexcept; void copy_bytes(const filter_core& x) noexcept;
    size_t range() const noexcept;
    static constexpr size_t space_for(size_t rng) noexcept;
    static unsigned char* array_for(unsigned char* p) noexcept;
    size_t used_array_size() const noexcept; static size_t used_array_size(size_t rng) noexcept;
    static size_t unadjusted_capacity_for(size_t n, double fpr);
    static double fpr_for_c(double c);
    bool get(const unsigned char* p, uint64_t hash) const;
    void set(unsigned char* p, uint64_t hash);
    [const] unsigned char* next_element(uint64_t& h) [const] noexcept;
    void combine<F>(const filter_core& x, F f);

public:
    using subfilter=Subfilter;
    static constexpr size_t k=K, stride=Stride?Stride:used_value_size;
    using allocator_type = Alloc;
    using size_type = size_t; using difference_type = ptrdiff_t;
    using pointer = unsigned char*; using const_pointer = const unsigned char*;

    explicit ctor(size_t m=0, const allocator_type& a={});
    ctor(size_t n, double fpr, const allocator_type& a); // m = unadjusted_capacity_for(n,fpr)
    //copy-ctor, move-ctor, and both with allocator argument, copy-assign, move-assign, dtor

    allocator_type get_allocator() const noexcept;
    size_t capacity() const noexcept;
    static size_t capacity_for(size_t n, double fpr);
    static double fpr_for(size_t n, size_t m); // m/n
    span<[const]unsigned char> array() [const] noexcept { return { ar.data? ar.array : nullptr, capacity()/CHAR_BIT }; }

    void insert(uint64_t hash);
    void swap(filter_core& x) noexcept;
    void clear() noexcept; void reset(size_t m=0); void reset(size_t n, double fpr);

    filter_core& operator&= (const filter_core& x);
    filter_core& operator|= (const filter_core& x);

    bool may_contain(uint64_t hash) const;
};
bool operator== <K,Subfilter,S,A> (const filter_core<K,Subfilter,S,A>& x, const filter_core<K,Subfilter,S,A>& y);

template<class T, size_t K, class Subfilter = block<unsigned char,1>, size_t Stride=0, class Hash=hash<T>, class Alloc=std::allocator<unsigned char>>
class filter : private filter_core<K,Subfilter,Stride,allocator_rebind_t<Alloc,unsigned char>>, private empty_value<Hash,0> {
    using mix_policy = hash_is_avalanching<Hash>::value && sizeof(size_t) >= sizeof(uint64_t) ? no_mix_policy : mulx64_mix_policy;
public:
    using value_type = T; // k, subfilter, stride, size_type, difference_type same as filter_core
    using hasher = Hash; using allocator_type = Alloc;
    using reference = value_type&; using const_reference = const value_type&;
    using pointer = value_type*; using const_pointer = const value_type*;

    explicit ctor(const allocator_type& al={});
    ctor(const filter& x, const allocator_type& al={});
    ctor(filter&& x, const allocator_type& al={});
    explicit ctor(size_t m, [const hasher& h={}], const allocator_type& al={});
    ctor(size_t n, double fpr, [const hasher& h={}], const allocator_type& al={});
    ctor <It> (It f, It l, size_t m, [const hasher& h={}], const allocator_type& al={});
    ctor <It> (It f, It l, size_t n, double fpr, [const hasher& h={}], const allocator_type& al={});
    ctor(std::initializer_list<value_type> il, size_t m, [const hasher& h={}], const allocator_type& al={});
    ctor(std::initializer_list<value_type> il, size_t n, double fpr, [const hasher& h={}], const allocator_type& al={});

    filter& operator=(const filter& x); filter& operator=(filter&& x);
    filter& operator=(std::initializer_list<vlaue_type> il);

    // get_allocator, capacity, capacity_for, fpr_for, array from filter_core
    void insert(const T& x);
    void insert <U,H=hasher> (const U& x) requires enable_if_transparent_t<H>;
    void insert <It> (It f, It l); void insert(std::initializer_list<value_type> il);
    void swap(filter& x);
    // clear, reset, operator&=, operator|= from filter_core

    hasher hash_function() const; { return h{}; }
    bool may_contain(const T& x) const;
    bool may_contain<U,H=hasher> (const U& x) const requires enable_if_transparent_t<H>;
};
bool operator== <T,K,SF,S,H,A> (const filter<T,K,SF,S,H,A>& x, const filter<T,K,SF,S,H,A>& y); // and !=
void swap<T,K,SF,S,H,A> (filter<T,K,SF,S,H,A>& x, filter<T,K,SF,S,H,A>& y);
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.ContainerHash

* `<boost/container_hash/hash.hpp>`
* `<boost/container_hash/hash_is_avalanching.hpp>`

#### Boost.Core

* `<boost/core/allocator_traits.hpp>`
* `<boost/core/empty_value.hpp>`
* `<boost/core/span.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/make_void.hpp>`

------
### Standard Facilities
