# Boost.Optional

* lib: `boost/libs/describe`
* repo: `boostorg/describe`
* commit: `bfdf0c7`, 2024-12-09

------
Header `<boost/describe.hpp>`

#### Describe Enumeration Types

* Header `<boost/describe/enum.hpp>`

```c++
struct detail::list<...T>{}; // just type wrapper
struct detail::enum_descriptor<D> { // capture value and name from D
    static constexpr decltype(D::value()) value = D::value();
    static constexpr decltype(D::name()) name = D::name();
};
auto detail::enum_descriptor_fn_impl<...T> (int, T...) { return list<enum_descriptor<T>...>{}; }

// make an local type wrapping `value()` and `name()` for each enumerator, wrap them into a type-list.
#define BOOST_DESCRIBE_ENUM(E, ...) \
    auto boost_enum_descriptor_fn(E**) { \
        return enum_descriptor_fn_impl(0 \
#foreach e in __VA_ARGS__ // Boost.Preprocessor
            , []{ struct _d{ static constexpr auto value() noexcept { return E::e; }
                static constexpr auto name() noexcept { return #e; } }; return d_{}; }()
#end foreach
        ); }
#define BOOST_DESCRIBE_NESTED_ENUM(E, ...) ... // define `friend` function

#define BOOST_DEFINE[_FIXED]_ENUM[_CLASS](E, {Base,} ...) enum {class} E {: Base} { __VA_ARGS__ }; BOOST_DESCRIBE_ENUM(E, __VA_ARGS__)
```

Usage:

```c++
enum E1 { a=5, b=10, c=15 };
BOOST_DESCRIBE_ENUM(E1, a, b, c)

struct O {
    enum E2 { a, b, c };
    BOOST_DESCRIBE_NESTED_ENUM(E2, a, b, c)
};

BOOST_DEFINE_ENUM_CLASS(E3, a, b, c)
```

----
* Header `<boost/describe/enumerators.hpp>`
* Header `<boost/describe/enum_to_string.hpp>`
* Header `<boost/describe/enum_from_string.hpp>`

```c++
using describe_enumerators<E> = decltype( boost_enum_descriptor_fn(static_cast<E**>(0)) ); // the type-list
using has_describe_enumerators<E> = {sfinae describe_enumerators<E>} ? true_type : false_type;

char const* enum_to_string <E,De=describe_enumerators<E>> (E e, char const* def) noexcept; // match value in list
bool enum_from_string <E,De=describe_enumerators<E>> (char const* name, E& e) noexcept; // match name by strcmp in list
bool enum_from_string <S,E,De=describe_enumerators<E>> (S const& name, E& e) noexcept; // for string types
```

Usage:

```c++
mp11::mp_for_each<describe_enumerators<E1>>([](auto de){ std::printf("%s, %d\n", de.name, de.value); });
static_assert(has_describe_enumerators<E1>::value);
static_assert(!has_describe_enumerators<int>::value);
assert(string("a") == enum_to_string(E3::a, "?"));
E3 e; assert(enum_from_string("a", e));
```

#### Class Template `optional`

Allow bind to reference type (but not rvalue-reference).

Optimized for trivally-copyable types.

Differences from STD proposal:

* Supports bind to lvalue-reference, specialized for reference and storing pointer for them. (C++26 included)
* Constructors/assignments for `optional<U>`, when `T` can construct from `U`
* Constructors/assignments for _InPlaceFactory_, and `in_place_init_if_t` constructors.
* Member and non-member `get()` and `get_ptr()`, to get reference/pointer to stored data.
* Supports conditional init/make (`cond, val`).
* `boost::none` vs. `std::nullopt`

#### Exception `bad_optional_access`

Thrown by `optional::value()` when it contains nothing.

#### I/O of `optional`

* Empty `optional` becomes `"--"`.
* Non-empty `optional` having a leading space and the content value's representation.

------
### Dependency

#### Boost.MP11

* `<boost/mp11/*.hpp>`

------
### Standard Facilities
