# Boost.Lambda2

* lib: `boost/libs/lambda2`
* repo: `boostorg/lambda2`
* commit: `dddfec6`, 2025-06-28

#### Class `source_location`

* `<boost/lambda2.hpp>`

```c++
struct detail::subscript { // wrapper for `__arg1__[__arg2__]`
  decltype(auto) operator() <T1,T2> (T1&& t1, T2&& t2) const
  { return std::forward<T1>(t1)[ std::forward<T2>(t2) ]; }
};
struct detail::get<I> { // wrapper for `get<I>(__arg__)`
  decltype(auto) operator() <T> (T&& t) const
  { return std::get<I>( std::forward<T>(t) ); }
};

struct lamgda2_arg<I> { // allow `_2(a,b,c)` => `b`, and _
  decltype(auto) operator() <...A> (A&&... a) const noexcept // get (I-1)th element of the pack (a...)
  { return std::get<I-1>( std::tuple<A&&...>( std::forward<A>(a)...) ); }
  auto operator[] <T> (T&& t) const // make a functor wrapping `(*this)[t]`
  { return std::bind( subscript(), *this, std::forward<T>(t) ); }
};

inline constexpr lamgda2_arg<1> _1{}; // up to _9

struct std::is_placeholder<lambda2_arg<I>> : integral_constant<int, I>{}; // placeholder enablement

using detail::is_lambda_expression<T, T2=remove_cvref_t<T>> =
  std::bool_constant<std::is_placeholder<T2>::value || std::is_bind_expression<T2>::value>;
using detail::enable_unary_lambda<A> = std::enable_if_t<is_lambda_expression<A>::value>;
using detail::enable_binary_lambda<A,B> = std::enable_if_t<is_lambda_expression<A>::value || is_lambda_expression<B>::value>;
using is_stream<T> = std::is_base_of<std::ios_base, remove_cvref_t<T>>;

// STL missing templates for these operators:
struct detail::unary_plus { decltype(auto) operator() <T> (T&& t) const { + std::forward<T>(t); } };
// also dereference(*), increment(++), decrement(--)
struct detail::postfix_increment { decltype(auto) operator() <T> (T&& t) const { std::forward<T>(t) ++; } };
// also postfix_decrement(--)
struct detail::left_shift { decltype(auto) operator() <T1,T2> (T1&& t1, T2&& t2) const { std::forward<T1>(t1) << std::forward<T2>(t2); } };
// also right_shift(>>), plus_equal(+=), minus_equal(-=), multiplies_equal(*=), divides_equal(/=), modulus_equal(%=),
//    bit_and_equal(&=), bit_or_equal(|=), bit_xor_equal(^=), left_shift_equal(<<=), right_shift_equal(>>=)

// unary:
auto operator + <A,=enable_unary_lambda<A>> (A&& a)
{ return std::bind(unary_plus(), std::forward<A>(a)); }
// also: - (std::negate<>), * (dereference), ++ (increment), -- (decrement), ! (std::logical_not<>), ~ (std::bit_not<>)

// postfix unary:
auto operator ++ <A,=enable_unary_lambda<A>> (A&& a, int)
{ return std::bind(postfix_increment(), std::forward<A>(a)); }
// also: -- (postfix_decrement)

// binary:
auto operator + <A,B,=enable_binary_lambda<A,B>> (A&& a, B&& b)
{ return std::bind(std::plus<>(), std::forward<A>(a), std::forward<B>(b)); }
// also - (std::minus), * (std::multiples), / (std::divides), % (std::modulus),
//    == (std::equal_to<>), != (std::not_equal_to<>), > (std::greater<>), < (std::less<>), >= (std::greater_equal<>), <= (std::less_equal<>),
//    && (std::logical_and<>), || (std::logical_or<>), & (std::bit_and<>), | (std::bit_or<>), ^ (std::bit_xor<>),
//    += (plus_equal), -= (minus_equal), *= (multiples_equal), /= (divides_equal), %= (modulus_equal),
//    &= (bit_and_equal), |= (bit_or_equal), ^= (bit_xor_equal), <<= (left_shift_equal), >>= (right_shift_equal)

auto operator << <A,=enable_if_t<!is_stream<A>::value>,B,=enable_binary_lambda<A,B>> (A&& a, B&& b) // shifting
{ return std::bind( left_shift(), std::forward<A>(a), std::forward<B>(b) ); }
auto operator << <A,=enable_if_t<is_stream<A>::value>,B,=enable_binary_lambda<A,B>> (A& a, B&& b) // inserter
{ return std::bind( left_shift(), std::ref<A>(a), std::forward<B>(b) ); }
// also >> (right_shift)

// projections: allow use of `<expr>->*first` => `std::get<0>(<expr>)`
inline constexpr detail::get<0> first{};
inline constexpr detail::get<1> second{};
auto operator ->* <A,B,enable_unary_lambda<A>> (A&& a, B&& b)
{ return std::bind( std::forward<B>(b), std::forward<A>(a) ); }
```

------
### Standard Facilities

Language: lambda expression (C++11)
