# Boost.Optional

* lib: `boost/libs/optional`
* repo: `boostorg/optional`
* commit: `0755ab7b`, 2016-03-08

------
### Boost Optional

Header `<boost/optional.hpp>`
Header `<boost/optional/optional_io.hpp>`

#### Class Template `optional`

Allow bind to reference type (but not rvalue-reference)

Differences from STD proposal:

* Supports bind to lvalue-reference, specialized for reference and storing pointer for them.
* Constructor `optional(bool, T const&)` and `make_optional(bool, T const&)`
* Constructors/assignments for `optional<U>`, when `T` can construct from `U`
* Constructors/assignments for _InPlaceFactory_, but no `in_place_t` constructors.
* Member and non-member `get()` and `get_ptr()`, to get reference/pointer to stored data.
* Provide `value_or_eval(F)`
* `boost::none` vs. `std::nullopt`

#### Exception `bad_optional_access`

Thrown by `optional::value()` when it contains nothing.

#### I/O of `optional`

* Empty `optional` becomes `"--"`.
* Non-empty `optional` having a leading space and the content value's representation.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Core

* `<boost/core/addressof.hpp>`
* `<boost/core/enable_if.hpp>`
* `<boost/core/explicit_operator_bool.hpp>`
* `<boost/core/swap.hpp>`
* `<boost/type.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.MPL

* `<boost/mpl/if.hpp>`, `<boost/mpl/bool.hpp>`, `<boost/mpl/not.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/alignment_of.hpp>`, `<boost/type_traits/type_with_alignment.hpp>`
* `<boost/type_traits/*.hpp>` for const/reference, decay, etc.
* `<boost/type_traits/*.hpp>` for nothrow_xxx detection
* `<boost/type_traits/is_same.hpp>`, `<boost/type_traits/is_base_of.hpp>`

#### Boost.Move

* `<boost/move/utility.hpp>`

#### Boost.Utility

* `<boost/utility/compare_pointees.hpp>`

------
### Standard Facilities

Proposals:
  Library Fundamentals v1 - `<optional>`
