# Boost.TTI

* lib: `boost/libs/tti`
* repo: `boostorg/tti`
* commit: `6c82633c`, 2015-01-15

------
### Type Trait Introspection

Header `<boost/tti.hpp>`

* `BOOST_TTI_HAS_TYPE(name)`: `has_type_[name]<T,Pred=[T]{true;}>`
  * By default, find member type named `name` in `T`
  * `Pred` is a mpl lambda to check acceptable found type.
* `BOOST_TTI_HAS_TEMPLATE(name, ...)`: `has_template_[name]<T>`
  * By default, template should have all type-parameters, signature of template can be specified
  * In variadic variant, sample is `HAS_TEMPLATE(template_name)`
    or `HAS_TEMPLATE(template_name, class, int, template<class>class)`
  * In non-variadic variant, sample is `HAS_TEMPLATE(template_name, BOOST_PP_NIL)` for default all-type
    while `HAS_TEMPLATE(template_name, (3, (class, int, template<class>class)))` for signature
* `BOOST_TTI_HAS_MEMBER_DATA(name)`:
  * `has_member_data_[name]<T, Type>`
  * `has_member_data_[name]<Type (T::*)>`
* `BOOST_TTI_HAS_MEMBER_FUNCTION(name)`:
  * `has_member_function_[name]<T, R=deftype, mpl::vector<Args...>=vector<>, Tag=function_types::null_tag>`
  * `has_member_function_[name]<R (T::*)(Args...) [const] [volatile]>`
  * The first version can omit `R` when not caring the return type.
* `BOOST_TTI_HAS_STATIC_MEMBER_DATA(name)`: `has_static_member_data_[name]<T, Type>`
* `BOOST_TTI_HAS_STATIC_MEMBER_FUNCTION(name)`:
  * `has_static_member_function_[name]<T, R=deftype, mpl::vector<Args...>=vector<>, Tag=function_types::null_tag>`
* `BOOST_TTI_HAS_DATA(name)`: `has_data_[name]<T, Type>`, either non-static or static
* `BOOST_TTI_HAS_FUNCTION(name)`: either non-static or static
  * `has_function_[name]<T, R=deftype, mpl::vector<Args...>=vector<>, Tag=function_types::null_tag>`

* `BOOST_TTI_MEMBER_TYPE(name)`:
  * `member_type_[name]<T,marker=notype> : has_member_type_[name]<T> ? decltype(T::name) : notype`.
    * `marker` defined as nested `boost_tti_marker_type`.
  * Helper trait `valid_member_metafunction<T> : (! is_same<T::type,T::boost_tti_marker_type>)`.
  * Helper trait `valid_member_type<T,marker=notype> : (! is_same<T,marker>)`.

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
