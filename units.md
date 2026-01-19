# Boost.Units

* lib: `boost/libs/units`
* repo: `boostorg/units`
* commit: `605ebf7`, 2025-05-03

------
### Common Unit System

```c++
// dim, is_dim
struct detail::dim_tag{};
struct detail::get_tag<T>; // tag_type
struct detail::get_value<T>; // value_type
struct detail::is_empty_dim<T>; //dim<(0/1)>
struct dim<T,V> { using type=self; using tag=dim_tag; using tag_type=T; using value_type=V; };
struct is_dim<T>; // dim<T,V>

// dimensionless_type, dimension list
struct detail::dimension_list_tag{};
struct dimensionless_type{ using type=self; using tag=dimension_list_tag; using size=long_<0>; };
struct list<Item,Next> { using tag=dimension_list_tag; using type=self; using item=Item; using next=Next; using size=mpl::next<Next::size>::type; };
struct make_dimension_list<Seq>; // sort_dims<Seq>

struct is_dimension_list<Seq>; // true for list<I,N> and dimensionless_type;

struct is_dimensionless<T>; // default false_
struct is_dimensionless<unit<dimensionless_type,System>>; // true_
struct is_dimensionless<quantity<Unit,Y>> : is_dimensionless<Unit>{};
struct is_implicitly_convertible<S1,S2> : is_same<reduce_unit<S1>::type, reduce_unit<S2>::type>{};
struct get_dimension<T>{};
struct get_dimension<unit<Dim,System>>{using type=Dim;};
struct get_dimension<<Dim,System>>{using type=Dim;};
struct get_system<T>;

class absolute<Y>;

struct homogeneous_system<L> { using type=L; };
struct static_{power,root}<homogeneous_system<L>, static_rational<n,d>>; // homogeneous_system<L>

struct heterogeneous_system<T> : T{};
struct detail::make_heterogeneous_system<Dimensions,System>;
struct {multiply,divide}_systems<T0,T1>;
struct static_{power,root}<heterogeneous_system<S>,static_rational<n,d>>;

struct unit<Dim,System,Enable=void> {
    using unit_type=self; using this_type=self; using dimension_type=Dim; using system_type=System;
};
struct reduce_unit<U>{using type=U;};
struct reduce_unit<unit<Dim,System>>{using type=unit<Dim,make_heterogeneous_system<Dim,System>::type>; };
struct unary_{plus,minus}_typeof_helper<unit<D,S>>; // unit<D,S>
struct {add,subtract}_typeof_helper<unit<D,S>,unit<D,S>>; // unit<D,S>
struct {multiply,divide}_typeof_helper<unit<D1,homogeneous_system<S>>,unit<D2,{homo,hetero}geneous_system<S>>>;
    // unit<mpl::{times,divides}<D1,D2>::type,homogeneous_system<S>>
struct {multiply,divide}_typeof_helper<unit<D1,{homo,hetero}geneous_system<S1>>,unit<D2,{homo,hetero}geneous_system<S2>>>;
    // unit<mpl::{times,divides}<D1,D2>::type, (homo/hetero: 2x2) \
            {multiply,divide}_systems<{homogeneous_system<S1>,make_heterogeneous_system<D1,S1>::type}, \
                                      {homogeneous_system<S2>,make_heterogeneous_system<D2,S2>::type}>::type>
struct {power,root}_typeof_helper<unit<D,S>,static_rational<n,d>>;
    // unit <static_{power,root}<D,static_rational<n,d>>::type, static_{power,root}<S,static_rational<n,d>>::type>
unary_{plus,minus}_typeof_helper<unit<D,S>>::type operator{+,-} <D,S> (const unit<D,S>&);
unary_{add,subtract,multiply,divide}_typeof_helper<unit<D1,S1>,unit<D2,S2>>::type operator{+,-,*,/} <D1,S1,D2,S2> <D1,D2,S1,S2> (const unit<D1,S1>&, const unit<D2,S2>&);
bool operator{==,!=} <D1,D2,S1,S2> (const unit<D1,S1>&, const unit<D2,S2>&);

struct scale<base,Exponent> {
    static constexpr long base=base;
    using exponent=Exponent; using value_type=double;
    static constexpr value_type value();
};
struct scale<base,static_rational<0>> {
    static constexpr long base=base;
    using exponent=static_rational<0>; using value_type=one;
    static constexpr value_type value();
    static std::string name(), symbol(); // "", ""
};
// for (EXP,VAL,NAME,SYMBOL):
//  (-24,1e-24,yocto,y), (-21,1e-21,zepto,z), (-18,1e-18,atto,a), (-15,1e-15,femto,f), (-12,1e-12,pico,p),
//  (-9,1e-9,nano,n), (-6,1e-6,micro,u), (-3,1e-3,milli,m), (-2,1e-2,centi,c), (-1,1e-1,deci,d),
//  (1,1e1,deka,da), (2,1e2,hecto,h), (3,1e3,kilo,k), (6,1e6,mega,M), (9,1e9,giga,G),
//  (12,1e12,tera,T), (15,1e15,peta,P), (18,1e18,exa,E), (21,1e21,zetta,Z), (24,1e24,yotta,Y)
struct scale<10,static_rational<{EXP}>> {
    static constexpr long base=base;
    using exponent=static_rational<{EXP}>; using value_type=double;
    static constexpr value_type value(); // {VAL}
    static std::string name(), symbol(); // "{NAME}", "{SYMBOL}"
};
// for (EXP,VAL,NAME,SYMBOL):
//  (10,2^10,kibi,Ki), (20,2^20,mebi,Mi), (30,2^30,gibi,Gi), (40,2^40,tebi,Ti),
//  (50,2^50,pebi,Pi), (60,2^60,exbi,Ei), (70,2^70,zebi,Zi), (80,2^80,yobi,Yi)
struct scale<2,static_rational<{EXP}>> {
    static constexpr long base=base;
    using exponent=static_rational<{EXP}>; using value_type=double;
    static constexpr value_type value(); // {VAL}
    static std::string name(), symbol(); // "{NAME}", "{SYMBOL}"
};

struct unscale<T> {using type=T;};
struct unscale<scaled_base_unit<S,Scale>> {using type=unscale<S>::type;};
struct unscale<unit<D,S>>{using type=unit<D,unscale<S>::type>;};

struct get_scale_list<T>;

struct base_unit_info<BaseUnitTag>;
struct dimensionless_unit<System>;
struct is_unit<T>;
struct is_unit_of_dimension<T,Dim>;
struct is_unit_of_system<T,System>;

class quantity<Unit,Y=double>;

struct dimensionless_quantity<System,Y>;
struct is_quantity<T>;
struct is_quantity_of_dimension<T,Dim>;
struct is_quantity_of_system<T,System>;

struct conversion_helper<From,To>;

std::string to_string<T>(const T&);
std::string name_string<T>(const T&);
std::string symbol_string<T>(const T&);
std::string raw_string<T>(const T&);
std::string typename_string<T>(const T&);
```

#### Details

```c++
struct unary_{plus,minus}_typeof_helper<X>; // decltype(+declval<X>())
struct {add,subtract,multiply,divide}_typeof_helper<X>; // decltype(declval<X>()+declval<X>())
struct {power,root}_typeof_helper<X,Y> { using type = /*...*/; static constexpr type value(const X&); };

struct detail::static_rational_tag{};
using integer_type = long;
struct static_abs<v>;
struct static_rational<n,d=1> {
    using tag = static_rationale_tag;
    constexpr static integer_type Numerator=n/den, Denominator=d/den;
    using this_type = self; using type = self<Numerator,Denominator>;
    static constexpr integer_type numerator(), denominator();
};
constexpr divide_typeof_helper<T,T>::type value<T,n,d>(const static_rational<n,d>&) { return T(n)/T(d); }
struct static_power<DL,Ex>;
struct static_root<DL,Rt>;

struct detail::push_front_if<cond>::apply<L,T>; // cond ? list<T,L> : L
struct detail::push_front_or_add<Seq,T>;

struct detail::insertion_sort<T>;

// linear algebra
struct detail::eliminate_from_pair_of_equations<E1,E2>;
struct detail::substitute<Equation,Vars>;
struct detail::solve<T>;
struct detail::inconsistent{};
struct detail::divide_equation<n>;
struct detail::wrap<T>{};
struct detail::set_end;
struct detail::set<T,Next>;
struct detail::set_insert<hasKey>;
struct detail::has_key<Set,T>;
struct detail::find_base_dimensions<T>;
struct detail::begins_with_dimension<It>;
struct detail::expand_dimensions<n>;
struct detail::create_unit_matrix<n>;
struct detail::normalize_units<T>;
struct detail::multiply_add_units<n>;
struct detail::is_base_dimension_unit<T>;
struct detail::is_simple_system<T>;
struct detail::calculate_base_unit_exponents<T,Dimensions>;
```

------
### Configuration

* `BOOST_UNITS_REQUIRE_LAYOUT_COMPATIBILITY`
* `BOOST_UNITS_NO_COMPILER_CHECK`
* `BOOST_UNITS_CHECK_HOMOGENEOUS_UNITS`

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/no_tr1/cmath.hpp>`
* `<boost/version.hpp>`

#### Boost.Core

* `<boost/core/demangle.hpp>`
* `<boost/core/nvp.hpp>`
* `<boost/utility/enable_if.hpp>`

#### Boost.Integer

* `<boost/integer/common_factor_ct.hpp>`

#### Boost.IO

* `<boost/io/io_state.hpp>`

#### Boost.Lambda

* `<boost/lambda/lambda.hpp>`

#### Boost.Math/TR1

* `<boost/math/special_functions/fpclassify.hpp>`
* `<boost/math/special_functions/hypot.hpp>`
* `<boost/math/special_functions/next.hpp>`
* `<boost/math/special_functions/round.hpp>`
* `<boost/math/special_functions/sign.hpp>`

#### Boost.Move

#### Boost.MPL

* `<boost/mpl/*.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

#### Boost.TypeOf

* `<boost/typeof/typeof.hpp>`

------
### Standard Facilities
