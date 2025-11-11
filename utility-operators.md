# Boost.Utility/Operators

* lib: `boost/libs/utility`
* repo: `boostorg/utility`
* commit: `dda83c5`, 2025-08-26

------
### Operators

```c++
struct detail::empty_base<T>{};

// relationship operators
struct less_than_comparable2<T,U,B=empty_base<T>> : B { // requires T<U and T>U
    friend constexpr bool operator<=(const T& x, const U& y) { return !(x>y); } // and so on.
    // impl <= by >, >= by <, and inversed version of four ops
};
struct less_than_comparable1<T,B=empty_base<T>> : B { // requires T<U
    friend constexpr bool operator<=(const T& x, const T& y) { return !(x>y); } // and so on.
    // impl >, <=, >= all by <
};
struct equality_comparable2<T,U,B=empty_base<T>> : B { // requires T==U
    friend constexpr bool operator!=(const T& x, const U& y) { return !(x==y); } // and so on.
    // impl != on ==, and inversed version of ==, !=
};
struct equality_comparable1<T,B=empty_base<T>> : B // requires T==T
{   friend constexpr bool operator!=(const T& x, const T& y) { return !(x==y); } };

// commutative operators (NAME,OP) : (multipliable, *), (addable, +), (xorable, ^), (andable, &), (orable, |)
struct NAME2<T,U,B=empty_base<T>> : B { // requires T OP= U
    friend T operator OP(const T& x, const U& y) { T nrv{x}; nrv OP= y; return nrv; }
    friend T operator OP(const U& y, const T& x) { T nrv{x}; nrv OP= y; return nrv; }
};
struct NAME1<T,B=empty_base<T>> : B // requires T OP= T
{   friend T operator OP(const T& x, const T& y) { T nrv{x}; nrv OP= y; return nrv; } };
// non-commutative operators (NAME,OP) : (subtractable, -), (dividable, /), (modable, %)
struct NAME2<T,U,B=empty_base<T>> : B // requires T OP= U
{   friend T operator OP(const T& x, const U& y) { T nrv{x}; nrv OP= y; return nrv; } };
struct NAME2_left<T,U,B=empty_base<T>> : B // requires U OP= T
{   friend T operator OP(const U& y, const T& x) { T nrv{x}; nrv OP= y; return nrv; } };
struct NAME1<T,B=empty_base<T>> : B // requires T OP= T
{   friend T operator OP(const T& x, const T& y) { T nrv{x}; nrv OP= y; return nrv; } };

// ++/--/*/[]
struct incrementable<T,B=empty_base<T>> // requires ++T
{   friend T operator++(T& x, int) { T nrv{x}; ++x; return nrv; } };
struct decrementable<T,B=empty_base<T>> // requires --T
{   friend T operator++(T& x, int) { T nrv{x}; --x; return nrv; } };
struct dereferenceable<T,P,B=empty_base<T>>
{   P operator->() const { return addressof(*static_cast<const T&>(*this)); } };
struct indexable<T,I,R,B=empty_base<T>>
{   P operator[](I n) const { return *(static_cast<const T&>(*this) + n); } };

// binary (NAME,OP) : (left_shiftable, <<), (right_shiftable, >>)
struct NAME2<T,U,B=empty_base<T>> : B // requires T OP= U
{   friend T operator OP (const T& x, const U& y) { T nrv{x}; nrv OP= y; return nrv; } };
struct NAME1<T,B=empty_base<T>> : B // requires T OP= T
{   friend T operator OP (const T& x, const T& y) { T nrv{x}; nrv OP= y; return nrv; } };

// equivalent, ordering
struct equivalent2<T,U,B=empty_base<T>> : B // requires T < U and T > U
{   friend T operator==(const T& x, const U& y) { return !(x < y) && !(x > y); } };
struct equivalent1<T,B=empty_base<T>> : B // requires T < T
{   friend T operator==(const T& x, const T& y) { return !(x < y) && !(y < x); } };
struct partially_ordered2<T,U,B=empty_base<T>> : B { // requires T<U, T>U, and T==U
    friend constexpr bool operator<=(const T& x, const U& y) { return x<y || x==y; } // and so on.
    // x<=y := x>y||x==y; y>x := x<y; y<x := x>y; y<=x : x>y||x==y; y>=x: x<y||x==y;
};
struct partially_ordered1<T,B=empty_base<T>> : B { // requires T<T, and T==T
    friend constexpr bool operator>(const T& x, const T& y) { return y < x; } // and so on.
    // x<=y := x<y||x==y; x>=y := x>y||x==y
};

// combinations
struct totally_ordered2<T,U,B=empty_base<T>> : less_than_comparable2<T,U, equality_comparable2<T,U,B>> {};
struct totally_ordered1<T,B=empty_base<T>> : less_than_comparable1<T, equality_comparable1<T,B>> {};
struct additive2<T,U,B=empty_base<T>> : addable2<T,U, subtractable2<T,U,B>> {};
struct additive1<T,B=empty_base<T>> : addable1<T, subtractable1<T,B>> {};
struct multiplicative2<T,U,B=empty_base<T>> : multipliable2<T,U, dividable2<T,U,B>> {};
struct multiplicative1<T,U,B=empty_base<T>> : multipliable1<T,U, dividable1<T,U,B>> {};
struct integer_multiplicative2<T,U,B=empty_base<T>> : multiplicative2<T,U, modable2<T,U,B>> {};
struct integer_multiplicative1<T,B=empty_base<T>> : multiplicative1<T, modable1<T,B>> {};
struct arithmetic2<T,U,B=empty_base<T>> : additive2<T,U, multiplicative2<T,U,B>> {};
struct arithmetic1<T,B=empty_base<T>> : additive1<T, multiplicative2<T,B>> {};
struct integer_arithmetic2<T,U,B=empty_base<T>> : additive2<T,U, integer_multiplicative2<T,U,B>> {};
struct integer_arithmetic1<T,B=empty_base<T>> : additive1<T, integer_multiplicative2<T,B>> {};
struct bitwise2<T,U,B=empty_base<T>> : xorable2<T,U, andable2<T,U, orable2<T,U,B>>> {};
struct bitwise1<T,B=empty_base<T>> : xorable1<T, andable1<T, orable1<T,B>>> {};
struct unit_steppable<T,B=empty_base<T>> : incrementable<T, decrementable<T,B>> {};
struct shiftable2<T,U,B=empty_base<T>> : left_shiftable2<T,U, right_shiftable2<T,U,B>> {};
struct shiftable1<T,B=empty_base<T>> : left_shiftable1<T, right_shiftable1<T,B>> {};
struct ring_operators2<T,U,B=empty_base<T>> : additive2<T,U, subtractable2_left<T,U, multipliable2<T,U,B>>> {};
struct ring_operators1<T,B=empty_base<T>> : additive1<T, multipliable1<T,B>> {};
struct ordered_ring_operators2<T,U,B=empty_base<T>> : ring_operators2<T,U, totally_ordered2<T,U,B>> {};
struct ordered_ring_operators1<T,B=empty_base<T>> : ring_operators1<T, totally_ordered1<T,B>> {};
struct field_operators2<T,U,B=empty_base<T>> : ring_operators2<T,U, dividable2<T,U, dividable2_left<T,U,B>>> {};
struct field_operators1<T,B=empty_base<T>> : ring_operators1<T, dividable1<T,B>> {};
struct ordered_field_operators2<T,U,B=empty_base<T>> : field_operators2<T,U, totally_ordered2<T,U,B>> {};
struct ordered_field_operators1<T,B=empty_base<T>> : field_operators1<T, totally_ordered1<T,B>> {};
struct euclidian_ring_operators2<T,U,B=empty_base<T>> : ring_operators2<T,U, dividable2<T,U, dividable2_left<T,U, modable2<T,U, modable2_left<T,U,B>>>>> {};
struct euclidian_ring_operators1<T,B=empty_base<T>> : ring_operators1<T, dividable1<T, modable1<T,B>>> {};
struct ordered_euclidian_ring_operators2<T,U,B=empty_base<T>> : totally_ordered2<T,U, euclidian_ring_operators2<T,U,B>> {};
struct ordered_euclidian_ring_operators1<T,B=empty_base<T>> : totally_ordered1<T, euclidian_ring_operators1<T,B>> {};
using euclidean_ring_operators2<T,U,B=empty_base<T>> = euclidian_ring_operators2<T,U,B>; //...
struct input_iteratable<T,P,B=empty_base<T>> : equality_comparable1<T, incrementable<T, dereferenceable<T,P,B>>> {};
struct output_iteratable<T,B=empty_base<T>> : incrementable<T,B> {};
struct forward_iteratable<T,P,B=empty_base<T>> : input_iteratable<T,P,B> {};
struct bidirectional_iteratable<T,P,B=empty_base<T>> : forward_iteratable<T,P,decrementable<T,B>> {};
struct random_access_iteratable<T,P,D,R,B=empty_base<T>> : bidirectional_iteratable<T,P, less_than_comparable1<T, additive2<T,D, indexable<T,D,R,B>>>> {};

// generic version
struct is_chained_base<T>;
struct addable<T,U=T,B=empty_base<T>,O=is_chained_base<U>::value>;
template<class T, class U, class B>
struct addable<T,U,B,false_t> : addable2<T, U, B> {};
struct addable<T,U,empty_base<T>,true_t> : addable1<T, U> {};
struct addable<T,T,B,false_t> : addable1<T, B> {};
struct is_chained_base<addable<T,U,B,O>> { using value = true_t; };
struct is_chained_base<addable2<T,U,B>> { using value = true_t; };
struct is_chained_base<addable1<T,B>> { using value = true_t; };
// generic versions for all base classes with -2/-1 tail

// overall version
struct operators2<T,U> : totally_ordered2<T,U, integer_arithmetic2<T,U, bitwise2<T,U>>> {};
struct operators<T,U=T> : operators2<T,U> {};
struct operators<T,T> : totally_ordered<T, integer_arithmetic<T, bitwise<T, unit_steppable<T>>>> {};

struct iterator_helper<Cat,T,D=ptrdiff_t,P=T*,R=T&> {
    using iterator_category = Cat; using value_type = T; using difference_type = D; using pointer = P; using reference = R;
};
struct input_iterator_helper<T,V,D=ptrdiff_t,P=V const*,R=V const&> : input_iteratable<T,P, iterator_helper<std::input_iterator_tag,V,D,P,R>> {};
struct output_iterator_helper<T> : output_iteratable<T, iterator_helper<std::output_iterator_tag,void,void,void,void>> {
    T& operator*() { return static_cast<T&>(*this); }
    T& operator++() { return static_cast<T&>(*this); }
};
struct bidirectional_iterator_helper<T,V,D=ptrdiff_t,P=V*,R=V&> : bidirectional_iteratable<T,P, iterator_helper<std::bidirectional_iterator_tag,V,D,P,R>> {};
struct random_access_iterator_helper<T,V,D=ptrdiff_t,P=V*,R=V&> : random_access_iteratable<T,P, iterator_helper<std::random_access_iterator_tag,V,D,P,R>> {
    friend D requires_difference_operator(const T& x, const T& y) { return x - y; }
};
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Core

* `<boost/core/addressof.hpp>`

------
### Standard Facilities
