# Boost.Utility

* lib: `boost/libs/utility`
* repo: `boostorg/utility`
* commit: `dda83c5`, 2025-08-26

------
### `base_from_member`

* Header `<boost/utility/base_from_member.hpp>`

```c++
class base_from_member<MemberType,UniqueID=0> {
protected:
    MemberType  member;
    explicit constexpr ctor<...T>(T&& ...x) noexcept(...);
};

class base_from_member<MemberType&, UniqueID> {
protected:
    MemberType& member;
    explicit constexpr ctor(MemberType& x) noexcept;
};
```

Usage: Utilize EBO. Use `base_from_member` as base class to store a member, use `UniqueID` to differentiate members of same type.

------
### Binary Literals

* Header `<boost/utility/binary.hpp>`

* `int` and modified types: `BOOST_BINARY(bit_groupings)`, `BOOST_BINARY_U(bit_groupings)`, `BOOST_BINARY_L(bit_groupings)`, `BOOST_BINARY_{UL|LU}(bit_groupings)`, `BOOST_BINARY_{ULL|LLU}(bit_groupings)`
* Separate suffix version: `BOOST_SUFFIXED_BINARY_LITERAL(bit_groupings, suffix)`

------
### Pointee comparation

* Header `<boost/utility/compare_pointees.hpp>`

```c++
bool equal_pointees<OptT>(OptT const& x, OptT const& y) { return (!x)!=(!y) ? false : !x ? true : (*x)==(*y); }
struct equal_pointees_t<OptT> {
    using result_type = bool; using first_argument_type = OptT; using second_argument_type = OptT;
    bool operator()(OptT const& x, OptT const& y) const { return equal_pointees(x,y); }
};

bool less_pointees<OptT>(OptT const& x, OptT const& y) { return !y ? false : !x ? true : (*x)<(*y); }
struct less_pointees_t<OptT> {
    using result_type = bool; using first_argument_type = OptT; using second_argument_type = OptT;
    bool operator()(OptT const& x, OptT const& y) const { return less_pointees(x,y); }
};
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Core

* `<boost/core/enable_if.hpp>`
* `<boost/core/invoke_swap.hpp>` - by `value_init`

#### Boost.IO

* `<boost/io/ostream_put.hpp>` - by `string_ref` and `string_view`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>` - by `string_ref` and `string_view`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`
* `<boost/type_traits/fucntion_traits.hpp>` - by `BOOST_IDENTITY_TYPE`

------
### Standard Facilities
