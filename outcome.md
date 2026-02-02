# Boost.Outcome

* lib: `boost/libs/outcome`
* repo: `boostorg/outcome`
* commit: `726661e`, 2025-05-21

------
### Macros

* Constrained template macros
  * `TEMPLATE(template args ...)`
  * `TREQUIRES(requirements ...)`
  * `TEXPR(expression)`
  * `TPRED(boolean)`
* Version macros
  * `VERSION_MAJOR`, `VERISON_MINOR`, `VERISON_PATCH`, `VERSION_REVISION`, `UNSTABLE_VERSION`, `V2`, `V2_NAMESPACE`
* `CO_TRY(var, expr)`, `CO_TRYV(expr)`, `CO_TRY(expr)`, `CO_TRYV2(spec,expr)`,
    `CO_TRYV2_FAILURE_LIKELY(spec, expr)`, `CO_TRYV_FAILURE_LIKELY(expr)`, `CO_TRY_FAILURE_LIKELY(expr)`,
    `CO_TRYX(expr)`, `CO_TRYX_FAILURE_LIKELY(expr)`, `CO_TRY_FAILURE_LIKELY(var, expr)`
* `DISABLE_EXECINFO`, `ENABLE_LEGACY_SUPPORT_FOR`, `NODISCARD`, `REQUIRES(...)`
* `SYMBOL_VISIBLE`, `THREAD_LOCAL`, `THROW_EXCEPTION(expr)`
* `TRY(var, expr)`, `TRYV(expr)`, `TRY(expr)`, `TRYV2(spec, expr)`, `TRYV2_FAILURE_LIKELY(spec, expr)`,
    `TRYV_FAILURE_LIKELY(expr)`, `TRY_FAILURE_LIKELY(expr)`,
    `TRYX(expr)`, `TRYX_FAILURE_LIKELY(expr)`, `TRY_FAILURE_LIKELY(var, expr)`
* `USE_STD_IN_PLACE_TYPE`, `USE_STD_IS_NOTHROW_SWAPPABLE`

------
### Concepts

```c++
concept basic_outcome<T>;
concept basic_result<T>;
concept value_or_error<T>;
concept value_or_none<T>;
```

------
### Converters

```c++
struct value_or_error<T, U>
{
  static constexpr bool enable_result_inputs = false, enable_outcome_inputs = false;
  template<class X> constexpr T operator()(X &&v);
};
```

------
### Traits

```c++
struct is_basic_outcome<T>;
struct is_basic_result<T>;
struct is_error_code_available<T>;
struct is_error_type<E>;
struct is_error_type_enum<E, Enum>;
struct is_exception_ptr_available<T>;
struct is_failure_type<T>;
struct is_move_bitcopying<T>;
struct is_success_type<T>;
struct type_can_be_used_in_basic_result<R>;
```

bad_access, basic_outcome, basic_result, boost_outcome, boost_result, convert, 
iostream_support_<result>, outcome_gdb, outcome, result, std_outcome, std_result, success_failure, trait, try, utils
detail/: basic_outcome_{exception_observers_<impl>,failure_observers}, basic_result_{error,value}_observers,
	basic_result_{final,storage}, revision, trait_std_{error_code,exception}, try, value_storage
policy/: all_narrow, base, fail_to_compile_observers, {outcome,result}_error_code_throw_as_system_error,	
	{outcome,result}_exception_ptr_rethrow, terminate, throw_bad_result_access

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/version.hpp>`

#### Boost.Exception

* `<boost/exception_ptr.hpp>`

#### Boost.System

x* `<boost/system/error_code.hpp>`
x* `<boost/system/system_error.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

------
### Standard Facilities
