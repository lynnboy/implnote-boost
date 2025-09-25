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
    auto boost_enum_descriptor_fn(E**) \
    { return enum_descriptor_fn_impl(0 \
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

#### Describe Classes

* Header `<boost/describe/class.hpp>`
* Header `<boost/describe/modifier_description.hpp>`

```c++
enum modifiers {
    mod_public = 1, mod_protected = 2, mod_private = 4, mod_virtual = 8, mod_static = 16,
    mod_function = 32, mod_any_member = 64, mod_inherited = 128, mod_hidden = 256
};
constexpr modifiers mod_any_access = 7; // public | protected | private
BOOST_DESCRIBE_ENUM(modifiers, mod_public, ...) // all mod_xxx

using detail::is_public_base_of<T,U> = std::is_convertible<U*,T*>;
using detail::is_protected_base_of<T,U> = (std::is_final<U> || is_union<U>) ? false_type
    : requires { /* detects `U* -> T*` works in derived class of U */ };
using detail::can_cast<T,U> = requires {{(U*)(T*)0};};
using detail::is_virtual_base_of<T,U> = can_cast<U,T> && !can_cast<T,U>;
constexpr unsigned detail::compute_base_modifiers <C,B> () noexcept {
    return (is_public_base_of<B,C> ? mod_public :
        is_protected_base_of<B,C> ? mod_protected : mod_private)
        | (is_virtual_base_of<B,C> ? mod_virtual : 0);
}

struct detail::base_descriptor<C,B> { // requires is_base_of<B,C>.
    using type = B; static constexpr unsigned modifiers = compute_base_modifiers<C,B>();
};
struct detail::bases_descriptor_impl<...T>;
using detail::bases_descriptor_impl<C,list<Bs...>>::type = list<base_descriptor<C,Bs>...>;

#define BOOST_DESCRIBE_BASES(C,...) \
    inline auto boost_base_descriptor_fn( C** ) \
    { return bases_descriptor_impl<C,list<__VA_ARGS__>>::type{}; }

constexpr unsigned detail::add_static_modifier <Pm> (Pm)
{ return std::is_member_pointer_v<Pm> ? 0 : mod_static; }
constexpr unsigned detail::add_function_modifier <Pm> (Pm)
{ return std::is_member_function_pointer_v<Pm> || std::is_function_v<std::remove_pointer_t<Pm>> ? mod_function : 0; }

struct detail::member_descriptor { // capture member pointer, name, and modifiers
    static constexpr decltype(D::pointer()) pointer = D::pointer();
    static constexpr decltype(D::name()) name = D::name();
    static constexpr unsigned modifiers = M | add_static_modifier(D::pointer()) | add_function_modifier(D::pointer());
};
auto detail::member_descriptor_fn_impl<M,...T> (int, T...) { return list<member_descriptor<T,M>...>{}; }
constexpr auto detail::mfn<C,F>(F C::* p) {return p;} // non-static member function
constexpr auto detail::mfn<C,F>(F* p) {return p;} // static member function

#define BOOST_DESCRIBE_{PUBLIC|PROTECTED|PRIVATE}_MEMBERS(C, ...) \
    inline auto boost_{public|protected|private}_member_descriptor_fn( C** ) \
    { return member_descriptor_fn_impl<mod_{public|protected|private}>(0
#foreach [sig] m in __VA_ARGS__ // Boost.Preprocessor
            , []{ struct _d{ static constexpr auto pointer() noexcept { return &C::m; /* (C::*)sig */ } \
                static constexpr auto name() noexcept { return #m; } }; return d_{}; }()
#end foreach
            ); }

#define UNPACK(...) __VA_ARGS__
#define BOOST_DESCRIBE_CLASS(C, Bases, Public, Protected, Private) \
    friend BOOST_DESCRIBE_BASES(C, UNPACK Bases) \
    friend BOOST_DESCRIBE_PUBLIC_MEMBERS(C, UNPACK Public) \
    friend BOOST_DESCRIBE_PROTECTED_MEMBERS(C, UNPACK Protected) \
    friend BOOST_DESCRIBE_PRIVATE_MEMBERS(C, UNPACK Private)
#define BOOST_DESCRIBE_STRUCT(C, Bases, Public) \
    BOOST_DESCRIBE_BASES(C, UNPACK Bases) \
    BOOST_DESCRIBE_PUBLIC_MEMBERS(C, UNPACK Public) \
    BOOST_DESCRIBE_PROTECTED_MEMBERS(C) \
    BOOST_DESCRIBE_PRIVATE_MEMBERS(C)
```

Usage:

```c++
struct B {}; struct B2 {};
struct X : B, B2 { int m1; static const int m2; };
BOOST_DESCRIBE_STRUCT(B, (), ())
BOOST_DESCRIBE_STRUCT(X, (B,B2), (m1,m2))

struct Y : protected virtual B, private B2 {
private: int m1;
protected: static const int m2; int f1(int);
public: int f2(); static void f1();
BOOST_DESCRIBE_CLASS(Y, (B,B2), (f2, (void()) f1), (m2, (int(int)) f1), (m1))
}
```

------

* Header `<boost/describe/bases.hpp>`
* Header `<boost/describe/members.hpp>`

```c++
using detail::_describe_bases<T> = decltype(boost_base_descriptor_fn(static_cast<T**>(0)));
using describe_bases<T,Mod> = mp_copy_if_q<_describe_bases<T>, Mod & mod_any_access & _1::modifiers>; // filter by Mod
using has_describe_bases<T> = {SFINAE{_describe_bases<T>}} ? true_type : false_type;

using detail::_describe_{public|protected|private}_members = decltype(boost_{public|protected|private}_member_descriptor_fn(static_cast<T**>(0)));
using detail::_describe_members<T> = mp_append<_describe_public_members<T>, _describe_protected_members<T>, _describe_private_members<T>>;
using detail::gather_virtual_bases<D> = ({ using virt_of_bases = mp_transform<gather_virtual_bases, _describe_bases<D::type>> })
    D::modifiers & mod_virtual ? mp_push_front<virt_of_bases, D> : virt_of_bases; //recursion
using detail::describe_inherited_members<T,VBases> = mp_append<describe_inherited_members2<describe_bases<T,mod_any_access>, T, V>, _describe_members<T>>;
using detail::describe_inherited_members2<L<...>,T,VBases> = L<>; // primary, no base / recursion end
using detail::name_is_hidden<D,L> = mp_any_of_q<L, strcmp(D::name,_1::name)==0>; // name hidden
struct detail::update_modifiers<T,bm>::fn<D> {
    static constexpr auto pointer = D::pointer; static constexpr auto name = D::name;
    static constexpr unsigned modifiers = (mods &~ mod_any_access) | max(mods & mod_any_access, bm & mod_any_access) // update access
        | name_is_hidden<D,_describe_members<T>>::value ? mod_hidden : 0; // detect hidden flag
};
using detail::describe_inherited_members2<L<D1,D...>,T,VBases> = mp_append<
    ({  using inh_of_D1 = D1::modifiers & mod_virtual && mp_contains<VBases, B> ? L<> : describe_inherited_members<D1::type, VBases>;
        mp::transform<inh_of_D1, update_modifiers<D1::modifiers>> }),
    describe_inherited_members2<L<D...>,T,mp_append<VBases,gather_virtual_bases<D1>>>>;
using describe_members<T,Mod> = mp_copy_if_q<_describe_members<T>,
    (Mod & mod_any_access & _1::modifiers) && // Mod filter by access
    (Mod & mod_any_member || Mod & mod_static == _1::modifiers & mod_static || Mod & mod_function == _1::modifiers & mod_function) && // filter by member type
    (Mod & mod_hidden >= _1::modifiers & mod_hidden) // Option to get hidden members
    >
using has_describe_members<T> = {SFINAE{_describe_members<T>}} ? true_type : false_type;
```

* Header `<boost/describe/descriptor_by_name.hpp>`
* Header `<boost/describe/descriptor_by_pointer.hpp>`

```c++
#define BOOST_DESCRIBE_MAKE_NAME(s) struct _boost_name_##s##_##__LINE__ { static constexpr char const* name() { return #s; }}

using descriptor_by_name<L,N> = mp_at<L, mp_find_if_q<L, strcmp(N::name(), _1::name) == 0>>;
using descriptor_by_pointer<L,Pm> = mp_at<L, mp_find_if_q<L, _1::pointer == Pm>>;
```

* Header `<boost/describe/operators.hpp>`

```c++
bool detail::eq<T, Bd=describe_bases<T,mod_any_access>, Md=describe_members<T,mod_any_access>> (T const& t1, T const& t2);
bool detail::lt<T, Bd=describe_bases<T,mod_any_access>, Md=describe_members<T,mod_any_access>> (T const& t1, T const& t2);
void detail::print<Os, T, Bd=describe_bases<T,mod_any_access>, Md=describe_members<T,mod_any_access>> (Os & os, T const& t);

bool operators::operator== <T> (T const& t1, T const& t2) requires has_describe_bases<T> && has_describe_members && !std::is_union<T>;
// also !=, <, >, <=, >=
std::basic_ostream<Ch,Tr>& operators::operator<< <T,Ch,Tr>  (std::basic_ostream<Ch,Tr>& os, T const& t) requires has_describe_bases<T> && has_describe_members && !std::is_union<T>;
```

------
### Dependency

#### Boost.MP11

* `<boost/mp11/*.hpp>`

------
### Standard Facilities
