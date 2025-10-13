# Boost.TTI - Type Trait Introspection

* lib: `boost/libs/tti`
* repo: `boostorg/tti`
* commit: `da15e74`, 2025-05-03

------
### Type Trait Introspection

Header `<boost/tti.hpp>`

Each macro have a trait-prenamed version `BOOST_TTI_TRAIT_HAS_XXX(trait,name,...)`, and a trait name generate macro `BOOST_TTI_HAS_XXX_GEN(name)`.

For template traits, another usage is allowed:
- use `HAS_XX_TEMPLATE(name, BOOST_PP_NIL)` is same as `HAS_XX_TEMPLATE(name)`
- use `HAS_XX_TEMPLATE(name, (4,class,int,class<int,class>))` is same as variadic `HAS_XX_TEMPLATE(name, class,int,class<int,class>)`

* `BOOST_TTI_HAS_TYPE(name)` : `struct has_type_'name'<T,Pred=<T>{true}>;`
* `BOOST_TTI_HAS_CLASS(name)` : `struct has_class_'name'<T,Pred=<T>{true}>;`
* `BOOST_TTI_HAS_ENUM(name)` : `struct has_enum_'name'<T,Pred=<T>{true}>;`
* `BOOST_TTI_HAS_UNION(name)` : `struct has_union_'name'<T,Pred=<T>{true}>;`
  * By default, find member type/class/enum/union named `name` in `T`
  * `Pred` is a MPL lambda to check acceptable found type.

* `BOOST_TTI_HAS_TEMPLATE(name)` : `struct has_template_'name'<T>`
  * Find member class template named `name` in `T`, all template parameters are `class`/`typename`.
* `BOOST_TTI_HAS_TEMPLATE(name,...temp-param-list)` : `struct has_template_'name'<T>`
  * Find member class template named `name` in `T`, with specified template parameter list.

* `BOOST_TTI_HAS_MEMBER_DATA(name)` : `struct has_member_data_'name'<T_or_MPtr, Type=void>`
  * Find non-static member data with name `name` and type `Type`.
  * Can provide member pointer type (`Type (T::*)`) as `T_or_MPtr`, then `Type` should be omitted.
* `BOOST_TTI_HAS_STATIC_MEMBER_DATA(name)` : `struct has_static_member_data_'name'<T, Type>`
  * Find static data member with name `name` and type `Type` in `T`.
* `BOOST_TTI_HAS_DATA(name)` : `struct has_data_'name'<T,Type>;`
  * Find non-static/static data member with name `name` and type `Type` in `T`.

* `BOOST_TTI_HAS_MEMBER_FUNCTION(name)` : `struct has_member_function_'name'<T_or_Sig, R=deftype, Fs=mpl::vector<>, Tag=function_types::null_tag>;`
  * Find non-static member function with specified signature within `T`.
  * Can provide member function pointer type (`(R (T::*)(Fs...) Tag)`) as `T_or_Sig`, then other parameters should be omitted.
* `BOOST_TTI_HAS_STATIC_MEMBER_FUNCTION(name)` : `struct has_static_member_function_'name'<T, R=deftype, Fs=mpl::vector<>, Tag=function_types::null_tag>`
  * Find static member function with specified signature within `T`.
* `BOOST_TTI_HAS_FUNCTION(name)` : `struct has_function_'name'<T, R=deftype, Fs=mpl::vector<>, Tag=function_types::null_tag>`
  * Find any member function with specified signature within `T`.

* `BOOST_TTI_HAS_MEMBER_FUNCTION_TEMPLATE(name,...temp-arg-list)` : `struct has_member_function_template_'name'<T_or_Sig,R,FS,Tag>;`
* `BOOST_TTI_HAS_STATIC_MEMBER_FUNCTION_TEMPLATE(name,...temp-arg-list)` : `struct has_static_member_function_template_'name'<T,R,FS,Tag>;`
* `BOOST_TTI_HAS_FUNCTION_TEMPLATE(name,...temp-arg-list)` : `struct has_function_template_'name'<T,R,FS,Tag>;`
  * Find static/non-static/any member function template named `name`, instanted with `temp-arg-list`, within `T`.
  * Can provide member function pointer type (`(R (T::*)(Fs...) Tag)`) as `T_or_Sig`, then other parameters should be omitted.


#### `BOOST_TTI_MEMBER_TYPE(name)`

```c++
using member_type_'name'<T,marker=notype>::type = has_type_'name'<T> ? decltype(T::name) : marker;
```

* `marker` defined as nested `boost_tti_marker_type`.

Helper traits:

* `valid_member_metafunction<T> : (! is_same<T::type,T::boost_tti_marker_type>)`
* `valid_member_type<T,marker=notype> : (! is_same<T,marker>)`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/remove_const.hpp>`
* `<boost/type_traits/is_class.hpp>`, `<boost/type_traits/is_same.hpp>`
* `<boost/type_traits/detail/yes_no_type.hpp>`

#### Boost.MPL

* `<boost/mpl/*.hpp>`

#### Boost.FunctionTypes

* `<boost/function_types/parameter_types.hpp>`
* `<boost/function_types/is_function.hpp>`
* `<boost/function_types/member_function_pointer.hpp>`
* `<boost/function_types/property_tags.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/*.hpp>`

------
### Standard Facilities

* Proposals:
  * N4428 - Type Property Queries (rev 4)
  * N4447 - From a type T, gather members named and type information, via variadic template expansion
  * N4451 - Static reflection
