# Boost.Contract

* lib: `boost/libs/contract`
* repo: `boostorg/contract`
* commit: `4a4047f`, 2025-06-09

contract, contract_macro
base_types, call_if, check, constructor, destructor, function, old, override, public_function
core/: access, check_macro, constructor_precondition, exception, specify, virtual
detail/: assert, auto_ptr, check, checking, config, debug, decl, declspec, inlined, name, none, operator_safe_bool, static_local_var, tvariadic
detail/condition/: cond_base, cond_inv, cond_post, cond_subcontracting
detail/inlined/: core/exception, detail/checking, old
detail/operation/: constructor, destructor, function, public_function, static_public_function
detail/preprocessor/: utility/is, private, protected, public, virtual
detail/type_traits/: member_function_types, mirror, optional

------
### Asserts

* `ASSERT(cond)`, `ASSERT_AUDIT(cond)`, `ASSERT_AXIOM(cond)`

-----
### Configuration

* `DYN_LINK`, `STATIC_LINK`, `HEADER_ONLY`, `DISABLE_THREADS`
* `MAX_ARGS`: 10, `BASES_TYPEDEF`: `base_types`, `INVARIANT_FUNC`: `invariant`, `STATIC_INVARIANT_FUNC`: `static_invariant`
* `PERMISSIVE`, `ON_MISSING_CHECK_DECL`, `PRECONDITIONS_DISABLE_NO_ASSERTION`, `ALL_DISABLE_NO_ASSERTION`, `AUDITS`
* `NO_CHECKS`, `NO_PRECONDITIONS`, `NO_POSTCONDITIONS`, `NO_EXCEPTS`, `NO_ENTRY_INVARIANTS`, `NO_EXIT_INVARIANTS`, `NO_INVARIANTS`

-----
### Dependencies

#### Boost.Any

* `<boost/any.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/config/auto_link.hpp>`

#### Boost.Core

* `<boost/noncopyable.hpp>`
* `<boost/utility/enable_if.hpp>`

#### Boost.Exception

* `<boost/exception/diagnostic_information.hpp>`

#### Boost.Function

* `<boost/function.hpp>`

#### Boost.FunctionTypes

* `<boost/function_types/function_arity.hpp>`
* `<boost/function_types/parameter_types.hpp>`
* `<boost/function_types/property_tags.hpp>`
* `<boost/function_types/result_type.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.Optional

* `<boost/optional.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.SmartPtr

* `<boost/make_shared.hpp>`
* `<boost/shared_ptr.hpp>`
* `<boost/smart_ptr/allocate_unique.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Thread

* `<boost/thread/lock_guard.hpp>`
* `<boost/thread/mutex.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`
* `<boost/utility/declval.hpp>`

#### Boost.TypeOf

* `<boost/typeof/typeof.hpp>`

#### Boost.Utility

* `<boost/utility/result_of.hpp>`

------
### Standard Facilities
