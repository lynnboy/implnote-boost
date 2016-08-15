# Boost.Detail

* lib: `boost/libs/detail`
* repo: `boostorg/detail`
* commit: `94f1e6b0`, 2016-08-07

------
### Blank Value Type

Header `<boost/blank.hpp>`

* Empty `struct blank`, declared as `is_pod`, `is_empty`, `is_stateless`.
* `==`, `<=`, `>=` always true, `!=`, `<`, `>` always false.
* Provided `<<` to ostream, do nothing.

* Used by **Boost.FunctionTypes** and **Boost.Variant**.

#### Dependency

##### Boost.Config

* `<boost/config.hpp>`

##### Boost.MPL

* `<boost/mpl/bool.hpp>`

##### Boost.TypeTraits

* `<boost/type_traits/is_empty.hpp>`, `<boost/type_traits/is_pod.hpp>`, `<boost/type_traits/is_stateless.hpp>`

------
### Predefined Return Status

Header `<boost/cstdlib.hpp>`

* Define `exit_success`, `exit_failure`, `exit_exception_failure`, `exit_test_failure`.

* Used by **Boost.Test** for the return values.

------
### Allocator Workaround

Header `<boost/detail/allocator_utilities.hpp>`

* Used by **Boost.Flyweight**, **Boost.MultiIndex**, **Boost.StateChart**.

#### Dependency

##### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

##### Boost.MPL

* `<boost/mpl/eval_if.hpp>`

##### Boost.TypeTraits

* `<boost/type_traits/is_same.hpp>`

------
### Binary Search Algorithms

Header `<boost/detail/binary_search.hpp>`

* Implement `lower_bound`, `upper_bound`, `equal_range`, and `binary_search`.
* The same as STL version.

* Used by **Boost.Python** and **Boost.Test**

#### Dependency

##### Boost.Core

* `<boost/detail/iterator.hpp>` - deprecated, just introduces STL version.

------
### Bitmask Macro

Header `<boost/detail/bitmask.hpp>`

* Macro `BOOST_BITMASK(Bitmask)`, Add `|`, `&`, `^`, `~`, `&=`, `|=`, and `^=` for existing enum type.
* The same as STL version.

* Used by **Boost.FileSystem**.

#### Dependency

##### Boost.Core

* `<boost/cstdint.hpp>` - for `int_least32_t`.

------
### Unhandled Exception Catching

Header `<boost/detail/catch_exceptions.hpp>`

* Dump every exception thrown by specified functor to an ostream and return status code.
* None usage.

------
### Container Forward Declaration

Header `<boost/detail/container_fwd.hpp>`

* Try to forward declare known standard container types, or include the STL headers

* Used by **Boost.Functional/Hash**, **Boost.Lambda**, and **Boost.Phoenix**

#### Dependency

##### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

------
### `<cfenv>`

Header `<boost/detail/fenv.hpp>`

* Same as `<cfenv>` of standard library (C++11).

* Used by **Boost.Math**

#### Dependency

##### Boost.Config

* `<boost/config.hpp>`

------
### `has_default_constructor`

Header `<boost/detail/has_default_constructor.hpp>`

* No usage.

------
### Identifier

Header `<boost/detail/identifier.hpp>`

* A class template `identifier` used tobe base class to distinguish from each other.

* No usage.

#### Dependency

##### Boost.Core

* `<boost/utility/enable_if.hpp>`

##### Boost.TypeTraits

* `<boost/type_traits/is_base_of.hpp>`

------
### Type Traits For References And Pointers

Header `<boost/detail/indirect_traits.hpp>`

* `is_reference_to_const`, `is_reference_to_non_const`, `is_reference_to_volatile`
* `is_reference_to_pointer`, `is_reference_to_class`, `is_pointer_to_class`
* `is_reference_to_function`, `is_pointer_to_function`
* `is_reference_to_member_function_pointer`, `is_reference_to_function_pointer`

* Used by **Boost.Iterator** and **Boost.Python**

#### Dependency

##### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

##### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

##### Boost.MPL

* `<boost/mpl/*.hpp>`

------
### Trait `is_incrementable`

Header `<boost/detail/is_incrementable.hpp>`

* Define `is_incrementable` and `is_postfix_incrementable`.

* Used by **Boost.ICL**, **Boost.IOStreams**, and **Boost.Iterator**

#### Dependency

##### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

##### Boost.TypeTraits

* `<boost/type_traits/integral_constant.hpp>`
* `<boost/type_traits/remove_cv.hpp>`

##### Boost.MPL

* `<boost/mpl/bool.hpp>`
* `<boost/mpl/aux_/lambda_support.hpp>`

------
### Algorithm `is_sorted`

Header `<boost/detail/is_sorted.hpp>`

* Define `is_sorted` and `is_sorted_until` algorithms.
* Same as STL version (C++11).

* Used by **Boost.Graph.Parallel** and **Boost.Range**

#### Dependency

##### Boost.Core

* `<boost/detail/iterator.hpp>` - deprecated, just introduce standard version.

------
### Template Specialization Detection Trait Making Macro

Header `<boost/detail/is_xxx.hpp>`

* Macro `BOOST_DETAIL_IS_XXX_DEF(name, qualified_name, nargs)`
* Generate trait `is_<name>`, detect a type is a specialization of
  template `<qualified_name>` with `nargs` type parameters.

* Used by **Boost.Parameter** and **Boost.Python**

#### Dependency

##### Boost.Config

* `<boost/config.hpp>`

##### Boost.MPL

* `<boost/mpl/bool.hpp>`

##### Boost.Preprocessor

* `<boost/preprocessor/enum_params.hpp>`

------
### Lightweight Test Facilities

Header `<boost/detail/lightweight_main.hpp>` and `<boost/detail/lightweight_test_report.hpp>`

* A `main` function calling `cpp_main` and catching all `std::exception`.
* A `cpp_main` function dumping brief BOOST version and command line, then call `test_main`.
* Either one may be included by a C++ source file for testing purpose.

#### Dependency

##### Boost.Config

* `<boost/config.hpp>`
* `<boost/version.hpp>`

##### Boost.Core

* `<boost/detail/lightweight_test.hpp>`

------
### Facility For Named Template Parameter

Header `<boost/detail/named_template_params.hpp>`

* Macro `BOOST_NAMED_TEMPLATE_PARAM(TYPE)`, makes `get_<TYPE>`

* No usage.

#### Dependency

##### Boost.TypeTraits

* `<boost/type_traits/conversion_traits.hpp>`
* `<boost/type_traits/composite_traits.hpp>`

------
### Numeric Type Traits

Header `<boost/detail/numeric_traits.hpp>`

* Traits `is_signed`, `digit_traits`, `integer_traits`, `numeric_traits`
* Algorithm `numeric_distance(Number, Number)`

* Used by **Boost.Graph** and **Boost.Iterator**

#### Dependency

##### Boost.Config

* `<boost/config.hpp>`
* `<boost/cstdint.hpp>`
* `<boost/limits.hpp>`

##### Boost.StaticAssert

* `<boost/static_assert.hpp>`

##### Boost.TypeTraits

* `<boost/type_traits.hpp>`

------
### Reference Data Storage

Header `<boost/detail/reference_content.hpp>`

* Class `reference_content`, wraps a reference, disallow assignment.
* Generator trait `make_reference_content`, just get argument type for non-reference.

* Used by **Boost.Optional** and **Boost.Variant**

#### Dependency

##### Boost.Config

* `<boost/config.hpp>`

##### Boost.MPL

* `<boost/mpl/bool.hpp>`
* `<boost/mpl/void.hpp>`

##### Boost.TypeTraits

* `<boost/type_traits/has_nothrow_copy.hpp>`

------
### Conditionally Select Type

Header `<boost/detail/select_type.hpp>`

* Meta function `if_true<bool>::template then<T,F>::type`

* Used by **Boost.Unordered**, and **Boost.Detail/numeric_traits**

------
### Adaptor Macros For Old And Templated IOStream Classes

Header `<boost/detail/templated_streams.hpp>`

* A set of macros to bridge old and standard `iostream` classes.

* Used by **Boost.Flyweight**, and **Boost.Variant**, and **Boost.Detail/blank**

------
### UTF-8 Conversion Facet

Header `<boost/detail/utf8_codecvt_facet.hpp>`.
Include `<boost/detail/utf8_codecvt_facet.ipp>` in source

* Similar to standard `codecvt_utf8<wchar_t>` facet.

* Used by **Boost.FileSystem**, and **Boost.Program_Options**, and **Boost.Serialization**

#### Dependency

##### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`

------
### Standard Facilities

* Standard Library:
  * `binary_search` etc.
  * `is_sorted`, etc. (C++11)
  * `<cfenv>`
  * `codecvt_utf8` (C++11)
