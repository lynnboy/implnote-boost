# Boost.Hash2

* lib: `boost/libs/hash2`
* repo: `boostorg/hash2`
* commit: `5cf695e`, 2025-07-04

------
### Hash Algorithms

#### FNV-1a

* Header `<boost/hash2/fnv1a.hpp>`

```c++
struct detail::fnv1a_const<uint32_t> { static constexpr uint32_t basis=0x811C9DC5ul, prime=0x01000193ul; };
struct detail::fnv1a_const<uint64_t> { static constexpr uint64_t basis=0xCBF29CE484222325ull, prime=0x00000100000001B3ull; };
class detail::fnv1a<T> { T st_ = fnv1a_const<T>::basis;
public: using result_type = T;
    ctor() =default;
    ctor(uint64_t seed) { if (seed) { byte tmp[8]={}; write64le(tmp,seed); update(tmp,8); } }
    ctor(byte const* p, size_t n) { if (n) { update(p,n); byte tmp[4]={}; write32le(tmp,(uint32_t)n); update(tmp,4); }}
    ctor(void const* p, size_t n) : self((byte const*)p, n) {}
    void update(byte const* p, size_t n) { T h = st_; for (size_t i = 0; i < n; ++i) { h ^= (T)p[i]; h*= fnv1a_const<T>::prime; } st_ = h; }
    void update(void const* pv, size_t n) { return update((byte const*)p, n); }
    T result() { T r = st_; st_ = (st_^0xFF)*fnv1a_const<T>::prime; return r;}
};
using fnv1a_32 = fnv1a<uint32_t>;
using fnv1a_64 = fnv1a<uint64_t>;
```

------
#### XXH32 / XXH64

* Header `<boost/hash2/xxhash.hpp>`

```c++
class xxhash_32 {
    static constexpr uint32_t P1=2654435761U, P2=2246822519U, P3=3266489917U, P4=668265263U, P5=374761393U;
    uint32_t v1_ = P1+P2, v2 = P2, v3_ = 0, v4_ = 0U - P1;
    byte buffer_[16] = {}; size_t m_ = 0, n_ = 0;
    void init(uint32_t seed) { v1_ = seed+P1+P2; v2_ = seed+P2; v3_ = seed; v4_ = seed-P1; }
    static uint32_t round(uint32_t seed, uint32_t input) { seed+=input*P2; seed=rotl(seed,13); seed*=P1; return seed; }
    void update_(byte const* p, size_t k) { uint32_t v1=v1_, v2=v2_, v3=v3_, v4=v4_;
        for (size_t i = 0; i < k; ++i, p += 16)
        { v1=round(v1, read32le(p+0)); v2=round(v2, read32le(p+4)); v3=round(v3, read32le(p+8)); v4=round(v4, read32le(p+12)); }
        v1_=v1, v2_=v2, v3_=v3, v4_=v4;
    }
public: using result_type = uint32_t;
    ctor()=default;
    explicit ctor(uint64_t seed) {
        uint32_t s0 = uint32_t(seed), s1 = (uint32_t)(seed >> 32);
        init(s0);
        if (s1) { v1_=round(v1_,s1); v2_=round(v2_,s1); v3_=round(v3_,s1); v4_=round(v4_,s1); }
    }
    ctor(byte const* p, size_t n) { if (n) { update(p,n); result(); } }
    ctor(void const* p, size_t n) :self((byte const*)p,n) {}
    void update(byte const* p, size_t n);
    void update(void const* p, size_t n) { update((byte const*)p, n); }
    uint32_t result();
};
class xxhash_64 {
    static constexpr uint64_t P1=11400714785074694791ULL, P2=14029467366897019727ULL, P3=1609587929392839161ULL, P4=9650029242287828579ULL, P5=2870177450012600261ULL;
    uint64_t v1_ = P1+P2, v2 = P2, v3_ = 0, v4_ = 0UL - P1;
    byte buffer_[16] = {}; size_t m_ = 0, n_ = 0;
    void init(uint64_t seed) { v1_ = seed+P1+P2; v2_ = seed+P2; v3_ = seed; v4_ = seed-P1; }
    static uint64_t round(uint64_t seed, uint64_t input) { seed+=input*P2; seed=rotl(seed,31); seed*=P1; return seed; }
    static uint64_t merge_round(uint64_t acc, uint64_t val) { val = round(0,val); acc^=val; acc=acc*P1+P4; return acc; }
    void update_(byte const* p, size_t k) { uint64_t v1=v1_, v2=v2_, v3=v3_, v4=v4_;
        for (size_t i = 0; i < k; ++i, p += 32)
        { v1=round(v1, read64le(p+0)); v2=round(v2, read64le(p+8)); v3=round(v3, read64le(p+16)); v4=round(v4, read64le(p+24)); }
        v1_=v1, v2_=v2, v3_=v3, v4_=v4;
    }
public: using result_type = uint64_t;
    ctor()=default;
    explicit ctor(uint64_t seed) { init(seed); }
    ctor(byte const* p, size_t n) { if (n) { update(p,n); result(); } }
    ctor(void const* p, size_t n) :self((byte const*)p,n) {}
    void update(byte const* p, size_t n);
    void update(void const* p, size_t n) { update((byte const*)p, n); }
    uint64_t result();
};
```

------
#### XXH3-128

* Header `<boost/hash2/xxh3.hpp>`

```c++
struct xh3_128_constants { constexpr static byte const default_secret[192]={...}; };
class xxh3_128 {
    static constexpr size_t default_secret_len=192, min_secret_len=136, buffer_size=256;
    static constexpr uint64_t P32_1=.., P32_2=.., P32_3=.., P64_1=.., P64_2=.., P64_3=.., P64_4=.., P64_5=.., PRIME_MX1=.., PRIME_MX2=..;
    byte secret_[default_secret_len]={}, buffer_[buffer_size]={};
    uint64_t seed_=0; bool with_secret_=false;
    uint64_t acc_[8] = {P32_3,P64_1,P64_2,P64_3,P64_4,P32_2,P64_5,P32_1};
    size_t n_=0, m_=0, secret_len_=default_secret_len, num_stripes_=0;

    ctor(uint64_t seed, byte const* p, size_t n, bool with_secret);
    void init_secret_from_seed(uint64_t seed) {
        for (size_t i=0; i < default_secret_len/16; ++i) auto p = secret+16*i;
        { auto low = read64le(p)+seed, high = read64le(p+8) - seed; write64le(p,low); write64le(p+8, high); }
    }
    static uint64_t avalanche(uint64_t x) { x^=x>>37; x*=PRIME_MX1; x^=x>>32; return x; }
    static uint64_t avalanche_xxh64(uint64_t x) { x^=x>>33; x*=P64_2; x^=x>>29; x*=P64_3; x^=x>>32; return x; }
    uint64_t mix_step(byte const* data, size_t secret_offset, uint64_t seed) {
        auto secret = with_secret_? secret_ : default_secret;
        uint64_t data_words[2]={}, secret_words[2]={};
        for (int i = 0; i < 2; ++i) { data_words[i] = read64le(data+8*i); secret_words[i] = read64le(secret + secret_offset + 8*i); }
        uint128 r = mul128(data_words[0]^(secret_words[0]+seed), data_words[1]^(secret_words[1]-seed));
        return r.low^r.high;
    }
    void mix_two_chunks(byte const* x, byte const* y, size_t secret_offset, uint64_t seed, uint64_t (&acc)[2]) {
        uint64_t data_words1[2]={}, data_words2[2]={};
        for (int i = 0; i < 2; ++i) { data_words1[i] = read64le(x+8*i); data_words2[i] = read64le(y+8*i); }
        acc[0]+=mix_step(x, secret_offset, seed); acc[1]+=mix_step(y, secret_offset+16, seed);
        acc[0]^=(data_words2[0]+data_words2[1]); acc[1]^=(data_words1[0]+data_words1[1]);
    }
    void accumulate(uint64_t stripe[8], size_t secret_offset) { uint64_t secret_words[8] = {};
        for (int i=0; i<8; ++i) secret_words[i] = read64le(secret_+secret_offset + 8*i);
        for (int i=0; i<8; ++i) { uint64_t value = stripe[i]^secret_words[i];
            acc_[i^1]=acc_[i^1]+stripe[i]; acc_[i]=acc_[i]+(value&0xffffffff)*(value>>32); }
    }
    void scramble() { uint64_t secret_words[8] = {};
        for (int i=0; i<8; ++i) secret_words[i] = read64le(secret_ + secret_len_-64 + 8*i);
        for (int i=0; i<8; ++i) { acc_[i]^=acc_[i]>>47; acc_[i]^=secret_words[i]; acc_[i]*=P32_1; }
    }
    void last_round() { byte last_stripe[64]={}; byte* last_strpie_ptr=nullptr;
        if (m_>=64) { size_t num_stripes = (m_==0?0:(m_-1)/64);
            for (size_t n=0; n<num_stripes; ++n) { uint64_t stripe[8]={};
                for (int i=0; i<8; ++i) stripe[i] = read64le(buffer_ +(64*n)+(8*i));
                accumulate(stripe, 8*num_stripes_++);
            } last_strpie_ptr = buffer_ + m_ - 64;
        } else { size_t len = 64 - m_;
            memcpy(last_stripe, buffer_+buffer_size-len, len); memcpy(last_stripe+len, buffer_, m_);
            last_strpie_ptr = last_stripe;
        }
        uint64_t stripe[8]={};
        for (int i=0; i<8; ++i) stripe[i] = read64le(last_strpie_ptr + 8*i);
        accumulate(stripe, secret_len_-71);
    }
    uint64_t final_merge(uint64_t init_value, size_t secret_offset) { uint64_t secret_words[8] = {};
        for (int i=0; i<8; ++i) secret_words[i] = read64le(secret_ + secret_offset + 8*i);
        uint64_t result = init_value;
        for (int i=0; i<4; ++i) { auto mul_result = mul128(acc_[2*i]^secret_words[2*i], acc_[2*i+1]^secret_words[2*i+1]);
            result += mul_result.low ^ mul_result.high; }
        return avalanche(result);
    }
    digest<16> xxh3_128_digest_empty();
    digest<16> xxh3_128_digest_1to3();
    digest<16> xxh3_128_digest_4to8();
    digest<16> xxh3_128_digest_9to16();
    digest<16> xxh3_128_digest_17to128();
    digest<16> xxh3_128_digest_129to240();
    digest<16> xxh3_128_digest_long();
    uint64_t combine(uint64_t v1, uint64_t v2) { return avalanche(v1+v2); }
public: using result_type = digest<16>;
    ctor() { memcpy(secret_, default_secret, default_secret_len); }
    explicit ctor(uint64_t seed) : seed_{seed} { init_secret_from_seed(seed); }
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n) :self((byte const*)p,n) {}

    static self with_seed(uint64_t seed) { return {seed}; }
    static self with_secret(byte const* p, size_t n) { return {0, p, n, true}; }
    static self with_secret(void const* p, size_t n) { return {(byte const*)p, n}; }
    static self with_secret_and_seed(byte const* p, size_t n uint64_t seed) { return {seed, p, n, false}; }
    static self with_secret_and_seed(void const* p, size_t n uint64_t seed) { return with_secret_and_seed((byte const*)p, n, seed); }
    void update(byte const* p, size_t n);
    void update(void const* p, size_t n) { update((byte const*)p, n); }
    result_type result();
};
```

------
#### SipHash and HalfSipHash

* Header `<boost/hash2/siphash.hpp>`

```c++
class siphash_64 {
    uint64_t v0=..., v1=..., v2=..., v3=...;
    byte buffer[8] ={}; size_t m_=0; uint64_t n_=0;

    void sipround();
    void update_(byte const* p);
public: using result_type = uint64_t;
    ctor()=default;
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);

    void update(byte const* p, size_t n);
    void update(void const* p, size_t n);
    result_type result();
};

class siphash_32 {
    uint32_t v0=..., v1=..., v2=..., v3=...;
    byte buffer[4] ={}; size_t m_=0; uint32_t n_=0;

    void sipround();
    void update_(byte const* p);
public: using result_type = uint32_t;
    ctor()=default;
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);

    void update(byte const* p, size_t n);
    void update(void const* p, size_t n);
    result_type result();
};
```

------
#### HMAC

* Header `<boost/hash2/hmac.hpp>`

```c++
class hmac<H> {
    H outer_, inner_;
    void init(byte const* p, size_t n) { byte key[block_size] = {};
        if (n==0); else if (n<=block_size) memcpy(key,p,n); else
        { H h; h.update(p,n); result_type r = h.result(); memcpy(key, &r[0], m<r.size()?block_size:r.size()); }
        for (size_t i = 0; i < block_size; ++i) key[i] = (byte)(key[i]^0x36);
        inner_.update(key,block_size);
        for (size_t i = 0; i < block_size; ++i) key[i] = (byte)(key[i]^0x36^0x5C);
        inner_.update(key,block_size);
    }
public: using result_type = H::result_type;
    static constexpr size_t block_size = H::block_size;

    ctor() {init(0,0);}
    explicit ctor(uint64_t seed) { if(seed==0) init(0,0); else { byte tmp[8]={}; write64le(tmp,seed); init(tmp,8); } }
    ctor(byte const* p, size_t n) {init(p,n);}
    ctor(void const* p, size_t n) {init((byte const*)p,n);}

    void update(byte const* p, size_t n) {inner_.update(p,n);}
    void update(void const* p, size_t n) {inner_.update(p,n);}
    result_type result(){ result r = inner_.result(); outer_.update(&r[0], r.size()); return outer_.result(); }
};
```

------
#### MD5

* Header `<boost/hash2/md5.hpp>`

```c++
class md5_128 {
    uint32_t state_[4] =...;
    static constexpr int N = 64; byte buffer_[N] = {};
    size_t m_ = 0; uint64_t n_ = 0;
    static uint32_t {F|G|H|I}(uint32_t x, uint32_t y, uint32_t z);
    static void {FF|GG|HH|II}(uint32_t& a, uint32_t b, uint32_t c, uint32_t d, uint32_t x, int s, uint32_t ac);
    static constexpr int S{1..4}{1..4} =...;
    void transform(byte const block[N]);
public: using result_type = digest<16>;
    static constexpr size_t block_size = N;

    ctor() =default;
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);

    void update(byte const* p, size_t n);
    void update(void const* p, size_t n);
    result_type result();
};
using hmac_md5_128 = hmac<md5_128>;
```

------
#### SHA-1

* Header `<boost/hash2/sha1.hpp>`

```c++
class sha1_160 {
    uint32_t state_[5] =...;
    static constexpr int N = 64; byte buffer_[N] = {};
    size_t m_ = 0; uint64_t n_ = 0;
    static void R1(uint32_t a, uint32_t& b, uint32_t c, uint32_t d, uint32_t& e, uint32_t w[], byte const block[N], int i);
    static uint32_t W(uint32_t w[], int i);
    static void R{2..5}(uint32_t a, uint32_t& b, uint32_t c, uint32_t d, uint32_t& e, uint32_t w[], int i);
    void transform(byte const block[N]);
public: using result_type = digest<20>;
    static constexpr size_t block_size = N;

    ctor() =default;
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);

    void update(byte const* p, size_t n);
    void update(void const* p, size_t n);
    result_type result();
};
using hmac_sha1_160 = hmac<sha1_160>;
```

------
#### SHA-2 Family

* Header `<boost/hash2/sha2.hpp>`

```c++
struct detail::sha2_base<Word,Algo,m> {
    Word state_[8] =...;
    static constexpr int N = m; byte buffer_[N] = {};
    size_t m_ = 0; uint64_t n_ = 0;

    ctor()=default;
    void update(byte const* p, size_t n);
    void update(void const* p, size_t n);
    result_type result();
};
struct detail::sha2_256_constants<_=void>{ constexpr static uint32_t constK[64]={...}; };
struct detail::sha2_512_constants<_=void>{ constexpr static uint64_t constK[80]={...}; };

struct detail::sha2_256_base : sha2_base<uint32_t, self, 64> {
    static uint32_t {Sigma|sigma}{0|1}(uint32_t x) noexcept;
    static uint32_t {Ch|Maj}(uint32_t x, uint32_t y, uint32_t z) noexcept;
    static uint32_t W(uint32_t w[], int t);
    static void R{1|2}(uint32_t a, uint32_t b, uint32_t c, uint32_t& d,
                        uint32_t e, uint32_t f, uint32_t g, uint32_t& h,
                        byte const block[64], uint32_t const* K, uint32_t w[], int t);
    static void transform(byte const block[N], uint32_t state[8]);
};
struct detail::sha2_512_base : sha2_base<uint64_t, self, 128> {
    static uint64_t {Sigma|sigma}{0|1}(uint64_t x) noexcept;
    static uint64_t {Ch|Maj}(uint64_t x, uint64_t y, uint64_t z) noexcept;
    static uint64_t W(uint64_t w[], int t);
    static void R{1|2}(uint64_t a, uint64_t b, uint64_t c, uint64_t& d,
                        uint64_t e, uint64_t f, uint64_t g, uint64_t& h,
                        byte const block[64], uint64_t const* K, uint64_t w[], int t);
    static void transform(byte const block[N], uint64_t state[8]);
};

class sha2_{256|224} : sha2_256_base {
    void init();
public: using result_type = digest<{32|28}>;
    static constexpr size_t block_size = 64;

    ctor();
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);
    result_type result();
};

class sha2_{512|384} : sha2_512_base {
    void init();
public: using result_type = digest<{64|48}>;
    static constexpr size_t block_size = 128;

    ctor();
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);
    result_type result();
};

class sha2_512_{224|256} : sha2_512_base {
    void init();
public: using result_type = digest<28|32>;
    static constexpr size_t block_size = 128;

    ctor();
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);
    result_type result();
};

using hmac_sha2_256 = hmac<sha2_256>;
using hmac_sha2_224 = hmac<sha2_224>;
using hmac_sha2_512 = hmac<sha2_512>;
using hmac_sha2_384 = hmac<sha2_384>;
using hmac_sha2_512_224 = hmac<sha2_512_224>;
using hmac_sha2_512_256 = hmac<sha2_512_256>;
```

------
#### SHA-3 Family

* Header `<boost/hash2/sha3.hpp>`

```c++
class detail::keccak_base<paddingDelim, c, d> { static constexpr int R = 1600-c;
    byte state_[200]={}; size_t m_=0; bool finalized_=false;
public: using result_type = digest<d/8>;
    static constexpr size_t block_size = R/8;
    void update(byte const* p, size_t n);
    void update(void const* p, size_t n);
    result_type result();
};

struct sha3_{224|256|384|512} : keccak_base<0x06, 2*{224|256|384|512}, {224|256|384|512}> {
    ctor();
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);
};

struct shake_{128|256} : keccak_base<0x1f, 2*{128|256}, 1600-2*{128|256}> {
    ctor();
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);
};

using hmac_sha3_224 = hmac<sha3_224>;
using hmac_sha3_256 = hmac<sha3_256>;
using hmac_sha3_384 = hmac<sha3_384>;
using hmac_sha3_512 = hmac<sha3_512>;
```

------
#### RIPEMD Family

* Header `<boost/hash2/ripemd.hpp>`

```c++
class ripemd_128 {
    uint32_t state_[4] =...;
    static constexpr int N = 64; byte buffer_[N] = {};
    size_t m_ = 0; uint64_t n_ = 0;
    static uint32_t F{1..4}(uint32_t x, uint32_t y, uint32_t z);
    static void R{1..4}(uint32_t& a, uint32_t b, uint32_t& c, uint32_t d, uint32_t x, uint32_t s);
    static void RR{1..4}(uint32_t& a, uint32_t b, uint32_t& c, uint32_t d, uint32_t x, uint32_t s);
    void transform(byte const block[N]);
public: using result_type = digest<16>;
    static constexpr size_t block_size = N;

    ctor() =default;
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);

    void update(byte const* p, size_t n);
    void update(void const* p, size_t n);
    result_type result();
};

class ripemd_160 {
    uint32_t state_[5] =...;
    static constexpr int N = 64; byte buffer_[N] = {};
    size_t m_ = 0; uint64_t n_ = 0;
    static uint32_t F{1..5}(uint32_t x, uint32_t y, uint32_t z);
    static void R{1..5}(uint32_t& a, uint32_t b, uint32_t& c, uint32_t d, uint32_t e, uint32_t x, uint32_t s);
    static void RR{1..5}(uint32_t& a, uint32_t b, uint32_t& c, uint32_t d, uint32_t e, uint32_t x, uint32_t s);
    void transform(byte const block[N]);
public: using result_type = digest<20>;
    static constexpr size_t block_size = N;

    ctor() =default;
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);

    void update(byte const* p, size_t n);
    void update(void const* p, size_t n);
    result_type result();
};

using hmac_ripemd_160 = hmac<ripemd_160>;
using hmac_ripemd_128 = hmac<ripemd_128>;
```

------
#### BLAKE2 Faimly

* Header `<boost/hash2/blake2.hpp>`

```c++
struct detail::blake2b_constants<_=void> { constexpr static uint64_t iv[8]={...}; };
struct detail::blake2s_constants<_=void> { constexpr static uint32_t iv[8]={...}; };

class blake2b_512 {
    byte b_[128]={}; uint64_t h_[8]={}, t_[2]={}; size_t m_=0;
    void init(uint64_t keylen=0);
    static void G(uint64_t v[16], int a, int b, int c, int d, uint64_t x, uint64_t y);
    void transform(byte const block[128], bool is_final=false);
    void incr_len(size_t n);
public: using result_type = digest<64>;
    static constexpr size_t block_size = 128;

    ctor() =default;
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);

    void update(byte const* p, size_t n);
    void update(void const* p, size_t n);
    result_type result();
};

class blake2s_256 {
    byte b_[64]={}; uint32_t h_[8]={}, t_[2]={}; size_t m_=0;
    void init(uint32_t keylen=0);
    static void G(uint32_t v[16], int a, int b, int c, int d, uint32_t x, uint32_t y);
    void transform(byte const block[64], bool is_final=false);
    void incr_len(size_t n);
public: using result_type = digest<32>;
    static constexpr size_t block_size = 64;

    ctor() =default;
    explicit ctor(uint64_t seed);
    ctor(byte const* p, size_t n);
    ctor(void const* p, size_t n);

    void update(byte const* p, size_t n);
    void update(void const* p, size_t n);
    result_type result();
};

using hmac_blake2b_512 = hmac<blake2b_512>;
using hmac_blake2s_256 = hmac<blake2s_256>;
```

------
### Common Bits

```c++
enum class endian { little, big, native };
struct default_flavor { using size_type = uint32_t; static constexpr endian byte_order = native; };
struct little_endian_flavor { using size_type = uint32_t; static constexpr endian byte_order = little; };
struct big_endian_flavor { using size_type = uint32_t; static constexpr endian byte_order = big; };

constexpr uint{32|64}_t detail::read{32|64}{le|be}(byte* p) noexcept;
constexpr void detail::write{16|32|64}{le|be}(byte* p, uint{16|32|64}_t v) noexcept;
constexpr void detail::write <T> (T v, endian e, byte(&w)[sizeof(T)]) noexcept requires std::is_integral_v<T>;

uint{32|64}_t detail::rot{l|r}(uint{32|64}_t v, int k) noexcept;
uint{32|64}_t detail::byteswap(uint{32|64}_t x) noexcept;

To detail::bit_cast<To,From>(From const& from) noexcept;

struct detail::uint128 { uint64_t low, high; };
uint128 detail::mul128(uint64_t x, uint64_t y) noexcept;

uint64_t detail::read_lane(byte const (&state)[200], int x, int y);
void detail::write_lane(byte const (&state)[200], int x, int y, uint64_t v);
void detail::xor_lane(byte const (&state)[200], int x, int y, uint64_t v);
void detail::rho_and_pi_round(byte const (&state)[200], uint64_t& lane, int rot, int x, int y);
void detail::keccak_round(byte (&state)[200]);
struct detail::iota_rc_holder<_=void> { static constexpr uint64_t data[24] = {...}; };
void detail::keccak_permute(byte (&state)[200]);

class digest<n> { byte data_[n] = {};
public: ctor()=default;
    ctor(byte const(&v)[n]) noexcept { memcpy(data_,v,n); }
    using value_type = byte; using size_type = size_t; using difference_type = ptrdiff_t;
    using <const>_reference = byte<const>&; using <const>_iterator = byte<const>*;

    <const>_iterator begin() <const> noexcept { return data_; }
    <const>_iterator end() <const> noexcept { return data_+n; }
    byte<const>* data() <const> noexcept { return data; }
    size_type {size|max_size}() const noexcept { return n; }
    <const>_reference operator[](size_t i) <const> { return data_[i]; }
    <const>_reference front() <const> noexcept { return data_[0]; }
    <const>_reference back() <const> noexcept { return data_[n-1]; }

    friend bool operator==(self const& a, self const& b) noexcept { return memcmp(a.data(),b.data(),n) == 0; } // also !=
    friend char* to_chars(self const& v, char* first, char* last) noexcept;
    friend char* to_chars<m>(self const& v, char(&w)[m]) noexcept;
    friend std::ostream& operator<<(std::ostream& os, self const& v);
    friend std::string to_string(self const& v);
};

uint32_t detail::get_result_multiplier<R>() requires sizeof(R) <= 4 { return 0xBF3D6763u; }
uint64_t detail::get_result_multiplier<R>() requires sizeof(R) == 8 { return 0x99EBE72FE70129CBull; }
T get_integral_result<T,Hash,R=Hash::result_type>(Hash& h) {
    if constexpr (std::is_integral_v<R>) { using U = std::make_unsigned_t<T>; using lr = numeric_limits<R>; using lu = numeric_limits<U>;
        if constexpr (sizeof(R) > sizeof(T)) return (T)(U)( (h.result() * get_result_multiplier<R>()) >> (lr::digits - lu::digits) );
        else if cnstexpr (sizeof(R) == sizeof(T)) return (T)(U) h.result();
        else { U u=0; for (int i=0; i<lu::digits; i += lr::digits) u += (U)h.result() << i; return (T)u; }
    } else return (T) read64le(h.result().data());
}

struct is_trivially_equality_comparable<T> : std::bool_constant<is_integral_v<T> || is_enum_v<T> || is_pointer_v<T>>{};
struct is_trivially_equality_comparable<T const> : is_trivially_equality_comparable<T>{};

struct is_endian_independent<T> : std::bool_constant<sizeof(T)==1> {};
struct is_endian_independent<T const> : is_endian_independent<T> {};

struct is_contiguously_hashable<T,e> : std::bool_constant<is_trivially_equality_comparable<T>::value && (e==native||is_endian_independent<T>::value)>{};
struct is_contiguously_hashable<T[n],e> : struct is_contiguously_hashable<T,e>{};

struct detail::has_tuple_size<T>; // std::tuple_size<T>::value is valid
struct has_constant_size<T> : has_tuple_size<T> {};
struct has_constant_size<array<T,n>> : std::true_type {};
struct has_constant_size<digest<n>> : std::true_type {};
struct has_constant_size<T const> : has_constant_size<T> {};

struct hash_append_tag{};
struct detail::provider_archetype {
    static void hash_append<H,F,T>(H& h, F const& f, T const& v);
    static void hash_append_range<H,F,It>(H& h, F const& f, It b, It e);
    static void hash_append_size<H,F,T>(H& h, F const& f, T const& v);
    static void hash_append_range_and_size<H,F,It>(H& h, F const& f, It b, It e);
    static void hash_append_unordered_range<H,F,It>(H& h, F const& f, It b, It e);
};
struct detail::hash_archetype {
    using result_type = uint64_t;
    ctor(); explicit ctor(uint64_t); ctor(void const*, size_t);
    void update(void const*; size_t);
    result_type result();
};
struct detail::flavor_archetype { using size_type = uint32_t; static constexpr endian byte_order = native; };
struct detail::has_tag_invoke<T> = requires(hash_append_tag tag, provider_archetype& p, hash_archetype& h, flavor_archetype const& f, T const* t) {tag_invoke(h,p,h,f,t); };
```

------
### Object Hashing

* Header `<boost/hash2/hash_append.hpp>`, `<boost/hash2/hash_append_fwd.hpp>`

```c++
void hash_append <H,F=default_flavor,T> (H& h, F const& f, T const& v);
void hash_append_range <H,F=default_flavor,It> (H& h, F const& f, It b, It e);
void hash_append_size <H,F=default_flavor,T> (H& h, F const& f, T const& v);
void hash_append_range_and_size <H,F=default_flavor,It> (H& h, F const& f, It b, It e);
void hash_append_unordered_range <H,F=default_flavor,It> (H& h, F const& f, It b, It e);

struct hash_append_provider {
    void hash_append <H,F,T> (H& h, F const& f, T const& v) {hash_append(h,f,v);}
    void hash_append_range <H,F,It> (H& h, F const& f, It b, It e) {hash_append_range(h,f,b,e);}
    void hash_append_size <H,F,T> (H& h, F const& f, T const& v) {hash_append_size(h,f,v);}
    void hash_append_range_and_size <H,F,It> (H& h, F const& f, It b, It e) {hash_append_range_and_size(h,f,b,e);}
    void hash_append_unordered_range <H,F,It> (H& h, F const& f, It b, It e) {hash_append_unordered_range(h,f,b,e);}
};
void detail::do_hash_append<H,F,T>(H& h, F const& f, T const& v){
    if constexpr (std::is_integral_v<T>) { byte tmp[sizeof(T)]={}; write(v,F::byte_order, tmp); h.update(tmp,sizeof(T)); }
    else if constexpr (std::is_enum_v<T>) hash_append(h, f, (std::underlying_type_t<T>)v);
    else if constexpr (std::is_pointer_v<T>) hash_append(h, f, (uintptr_t)v);
    else if constexpr (std::is_floating_point_v<T> && sizeof(T) == 4) hash_append(h, f, bit_cast<uint32_t>(v+0));
    else if constexpr (std::is_floating_point_v<T> && sizeof(T) == 8) hash_append(h, f, bit_cast<uint64_t>(v+0));
    else if constexpr (std::is_same_v<T,nullptr_t>) hash_append(h, f, (void*)v);
    else if constexpr (std::is_bounded_array<T>) { hash_append_range(h,f,v+0,v+std::extend_v<T>); }
    else if constexpr (container_hash::is_contiguous_range<T>::value) {
        if constexpr (!has_constant_size<T>::value) { hash_append_range(h,f,v.data(),v.data()+v.size()); hash_append_size(h,f,v.size()); }
        else { if (v.size()==0) hash_append(h,f,'\x00'); else hash_append_range(h,f,&v.front(),&v.front()+v.size()); }
    } else if constexpr (container_hash::is_unordered_range<T>::value) hash_append_unordered_range(h,f,v.begin(),v.end());
    else if constexpr(container_hash::is_range<T>::value) {
        if constexpr (!has_constant_size<T>::value) { hash_append_range_and_size(h,f,v.begin(),v.end()); }
        else { if (v.begin()==v.end()) hash_append(h,f,'\x00'); else hash_append_range(h, f, v.begin(), v.end()); }
    } else if constexpr (has_tag_invoke<T>::value) tag_invoke(hash_append_tag{}, hash_append_provider{}, h, f, &v);
}
void detail::do_hash_append<H,F,T,n>(H& h, F const& f, T const(&v)[n]) { hash_append_range(h,f,v+0,v+n); }
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/workaround.hpp>`

#### Boost.ContainerHash

* `<boost/container_hash/**.hpp>`

#### Boost.Describe

* `<boost/describe/**.hpp>`

#### Boost.MP11

* `<boost/mp11/**.hpp>`

------
### Standard Facilities
