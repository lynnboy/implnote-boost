# Boost.UUID

* lib: `boost/libs/uuid`
* repo: `boostorg/uuid`
* commit: `23783801`, 2015-9-13

------
### UUID library

#### Class `uuid`

##### Header

* `<boost/uuid.hpp>`

##### Partial STL container interface (as fixed byte array)

* Member types `value_type`, `iterator`, `size_type`, etc.
* Methods `begin`, `end`, `size`, `swap`, `static_size`.
* Operators `==`, `<`, `!=`, `>`, `<=`, `>=`
* Non members `swap` and `hash_value`

##### UUID specific

* `bool is_nil() const noexcept`

* `variant_type variant() const noexcept`
  `enum variant_type { variant_ncs, variant_rfc_4122, variant_microsoft, variant_future }`

* `version_type version() const noexcept`
  `enum version_type { version_unknown, version_time_based, version_dce_security,
    version_name_based_md5, version_random_number_based, version_name_based_sha1 }`

##### Optimization

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
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/cstdint.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_pod.hpp>`, `<boost/type_traits/integral_constant.hpp>`, register `uuid` as POD.

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>` for conversion parsing error and SHA1 bit count overflow.

#### Boost.IO

* `<boost/io/ios_state.hpp>` for I/O operators.

#### Boost.Serialization

* `<boost/serialization/level.hpp>`, to register `uuid` as primitive type.

#### Boost.Core

* `<boost/core/noncopyable.hpp>`, `seed_rng` generator class is non-copyable.

#### Boost.Iterator

* `<boost/iterator/iterator_facade.hpp>`, internal used by `seed`.

#### Boost.WinAPI

* `<boost/detail/winapi/*.hpp>` on Windows, access system entropy resources by `seed_rng`.

#### Boost.SmartPtr

* `<boost/shared_ptr.hpp>`, used to hold URNG in `basic_random_generator`.

#### Boost.Random

* Used by `random_generator`.
* `<boost/random/uniform_int.hpp>`, `<boost/random/variate_generator.hpp>`
* `<boost/random/mersenne_twister.hpp>`, for `mt19937`

------
### Standard Facilities

