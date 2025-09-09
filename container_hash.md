# Boost.ContainerHash

* lib: `boost/libs/container_hash`
* repo: `boostorg/container_hash`
* commit: `060d4ae`, 2025-08-22

------
### Hash

Header `<boost/container_hash/hash_fwd.hpp>`, `<boost/container_hash/hash.hpp>`

```c++
size_t hash_value(type); // impl
namespace hash_detail {
  uint32_t hash_mix(uint32_t);
  uint64_t hash_mix(uint64_t);
}

namespace container_hash {
  struct is_range<T>;
  struct is_contiguous_range<T>;
  struct is_unordered_range<T>;
  struct is_described_class<T>; // Boost.Describe support
  struct is_tuple_like<T>;
}

struct hash<T>{ std::size_t operator()(T const& val) const { return hash_value(val); } };

void hash_combine<T>(size_t& seed, T const& v) {
  seed = hash_detail::hash_mix(seed + 0x9e3779b9 + boost::hash<T>()(v));
}
void hash_range(size_t& seed, It f, It l) { // optimized for character types
  for (; f != l; ++f) hash_combine<std::iterator_traits<It>::value_type>(seed, *f);
}
size_t hash_range(It f, It l) { size_t s{0}; hash_range(s, f, l); return s; }
void hash_unordered_range(size_t& seed, It f, It l) {
  size_t r{0}, const s2(seed);
  for (; f != l; ++f) { // each time use same seed
    size_t s3{s2}; hash_combine<std::iterator_traits<It>::value_type>(s3, *f); r += s3;
  }
  seed += r;
}
size_t hash_unordered_range(It f, It l) { size_t s{0}; hash_range(s, f, l); return s; }

struct hash_is_avalanching<Hash>;
```

* Calculating combined hash values based on 'MurMurHash3' (`hash_mix`).
* `hash_value` overloaded for all basic types, and `basic_string`, `pair`, `tuple`, `type_index`,
  and STL containers, and smart pointers
  - Integral types: combine each part with `hash_mix`
  - Float Numbers: copy to `uint[32|64]_t` (2 unit64_6 for 80/128 bit fp type), then combine hashvalue of each part with `hash_mix`
  - Enums: use underlying value.
  - Pointer: `p -> ptrdiff_t -> size_t`, then `p + (p >> 3)`, `nullptr` handled as `void*`
  - C-array, `std::basic_string`, `std::basic_string_view`: call `hash_range`
  - `std::complex`: `re + hash_mix(im)`
  - `std::pair`, `std::tuple`, Tuple-like: `hash_combine` (`first` and `second`'s, or each component's hash value)
  - Contiguous Range: `hash_range(r.data(), r.data() + r.size())`
  - Unordered Range: `hash_unordered_range(r.begin(), r.end())`
  - Range: `hash_range(r.begin(), r.end())`
  - `std::shared_ptr`, `std::unique_ptr`: `hash_value(p.get())`
  - `std::type_index`: call its own `hash_code`
  - `std::error_code`, `std::error_condition`: `hash_combine` `value()` and address of `category()`
  - `std::optional`: `0x12345678` for empty, value's hash value otherwise.
  - `std::variant`: `0x87654321` for `std::monostate`, hash_combine `index()` and value
  - (Boost.Describe): `hash_combine` each base and data member.
* (Boost.Unordered): Register `hash_is_avalanching` for `hash<std::basic_string<char8_t>>` and `hash<std::basic_string_view<char8_t>>`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.Describe

* `<boost/describe/bases.hpp>`, `<boost/describe/members.hpp>`

#### Boost.MP11

* `<boost/mp11/algorithm.hpp>` // mp_for_each

------
### Standard Facilities

* Standard Library: `hash` in `<functional>` (C++11)
* Proposal
  * P0029R0 - A Unified Proposal for Composable Hashing
