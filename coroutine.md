# Boost.Coroutine2

* lib: `boost/libs/coroutine`
* repo: `boostorg/coroutine`
* commit: `8b09bf7`, 2024-08-28

------
### Boost Asymmetric Coroutine

Header `<boost/coroutine/all.hpp>`

#### API

```c++
template <typename T>
struct asymmetric_coroutine {
  typedef pull_coroutine<T> pull_type;
  typedef push_coroutine<T> push_type;
};

template< typename T >
struct symmetric_coroutine {
    typedef detail::symmetric_coroutine_call<T>  call_type;
    typedef detail::symmetric_coroutine_yield<T> yield_type;
};
```

#### Class `push_coroutine<Arg>`

* Constructor: `push_coroutine(Fn&&, [attributes const&], [StackAllocator])`
* PImpl idiom, moveable, not copyable, `swap`
* `explicit operator bool`, `!` => impl's `is_complete()`
* `operator()(Arg)` push arg. Member class `iterator` call `push(arg)` on `operator=`.
* Specialize for `Arg&` and `void`.

#### Class `pull_coroutine<R>`

* Constructor: `pull_coroutine(Fn&&, [attributes const&], [StackAllocator])`
* PImpl idiom, moveable, not copyable, `swap`
* `explicit operator bool`, `!` => impl's `is_complete()`
* `operator()(Arg)` push arg. `get` return `R` result value from context. Member class `iterator` call `pull()` on `operator++`, then cache ptr to result value.
* Specialize for `R&` and `void`.

#### Stack Context

* store stack `size`, pointer `sp`, If support segmented stack, store `segments_ctx`
* put an object wrapping pull/push context and the impl object into stack.

#### Stack Allocator

```c++
struct segmented_stack_allocator;
struct standard_stack_allocator;

using stack_allocator = ...
```

* Whether use segmented stack (`BOOST_USE_SEGMENTED_STACKS` macro switch).

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/cstdint.hpp>`
* `<boost/config/auto_link.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Exception

* `<boost/exception_ptr.hpp>`

#### Boost.Move

* `<boost/move/move.hpp>`

#### Boost.Core

* `<boost/utility/explicit_operator_bool.hpp>`
* `<boost/utility/enable_if.hpp>`
* `<boost/core/scoped_enum.hpp>`

#### Boost.System

* `<boost/system/error_code.hpp>`
* `<boost/system/system_error.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/integral_constant.hpp>`
* `<boost/type_traits/decay.hpp>`
* `<boost/type_traits/is_convertible.hpp>`
* `<boost/type_traits/is_same.hpp>`

#### Boost.Context

* `<boost/context/detail/config.hpp>`
* `<boost/context/detail/fcontext.hpp>`

### Boost.Utility

* `<boost/utility.hpp>`

------
### Standard Facilities

Proposals:
  N4402 - Resumable Functions (revision 4)
  N4403 - Draft Wording for Resumable Functions
  P0057R00 - Wording for Coroutines (Revision 3)
