# Boost.UUID

* lib: `boost/libs/uuid`
* repo: `boostorg/uuid`
* commit: `bbbb204`, 2025-09-19

------
### UUID library

#### Class `uuid`

##### Header

* `<boost/uuid.hpp>`

```c++
// byte order
uint{16|32|64}_t detail::byteswap(uint{16|32|64}_t) noexcept;
uint{16|32|64}_t detail::load_{native|little|big}_u{16|32|64}(void const* p) noexcept;
void detail::store_{native|little|big}_u{16|32|64}(void const* p, uint{16|32|64}_t) noexcept;
// also support for __uint128_t if available

struct uuid_clock { // API for v1, v6 timestamp/clock, meets std's Clock concept, epoch is 1582/10/15
  using rep = uint64_t; using period = std::ratio<1,10'000'000>; // count by 100ns
  using duration = std::duration<rep, period>; using time_point = std::time_point<uuid_clock, duration>;
  static constexpr bool is_steady = false;
  static time_point now() noexcept; // from_sys(system_clock::now())

  static time_point from_sys<D>(std::time_point<std::system_clock,D> const&) noexcept; // + 141427 days offset
  static std::time_point<std::system_clock,duration> to_sys(time_point const&) noexcept;
  static time_point from_timestamp(uint64_t) noexcept; // factory conv
  static uint64_t to_timestamp(time_point const&) noexcept;
};

struct uuid {
  using repr_type = uint8_t[16];
  struct data_type {
    uint8_t alignas(uint64_t) repr_[16] = {};
    constexpr operator repr_type [const] & () [const] noexcept; // impl convert to uint8_t[16]
    constexpr uint8_t [const]* operator() () [const] noexcept; // () -> uint8_t*
  };
  data_type data={};
public: // container API
  using value_type = uint8_t; // [const_]reference and [const_]iterator
  using size_type = size_t; using difference_type = ptrdiff_t;
  ctor(); constexpr ctor(repr_type const&) noexcept; // copy-ctor

  [const_]iterator begin() [const] noexcept;
  [const_]iterator end() [const] noexcept;
  constexpr size_type size() const noexcept; // static_size, 16
  static constexpr size_type static_size() noexcept; // 16

  bool is_nil() const noexcept; // impl
  enum variant_type { variant_ncs, variant_rfc_4122, variant_microsoft, variant_future };
  variant_type variant() const noexcept; // based on data[8]'s high 4 bits: 0xxx, 10xx, 110x, 111x
  enum version_type { version_unknown = -1, // version_XXX
    time_based = 1, dce_security, name_based_md5, random_number_based,
    name_based_sha1, time_based_v6, time_based_v7, custom_v8 };
  version_type version() const noexcept; // based on data[6]'s high 4 bits

  using timestamp_type = uint64_t;
  timestamp_type timestamp_v1() const noexcept; // data[6:7]/12 ++ data[4:5]/8 ++ data[0:3]/8
  timestamp_type timestamp_v6() const noexcept; // data[0:3]/32 ++ data[4:5]/16 ++ data[6:7]/12
  timestamp_type timestamp_v7() const noexcept; // data[0:7] >> 16
  uuid_clock::time_point time_point_v1() const noexcept; // and v6, uuid_clock::from_timestamp
  std::chrono::time_point<system_clock,milliseconds> time_point_v7() const noexcept;

  using clock_seq_type = uint16_t;
  clock_seq_type clock_seq() const noexcept; // data[8:9]/14

  using node_type = std::array<uint8_t, 6>;
  node_type node_identifier() const noexcept; // data[10:15]

  void swap(uuid&) noexcept; // impl
};
bool operator== (uuid const&, uuid const&) noexcept; // and !=, <, >, <=, >=, <=>
void swap(uuid&, uuid&) noexcept;

uint64_t detail::hash_mix_mx(uint64_t x) noexcept { x *= 0xD96AAA55; x ^= x>>16; }
uint64_t detail::hash_mix_fmx(uint64_t x) noexcept { x *= 0x7DF954AB; x ^= x>>16; }
size_t hash_value(uuid const&) noexcept;
// std hash support
struct std::hash<uuid> { size_t operator()( uuid const&) const noexcept; }; // hash_value
```

Implementations for `is_nil`, `swap`, and comparations:
- On x86, optimized for SSE2/SSE3/SSE41/AVX
- If supports `__uint128_t`, implements based on this type.
- Generic implementation: treate data as two `uint64_t`.

------
#### Generators(Factories) for `uuid`

##### Header

* `<boost/uuid/uuid_generators.hpp>`
* `<boost/uuid/xxx_generator.hpp>`

```c++
struct nil_generator {
  using result_type = uuid;
  uuid operator()() const noexcept { return {{}}; }
};
constexpr uuid nil_uuid() noexcept { return {{}}; }

class string_generator {
  [[noreturn]] void throw_invalid(int ipos, char const* err) const; // throw runtime_error
  constexpr It::value_type get_next_char<It> (It& b It e, int& ipos) const;
  // get_value(c) for hex digit, is_dash, is_open_brace, is_close_brace, all for both char and wchar_t
public:
  using result_type = uuid;
  constexpr uuid operator() <It> (It b, It e) const;
  constexpr uuid operator() <Ch,Tr,A> (std::basic_string<Ch,Tr,A> const& s) const;
  constexpr uuid operator() (char const*) const; // and wchar_t
};

constexpr uuid ns::dns() noexcept; // '6ba7b810-9dad-11d1-80b4-00c04fd430c8'
constexpr uuid ns::url() noexcept; // '6ba7b811-9dad-11d1-80b4-00c04fd430c8'
constexpr uuid ns::oid() noexcept; // '6ba7b812-9dad-11d1-80b4-00c04fd430c8'
constexpr uuid ns::x500dn() noexcept; // '6ba7b814-9dad-11d1-80b4-00c04fd430c8'

class basic_name_generator<HashAlgo> {
  uuid namespace_uuid_;
  void process_characters(HashAlgo& h, char const* p, size_t n) const noexcept; // for all char types
  uuid hash_to_uuid(HashAlgo& h) const noexcept; // Use 16 bytes from digest, set variant, set version to h.get_version()
public:
  using result_type = uuid;
  explicit ctor(uuid const& namespace_uuid) noexcept;
  uuid operator() <Ch> (Ch const* name) const noexcept; // hash_to_uuid(has of namespace + name)
  uuid operator() <Ch,Tr,A> (std::basic_string<Ch,Tr,A> const& name) const noexcept;
  uuid operator() (void const* b, size_t c) const noexcept;
};
struct name_generator_md5 : basic_name_generator<md5> {using base::ctor;};
struct name_generator_sha1 : basic_name_generator<sha1> {using base::ctor;};

class detail::random_provider {
  random_device dev_;
public:
  using result_type = uint32_t;
  void generate<It> (It f, It l); // fill with [dev_ -> uniform_int_distribution]
  const char* name() const { return "std::random_device"; }
};
class basic_random_generator<URNG> {
  URNG* p_; URNG g_; // use g_ if p_ is not provided
public:
  using result_type = uuid;
  ctor(); // call g_.seed() if it has member `seed()`
  explicit ctor(URNG& gen); explicit ctor(URNG* gen); // both pass already seeded URNG, and set p_;
  uuid operator()(); // fill with random bytes,  set variant and version
};
struct random_generator : basic_random_generator<chacha20_12> {};
using random_generator_mt19937 = basic_random_generator<std::mt19937>;
using random_generator_pure = basic_random_generator<std::random_device>;
```

* `nil_generator` - Just creates NIL uuid values.
* `string_generator` - String can optionally have `{}` braces, and optionally have `-` dashes
* `name_generator_xxx` - Resulting variant is `rfc_4122` (0b10).
  Forcely treat `wchar_t` as little-endian `uint32_t` for portability.
  `char16_t` and `char32_t` are handled by UTF-32 codepoints.
* `random_generator` use ChaCha20, also provides `random_generator_mt19937` and `random_generator_pure`.
  Resulting variant and version is fixed as `rfc_4122` and `random_number_based`.

------
#### UUID I/O & Serialization

##### Header

* `<boost/uuid/uuid_io.hpp>`

```c++
T detail::numeric_cast<T,U>(U); // throw if out-of-range
Ch* detail::to_chars<Ch> (uuid const& u, Ch* out) noexcept; // "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"

OutIt to_chars<OutIt> (uuid const&, OutIt out); // fill
bool to_chars<Ch> (uuid const&, Ch* f, Ch* l) noexcept; // check l-f >= 36 and fill
Ch* to_chars<Ch,N> (uuid const&, Ch(*)[N]) noexcept; // requires N >= 37

std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>&, uuid const&);
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr> (std::basic_istream<Ch,Tr>&, uuid const&);

std::string to_string(uuid const&);
std::wstring to_wstring(uuid const&);

// Boost.Serialization support
struct ::boost::serialization::implementation_level_impl<const uuid> : integral_constant<int, 1>;
```

##### Operations

* `<<` and `>>` - no braces, but have dashes.
* `to_string(uuid const&) -> std::string` and `to_wstring(uuid const&) -> string::wstring`
* On serialization, `uuid` is treated as primitive value type.

------
#### Valuable Utilities

##### MD5, SHA1 Hash Calculator

```c++
class detail::md5 {
  struct MD5_CTX { uint32_t lo, hi, a, b, c, d; unsigned char buffer[64]; uint32_t block[16]; } ctx_;

public:
  using digest_type = unsigned char[16];
  ctor(); // init
  void process_byte(unsigned char); void process_bytes(void const* b, size_t n); // update
  void get_digest(digest_type& digest); // final

  unsigned char get_version() const { return uuid::version_name_based_md5; } // for uuid version
};

class detail::sha1 {
  unsigned int h_[5]; unsigned char block_[64]; size_t block_byte_index_, bit_count_low, bit_count_high;
public:
  using digest_type = unsigned char[20];
  ctor(); void reset();
  void process_byte(unsignec char); void process_block(void const* b, void const* e); void process_bytes(void const* b, size_t c);
  void get_digest(digest_type& digest);
  unsigned char get_version() const { return uuid::version_name_based_sha1; }
};
```

##### ChaCha20 Random Seed Generator

```c++
class detail::chacha20_12 {
  uint32_t state_[16], block_[16]; size_t index_;
public:
  using result_type = uint32_t;
  chacha20_12() noexcept;
  chacha20_12(uint32_t const(& key)[8], uint32_t const(& nonce)[2]) noexcept;
  void seed() noexcept; void seed<Seq>(Seq& seq);
  void perturb() noexcept;
  static constexpr result_type min() noexcept;
  static constexpr result_type max() noexcept;
  result_type operator()() noexcept;
};
```

### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>` for conversion parsing error and SHA1 bit count overflow.

#### Boost.TypeTraits

* `<boost/type_traits/integral_constant.hpp>`.

------
### Standard Facilities

