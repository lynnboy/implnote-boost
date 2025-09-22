# Boost.TypeIndex

* lib: `boost/libs/type_index`
* repo: `boostorg/type_index`
* commit: `af61d6b8`, 2017-02-28

------
### Type index solution

#### Header

* `<boost/type_index.hpp>`

#### API

```c++
#ifdef BOOST_TYPE_INDEX_USER_TYPEINDEX
#include BOOST_TYPE_INDEX_USER_TYPEINDEX
#elif BOOST_NO_RTTI || defined(BOOST_TYPE_INDEX_FORCE_NO_RTTI_COMPATIBILITY)
using type_index = ctti_type_index;
#else
using type_index = stl_type_index;
#endif
type_index type_id<T>() noexcept { return type_index::type_id<T>(); }
type_index type_id_with_cvr<T>() noexcept { return type_index::type_id_with_cvr<T>(); }
type_index type_id_runtime<T>(const T& runtime_val) noexcept { return type_index::type_id_runtime(runtime_val); }
```

* `typedef xxxxx type_index`
  * `raw_name() const noexcept -> const char*`
  * `name() const noexcept -> const char*`
  * `pretty_name() const -> std::string`
  * `equal(const type_index&) const noexcept -> bool` - *Compared by `raw_name()`*
  * `before(const type_index&) const noexcept -> bool` - *Compared by `raw_name()`*
  * `hash_code() const noexcept -> std::size_t`
* `typedef type_index::type_info_t type_info`
* `type_id<T>() noexcept -> type_index`
* `type_id_with_cvr<T>() noexcept -> type_index`, which don't remove constenss and reference.
* `type_id_runtime<T>(const T&) noexcept -> type_index`;
* Operators: `==`, `!=`, `<`, `<=`, `>`, `>=`
* I/O: `<<`, `>>`
* `hash_value(const type_index&) noexcept -> std::size_t`
* Macro `BOOST_TYPE_INDEX_REGISTER_CLASS`, used to declare a hidden virtual function
  in user's polymorphic classes returning its `type_info`, used when RTTI is unusable.

#### Configurations

* Detected **RTTI** support, to select `std::type_index` or simulated **CTTI** solution.
* Macro `BOOST_TYPE_INDEX_FORCE_NO_RTTI_COMPATIBILITY`,
  force use **CTTI** even if **RTTI** is available.
* Macro `BOOST_TYPE_INDEX_USER_TYPEINDEX`, path to a header, including user-provided
  `type_index` implementation.

#### Extensibility

Compatible `type_index` class should derive from `type_index_facade`:

```c++
template <class Derived, class TypeInfo>
class type_index_facade { // CRTP base
  const Derived& derived() const noexcept; // cast *this to Derived
public:
  using type_info_t TypeInfo;
  const char* name() const noexcept { return derived().raw_name(); }
  std::string pretty_name() const { return derived().name(); }
  bool equal(const Derived&) const noexcept; // compare raw_name()
  bool before(const Derived&) const noexcept; // compare raw_name()
  size_t hash_code() const noexcept; // hash_range on raw_name;
};
constexpr bool operator== <D,TI> (const type_index_facade<D,TI>&, const type_index_facade<D,TI>&) noexcept;
constexpr bool operator== <D,TI> (const TI&, const type_index_facade<D,TI>&) noexcept;
constexpr bool operator== <D,TI> (const type_index_facade<D,TI>&, const TI&) noexcept;
// and also !=, <, >, <=, >=
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>&, const type_index_facade<D,TI>&); // pretty_name
size_t hash_value<D,TI> (const type_index_facade<D,TI>&) noexcept; // hash_code
```

User may provide its own `type_info` class, and at least these members of `type_index`:
* `raw_name() const noexcept -> const char*`
* `type_info() const noexcept -> const type_info&`
* `static type_id<T>() noexcept -> type_index`
* `static type_id_with_cvr<T>() noexcept -> type_index`
* `static type_id_runtime<T>(const T&) noexcept -> type_index`

#### Implementation based on `std::type_info`

```c++
class detail::cvr_saver<T>{};

class stl_type_index : public type_index_facade<stl_type_index, std::type_info> {
  const std::type_info* data_;
public:
  using type_info_t = std::type_info;
  ctor(const type_info_t& data=typeid(void)) noexcept; //
  const type_info_t& type_info() const noexcept;
  const char* raw_name() const noexcept;
  const char* name() const noexcept;
  std::string pretty_name() const; // use core::scoped_demangled_name, treate cvr_saver

  size_t hash_code() const noexcept;
  bool equal(const stl_type_index&) const noexcept;
  bool before(const stl_type_index&) const noexcept;

  static stl_type_index type_id<T> () noexcept;
  static stl_type_index type_id_with_cvr<T> () noexcept; // wrap with cvr_saver if T has any of cvr
  static stl_type_index type_id_runtime<T> (const T& value) noexcept;
};

const std::type_info& detail::stl_construct_typeid_ref<T> (const T*) noexcept { return typeid(T); }

#define BOOST_TYPE_INDEX_REGISTER_CLASS \
    virtual const std::type_info& boost_type_index_type_id_runtime_() const noexcept \
    { return boost::typeindex::detail::stl_construct_typeid_ref(this); }
```

* Use `boost::core::demangled_name()` to get pretty name.
* Use a internal wrapper class to mark for `_with_cvr` case.
* Call hidden virtual function in `type_id_runtime` when RTTI is unavailable.

#### Implementation based on compile-time simulation

```c++
struct detail::ctti<T> {
  static const char* n() noexcept; // BOOST_TYPE_INDEX_FUNCTION_SIGNATURE, __FUNCSIG__, or __PRETTY_FUNCTION__, known actual place within it
};
class detail::ctti_data {}; // non-constructible, non-copyable type, used for type info ptr;
const ctti_data& ctti_construct<T>() noexcept() { return *reinterpret_cast<const ctti_data*>(ctti<T>::n()); }

class ctti_type_index : public type_index_facade<ctti_type_index, std::type_info> {
  const char* data_ = ctti<void>::n(); // use name pointer as type identifier information
  size_t get_raw_name_length() const noexcept; // strlen of raw_name excluding markers
  explicit ctor(const char* data) noexcept;
public:
  using type_info_t = ctti_data;
  constexpr ctor() noexcept;
  ctor(const type_info_t& data) noexcept;
  const type_info_t& type_info() const noexcept;
  constexpr const char* raw_name() const noexcept;
  constexpr const char* name() const noexcept;
  std::string pretty_name() const; // raw_name trimmed out ending spaces

  size_t hash_code() const noexcept; // hash_range on raw_name
  constexpr bool equal(const ctti_type_index&) const noexcept;
  constexpr bool before(const ctti_type_index&) const noexcept;

  constexpr static ctti_type_index type_id<T> () noexcept;
  constexpr static ctti_type_index type_id_with_cvr<T> () noexcept; // wrap with cvr_saver if T has any of cvr
  static ctti_type_index type_id_runtime<T> (const T& value) noexcept;
};

const ctti_data& detail::ctti_construct_typeid_ref<T> (const T*) noexcept { return ctti_construct<T>(); }

#define BOOST_TYPE_INDEX_REGISTER_CLASS \
    virtual const boost::typeindex::detail::ctti_data& boost_type_index_type_id_runtime_() const noexcept \
    { return boost::typeindex::detail::ctti_construct_typeid_ref(this); }
```

* `type_info` is an unusable type, `reinterpret_cast`ed from a C-string, which
  is extracted from a function's `BOOST_CURRENT_FUNCTION` (by default).
* `raw_name()` just cast back the `type_info` data to string.
* Customize macro `BOOST_TYPE_INDEX_CTTI_USER_DEFINED_PARSING` and
  `BOOST_TYPE_INDEX_FUNCTION_SIGNATURE` to override default compiler handling
  for extracting type from function name.

### `runtime_cast` (mimic `dynamic_cast`)

Header `<boost/type_index/runtime_cast.hpp>`

```c++
type_index detail::runtime_class_construct_type_id<T> (T const*) { return type_id<T>(); }
constexpr const void* detail::find_instance<Self> (type_index const&, const Self*) noexcept { return nullptr; }
const void* detail::find_instance<Base,...OtherBases,Self> (type_index const& idx, const Self*) noexcept {
  if (const void* ptr = self->Base::boost_type_index_find_instance_(idx)) return ptr;
  return find_instance<OtherBases...>(idx, self); // find in other branch
}



struct bad_runtime_cast : exception;

T runtime_cast<T,U>(U [const]* u) noexcept;
T [const]* runtime_cast<T,U>(U [const]* u) noexcept;

add_reference_t<[const] T> runtime_cast<T,U>(U [const]& u); // throws

std::shared_ptr<T> runtime_pointer_cast<T,U>(std::shared_ptr<U> const& u);
boost::shared_ptr<T> runtime_pointer_cast<T,U>(boost::shared_ptr<U> const& u);
```

* `BOOST_TYPE_INDEX_REGISTER_RUNTIME_CLASS` adds a recursive lookup virtual function.
  Arguments can be `BOOST_TYPE_INDEX_NO_BASE_CLASS` for nobase, and `(base1)(base2)...` for bases.
  It uses `type_id<T>()` internally.
* `runtime_cast` supports pointers, references
* `runtime_pointer_cast`, supports std/boost `shared_ptr`.
  It perform lookup for downcasting or sideway casting.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`.

#### Boost.ContainerHash

* `<boost/container_hash/hash_fwd.hpp>`, `<boost/container_hash/hash.hpp>`.

#### Boost.Core

* `<boost/core/demangle.hpp>`, when `std::type_info` based implementation is chosen.

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`, when `std::type_info` based implementation is chosen.

------
### Standard Facilities

* Standard Library: `<typeindex>` (C++11).
