# Boost.Function

* lib: `boost/libs/function`
* repo: `boostorg/function`
* commit: `dd047072`, 2017-01-07

------
### Polymorphic Function Wrapper

Header `<boost/function.hpp>`

#### API Compared With `std::function`

* Provide old-style `functionN` for old compilers.
* `bad_function_call` inherites from `std::runtime_error`, not `std::exception`
* Have non-template base `function_base`
* Provides `argN_type`, `arity`, and nested `sig` type (for Boost.Lambda)
* Provides `empty`, `contains`, `clear`

#### Implementation

```c++
union function_buffer {
  union function_buffer_members {
    void* obj_ptr;          // to store functor objects, mem_fn wrapped member function pointers
    struct { type_info* type; bool const_q; bool volatile_q; } type;
    void (*func_ptr)();     // to store function pointers
    struct { void (X::*memfunc_ptr)(int); void* obj_ptr; } bounded_memfun_ptr; // not used
    struct { void* obj_ptr; bool const_q; bool volatile_q; } obj_ref;  // store reference_wrapper, remember cv
  } members;
  char data[sizeof(members)];	    // used for small-obj-opt, store full functor object
};
enum functor_manager_operation_type { clone, move, destroy, check_functor_type, get_functor_type };
struct vtable_base {
  // handling of signature-netural tasks - everything other than the invocation.
  void (*manager)(const function_buffer& in, function_buffer& out, functor_manager_operation_type op);
};
struct function_base {
  vtable_base* vtable;    // nullptr means empty, use 0x01 as flag for trival copy and dtor for optimize
  function_buffer functor;
};

template<typename R, typename ... Ts> struct basic_vtable {
  vtable_base base;     // mimic inheritance
  invoker_type invoker; // dispatch to invocation code of each case (funcptr, ref, functor, memptr, smallobj)
};
template<typename R(Ts...)>
class function : public function_base {
  basic_vtable<R, Ts...>* get_vtable() { return reinterpret_cast<>(vtable);
};
```

* Optimize for small objects (which can be hold by a `functor_buffer`).
* Optimize for trivially_copyable and trivially_destructible.
* Different `manager` functions are used for cases:
  * function pointer
  * function object reference wrapped by `reference_wrapper`
  * member function pointer (which is wrapped by `mem_fn` as a function object)
  * function object
  * function object with specified `Allocator`
  * function object with small size

#### Cooperation With Other Boost Libraries

* For Boost.Lambda, `sig` nested template is provided.
* For Boost.TypeOf, type `bad_function_call` and templates `function`, `functionN` are registered.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Core

* `<boost/detail/no_exception_support.hpp>`
* `<boost/ref.hpp>`
* `<boost/utility/enable_if.hpp>`

#### Boost.Bind

* `<boost/mem_fn.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/*.hpp>`

#### Boost.Integer

* `<boost/integer.hpp>`

#### Boost.MPL

* `<boost/mpl/if.hpp>`, `<boost/mpl/bool.hpp>`

#### Boost.TypeIndex

* `<boost/type_index.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

#### Boost.Move

* `<boost/move/utility_core.hpp>` - for `forward`.

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeOf

* `<boost/typeof/typeof.hpp>` - for registering types.

------
### Standard Facilities

Standard Library: `function` (C++11).
