# Boost.FunctionTypes

* lib: `boost/libs/function_types`
* repo: `boostorg/function_types`
* commit: `6fba35a`, 2024-09-18

------
### Type classification

```c++
struct is_function<T,Tag=null_tag>;
struct is_function_pointer<T,Tag=null_tag>;
struct is_function_reference<T,Tag=null_tag>;
struct is_member_pointer<T,Tag=null_tag>;
struct is_member_object_pointer<T,Tag=null_tag>;
struct is_member_function_pointer<T,Tag=null_tag>;
struct is_callable_builtin<T,Tag=null_tag>;
struct is_nonmember_callable_builtin<T,Tag=null_tag>;
```

### Type decomposition

```c++
struct result_type<F>;
struct parameter_types<F,ClassTransform=add_reference<_>>;
struct function_arity<F>;
struct components<T,ClassTransform=add_reference<_>>;
```

### Type synthesis

```c++
struct function_type<Types,Tag=null_tag>;
struct function_pointer<Types,Tag=null_tag>;
struct function_reference<Types,Tag=null_tag>;
struct member_function_pointer<Types,Tag=null_tag>;
```

### Tag types

```c++
using detail::bits_t = long;
struct detail::constant<bits> : integral_constant<bits_t,bits>{};
struct detail::property_tag<bits_,mask_> { using bits=constant<bits_>; using mask=constant<mask_>; };
struct detail::bits<T> : T::bits{}; struct detail::mask<T> : T::mask {};
using detail::full_mask=constant<0x00ff0fff>;
struct detail::encode_bits_impl<flags,ccid> { constexpr static bits_t value = flags|(0x8000*ccid)<<1; };
struct detail::encode_charr_impl<flags,ccid,arity> { constexpr static size_t value = (size_t)(1+flags|(0x8000*ccid)<<1|arity<<24); };
struct detail::encode_bits<bits_,ccid> : constant<encode_bits_impl<bits_,ccid>::value>{};
struct detail::decode_bits<bits> {
    constexpr static bits_t flags=bits&0x0fff, cc_id=((bits&0x00ff0fff)/0x8000)>>1, tag_bits=(bits&0x00ff0fff);
    constexpr static size_t arity=(size_t)(bits>>24);
};
struct detail::tag_ice<lhs_bits,lhs_mask,rhs_bits,rhs_mask> {
    constexpr static bool match=rhs_bits==(lhs_bits&rhs_mask&(rhs_bits|~0xff));
    constexpr static bits_t combined_bits=(lhs_bits&~rhs_mask)|rhs_bits, combined_mask=lhs_mask|rhs_mask, extracted_bits=lhs_bits&rhs_mask;
};
struct detail::retag_default_cc<Tag,ReTag=Tag>;
struct detail::compound_tag<LHS,RHS> {
    using bits = constant<tag_ice<bits<LHS>::value,mask<LHS>::value,bits<RHS>::value,mask<RHS>::value>::combined_bits>;
    using mask = constant<tag_ice<bits<LHS>::value,mask<LHS>::value,bits<RHS>::value,mask<RHS>::value>::combined_mask>;
};
struct detail::changed_tag<Base,PropOld,PropNew> : Base { using bits=mpl::bitxor_<Base::bits,PropOld::bits,PropNew::bits>; };
struct detail::represents_impl<Tag,QueryTag> : integral_constant<bool, tag_ice<bits<Tag>::value,mask<Tag>::value,bits<QueryTag>::value,mask<QueryTag>::value>::match>{};

using null_tag=property_tag<0,0>;

struct tag<Tag1,Tag2,Tag3=null_tag,Tag4=null_tag> : compound_tag<compound_tag<Tag1,Tag2>, compound_tag<Tag3,Tag4>>{};
struct tag<Tag1,Tag2,Tag3,null_tag> : compound_tag<compound_tag<Tag1,Tag2>, Tag3>{};
struct tag<Tag1,Tag2,null_tag,null_tag> : compound_tag<Tag1,Tag2>{};
struct tag<Tag1,null_tag,null_tag,null_tag> : Tag1{};

struct represents<Tag,QueryTag> : represents_impl<Tag,retag_default_cc<QueryTag,Tag>>{};
struct extract<Tag,QueryTag> {
    using bits = constant<tag_ice<bits<Tag>::value,mask<Tag>::value,bits<QueryTag>::value,mask<QueryTag>::value>::extracted_bits>;
    using mask = constant<mask<QueryTag>::value>;
};
struct has_property_tag<Tag,PropertyTag> : represents_impl<Tag,PropertyTag>{};
struct has_variadic_property_tag : has_property_tag<Tag,variadic>{};
struct has_default_cc_property_tag : has_property_tag<Tag,default_cc>{};
struct has_const_property_tag : has_property_tag<Tag,const_qualified>{};
struct has_volatile_property_tag : has_property_tag<Tag,volatile_qualified>{};
struct has_cv_property_tag : has_property_tag<Tag,cv_qualified>{};
struct has_null_property_tag : has_property_tag<Tag,null_tag>{};

using variadic=property_tag<0x0100,0x0300>; using non_variadic=property_tag<0x00000200,0x00000300>;
using non_const=property_tag<0,0x0400>; using const_qualified=property_tag<0x0400,0x0400>;
using non_volatile=property_tag<0,0x0800>; using volatile_qualified=property_tag<0x0800,0x0800>;
using non_cv=property_tag<0,0x0c00>; using const_non_volatile=property_tag<0x0400,0x0c00>;
using volatile_non_const=property_tag<0x0800,0x0c00>; using cv_qualfied=property_tag<0x0c00,0x0c00>;
using default_cc=property_tag<0,0x00ff8000>;

using callable_builtin_tag=property_tag<0x01,0xff>;
using nonmember_callable_builtin_tag=property_tag<0x03,0xff>;
using function_tag=property_tag<0x07,0xff>;
using reference_tag=property_tag<0x13,0xff>;
using pointer_tag=property_tag<0x0b,0xff>;
using member_function_pointer_tag=property_tag<0x61,0xff>;
using member_object_pointer_tag=property_tag<0xa3,0xff>;
using member_pointer_tag=property_tag<0x20,0xff>;
using member_object_pointer_base=property_tag<0x02a3,0x00ff0fff>;
using nv_dcc_func=property_tag<0x8207,0x00ff83ff>; using nv_dcc_mfp=property_tag<0x8261,0x00ff83ff>;
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.Core

* `<boost/type.hpp>`

#### Boost.Detail

* `<boost/blank.hpp>`

#### Boost.MPL

* `<boost/mpl/*.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

------
### Standard Facilities
