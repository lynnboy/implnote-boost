# Boost.Functional/Hash

* lib: `boost/libs/functional/hash`
* repo: `boostorg/functional`
* commit: `45eeb170`, 2016-02-28

------
### Hash

Header `<boost/functional/hash/hash_fwd.hpp>`, `<boost/functional/hash/hash.hpp>`, and `<boost/functional/hash/extensions.hpp>`

```c++
size_t hash_value(type);
struct hash<T>; // defined to call `hash_value` by extension
struct hash<type>; //specializations

void hash_combine<T>(size_t& seed, T const& v);
size_t hash_range<It>(It, It);  void hash_range(size_t&, It, It);
```

* Calculating hash values based on 'MurMurHash3'
* `hash_value` overloaded for all basic types, and `basic_string`, `pair`, `tuple`, `type_index`,
  and STL containers, and smart pointers
* `hash` is specialized for all supported types

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`
* `<boost/cstdint.hpp>`
* `<boost/config/no_tr1/cmath.hpp>`

#### Boost.Core

* `<boost/utility/enable_if.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_enum.hpp>`, `<boost/type_traits/is_integral.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Detail

* `<boost/detail/container_fwd.hpp>`

#### Boost.Integer

* `<boost/integer/static_log2.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/repetition/repeat_from_to.hpp>`, `<boost/preprocessor/repetition/enum_params.hpp>`

------
### Standard Facilities

* Standard Library: `hash` in `<functional>` (C++11)
* Proposal
  * P0029R0 - A Unified Proposal for Composable Hashing
