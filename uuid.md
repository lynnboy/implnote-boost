# Boost.UUID

* lib: `boost/libs/uuid`
* repo: `boostorg/uuid`
* commit: `59a01cd4`, 2017-01-11

------
### UUID library

#### Class `uuid`

##### Header

* `<boost/uuid.hpp>`

```c++
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

  bool is_nil() const noexcept;
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

  void swap(uuid&) noexcept;
};
bool operator== (uuid const&, uuid const&) noexcept; // and !=, <, >, <=, >=
void swap(uuid&, uuid&) noexcept;
size_t hash_value(uuid const&) noexcept;
// Boost.Serialization support
struct ::boost::serialization::implementation_level_impl<const uuid> : integral_constant<int, 1>;
// std hash support
struct std::hash<uuid> { size_t operator()( uuid const&) const noexcept; }; // hash_value
```

Provides SSE2/SSE3/SSE4.1 optimized version for compare and swap operations.

------
#### Generators(Factories) for `uuid`

##### Header

* `<boost/uuid/uuid_generators.hpp>`
* `<boost/uuid/xxx_generator.hpp>`

##### `nil_generator`

Just creates NIL uuid values.

* `class nil_generator`
* `nil_uuid() -> uuid`

##### `string_generator`

* Accepts `std::basic_string`s, C-style char/wchar_t strings, and iterator pair ranges.
* Allowed string can optionally have `{}` braces, and optionally have `-` dashes

##### `name_generator`

* Constructor `explicit name_generator(uuid const& namespace_uuid)`.
* Accepts `std::basic_string`s, C-style char/wchar_t strings, and byte buffers.
* Calculate sha1 for initial namespace uuid and provided content as result `uuid` value.
* Resulting variant and version is fixed as `rfc_4122` and `name_based_sha1`.

##### `random_generator`

* `class basic_random_generator<URNG>`
* `typedef basic_random_generator<mt19937> random_generator`
* Constructor accepts an lvalue or pointer of `URNG` instance
* Default constructor creates a new `URNG` instance, and seeds it with system entropy.
* Resulting variant and version is fixed as `rfc_4122` and `random_number_based`.

------
#### UUID I/O & Serialization

##### Header

* `<boost/uuid/uuid_io.hpp>`
* `<boost/uuid/uuid_serialize.hpp>`

##### Operations

* `<<` and `>>` - no braces, but have dashes.
* `to_string(uuid const&) -> std::string` and `to_wstring(uuid const&) -> string::wstring`
* On serialization, `uuid` is treated as primitive value type.

------
#### Valuable Utilities

##### SHA1 Hash Calculator

`<boost/uuid/sha1.hpp>`

* Default constructor
* `reset()`
* `process_byte(unsigned char)`, `process_block(void const*, void const*)`, and
  `process_bytes(void const*, std::size_t)`
* `get_digest(unsigned int (&)[5])`

##### Random Seed Generator

`<boost/uuid/seed_rng.hpp>`

* class `seed_rng`, similar to `random_device` of Random library.
  * Default constructor.
  * `operator()` generate result
  * `min` and `max`
  * Internally calculate SHA1 value from platform-specific entropy sources.
* `seed<URNG>(URNG& rng)`, seed the `rng` with `seed_rng` generated values.

------
#### Implementation Details

```c++
namespace uuids::detail;

// byte order
uint{16|32|64}_t byteswap(uint{16|32|64}_t) noexcept;
uint{16|32|64}_t load_{native|little|big}_u{16|32|64}(void const* p) noexcept;
void store_{native|little|big}_u{16|32|64}(void const* p, uint{16|32|64}_t) noexcept;
// also support for __uint128_t if available

uint64_t hash_mix_mx(uint64_t x) noexcept { x *= 0xD96AAA55; x ^= x>>16; }
uint64_t hash_mix_fmx(uint64_t x) noexcept { x *= 0x7DF954AB; x ^= x>>16; }

T numeric_cast<T,U>(U); // throw if out-of-range

Ch* to_chars<Ch> (uuid const& u, Ch* out) noexcept; // "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
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

