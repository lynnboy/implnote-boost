# Boost.Tuple

* lib: `boost/libs/tuple`
* repo: `boostorg/tuple`
* commit: `d04e4a6`, 2025-06-09

------
### Tuple library

#### Header

* `<boost/tuple/tuple.hpp>`
* `<boost/tuple/tuple_comparison.hpp>`
* `<boost/tuple/tuple_io.hpp>`

#### API for `tuple`

```c++
struct null_type {}; // empty tuple

struct cons<HT,TT> {
    using head_type = HT; using tail_type = TT;
    using stored_head_type = is_function_v<HT> ? type<T> : T;
    stored_head_type head; tail_type tail; // data

    auto get_head() -> access_traits<stored_head_type>::non_const_type;
    auto get_head() const -> access_traits<stored_head_type>::const_type;
    auto get_tail() -> access_traits<tail_type>::non_const_type;
    auto get_tail() const -> access_traits<tail_type>::const_type;

    cons(); cons(const cons& u); cons<HT2,TT2>(const cons<HT2,TT2>&);
    cons(access_traits<stored_head_type>::parameter_type h, const tail_type& t);
    cons<T1,...,T10>(T1& t1, ..., T10& t10) : head(t1), tail(t2,...,t10, null_type{}){}
    cons<T2,...,T10>(const null_type&, T2& t2, ..., T10& t10), head(), tail(t2,...,t10, null_type{}){}

    cons& operator= (const cons&);
    cons& operator= <HT2,TT2> (const cons<HT2,TT2>&);
    cons& operator= <T1,T2> (const std::pair<T1,T2>&);

    auto get<N>() -> access_traits<element<N,cons>::type>::non_const_type;
    auto get<N>() const -> access_traits<element<N,cons>::type>::const_type;
};

struct cons<HT,null_type> {
    using head_type = HT; using tail_type = null_type;
    using stored_head_type = is_function_v<HT> ? type<T> : T;
    stored_head_type head; // data, no tail

    auto get_head() -> access_traits<stored_head_type>::non_const_type;
    auto get_head() const -> access_traits<stored_head_type>::const_type;
    [const] null_type get_tail() [const];

    cons(); cons(const cons& u); cons<HT2>(const cons<HT2,null_type>&);
    cons(access_traits<stored_head_type>::parameter_type h, const tail_type& t={});
    cons<T1>(T1& t1, const null_type&.../*9*/) : head(t1){}
    cons(const null_type&.../*10*/) : head(){}

    cons& operator= (const cons&);
    cons& operator= <HT2> (const cons<HT2,null_type>&);

    auto get<N>() -> access_traits<element<N,cons>::type>::non_const_type;
    auto get<N>() const -> access_traits<element<N,cons>::type>::const_type;
};

using length<T> = integral_constant<1+length<T::tail_type>::value>; // and 0 for [const] tuple<> and null_type
struct element<N,T> { using type=...; };

struct access_traits<T> {
    using const_type = const T&; using non_const_type = T&;
    using parameter_type = const remove_cv_t<T>&;
};
struct access_traits<T&> {
    using const_type = T&; using non_const_type = T&; using parameter_type = T&;
};

auto get<N,HT,TT>(      cons<HT,TT>& c) -> access_traits<element<N,cons<HT,TT>>::type>::non_const_type;
auto get<N,HT,TT>(const cons<HT,TT>& c) -> access_traits<element<N,cons<HT,TT>>::type>::const_type;

struct map_tuple_to_cons<T0,...,T9> { using type = cons<T0, map_tuple_to_cons<T1,..,T9,null_type>; };
struct tuple<T0=null_type,...,T9=null_type> : map_tuple_to_cons<T0,...,T9>::type {
    tuple(); tuple<U1,U2>(const cons<U1,U2>&);
    tuple(access_traits<T0>::parameter_type t0, ... access_traits<T9>::parameter_type t9); // 1 ~ 10 args
    tuple& operator= <U1,U2> (const cons<U1,U2>&);
    tuple& operator= <U1,U2> (const std::pair<U1,U2>&);
};

using detail::ignore_t = void(swallow_assign::*)();
struct detail::swallow_assign {
    ctor(ignore_t(*)(ignore_t)) {}; swallow_assign const& operator= <T>(const T&) const{return this*;}
};
ignore_t ignore(ignore_t); // used for `tie`

using make_tuple_traits<T>::type = T;
using make_tuple_traits<T&>::type = ERROR;
using make_tuple_traits<[const] T[n]>::type = const T(&)[n];
using make_tuple_traits<[const] volatile T[n]>::type = const volatile T(&)[n];
using make_tuple_traits<[const] reference_wrapper<T>>::type = T&;
using make_tuple_traits<ignore_t(ignore_t)>::type = swallow_assign;
using detail::make_tuple_mapper<T0=null_type,...,T9=null_type>::type = tuple<make_tuple_mapper<T0>::type...>;
auto make_tuple() -> tuple<>;
auto make_tuple<T0,...,T9>(const T0& t0,...,const T9& t9) -> make_tuple_mapper<T0,...,T9>::type; // 1 ~ 10 args

using detail::tie_traits<T>::type = T&;
using detail::tie_traits<void>::type = null_type;
using detail::tie_traits<ignore_t(ignore_t)>::type = swallow_assign;
using detail::tie_mapper<T0=void,...,T9=void>::type = tuple<tie_traits<T0>::type,...,tie_traits<T9>::type>;
auto tie<T0,...,T9> (T0& t0,...,T9& t9) -> tie_mapper<T0,...,T9>::type; // 1 ~ 10 args

void swap (null_type&, null_type&);
void swap<HH> (cons<HH,null_type>& lhs, cons<HH,null_type>& rhs); // invoke_swap on head for ADL
void swap<HH,TT> (cons<HH,TT>& lhs, cons<HH,TT>& rhs);
void swap<T0,...,T9>(tuple<T0,...,T9>& lhs, tuple<T0,...,T9>& rhs);
```

##### Comparison

```c++
bool operator== (const null_type&, const null_type&); // and !=, <, >, <=, >=
bool operator==<T1,T2,S1,S2> (const cons<T1,T2>& lhs, const cons<H1,H2>& rhs); // and !=, <, >, <=, >=
```

##### I/O Support

```c++
struct detail::format_info { // use iword to get/set info into stream
    enum manipulator_type {open, close, delimiter};
    static Ch get_manipulator<Ch,Tr> (std::basic_ios<Ch,Tr>&, manipulator_type);
    static void set_manipulator<Ch,Tr> (std::basic_ios<Ch,Tr>&, manipulator_type, Ch c);
};
class tuple_manipulator<Ch> {
    format_info::manipulator_type mt; Ch f_c; // type and value
public:
    explicit ctor(format_info::manipulator_type m, Ch c={});
    void set<Tr> (std::basic_ios<Ch,Tr>&) const;
};
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>&, const tuple_manipulator<Ch>&);
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr> (std::basic_istream<Ch,Tr>&, const tuple_manipulator<Ch>&);

tuple_manipulator<Ch> set_open<Ch> (Ch);
tuple_manipulator<Ch> set_close<Ch> (Ch);
tuple_manipulator<Ch> set_delimiter<Ch> (Ch);

std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>&, const tuple_manipulator<Ch>&, const null_type&);
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,T1,T2> (std::basic_ostream<Ch,Tr>&, const tuple_manipulator<Ch>&, const cons<T1,T2>&);
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr> (std::basic_istream<Ch,Tr>&, const tuple_manipulator<Ch>&, null_type&);
std::basic_istream<Ch,Tr>& operator>> <Ch,Tr,T1,T2> (std::basic_istream<Ch,Tr>&, const tuple_manipulator<Ch>&, cons<T1,T2>&);
```

* Custom manipulators `set_open(CharT)`, `set_close(CharT)`, and `set_delimiter(CharT)`.
* Operators `<<` and `>>`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Core

* `<boost/core/ref.hpp>`.
* `<boost/core/invoke_swap.hpp>`.

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/cv_traits.hpp>`
* `<boost/type_traits/function_traits.hpp>`
* `<boost/type_traits/integral_constant.hpp>`

------
### Standard Facilities

Standard Library: `<tuple>` (C++11)
