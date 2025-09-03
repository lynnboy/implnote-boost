# Boost.Conversion

* lib: `boost/libs/conversion`
* repo: `boostorg/conversion`
* commit: `9f285ef`, 2024-01-21

------
### implicit_cast

#### Header

* `<boost/implicit_cast.hpp>`

#### Function

* `constexpr boost::implicit_cast<T>(x)`;

------
### polymorphic_cast, polymorphic_pointer_cast

#### Header

* `<boost/polymorphic_cast.hpp>`
* `<boost/polymorphic_pointer_cast.hpp>`

#### Function

* `boost::polymorphic_cast<T>(x)`;
* `boost::polymorphic_downcast<T>(x)`;

#### Rationale

`polymorphic_cast` is a throwing version of pointer `dynamic_cast`.
`polymorphic_downcast` is a pointer/reference `static_cast` with assertion for `dynamic_cast`.
`polymorphic_pointer_cast` and `polymorphic_pointer_downcast` is smartptr version of
above casts, using `static_pointer_cast` and `dynamic_pointer_cast` instead.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`.

#### Boost.Assert

* `<boost/assert.hpp>`.

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`.

#### Boost.SmartPtr

* `<boost/pointer_cast.hpp>`.

#### Boost.TypeTraits

* `<boost/type_traits/is_reference.hpp>`, `<boost/type_traits/remove_reference.hpp>`.
* `<boost/utility/declval.hpp>`.

#### Boost.Core

* `<boost/core/addressof.hpp>`
* `<boost/core/enable_if.hpp>`

#### Boost.TypeOf

* `<boost/typeof.hpp>`, if compiler don't supports `decltype`.

------
### Standard Facilities

* Language: `static_cast`, `dynamic_cast`.
