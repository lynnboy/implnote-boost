# Boost.Coroutine2

* lib: `boost/libs/coroutine2`
* repo: `boostorg/coroutine2`
* commit: `b8f1065`, 2024-01-04

------
### Boost Asymmetric Coroutine (C++14)

Header `<boost/coroutine2/all.hpp>`

#### API

```c++
template <typename T>
struct coroutine {
  typedef pull_coroutine<T> pull_type;
  typedef push_coroutine<T> push_type;
};
```

#### Class `pull_coroutine<T>` and `push_coroutine<T>`

Wrappers around a `control_block<T>` pointer.

* `ctor([StackAllocator,] Fn &&)`
  Constructor for the master side, allocate and create `control_block`
  By default use `fixedsize_stack` allocator.
* `ctor(control_block*)`
  Used to create synthesized slave side.
* `dtor()` - call dtor of `control_block`
* Move-only type, transfer pointer of `control_block`
* `operator()` will cause context-switch
  `pull` iterator's `operator++` and `push` iterator's `operator=` will cause context-switch
* `pull` type provide `get() ->T`, `push` type's `operator()` take `T` argument
* `operator bool` and `operator!` check for completion of coroutines.

#### Context `control_block`s

* On destruction of master control block, the slave stack is forced to unwind
* When slave throw exception, it is propagated and thrown at master side.
* Pull side will switch to its push counterpart to try to fetch the first data element.
* Data item is stored in pull-side control block, which is transferred(returned) via
  context-switch from the push-side.
* Implementated based on Boost.Context `fiber`.

#### Specialization

* Specialization for `T&` is provided to transfer reference instead of value.
* Specialization for `void` is also provided, no iterator is provided in this case.

#### Stack Allocator

```c++
concept bool StackAllocator<T> = Constructible_v<T,std::size_t> && requires(T a, stack_context sctx) {
    { a.allocate() } -> stack_context;
    { a.deallocate(sctx) } -> void;
};

using fixedsize_stack = ...; using pooled_fixedsize_stack = ...; using protected_fixedsize_stack = ...;
using segmented_stack = ...;
```

* The stack allocators are used as optional argument for coroutine types to specify how to allocate stack frames.
* The predefined allocator types are aliases to the allocators defined in **Boost.Context**.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Context

* `<boost/context/stack_context.hpp>`
* `<boost/context/fixedsize_stack.hpp>`
* `<boost/context/pooled_fixedsize_stack.hpp>`
* `<boost/context/protected_fixedsize_stack.hpp>`
* `<boost/context/segmented_stack.hpp>`
* `<boost/context/detail/invoke.hpp>`
* `<boost/context/fiber.hpp>`

------
### Standard Facilities

Proposals:
  N4402 - Resumable Functions (revision 4)
  N4403 - Draft Wording for Resumable Functions
  P0057R00 - Wording for Coroutines (Revision 3)
