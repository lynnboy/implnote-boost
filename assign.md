# Boost.Assignment

* lib: `boost/libs/assign`
* repo: `boostorg/assign`
* commit: `dcc4d364`, 2015-8-15

------
### `list_inserter` and functions

Header `<boost/assign/list_inserter.hpp>`

```c++
template <typename Function, typename Argument>
class list_inserter;
```

Provides common interface for element insertion.
Store a `Function` as inserter functor.

#### Constructors

* `list_inserter(Function fun)`
* `list_inserter(const list_inserter<Func2,Arg>&)`
* `list_inserter(const list_inserter&)`

#### Members

* `operator()()`
* `operator()<T>(const T& ...)`
* `operator=<T>(const T&)`, `operator=<T>(repeater<T>)`, `operator=<Fun>(const fun_repeater<Fun>&)`
* `operator,<T>(const T&)`, `operator,<T>(repeater<T>)`, `operator,<Fun>(const fun_repeater<Fun>&)`
* `repeat<T>(std::size_t sz, T)`, `repeat_fun<Fun>(std::size_t sz, Fun)`
* `range<SinglePassIterator>(first, last)`, `range<SinglePassRange>(const SinglePassRange&)`

If `Argument` is not provided, the arguments for `operator()` are passed to `Function`,
otherwise the arguments are used to construct an `Argument`, which is then passed to `Function`.

#### Functions

* `make_list_inserter<Function>(Function fun) -> list_inserter<Function>`
* `make_list_inserter<Function, Argument>(Function fun, Argument*) -> list_inserter<Function, Argument>`
* `push_back<C>(C&) -> list_inserter<call_push_back<C>, C::value_type>`
* `push_front<C>(C&) -> list_inserter<call_push_front<C>, C::value_type>`
* `insert<C>(C&) -> list_inserter<call_insert<C>, C::value_type>`
* `push<C>(C&) -> list_inserter<call_push<C>, C::value_type>`
* `add_edge<C>(C&) -> list_inserter<call_add_edge<C>>` - for Boost.Graph and like
* `repeat<T>(std::size_t sz, T) -> repeater<T>`
* `repeat_fun<T>(std::size_t sz, Fun) -> fun_repeater<Fun>`

------
### `list_of` and friends - Temporary value holder for `list_inserter`

Header `<boost/assign/list_of.hpp>`

#### Internal class `generic_list<T>` and `static_generic_list<T, size_t N>`

`generic_list` internally store `T` values in a `deque`.
`static_generic_list` internally store references in a fixed size array.

These classes are convertable to container, adapter classes, and array types.
If the array type accept less elements than the list, throw `assignment_exception`.

#### Functions

* `list_of<T>() -> generic_list<T>` - make a default `T()` value.
* `list_of<T>(const T&) -> generic_list<T>`
* `ref_list_of<N, T>(T&) -> static_generic_list<decay<T>, N>`
* `cref_list_of<N, T>(const T&) -> static_generic_list<const decay<T>, N>`
* `list_of<T, Un ...>(const Un& ... u) -> generic_list<T>`
* `tuple_list_of<Un ...>(Un ... u) -> generic_list<tuple<U...>>`
* `map_list_of<Key,T>(const Key&, const T&) -> generic_list<std::pair<Key,T>>`
* `pair_list_of<Key,T>(const Key&, const T&) -> generic_list<std::pair<Key,T>>` - same as `map_list_of`

------
### STL Container `+=` Support

Header `<boost/assign/std.hpp>`

Operator `+=` with an argument:
* `deque` - `push_back`
* `list` - `push_back`
* `map` and `multimap` - `insert` (for `pair`s)
* `queue` and `priority_queue` - `push`
* `set` and `multiset` - `insert`
* `slist` - `push_back` (when `slist` is available)
* `stack` - `push`
* `vector` - `push_back`

Each one returns `list_inserter` instance for chaining invocation.

------
### Exception

------
### Boost.PtrContainer Inserters

Header `<boost/assign/ptr_list_inserter.hpp>`
Header `<boost/assign/ptr_map_inserter.hpp>`

* Class `ptr_list_inserter<Func,Obj>`
* Class `ptr_map_inserter<Func,Obj>`
* Function `make_ptr_list_inserter`
* Function `ptr_push_back`, `ptr_push_front`, `ptr_insert`, and `ptr_map_insert`

API similar to `list_inserter`, but create an `Obj` by `new` when call `Func`.

------
### Boost.PtrContainer oriented temporary list

Header `<boost/assign/ptr_list_of.hpp>`

#### Internal class `generic_ptr_list<T>`

`generic_list` internally store `T` pointers in a `ptr_vector`.

#### Functions

* `ptr_list_of<T>() -> generic_ptr_list<T>`
* `ptr_list_of<T, U...>(const U& ...) -> generic_ptr_list<T>`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.MPL

* `<boost/mpl/if.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_same.hpp>`
* `<boost/type_traits/*.hpp>`

#### Boost.Range

* `<boost/range/begin.hpp>`, `<boost/range/end.hpp>`
* `<boost/range/iterator_range.hpp>`

#### Boost.Tuple

* `<boost/tuple/tuple.hpp>` - to support `tuple` oriented `list_of`.

#### Boost.PtrContainer

* `<boost/ptr_container/ptr_vector.hpp>` - used for `ptr_list_of` internal storage.

#### Boost.Preprocessor

* `<boost/preprocessor/*.hpp>` - used for variadic template emulation.

------
### Standard Facilities

Language: initializer list syntax (C++11)
Standard Library: `<initializer_list>` (C++11)
