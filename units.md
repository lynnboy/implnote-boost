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
struct get_dimension<absolute<Unit>>{using type=get_dimension<Unit>::type;};
struct get_system<quantity<Unit,Y>>{using type=get_dimension<Unit>::type;};
struct get_system<T>{};
struct get_system<unit<Dim,System>>{using type=System;};
struct get_system<absolute<Unit>>{using type=get_system<Unit>::type;};
struct get_system<quantity<Unit,Y>>{using type=get_system<Unit>::type;};

struct base_dimension_ordinal<n>{}; struct base_dimension_pair<T,n>{};
struct base_dimension<Derived,n> : public ordinal<n> {
    using dimension_type = list<dim<Derived,static_rational<1>>,dimensionless_type>; using type=Derived;
};
struct base_unit_ordinal<n>{}; struct base_unit_pair<T,n>{};
struct base_unit<Derived,Dim,n> : public ordinal<n> {
    using boost_units_is_base_unit_type=void; using dimension_type=Dim; using type=Derived;
    using unit_type = unit<Dim,heterogeneous_system<...>>;
};

struct derived_dimension<D1,...D,n1,...n> { using type=make_dimension_list<list<dim<D1,static_rational<1>>,derived_dimension<D...,n...>::type>>; };

struct absolute<Y> { // constexpr
    using this_type=self; using value_type=Y;
    ctor(); ctor(const value_type& val); ctor(const self& source); self& operator=(const self& source);
    const value_type& value() const;
    const this_type& operator{+=,-=}(const value_type& val);
private: value_type val_;
};
absolute<Y> operator+ <Y> (const absolute<Y>& aval, const Y& rval); // and inversed
absolute<Y> operator- <Y> (const absolute<Y>& aval, const Y& rval);
Y operator- <Y> (const absolute<Y>& aval1, const absolute<Y>& aval2);
quantity<absolute<unit<D,S>>,T> operator* <D,S,T> (const absolute<unit<D,S>>&, const T& t); // and inversed
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,Y> (std::basic_ostream<Ch,Tr>& os, const absolute<Y>& aval);

struct homogeneous_system<L> { using type=L; };
struct static_{power,root}<homogeneous_system<L>, static_rational<n,d>>; // homogeneous_system<L>

struct heterogeneous_system<T> : T{};
struct detail::make_heterogeneous_system<Dimensions,System>;
struct {multiply,divide}_systems<T0,T1>;
struct static_{power,root}<heterogeneous_system<S>,static_rational<n,d>>;
using heterogeneous_dimensionless_system = heterogeneous_system<heterogeneous_system_impl<dimensionless_type,dimensionless_type,dimensionless_type>>;
struct is_dimensionless_system<System>; //

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

struct scaled_base_unit_tag{};
struct scaled_base_unit<S,Scale> {
    using boost_units_is_base_unit_type=void; using type = self; using tag = scaled_base_unit_tag;
    using system_type = S; using scale_type = Scale;
    using dimension_type = S::dimension_type;
    using unit_type = unit<dimension_type, heterogeneous_system<...>>
    static std::string symbol(), name();
};

struct make_scaled_unit<Unit,Scale> {using type=make_scaled_unit<reduce_unit<Unit>::type,Scale>::type; };
struct make_scaled_unit<unit<Dim,heterogeneous_system<...>>,Scale> {using type=...;};
struct make_scaled_unit<unit<Dim,heterogeneous_system<...>>,scale<Base,static_rational<0>>> {using type=...;};

struct dimensionless_unit<System> {using type=unit<dimensionless_type,System>;};
struct is_unit<T>; // unit<D,S>
struct is_unit_of_dimension<T,Dim>; // unit<D,S>, absolute<unit<D,S>>
struct is_unit_of_system<T,System>; // unit<D,S>, absolute<unit<D,S>>
struct is_dimensionless_unit<T> : public is_unit_of_dimension<T,dimensionless_type>{};

// and lambda support for unit and quantity

struct detail::is_bas_unit<T>;
struct detail::is_non_narrowing_conversion<Source,Dest>;
struct quantity<Unit,Y=double> {
    using value_type=Y; using unit_type = U;
    ctor(); ctor(nullptr_t); ctor(const self& src); self& operator=(const self& src);
    ctor<YY>(const self<Unit,YY>& src); self& operator= <YY>(const self<Unit,YY>& src>);
    explicit ctor<Unit2,YY>(const self<Unit2,YY>& src); self& operator= <Unit2,YY>(const self<Unit2,YY>& src);

    const value_type& value() const;

    self& operator{+=,-=,*=,/=}<Unit2,YY>(const self<Unit2,YY>& src);
    self& operator{*=,/=}(const value_type& src);
    static self from_value(const value_type& val);
protected: explicit ctor(const value_type& val, int);
private: value_type val_;
};
struct quantity<unit<dimensionless_type, System>,Y>{
    using value_type=Y; using system_type=System; using dimension_type=dimensionless_type; using unit_type = unit<dimension_type,system_type>;
    ctor(); ctor(value_type val); ctor(const self& src); self& operator=(const self& src);
    ctor<YY>(const self<unit_type,YY>& src); self& operator= <YY>(const self<unit_type,YY)& src>);
    explicit ctor<System2,Y2>(const self<unit<dimensionless_type,System2>,Y2>& src);
    self& operator= <System2>(const self<unit<dimensionless_type,System2>,Y>& src);

    operator value_type() const; const value_type& value() const;

    self& operator{+=,-=}(const self& src);
    self& operator{*=,/=}(const value_type& val);
    static self from_value(const value_type& val);
private: value_type val_;
};
struct quantity<unit<int,System>,T>{};
class quantity<int,T>{};
constexpr X quantity_cast<X,Y>(<const> Y& source);
void swap<Unit,Y>(quantity<Unit,Y>& lhs, quantity<Unit,Y>& rhs);

struct unary_{plus,minus}_typeof_helper<quantity<Unit,Y>> {using type=quantity<...>;};
struct {add,subtract}_typeof_helper<quantity<Unit1,X>,quantity<Unit2,Y>>;
struct {add,subtract}_typeof_helper<quantity<unit<D1,S1>,X>,quantity<unit<D2,S2>,Y>>;
struct {add,subtract}_typeof_helper<quantity<unit<D,S>,X>,quantity<unit<D,S>,Y>>;
struct {multiply,divide}_typeof_helper<X,unit<D,S>>; struct {multiply,divide}_typeof_helper<unit<D,S>,X>;
struct {multiply,divide}_typeof_helper<X,quantity<U,Y>>; struct {multiply,divide}_typeof_helper<quantity<U,X>,Y>;
struct {multiply,divide}_typeof_helper<one,quantity<U,Y>>; struct {multiply,divide}_typeof_helper<quantity<U,X>,one>;
struct {multiply,divide}_typeof_helper<unit<D,S>,quantity<U,X>>; struct {multiply,divide}_typeof_helper<quantity<U,X>,unit<D,S>>;
struct {multiply,divide}_typeof_helper<quantity<U1,X>,quantity<U2,Y>>;
struct {power,root}_typeof_helper<quantity<U,Y>,static_rational<n,d>>;
{multiply,divide}_typeof_helper<unit<D,S>,Y>::type operator{*,/} <S,D,Y> (const unit<D,S>& const Y& rhs); // and inversed
{multiply,divide}_typeof_helper<quantity<U,X>,X>::type operator{*,/} <U,X> (const quantity<U,X>& const X& rhs); // and inversed
{multiply,divide}_typeof_helper<unit<D1,S1>,quantity<U2,Y>>::type operator{*,/} <D1,S1,U2,Y> (const unit<D1,S1>& const quantity<U2,Y>& rhs); // and inversed
unary_{plus,minus}_typeof_helper<quantity<U,Y>>::type operator{+,-} <U,Y>(const quantity<U,Y>& val);
{add,subtract,multiply,divide}_typeof_helper<quantity<U1,X>,quantity<U2,Y>>::type operator{+,-,*,/} <U1,U2,X,Y> (const quantity<U1,X>& lhs, const quantity<U2,Y>& rhs);
bool operator {==,!=,<,<=,>,>=} <U,X,Y> (const quantity<U,X>& val1, const quantity<U,Y>& val2);

struct std::numeric_limits<quantity<U,T>>;

struct dimensionless_quantity<System,Y>{using type=quantity<dimensionless_unit<System>::type,Y>;};
struct is_quantity<T>; // quantity<U,Y>
struct is_quantity_of_dimension<T,Dim>; // quantity<Unit,Y>: is_unit_of_dimension<U,Dim>
struct is_quantity_of_system<T,System>; // quantity<Unit,Y>: is_unit_of_system<U,Dim>
struct is_dimensionless_quantity<T> : is_quantity_of_dimension<T,dimensionless_type>{};

bool is{finite,inf,nan,normal}<U,Y>(const quantity<U,Y>& q);
bool isgreater<U,Y>(const quantity<U,Y>& q1, const quantity<U,Y>& q2);
bool is{greater,less}<equal> <U,Y> (const quantity<U,Y>& q1, const quantity<U,Y>& q2);
bool isunordered <U,Y> (const quantity<U,Y>& q1, const quantity<U,Y>& q2);
quantity<U,Y> {abs,ceil,fabs,floor,round,trunc}<U,Y>(const quantity<U,Y>& q);
quantity<U,Y> {copysign,fdim,fmax,fmin,nextafter,nexttoward,fmod} <U,Y> (const quantity<U,Y>& q1, const quantity<U,Y>& q2);
int {fpclassify,signbit} <U,Y>(const quantity<U,Y>& q);
root<add<power<q,sr<2>>,power<q,sr<2>>>,sr<2>>::type hypot <U,Y> (const quantity<U,Y>& q1, const quantity<U,Y>& q2);
quantity<U,Y> modf<U,Y> (const quantity<U,Y>& q1, quantity<U,Y>* q2);
quantity<U,Y> frexp<U,Y,Int> (const quantity<U,Y>& q1, Int* ex);
quantity<U,Y> ldexp<U,Y,Int> (const quantity<U,Y>& q1, const Int& ex);
quantity<unit<dimensionless_type,S>,Y> pow<S,Y>(const quantity<unit<dimensionless_type,S>,Y>& q1, const quantity<unit<dimensionless_type,S>,Y>& q2);
quantity<unit<dimensionless_type,S>,Y> {exp,log,log10}<S,Y>(const quantity<unit<dimensionless_type,S>,Y>& q);
root<q<U,Y>,sr<2>> sqrt<U,Y>(const quantity<U,Y>& q);

dimensionless_quantity<si::system,Y>::type {cos,sin,tan}<Y>(const quantity<si::plane_angle,Y>& theta);
dimensionless_quantity<S,Y>::type {cos,sin,tan}<S,Y>(const quantity<unit<plane_angle_dimension,S>,Y>& theta);
quantity<unit<plane_angle_dimension,homogeneous_system<S>>,Y> {acos,asin,atan}<Y,S> (const quantity<unit<dimensionless_type,homogeneous_system<S>>,Y>& val);
quantity<angle::radian_base_unit::unit_type,Y> {acos,asin,atan}<Y> (const quantity<unit<dimensionless_type,heterogeneous_dimensionless_system>,Y>& val);
quantity<unit<plane_angle_dimension,homogeneous_system<S>>,Y> atan2<Y,S> (const quantity<unit<dimensionless_type,homogeneous_system<S>>,Y>& y, &x);
quantity<angle::radian_base_unit::unit_type,Y> atan2<Y> (const quantity<unit<dimensionless_type,heterogeneous_dimensionless_system>,Y>& y, &x);

struct {power,root}_typeof_helper<T,static_rational<n,d>>; // type, value()
struct {power,root}_typeof_helper<float,static_rational<n,d>>; // type, value()
{power,root}_typeof_helper<Y,Rat>::type {pow,root}<Rat,Y>(const Y& x);
{power,root}_typeof_helper<Y,static_rational<n>>::type {pow,root}<n,Y>(const Y& x);

struct select_base_unit_converter<unscale<Source>::type,unscale<reduce_unit<Dest::unit_type>::type>::type>;
struct base_unit_converter<Source,reduce_unit<Dest::unit_type>::type>;
struct base_unit_converter<Src,make_heterogeneous_unit<...>::type>;
struct unscaled_get_default_conversion<T>;
one_to_double_type<conversion_factor_helper<FromUnit,ToUnit>::type>::type conversion_factor<FromUnit,ToUnit>(const FromUnit&,const ToUnit&);

struct na{};
struct make_system<...U>; // homogeneous_system<list<...,dimensionless_type>>>
```

#### IO

```c++
struct conversion_helper<From,To>;
std::string to_string<T>(const T&);  // static_rational
std::string name_string<T>(const T&); // unit<D,S>
std::string symbol_string<T>(const T&); // unit<D,S>
std::string raw_string<T>(const T&);
std::string typename_string<T>(const T&); // unit<D,S>
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,n,d>(std::basic_ostream<Ch,Tr>& os, const static_rational<n,d>& r);
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,D,S>(std::basic_ostream<Ch,Tr>& os, const unit<D,S>& u);
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,U,T>(std::basic_ostream<Ch,Tr>& os, const quantity<U,T>& q);

struct base_unit_info<BaseUnitTag> {
    using base_unit_info_primary_template=void;
    static std::string name(), symbol();
};

enum format_mode { symbol_fmt, name_fmt, raw_fmt, typename_fmt, fmt_mask=3 };
enum autoprefix_mode { autoprefix_none=0, autoprefix_engineering=4, autoprefix_binary=8, autoprefix_mask=12 };

void serialization::serialize<Ar,System,Dim>(Ar&, unit<Dim,System>&, unsigned){}
void serialization::serialize<Ar,Unit,Y>(Ar&, quantity<Unit,Y>& q, unsigned); // nvp("value", quantity_cast<Y&>(q))

long get_flags(std::ios_base& ios, long mask); void set_flags(std::ios_base& ios, lnog new_flags, long mask);
format_mode get_format(std::ios_base& ios); void set_format(std::ios_base& ios, format_mode new_mode);
std::ios_base& {typename,raw,symbol,name}_format(std::ios_base& ios);
autoprefix_mode get_autoprefix(std::ios_base& ios); void set_autoprefix(std::ios_base& ios, autoprefix_mode new_mode);
std::ios_base& {no,engineering,binary}_prefix(std::ios_base& ios);

double autoprefix_norm<T>(const T& arg);
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

struct one { ctor(); };
constexpr one make_one();
struct {multiply,divide}_typeof_helper<one,T>; // and <T,one>, <one,one>
constexpr T operator{*,/} <T>(const one&, const T& t); // and <T,one>, <one,one>
constexpr bool operator> (const one&, const T& t); // 1 > t
constexpr T one_to_double<T>(const T& t); // t
constexpr bool one_to_double(const one&); // 1.0
struct one_to_double_type<T>; // T
struct one_to_double_type<one>; // double

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

std::string detail::demangle(const char* name);
std::string simplify_typename<L>(const L&) { return demangle(typeid(L).name()); }

struct detail::no_solution{};
struct detail::make_homogeneous_system<Units>;
struct detail::extract_base_units<n>;

struct detail::ordinal_tag{};
struct ordinal<n> {using type=ordinal_tag; static constexpr long value=n;};

struct detail::no{char dummy;}; struct detail::yes{ no dummy[2]; };
no boost_units_is_registered<T>(const T&);
no boost_units_unit_is_registered<T>(const T&);
```

------
### Physical Dimensions

```c++
// Base dimensions: for (NAME,N,SYM):
struct {NAME}_base_dimension : base_dimension<self,{N}>{};
using {NAME}_dimension = {NAME}_base_dimension::dimension_type;
// plane and solid angle:
//  (solid_angle,-1,QS), (plane_angle,-2,QP)
// SI:
//  (luminous_intensity,-3,J), (amount,-4,N), (temperature,-5,Theta), (current,-6,I),
//  (time,-7,T), (mass,-8,M), (length,-9,L)
// Other
//  (information,-700)

// Derived dimensions: for (NAME,B1^n1...):
using {NAME}_dimension = derived_dimension<{B1},{n1},...>::type;
// (absorbed_dose, L^2 T^-2), (acceleration, L T^-2), (action, L^2 M T^-1), (activity, T^-1),
// (angular_acceleration, T^-2 QP), (angular_momentum, L^2 M T^-1 QP^-1), (angular_velocity, T^-1 QP),
// (area, L^2), (capacitance, L^-2 M^-1 T^4 I^2),
// (conductance, L^-2 M^-1 T^3 I^2), (conductivity, L^-3 M^-1 T^3 I^2),
// (dose_equivalent, L^2 T^-2), (dynamic_viscosity, M L^-1 T^-1),
// (electric_charge, T^1 I^1), (electric_potential, L^2 M T^-3 I^-1),
// (energy, L^2 M T^-2), (energy_density, L^-1 M^1 T^-2 ), (force, L M T^-2),
// (frequency, T^-1), (heat_capacity, L^2 M T^-2 Theta^-1), (illuminance, L^-2 I QS),
// (impedance, L^2 M T^-3 I^-2), (inductance, L^2 M T^-2 I^-2), (kinematic_viscosity, L^2 T^-1),
// (luminance, L^-2 I), (luminous_flux, I QS),
// (magnetic_field_intensity, L^-1 I), (magnetic_flux, L^2 M T^-2 I^-1), (magnetic_flux_density, M T^-2 I^-1),
// (mass_density, L^-3 M), (molar_energy, L^2 M T^-2 N^-1), (molar_heat_capacity, L^2 M T^-2 Theta^-1 N^-1),
// (moment_of_inertia, L^2 M QP^-2), (momentum, L M T^-1), (permeability, L M T^-2 I^-), (permittivity, L^-3 M^-1 T^4 I^2),
// (power, L^2 M T^-3), (pressure, L^-1 M T^-2), (reluctance, L^-2 M^-1 T^2 I^2),
// (resistance, L^2 M T^-3 I^-2), (resistivity, L^3 M T^-3 I^-2),
// (specific_energy, L^2 T^-2), (specific_heat_capacity, L^2 T^-2 Theta^-1), (specific_volume, L^3 M^-1),
// (stress, L^-1 M T^-2), (surface_density, L^-2 M), (surface_tension, M T^-2),
// (thermal_conductivity, L^1 M^1 T^-3 Theta^-1), (torque, L^2 M T^-2 QP^-1),
// (velocity, L T^-1), (volume, L^3), (wavenumber, L^-1),
```

------
### Base Units

#### Angle Base Units

```c++
namespace angle;

struct steradian_base_unit : base_unit<self, solid_angle_dimension, -1>; // "steradian", "sr"
struct radian_base_unit : base_unit<self, plane_angle_dimension, -2>; // "radian", "rad"
struct degree_base_unit : base_unit<self, radian_base_unit::dimension_type, -101>; // "degree", "deg"
struct gradian_base_unit : base_unit<self, radian_base_unit::dimension_type, -102>; // "gradian", "grad"
using arcminute_base_unit = scaled_base_unit<degree_base_unit, scale<60, static_rational<-1>>>; // "arcminute", "'"
using arcsecond_base_unit = scaled_base_unit<degree_base_unit, scale<3600, static_rational<-1>>>; // "arcsecond", "\""
using revolution_base_unit = scaled_base_unit<degree_base_unit, scale<360, static_rational<1>>>; // "revolution", "rev"
```

#### SI Base Units

```c++
namespace si;
struct candela_base_unit : public base_unit<candela_base_unit, luminous_intensity_dimension, -3>; // "candela", "cd"
struct mole_base_unit : public base_unit<mole_base_unit, amount_dimension, -4>; // "mole", "mol"
struct kelvin_base_unit : public base_unit<kelvin_base_unit, temperature_dimension, -5>; // "kelvin", "K"
struct ampere_base_unit : public base_unit<ampere_base_unit, current_dimension, -6>; // "ampere", "A"
struct second_base_unit : public base_unit<second_base_unit, time_dimension, -7>; // "second", "s"
using kilogram_base_unit = scaled_base_unit<cgs::gram_base_unit, scale<10, static_rational<3>>>;
struct meter_base_unit : public base_unit<meter_base_unit, length_dimension, -9>; // "meter", "m"
```

#### CGS Base Units

```c++
namespace cgs;
using biot_base_unit = scaled_base_unit<si::ampere_base_unit, scale<10, static_rational<+1>>>;
struct gram_base_unit : base_unit<gram_base_unit, mass_dimension, -8>; // "gram", "g"
using centimeter_base_unit =  scaled_base_unit<si::meter_base_unit, scale<10, static_rational<-2>>>;
```

#### Astronomical Base Units

```c++
namespace astronomical;
struct light_second_base_unit : base_unit<self, si::meter_base_unit::dimension_type, -201>; // "light second", "lsc"
using light_minute_base_unit = scaled_base_unit<light_second_base_unit, scale<60, static_rational<1>>>; // "light minute", "lmn"
using light_hour_base_unit = scaled_base_unit<light_second_base_unit, scale<3600, static_rational<1>>>; // "light hour", "lhr"
using light_day_base_unit = scaled_base_unit<light_second_base_unit, scale<86400, static_rational<1>>>; // "light day", "ldy"
using light_year_base_unit = scaled_base_unit<light_second_base_unit, scale<31557600, static_rational<1>>>; // "light year", "ly"
struct parsec_base_unit  : base_unit<self, si::meter_base_unit::dimension_type, -206>; // "parsec", "psc"
struct astronomical_unit_base_unit : base_unit<self, si::meter_base_unit::dimension_type, -207>; // "astronomical unit", "a.u."
```

#### Temperature Base Units

```c++
namespace temperature;
struct celsius_base_unit : base_unit<self, temperature_dimension, -1008>; // "celsius", "C"
struct fahrenheit_base_unit : base_unit<self, temperature_dimension, -1007>; // "fahrenheit", "F"
```

#### Information Base Units

```c++
namespace information;
struct bit_base_unit : base_unit<self, information_dimension, -700>; // "bit", "b"
using byte_base_unit = scaled_base_unit<bit_base_unit, scale<2, static_rational<3>>>; // "byte", "B"
struct nat_base_unit : base_unit<self, bit_base_unit::dimension_type, -702>; // "nat", "nat"
struct hartley_base_unit : base_unit<self, bit_base_unit::dimension_type, -703>; // "hartley", "Hart"
using shannon_base_unit = scaled_base_unit<bit_base_unit, scale<1, static_rational<1>>>; // "shannon", "Sh"
```

#### Imperial Base Units

```c++
namespace imperial;
struct yard_base_unit : base_unit<self, si::meter_base_unit::dimension_type, -301>; // "yard", "yd"
struct pound_base_unit : base_unit<self, cgs::gram_base_unit::dimension_type, -302>; // "pound", "lb"
struct pint_base_unit : base_unit<self, si::volume::dimension_type, -303>; // "pint (imp.)", "pt"
using grain_base_unit = scaled_base_unit<pound_base_unit, scale<7000, static_rational<-1>>>; // "grain", "grain"
using ounce_base_unit = scaled_base_unit<pound_base_unit, scale<2, static_rational<-4>>>; // "ounce", "oz"
using drachm_base_unit = scaled_base_unit<pound_base_unit, scale<16, static_rational<-2>>>; // "drachm", "drachm"
using stone_base_unit = scaled_base_unit<pound_base_unit, scale<14, static_rational<1>>>; // "stone", "st"
using quarter_base_unit = scaled_base_unit<pound_base_unit, scale<28, static_rational<1>>>; // "quarter", "quarter"
using hundredweight_base_unit = scaled_base_unit<pound_base_unit, scale<112, static_rational<1>>>; // "hundredweight", "cwt"
using ton_base_unit = scaled_base_unit<pound_base_unit, scale<2240, static_rational<1>>>; // "long ton", "t"
using fluid_ounce_base_unit = scaled_base_unit<pint_base_unit, scale<20, static_rational<-1>>>; // "fluid ounce (imp.)", "fl oz"
using gill_base_unit = scaled_base_unit<pint_base_unit, scale<4, static_rational<-1>>>; // "gill (imp.)", "gill"
using quart_base_unit = scaled_base_unit<pint_base_unit, scale<2, static_rational<1>>>; // "quart (imp.)", "qt"
using gallon_base_unit = scaled_base_unit<pint_base_unit, scale<8, static_rational<1>>>; // "gallon (imp.)", "gal"
using thou_base_unit = scaled_base_unit<yard_base_unit, scale<36000, static_rational<-1>>>; // "thou", "thow"
using inch_base_unit = scaled_base_unit<yard_base_unit, scale<36, static_rational<-1>>>; // "inch", "in"
using foot_base_unit = scaled_base_unit<yard_base_unit, scale<3, static_rational<-1>>>; // "foot", "ft"
using furlong_base_unit = scaled_base_unit<yard_base_unit, scale<220, static_rational<1>>>; // "furlong", "furlong"
using mile_base_unit = scaled_base_unit<yard_base_unit, scale<1760, static_rational<1>>>; // "mile", "mi"
using league_base_unit = scaled_base_unit<yard_base_unit, scale<5280, static_rational<1>>>; // "league", "league"
```

#### Metric Base Units

```c++
namespace metric;
struct are_base_unit : base_unit<self, si::area::dimension_type, 10>; // "are", "a"
struct barn_base_unit : base_unit<self, si::area::dimension_type, 11>; // "barn", "b"
struct hectare_base_unit : base_unit<self, si::area::dimension_type, 12>; // "hectare", "ha"
struct liter_base_unit : base_unit<self, si::volume::dimension_type, 13>; // "liter", "L"
struct bar_base_unit : base_unit<self, si::pressure::dimension_type, 14>; // "bar", "bar"
struct atmosphere_base_unit : base_unit<self, si::pressure::dimension_type, 33>; // "atmosphere", "atm"
struct torr_base_unit : base_unit<self, si::presure::dimension_type, -401>; // "torr", "Torr"
struct knot_base_unit : base_unit<self, si::velocity::dimension_type, -403>; // "knot", "kt"
struct mmHg_base_unit : base_unit<self, si::presure::dimension_type, -404>; // "millimeters mercury", "mmHg"
using minute_base_unit = scaled_base_unit<si::second_base_unit, scale<60, static_rational<1>>>; // "minute", "min"
using hour_base_unit = scaled_base_unit<si::second_base_unit, scale<60, static_rational<2>>>; // "hour", "h"
using day_base_unit = scaled_base_unit<si::second_base_unit, scale<86400, static_rational<1>>>; // "day", "d"
using year_base_unit = scaled_base_unit<si::second_base_unit, scale<31557600, static_rational<1>>>; // "Julian year", "yr"
using fermi_base_unit = scaled_base_unit<si::meter_base_unit, scale<10, static_rational<-15>>>; // "fermi", "fm"
using micron_base_unit = scaled_base_unit<si::meter_base_unit, scale<10, static_rational<-6>>>; // "micron", "u"
using nautical_mile_base_unit = scaled_base_unit<si::meter_base_unit, scale<1852, static_rational<1>>>; // "nautical mile", "nmi"
using angstrom_base_unit = scaled_base_unit<si::meter_base_unit, scale<10, static_rational<-10>>>; // "angstrom", "A"
using ton_base_unit = scaled_base_unit<si::kilogram_base_unit, scale<1000, static_rational<1>>>; // "metric ton", "t"
```

#### Imperial Base Units

```c++
namespace us;

struct yard_base_unit : base_unit<self, si::meter_base_unit::dimension_type, -501>; // "yard", "yd"
struct pound_base_unit : base_unit<self, cgs::gram_base_unit::dimension_type, -502>; // "pound", "lb"
struct pint_base_unit : base_unit<self, si::volume::dimension_type, -503>; // "pint (U.S.)", "pt"
struct pound_force_base_unit : base_unit<self, si::force::dimension_type, -600>; // "pound-force", "lbf"

using grain_base_unit = scaled_base_unit<pound_base_unit, scale<7000, static_rational<-1>>>; // "grain", "grain"
using dram_base_unit = scaled_base_unit<pound_base_unit, scale<16, static_rational<-2>>>; // "dram (U.S.)", "dr"
using ounce_base_unit = scaled_base_unit<pound_base_unit, scale<2, static_rational<-4>>>; // "ounce", "oz"
using hundredweight_base_unit = scaled_base_unit<pound_base_unit, scale<100, static_rational<1>>>; // "hundredweight (U.S.)", "cwt"
using ton_base_unit = scaled_base_unit<pound_base_unit, scale<2000, static_rational<1>>>; // "short ton", "t"
using minim_base_unit = scaled_base_unit<pint_base_unit, scale<7600, static_rational<-1>>>; // "minim (U.S.)", "minim"
using fluid_dram_base_unit = scaled_base_unit<pint_base_unit, scale<2, static_rational<-7>>>; // "fluid dram (U.S.)", "fl dr"
using teespoon_base_unit = scaled_base_unit<pint_base_unit, scale<96, static_rational<-1>>>; // "teespoon", "tsp"
using tablespoon_base_unit = scaled_base_unit<pint_base_unit, scale<2, static_rational<-5>>>; // "tablespoon", "tbsp"
using cup_base_unit = scaled_base_unit<pint_base_unit, scale<2, static_rational<-1>>>; // "cup", "c"
using fluid_ounce_base_unit = scaled_base_unit<pint_base_unit, scale<16, static_rational<-1>>>; // "fluid ounce (U.S.)", "fl oz"
using gill_base_unit = scaled_base_unit<pint_base_unit, scale<2, static_rational<-2>>>; // "gill (U.S.)", "gi"
using quart_base_unit = scaled_base_unit<pint_base_unit, scale<2, static_rational<1>>>; // "quart (U.S.)", "qt"
using gallon_base_unit = scaled_base_unit<pint_base_unit, scale<2, static_rational<3>>>; // "gallon (U.S.)", "gal"
using mil_base_unit = scaled_base_unit<yard_base_unit, scale<36000, static_rational<-1>>>; // "mil", "mil"
using inch_base_unit = scaled_base_unit<yard_base_unit, scale<36, static_rational<-1>>>; // "inch", "in"
using foot_base_unit = scaled_base_unit<yard_base_unit, scale<3, static_rational<-1>>>; // "foot", "ft"
using mile_base_unit = scaled_base_unit<yard_base_unit, scale<1760, static_rational<1>>>; // "mile", "mi"
```

------
### Unit Systems

```c++
struct constant<Base> {
    using value_type = Base::value_type;
    operator value_type() const; value_type value() const; // Base{}.value()
    value_type uncertainty() const, {lower,upper}_bound() const; // Base{}.xxx()
};
struct physical_constant<Base> {
    using value_type = Base::value_type;
    operator value_type() const; value_type value() const; // Base{}.value()
    value_type uncertainty() const, {lower,upper}_bound() const; // Base{}.xxx()
};

struct {add,subtract,multiply,divide}_typeof_helper<constant<T>,unit<T1,T2>>
{using type={add,subtract,multiply,divide}_typeof_helper<T::value_type,unit<T1,T2>>::type;}; // and inversed
{add,subtract,multiply,divide}_typeof_helper<T::value_type,unit<T1,T2>>::type operator{+,-,*,/} <T,T1,T2>(const constant<T>& t, const unit<T1,T2>& u); // and inversed

struct {add,subtract,multiply,divide}_typeof_helper<constant<T>,quantity<T1,T2>>
{using type={add,subtract,multiply,divide}_typeof_helper<T::value_type,quantity<T1,T2>>::type;}; // and inversed
{add,subtract,multiply,divide}_typeof_helper<T::value_type,quantity<T1,T2>>::type operator{+,-,*,/} <T,T1,T2>(const constant<T>& t, const quantity<T1,T2>& u); // and inversed

struct {add,subtract,multiply,divide}_typeof_helper<constant<T1>,constant<T2>>
{using type={add,subtract,multiply,divide}_typeof_helper<T1::value_type,T2::value_type>::type;};
{add,subtract,multiply,divide}_typeof_helper<T1::value_type,T2::value_type>::type operator{+,-,*,/} <T1,T2>(const constant<T1>& t, const constant<T2>& u);

struct {add,subtract,multiply,divide}_typeof_helper<constant<T1>,T2> // and inversed
{using type={add,subtract,multiply,divide}_typeof_helper<T1::value_type,T2>::type;};
{add,subtract,multiply,divide}_typeof_helper<T1::value_type,T2>::type operator{+,-,*,/} <T1,T2>(const constant<T1>& t, const T2& u); // and inversed

struct {multiply,divide}_typeof_helper<constant<T1>,one> // and inversed
{using type={multiply,divide}_typeof_helper<T1::value_type,one>::type;};
{multiply,divide}_typeof_helper<T1::value_type,one>::type operator{*,/} <T1>(const constant<T1>& t, const one& u); // and inversed

struct power_typeof_helper<constant<T1>,static_rational<n,d>>; // type, value()

std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,Y> (std::basic_ostream<Ch,Tr>& os, const physical_constant<Y>& val);
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,Y> (std::basic_ostream<Ch,Tr>& os, const constant<Y>& val);
```

#### SI System

```c++
namespace si {
// system
using system = make_system<meter_base_unit, kilogram_base_unit, second_base_unit,
    ampere_base_unit, kelven_base_unit, mole_base_unit, candela_base_unit,
    angle::radian_base_unit, angle::steradian_base_unit>::type;
using dimensionless = unit<dimensionless_type,system>;

// units
using absorbed_dose = unit<absorbed_dose_dimension, si::system>;
constexpr static absorbed_dose gray, grays;
using acceleration = unit<acceleration_dimension, si::system>;
constexpr static acceleration {meter,metre}<s>_per_second_squared;
using action = unit<action_dimension, si::system>;
using activity = unit<activity_dimension, si::system>;
constexpr static activity becquerel<s>;
using amount = unit<amount_dimension, si::system>;
constexpr static amount mole<s>;
using angular_acceleration = unit<angular_acceleration_dimension,si::system>;
using angular_momentum = unit<angular_momentum_dimension,si::system>;
using angular_velocity = unit<angular_velocity_dimension,si::system>;
constexpr static angular_velocity radian<s>_per_second;
using area = unit<area_dimension, si::system>;
constexpr static area square_{meter,metre}<s>;
using capacitance = unit<capacitance_dimension,si::system>;
constexpr static capacitance farad<s>;
using catalytic_activity_dim  = derived_dimension<time_base_dimension,-1,amount_base_dimension,1>::type;
using catalytic_activity = unit<si::catalytic_activity_dim,si::system>;
constexpr static catalytic_activity katal<s>;
using conductance = unit<conductance_dimension,si::system>;
constexpr static conductance siemen<s>, mho<s>;
using conductivity = unit<conductivity_dimension,si::system>;
using current = unit<current_dimension,si::system>;
constexpr static current ampere<s>;
static constexpr dimensionless si_dimensionless;
using dose_equivalent = unit<dose_equivalent_dimension,si::system>;
constexpr static dose_equivalent sievert<s>;
using dynamic_viscosity = unit<dynamic_viscosity_dimension,si::system>;
using electric_charge = unit<electric_charge_dimension,si::system>;
constexpr static electric_charge coulomb<s>;
using electric_potential = unit<electric_potential_dimension,si::system>;
constexpr static electric_potential volt<s>;
using energy = unit<energy_dimension,si::system>;
constexpr static energy joule<s>;
using force = unit<force_dimension,si::system>;
constexpr static force newton<s>;
using frequency = unit<frequency_dimension,si::system>;
constexpr static frequency hertz;
using illuminance = unit<illuminance_dimension,si::system>;
constexpr static illuminance lux;
using impedance = unit<impedance_dimension,si::system>;
using inductance = unit<inductance_dimension,si::system>;
constexpr static inductance henry<s>;
using kinematic_viscosity = unit<kinematic_viscosity_dimension,si::system>;
using length = unit<length_dimension,si::system>;
constexpr static length {meter,metre}<s>;
using luminous_flux = unit<luminous_flux_dimension,si::system>;
constexpr static luminous_flux lumen<s>;
using luminous_intensity = unit<luminous_intensity_dimension,si::system>;
constexpr static luminous_intensity candela<s>;
using magnetic_field_intensity = unit<magnetic_field_intensity_dimension,si::system>;
using magnetic_flux = unit<magnetic_flux_dimension,si::system>;
constexpr static magnetic_flux weber<s>;
using magnetic_flux_density = unit<magnetic_flux_density_dimension,si::system>;
constexpr static magnetic_flux_density tesla<s>;
using mass = unit<mass_dimension,si::system>;
constexpr static mass kilogram<me><s>;
using mass_density = unit<mass_density_dimension,si::system>;
constexpr static mass_density kilogram<me><s>_per_cubic_{meter,metre};
using moment_of_inertia = unit<moment_of_inertia_dimension,si::system>;
using momentum = unit<momentum_dimension,si::system>;
using permeability = unit<permeability_dimension,si::system>;
using permittivity = unit<permittivity_dimension,si::system>;
using plane_angle = unit<plane_angle_dimension,si::system>;
constexpr static plane_angle radian<s>;
using power = unit<power_dimension,si::system>;
constexpr static power watt<s>;
using pressure = unit<pressure_dimension,si::system>;
constexpr static pressure pascal<s>;
using reluctance = unit<reluctance_dimension,si::system>;
using resistance = unit<resistance_dimension,si::system>;
constexpr static resistance ohm<s>;
using resistivity = unit<resistivity_dimension,si::system>;
using solid_angle = unit<solid_angle_dimension,si::system>;
constexpr static solid_angle steradian<s>;
using surface_density = unit<surface_density_dimension,si::system>;
constexpr static surface_density kilogram<me><s>_per_square_{meter,metre};
using surface_tension = unit<surface_tension_dimension,si::system>;
constexpr static surface_tension newton<s>_per_meter;
using temperature = unit<temperature_dimension,si::system>;
constexpr static temperature kelvin<s>;
using time = unit<time_dimension,si::system>;
constexpr static time second<s>;
using torque = unit<torque_dimension,si::system>;
constexpr static torque newton_meter<s>;
using velocity = unit<velocity_dimension,si::system>;
constexpr static velocity {meter,metre}<s>_per_second;
using volume = unit<volume_dimension,si::system>;
constexpr static volume cubic_{meter,metre}<s>;
using wavenumber = unit<wavenumber_dimension,si::system>;
constexpr static wavenumber reciprocal_{meter,metre}<s>;

// prefixes: for (EXP,NAME):
using {NAME}_type = make_scaled_unit<dimensionless, scale<10,static_rational<{EXP}>>>::type;
constexpr static {NAME}_type {NAME};
// (-24,yocto), (-21,zepto), (-18,atto), (-15,femto), (-12,pico), (-9,nano), (-6,micro),
// (-3,milli), (-2,centi), (-1,deci), (1,deka), (2,hecto), (3,kilo), (6,mega), (9,giga),
// (12,tera), (15,peta), (18,exa), (21,zetta), (24,yotta)

// io: for (NAME,N,S)
std::string name_string(const reduce_unit<si::{NAME}>::type&) { return "{N}"; }
std::string symbol_string(const reduce_unit<si::{NAME}>::type&) { return "{S}"; }
// (absorbed_dose,gray,Gy), (capacitance,farda,F), (catalytic_activity,katal,kat),
// (conductance,siemen,S), (electric_charge,coulomb,C), (electric_potential,volt,V),
// (energy,joule,J), (force,newton,N), (frequency,hertz,Hz), (illuminance,lux,lx),
// (inductance,henry,H), (luminous_flux,lumen,lm), (magnetic_flux,weber,Wb),
// (magnetic_flux_density,tesla,T), (power,watt,W), (presure,pascal,Pa), (resistance,ohm,Ohm)

// constants
namespace constants::codata {
using frequency_over_electric_potential = divide_typeof_helper<frequency,electric_potential>::type;
using electric_charge_over_mass = divide_typeof_helper<electric_charge,mass>::type;
using mass_over_amount = divide_typeof_helper<mass,amount>::type;
using energy_over_magnetic_flux_density = divide_typeof_helper<energy,magnetic_flux_density>::type;
using frequency_over_magnetic_flux_density = divide_typeof_helper<frequency,magnetic_flux_density>::type;
using current_over_energy = divide_typeof_helper<current,energy>::type;
using inverse_amount = divide_typeof_helper<dimensionless,amount>::type;
using energy_over_temperature = divide_typeof_helper<energy,temperature>::type;
using energy_over_temperature_amount = divide_typeof_helper<energy_over_temperature,amount>::type;
using power_over_area_temperature_4 = divide_typeof_helper<divide_typeof_helper<power,area>::type,power_typeof_helper<temperature,static_rational<4> >::type>::type;
using power_area = multiply_typeof_helper<power,area>::type;
using power_area_over_solid_angle = divide_typeof_helper<power_area,solid_angle>::type;
using length_temperature = multiply_typeof_helper<length,temperature>::type;
using frequency_over_temperature = divide_typeof_helper<frequency,temperature>::type;
using force_over_current_squared = divide_typeof_helper<divide_typeof_helper<force,current>::type,current>::type;
using capacitance_over_length = divide_typeof_helper<capacitance,length>::type;
using volume_over_mass_time_squared = divide_typeof_helper<divide_typeof_helper<divide_typeof_helper<volume,mass>::type,time>::type,time>::type;
using energy_time = multiply_typeof_helper<energy,time>::type;
using electric_charge_over_amount = divide_typeof_helper<electric_charge,amount>::type;

// for (NAME,TYPE,VAL,UNCERTAINTY):
struct {NAME}_t {
    using value_type = {TYPE};
    operator value_type() const; value_type value() const; // {VAL}
    value_type uncertainty() const; // {UNCERTAINTY}
    value_type {lower,upper}_bound() const; // {VAL} +/- {UNCERTAINTY}
};

// ---- atomic-nuclear_constants
/// alpha particle mass: (m_alpha,quantity<mass>,6.64465620e-27*kilograms,3.3e-34*kilograms)
/// alpha-electron mass ratio: (m_alpha_over_m_e,quantity<dimensionless>,7294.2995365*dimensionless(),3.1e-6*dimensionless())
/// alpha-proton mass ratio: (m_alpha_over_m_p,quantity<dimensionless>,3.97259968951*dimensionless(),4.1e-10*dimensionless())
/// alpha molar mass: (M_alpha,quantity<mass_over_amount>,4.001506179127e-3*kilograms/mole,6.2e-14*kilograms/mole)

/// deuteron mass: (m_d,quantity<mass>,3.34358320e-27*kilograms,1.7e-34*kilograms)
/// deuteron-electron mass ratio: (m_d_over_m_e,quantity<dimensionless>,3670.4829654*dimensionless(),1.6e-6*dimensionless())
/// deuteron-proton mass ratio: (m_d_over_m_p,quantity<dimensionless>,1.99900750108*dimensionless(),2.2e-10*dimensionless())
/// deuteron molar mass: (M_d,quantity<mass_over_amount>,2.013553212724e-3*kilograms/mole,7.8e-14*kilograms/mole)
/// deuteron rms charge radius: (R_d,quantity<length>,2.1402e-15*meters,2.8e-18*meters)
/// deuteron magnetic moment: (mu_d,quantity<energy_over_magnetic_flux_density>,0.433073465e-26*joules/tesla,1.1e-34*joules/tesla)
/// deuteron-Bohr magneton ratio: (mu_d_over_mu_B,quantity<dimensionless>,0.4669754556e-3*dimensionless(),3.9e-12*dimensionless())
/// deuteron-nuclear magneton ratio: (mu_d_over_mu_N,quantity<dimensionless>,0.8574382308*dimensionless(),7.2e-9*dimensionless())
/// deuteron g-factor: (g_d,quantity<dimensionless>,0.8574382308*dimensionless(),7.2e-9*dimensionless())
/// deuteron-electron magnetic moment ratio: (mu_d_over_mu_e,quantity<dimensionless>,-4.664345537e-4*dimensionless(),3.9e-12*dimensionless())
/// deuteron-proton magnetic moment ratio: (mu_d_over_mu_p,quantity<dimensionless>,0.3070122070*dimensionless(),2.4e-9*dimensionless())
/// deuteron-neutron magnetic moment ratio: (mu_d_over_mu_n,quantity<dimensionless>,-0.44820652*dimensionless(),1.1e-7*dimensionless())

/// electron mass: (m_e,quantity<mass>,9.10938215e-31*kilograms,4.5e-38*kilograms)
/// electron-muon mass ratio: (m_e_over_m_mu,quantity<dimensionless>,4.83633171e-3*dimensionless(),1.2e-10*dimensionless())
/// electron-tau mass ratio: (m_e_over_m_tau,quantity<dimensionless>,2.87564e-4*dimensionless(),4.7e-8*dimensionless())
/// electron-proton mass ratio: (m_e_over_m_p,quantity<dimensionless>,5.4461702177e-4*dimensionless(),2.4e-13*dimensionless())
/// electron-neutron mass ratio: (m_e_over_m_n,quantity<dimensionless>,5.4386734459e-4*dimensionless(),3.3e-13*dimensionless())
/// electron-deuteron mass ratio: (m_e_over_m_d,quantity<dimensionless>,2.7244371093e-4*dimensionless(),1.2e-13*dimensionless())
/// electron-alpha particle mass ratio: (m_e_over_m_alpha,quantity<dimensionless>,1.37093355570e-4*dimensionless(),5.8e-14*dimensionless())
/// electron charge to mass ratio: (e_over_m_e,quantity<electric_charge_over_mass>,1.758820150e11*coulombs/kilogram,4.4e3*coulombs/kilogram)
/// electron molar mass: (M_e,quantity<mass_over_amount>,5.4857990943e-7*kilograms/mole,2.3e-16*kilograms/mole)
/// Compton wavelength: (lambda_C,quantity<length>,2.4263102175e-12*meters,3.3e-21*meters)
/// classical electron radius: (r_e,quantity<length>,2.8179402894e-15*meters,5.8e-24*meters)
/// Thompson cross section: (sigma_e,quantity<area>,0.6652458558e-28*square_meters,2.7e-37*square_meters)
/// electron magnetic moment: (mu_e,quantity<energy_over_magnetic_flux_density>,-928.476377e-26*joules/tesla,2.3e-31*joules/tesla)
/// electron-Bohr magenton moment ratio: (mu_e_over_mu_B,quantity<dimensionless>,-1.00115965218111*dimensionless(),7.4e-13*dimensionless())
/// electron-nuclear magneton moment ratio: (mu_e_over_mu_N,quantity<dimensionless>,-183.28197092*dimensionless(),8.0e-7*dimensionless())
/// electron magnetic moment anomaly: (a_e,quantity<dimensionless>,1.15965218111e-3*dimensionless(),7.4e-13*dimensionless())
/// electron g-factor: (g_e,quantity<dimensionless>,-2.0023193043622*dimensionless(),1.5e-12*dimensionless())
/// electron-muon magnetic moment ratio: (mu_e_over_mu_mu,quantity<dimensionless>,206.7669877*dimensionless(),5.2e-6*dimensionless())
/// electron-proton magnetic moment ratio: (mu_e_over_mu_p,quantity<dimensionless>,-658.2106848*dimensionless(),5.4e-6*dimensionless())
/// electron-shielded proton magnetic moment ratio: (mu_e_over_mu_p_prime,quantity<dimensionless>,-658.2275971*dimensionless(),7.2e-6*dimensionless())
/// electron-neutron magnetic moment ratio: (mu_e_over_mu_n,quantity<dimensionless>,960.92050*dimensionless(),2.3e-4*dimensionless())
/// electron-deuteron magnetic moment ratio: (mu_e_over_mu_d,quantity<dimensionless>,-2143.923498*dimensionless(),1.8e-5*dimensionless())
/// electron-shielded helion magnetic moment ratio: (mu_e_over_mu_h_prime,quantity<dimensionless>,864.058257*dimensionless(),1.0e-5*dimensionless())
/// electron gyromagnetic ratio: (gamma_e,quantity<frequency_over_magnetic_flux_density>,1.760859770e11/second/tesla,4.4e3/second/tesla)

/// helion mass: (m_h,quantity<mass>,5.00641192e-27*kilograms,2.5e-34*kilograms)
/// helion-electron mass ratio: (m_h_over_m_e,quantity<dimensionless>,5495.8852765*dimensionless(),5.2e-6*dimensionless())
/// helion-proton mass ratio: (m_h_over_m_p,quantity<dimensionless>,2.9931526713*dimensionless(),2.6e-9*dimensionless())
/// helion molar mass: (M_h,quantity<mass_over_amount>,3.0149322473e-3*kilograms/mole,2.6e-12*kilograms/mole)
/// helion shielded magnetic moment: (mu_h_prime,quantity<energy_over_magnetic_flux_density>,-1.074552982e-26*joules/tesla,3.0e-34*joules/tesla)
/// shielded helion-Bohr magneton ratio: (mu_h_prime_over_mu_B,quantity<dimensionless>,-1.158671471e-3*dimensionless(),1.4e-11*dimensionless())
/// shielded helion-nuclear magneton ratio: (mu_h_prime_over_mu_N,quantity<dimensionless>,-2.127497718*dimensionless(),2.5e-8*dimensionless())
/// shielded helion-proton magnetic moment ratio: (mu_h_prime_over_mu_p,quantity<dimensionless>,-0.761766558*dimensionless(),1.1e-8*dimensionless())
/// shielded helion-shielded proton magnetic moment ratio: (mu_h_prime_over_mu_p_prime,quantity<dimensionless>,-0.7617861313*dimensionless(),3.3e-8*dimensionless())
/// shielded helion gyromagnetic ratio: (gamma_h_prime,quantity<frequency_over_magnetic_flux_density>,2.037894730e8/second/tesla,5.6e-0/second/tesla)

/// muon mass: (m_mu,quantity<mass>,1.88353130e-28*kilograms,1.1e-35*kilograms)
/// muon-electron mass ratio: (m_mu_over_m_e,quantity<dimensionless>,206.7682823*dimensionless(),5.2e-6*dimensionless())
/// muon-tau mass ratio: (m_mu_over_m_tau,quantity<dimensionless>,5.94592e-2*dimensionless(),9.7e-6*dimensionless())
/// muon-proton mass ratio: (m_mu_over_m_p,quantity<dimensionless>,0.1126095261*dimensionless(),2.9e-9*dimensionless())
/// muon-neutron mass ratio: (m_mu_over_m_n,quantity<dimensionless>,0.1124545167*dimensionless(),2.9e-9*dimensionless())
/// muon molar mass: (M_mu,quantity<mass_over_amount>,0.1134289256e-3*kilograms/mole,2.9e-12*kilograms/mole)
/// muon Compton wavelength: (lambda_C_mu,quantity<length>,11.73444104e-15*meters,3.0e-22*meters)
/// muon magnetic moment: (mu_mu,quantity<energy_over_magnetic_flux_density>,-4.49044786e-26*joules/tesla,1.6e-33*joules/tesla)
/// muon-Bohr magneton ratio: (mu_mu_over_mu_B,quantity<dimensionless>,-4.84197049e-3*dimensionless(),1.2e-10*dimensionless())
/// muon-nuclear magneton ratio: (mu_mu_over_mu_N,quantity<dimensionless>,-8.89059705*dimensionless(),2.3e-7*dimensionless())
/// muon magnetic moment anomaly: (a_mu,quantity<dimensionless>,1.16592069e-3*dimensionless(),6.0e-10*dimensionless())
/// muon g-factor: (g_mu,quantity<dimensionless>,-2.0023318414*dimensionless(),1.2e-9*dimensionless())
/// muon-proton magnetic moment ratio: (mu_mu_over_mu_p,quantity<dimensionless>,-3.183345137*dimensionless(),8.5e-8*dimensionless())

/// neutron mass: (m_n,quantity<mass>,1.674927211e-27*kilograms,8.4e-35*kilograms)
/// neutron-electron mass ratio: (m_n_over_m_e,quantity<dimensionless>,1838.6836605*dimensionless(),1.1e-6*dimensionless())
/// neutron-muon mass ratio: (m_n_over_m_mu,quantity<dimensionless>,8.89248409*dimensionless(),2.3e-7*dimensionless())
/// neutron-tau mass ratio: (m_n_over_m_tau,quantity<dimensionless>,0.528740*dimensionless(),8.6e-5*dimensionless())
/// neutron-proton mass ratio: (m_n_over_m_p,quantity<dimensionless>,1.00137841918*dimensionless(),4.6e-10*dimensionless())
/// neutron molar mass: (M_n,quantity<mass_over_amount>,1.00866491597e-3*kilograms/mole,4.3e-13*kilograms/mole)
/// neutron Compton wavelength: (lambda_C_n,quantity<length>,1.3195908951e-15*meters,2.0e-24*meters)
/// neutron magnetic moment: (mu_n,quantity<energy_over_magnetic_flux_density>,-0.96623641e-26*joules/tesla,2.3e-33*joules/tesla)
/// neutron g-factor: (g_n,quantity<dimensionless>,-3.82608545*dimensionless(),9.0e-7*dimensionless())
/// neutron-electron magnetic moment ratio: (mu_n_over_mu_e,quantity<dimensionless>,1.04066882e-3*dimensionless(),2.5e-10*dimensionless())
/// neutron-proton magnetic moment ratio: (mu_n_over_mu_p,quantity<dimensionless>,-0.68497934*dimensionless(),1.6e-7*dimensionless())
/// neutron-shielded proton magnetic moment ratio: (mu_n_over_mu_p_prime,quantity<dimensionless>,-0.68499694*dimensionless(),1.6e-7*dimensionless())
/// neutron gyromagnetic ratio: (gamma_n,quantity<frequency_over_magnetic_flux_density>,1.83247185e8/second/tesla,4.3e1/second/tesla)

/// proton mass: (m_p,quantity<mass>,1.672621637e-27*kilograms,8.3e-35*kilograms)
/// proton-electron mass ratio: (m_p_over_m_e,quantity<dimensionless>,1836.15267247*dimensionless(),8.0e-7*dimensionless())
/// proton-muon mass ratio: (m_p_over_m_mu,quantity<dimensionless>,8.88024339*dimensionless(),2.3e-7*dimensionless())
/// proton-tau mass ratio: (m_p_over_m_tau,quantity<dimensionless>,0.528012*dimensionless(),8.6e-5*dimensionless())
/// proton-neutron mass ratio: (m_p_over_m_n,quantity<dimensionless>,0.99862347824*dimensionless(),4.6e-10*dimensionless())
/// proton charge to mass ratio: (e_over_m_p,quantity<electric_charge_over_mass>,9.57883392e7*coulombs/kilogram,2.4e0*coulombs/kilogram)
/// proton molar mass: (M_p,quantity<mass_over_amount>,1.00727646677e-3*kilograms/mole,1.0e-13*kilograms/mole)
/// proton Compton wavelength: (lambda_C_p,quantity<length>,1.3214098446e-15*meters,1.9e-24*meters)
/// proton rms charge radius: (R_p,quantity<length>,0.8768e-15*meters,6.9e-18*meters)
/// proton magnetic moment: (mu_p,quantity<energy_over_magnetic_flux_density>,1.410606662e-26*joules/tesla,3.7e-34*joules/tesla)
/// proton-Bohr magneton ratio: (mu_p_over_mu_B,quantity<dimensionless>,1.521032209e-3*dimensionless(),1.2e-11*dimensionless())
/// proton-nuclear magneton ratio: (mu_p_over_mu_N,quantity<dimensionless>,2.792847356*dimensionless(),2.3e-8*dimensionless())
/// proton g-factor: (g_p,quantity<dimensionless>,5.585694713*dimensionless(),4.6e-8*dimensionless())
/// proton-neutron magnetic moment ratio: (mu_p_over_mu_n,quantity<dimensionless>,-1.45989806*dimensionless(),3.4e-7*dimensionless())
/// shielded proton magnetic moment: (mu_p_prime,quantity<energy_over_magnetic_flux_density>,1.410570419e-26*joules/tesla,3.8e-34*joules/tesla)
/// shielded proton-Bohr magneton ratio: (mu_p_prime_over_mu_B,quantity<dimensionless>,1.520993128e-3*dimensionless(),1.7e-11*dimensionless())
/// shielded proton-nuclear magneton ratio: (mu_p_prime_over_mu_N,quantity<dimensionless>,2.792775598*dimensionless(),3.0e-8*dimensionless())
/// proton magnetic shielding correction: (sigma_p_prime,quantity<dimensionless>,25.694e-6*dimensionless(),1.4e-8*dimensionless())
/// proton gyromagnetic ratio: (gamma_p,quantity<frequency_over_magnetic_flux_density>,2.675222099e8/second/tesla,7.0e0/second/tesla)
/// shielded proton gyromagnetic ratio: (gamma_p_prime,quantity<frequency_over_magnetic_flux_density>,2.675153362e8/second/tesla,7.3e0/second/tesla)

/// tau mass: (m_tau,quantity<mass>,3.16777e-27*kilograms,5.2e-31*kilograms)
/// tau-electron mass ratio: (m_tau_over_m_e,quantity<dimensionless>,3477.48*dimensionless(),5.7e-1*dimensionless())
/// tau-muon mass ratio: (m_tau_over_m_mu,quantity<dimensionless>,16.8183*dimensionless(),2.7e-3*dimensionless())
/// tau-proton mass ratio: (m_tau_over_m_p,quantity<dimensionless>,1.89390*dimensionless(),3.1e-4*dimensionless())
/// tau-neutron mass ratio: (m_tau_over_m_n,quantity<dimensionless>,1.89129*dimensionless(),3.1e-4*dimensionless())
/// tau molar mass: (M_tau,quantity<mass_over_amount>,1.90768e-3*kilograms/mole,3.1e-7*kilograms/mole)
/// tau Compton wavelength: (lambda_C_tau,quantity<length>,0.69772e-15*meters,1.1e-19*meters)

/// triton mass: (m_t,quantity<mass>,5.00735588e-27*kilograms,2.5e-34*kilograms)
/// triton-electron mass ratio: (m_t_over_m_e,quantity<dimensionless>,5496.9215269*dimensionless(),5.1e-6*dimensionless())
/// triton-proton mass ratio: (m_t_over_m_p,quantity<dimensionless>,2.9937170309*dimensionless(),2.5e-9*dimensionless())
/// triton molar mass: (M_t,quantity<mass_over_amount>,3.0155007134e-3*kilograms/mole,2.5e-12*kilograms/mole)
/// triton magnetic moment: (mu_t,quantity<energy_over_magnetic_flux_density>,1.504609361e-26*joules/tesla,4.2e-34*joules/tesla)
/// triton-Bohr magneton ratio: (mu_t_over_mu_B,quantity<dimensionless>,1.622393657e-3*dimensionless(),2.1e-11*dimensionless())
/// triton-nuclear magneton ratio: (mu_t_over_mu_N,quantity<dimensionless>,2.978962448*dimensionless(),3.8e-8*dimensionless())
/// triton g-factor: (g_t,quantity<dimensionless>,5.957924896*dimensionless(),7.6e-8*dimensionless())
/// triton-electron magnetic moment ratio: (mu_t_over_mu_e,quantity<dimensionless>,-1.620514423e-3*dimensionless(),2.1e-11*dimensionless())
/// triton-proton magnetic moment ratio: (mu_t_over_mu_p,quantity<dimensionless>,1.066639908*dimensionless(),1.0e-8*dimensionless())
/// triton-neutron magnetic moment ratio: (mu_t_over_mu_n,quantity<dimensionless>,-1.55718553*dimensionless(),3.7e-7*dimensionless())

// ------electromagnetic_constants
/// elementary charge: (e,quantity<electric_charge>,1.602176487e-19*coulombs,4.0e-27*coulombs)
/// elementary charge to Planck constant ratio: (e_over_h,quantity<current_over_energy>,2.417989454e14*amperes/joule,6.0e6*amperes/joule)
/// magnetic flux quantum: (Phi_0,quantity<magnetic_flux>,2.067833667e-15*webers,5.2e-23*webers)
/// conductance quantum: (G_0,quantity<conductance>,7.7480917004e-5*siemens,5.3e-14*siemens)
/// Josephson constant: (K_J,quantity<frequency_over_electric_potential>,483597.891e9*hertz/volt,1.2e7*hertz/volt)
/// von Klitzing constant: (R_K,quantity<resistance>,25812.807557*ohms,1.77e-5*ohms)
/// Bohr magneton: (mu_B,quantity<energy_over_magnetic_flux_density>,927.400915e-26*joules/tesla,2.3e-31*joules/tesla)
/// nuclear magneton: (mu_N,quantity<energy_over_magnetic_flux_density>,5.05078324e-27*joules/tesla,1.3e-34*joules/tesla)

// ------physico-chemical_constants
/// Avogadro constant: (N_A,quantity<inverse_amount>,6.022140857e23/mole,7.4e15/mole);
/// atomic mass constant: (m_u,quantity<mass>,1.660539040e-27*kilograms,2.0e-35*kilograms);
/// Faraday constant: (F,quantity<electric_charge_over_amount>,96485.33289*coulombs/mole,5.9e-4*coulombs/mole);
/// molar gas constant: (R,quantity<energy_over_temperature_amount>,8.3144598*joules/kelvin/mole,4.8e-06*joules/kelvin/mole);
/// Boltzmann constant: (k_B,quantity<energy_over_temperature>,1.38064852e-23*joules/kelvin,7.9e-30*joules/kelvin);
/// Stefan-Boltzmann constant: (sigma_SB,quantity<power_over_area_temperature_4>,5.670367e-8*watts/square_meter/pow<4>(kelvin),1.3e-13*watts/square_meter/pow<4>(kelvin));
/// first radiation constant: (c_1,quantity<power_area>,3.741771790e-16*watt*square_meters,4.6e-24*watt*square_meters);
/// first radiation constant for spectral radiance: (c_1L,quantity<power_area_over_solid_angle>,1.191042953e-16*watt*square_meters/steradian,1.5e-24*watt*square_meters/steradian);
/// second radiation constant: (c_2,quantity<length_temperature>,1.43877736e-2*meter*kelvin,8.3e-9*meter*kelvin);
/// Wien displacement law constant : lambda_max T: (b,quantity<length_temperature>,2.8977729e-3*meter*kelvin,1.7e-9*meter*kelvin);
/// Wien displacement law constant : nu_max/T: (b_prime,quantity<frequency_over_temperature>,5.8789238e10*hertz/kelvin,3.4e4*hertz/kelvin);

// ------universal_constants
/// speed of light: (c,quantity<velocity>,299792458.0*meters/second,0.0*meters/second);
/// magnetic constant (exactly 4 pi x 10^(-7) - error is due to finite precision of pi): (mu_0,quantity<force_over_current_squared>,12.56637061435917295385057353311801153679e-7*newtons/ampere/ampere,0.0*newtons/ampere/ampere);
/// electric constant: (epsilon_0,quantity<capacitance_over_length>,8.854187817620389850536563031710750260608e-12*farad/meter,0.0*farad/meter);
/// characteristic impedance of vacuum: (Z_0,quantity<resistance>,376.7303134617706554681984004203193082686*ohm,0.0*ohm);
/// Newtonian constant of gravitation: (G,quantity<volume_over_mass_time_squared>,6.67428e-11*cubic_meters/kilogram/second/second,6.7e-15*cubic_meters/kilogram/second/second);
/// Planck constant: (h,quantity<energy_time>,6.62606896e-34*joule*seconds,3.3e-41*joule*seconds);
/// Dirac constant: (hbar,quantity<energy_time>,1.054571628e-34*joule*seconds,5.3e-42*joule*seconds);
/// Planck mass: (m_P,quantity<mass>,2.17644e-8*kilograms,1.1e-12*kilograms);
/// Planck temperature: (T_P,quantity<temperature>,1.416785e32*kelvin,7.1e27*kelvin);
/// Planck length: (l_P,quantity<length>,1.616252e-35*meters,8.1e-40*meters);
/// Planck time: (t_P,quantity<time>,5.39124e-44*seconds,2.7e-48*seconds);
}
}
```

#### CGS System

```c++
namespace cgs {
// system
using system = make_system<centimeter_base_unit, gram_base_unit, si::second_base_unit, biot_base_unit>::type;
using dimensionless = unit<dimensionless_type,system>;

// units
using acceleration = unit<acceleration_dimension,cgs::system>;
constexpr static acceleration gal<s>;
using area = unit<area_dimension,cgs::system>;
constexpr static area square_centi{meter,metre}<s>;
using current = unit<current_dimension,cgs::system>;
constexpr static current biot<s>;
static constexpr dimensionless cgs_dimensionless;
using dynamic_viscosity = unit<dynamic_viscosity_dimension,cgs::system>;
static constexpr dynamic_viscosity poise;
using energy = unit<energy_dimension,cgs::system>;
constexpr static energy erg<s>;
using force = unit<force_dimension,cgs::system>;
constexpr static force dyne<s>;
using frequency = unit<frequency_dimension,cgs::system>;
using kinematic_viscosity = unit<kinematic_viscosity_dimension,cgs::system>;
constexpr static kinematic_viscosity stoke<s>;
using length = unit<length_dimension,cgs::system>;
constexpr static length centi{meter,metre}<s>;
using mass = unit<mass_dimension,cgs::system>;
constexpr static mass gram<me><s>;
using mass_density = unit<mass_density_dimension,cgs::system>;
using momentum = unit<momentum_dimension,cgs::system>;
using power = unit<power_dimension,cgs::system>;
using pressure = unit<pressure_dimension,cgs::system>;
constexpr static pressure barye<s>;
using time = unit<time_dimension,cgs::system>;
constexpr static time second<s>;
using velocity = unit<velocity_dimension,cgs::system>;
constexpr static velocity centi{meter,metre}<s>_per_second;
using volume = unit<volume_dimension,cgs::system>;
constexpr static volume cubic_centi{meter,metre}<s>;
using wavenumber = unit<wavenumber_dimension,cgs::system>;
constexpr static wavenumber kayser<s>, reciprocal_centi{meter,metre}<s>;

// io: for (NAME,N,S)
std::string name_string(const reduce_unit<si::{NAME}>::type&) { return "{N}"; }
std::string symbol_string(const reduce_unit<si::{NAME}>::type&) { return "{S}"; }
// (acceleration,galileo,Gal), (current,biot,Bi), (dynamic_viscosity,poise,P),
// (energy,erg,erg), (force,dyne,dyn), (kinematic_viscosity,stoke,St),
// (pressure,barye,Ba), (wavenumber,kayser,K)
}
```

#### Trigonometry and Angle System

```c++
namespace degree {
using system = make_system<angle::degree_base_unit>::type;
using dimensionless = unit<dimensionless_type,system>;
using plane_angle = unit<plane_angle_dimension,system>;
constexpr static plane_angle degree<s>;
}

namespace gradian {
using system = make_system<angle::gradian_base_unit>::type;
using dimensionless = unit<dimensionless_type,system>;
using plane_angle = unit<plane_angle_dimension,system>;
constexpr static plane_angle gradian<s>;
}

namespace revolution {
using system = make_system<angle::revolution_base_unit>::type;
using dimensionless = unit<dimensionless_type,system>;
using plane_angle = unit<plane_angle_dimension,system>;
constexpr static plane_angle revolution<s>;
}
```

#### Temperature System

```c++
namespace celsius {
using system = make_system<temperature::celsius_base_unit>::type;
using temperature = unit<temperature_dimension,system>;
constexpr static temperature degree<s>;
}

namespace fahrenheit {
using system = make_system<temperature::fahrenheit_base_unit>::type;
using temperature = unit<temperature_dimension,system>;
constexpr static temperature degree<s>;
}
```

#### Information System

```c++
namespace information {
using system = make_system<byte_base_unit>::type;
using dimensionless = unit<dimensionless_type,system>;

using hu::bit::info = unit<information_dimension, make_system<bit_base_unit>::type>;
constexpr static hu::bit::info bit<s>;
using hu::byte::info = unit<information_dimension,system>;
constexpr static hu::byte::info byte<s>;
using hu::hartley::info = unit<information_dimension, make_system<hartley_base_unit>::type>;
constexpr static hu::hartley::info hartley<s>;
using hu::nat::info = unit<information_dimension, make_system<nat_base_unit>::type>;
constexpr static hu::nat::info nat<s>;
using hu::shannon::info = unit<information_dimension, make_system<shannon_base_unit>::type>;
constexpr static hu::shannon::info shannon<s>;

// prefixes: for (EXP,NAME):
using {NAME}_pf_type = make_scaled_unit<dimensionless, scale<2,static_rational<{EXP}>>>::type;
constexpr static {NAME}_pf_type {NAME};
// (10,kibi), (20,mebi), (30,gibi), (40,tebi), (50,pebi), (60,exbi), (70,zebi), (80,yobi)
}
```

#### Abstract System

```c++
namespace abstract {
struct length_unit_tag : base_unit<length_unit_tag, length_dimension, -30> { };
struct mass_unit_tag : base_unit<mass_unit_tag, mass_dimension, -29> { };
struct time_unit_tag : base_unit<time_unit_tag, time_dimension, -28> { };
struct current_unit_tag : base_unit<current_unit_tag, current_dimension, -27> { };
struct temperature_unit_tag : base_unit<temperature_unit_tag, temperature_dimension, -26> { };
struct amount_unit_tag : base_unit<amount_unit_tag, amount_dimension, -25> { };
struct luminous_intensity_unit_tag : base_unit<luminous_intensity_unit_tag, luminous_intensity_dimension, -24> { };
struct plane_angle_unit_tag : base_unit<plane_angle_unit_tag, plane_angle_dimension, -23> { };
struct solid_angle_unit_tag : base_unit<solid_angle_unit_tag, solid_angle_dimension, -22> { };

using system = make_system<length_unit_tag,mass_unit_tag,time_unit_tag,current_unit_tag,temperature_unit_tag,
    amount_unit_tag,luminous_intensity_unit_tag,plane_angle_unit_tag,solid_angle_unit_tag>::type;

using length = unit<length_dimension,system>;
using mass = unit<mass_dimension,system>;
using time = unit<time_dimension,system>;
using current = unit<current_dimension,system>;
using temperature = unit<temperature_dimension,system>;
using amount = unit<amount_dimension,system>;
using luminous_intensity = unit<luminous_intensity_dimension,system>;
using plane_angle = unit<plane_angle_dimension,system>;
using solid_angle = unit<solid_angle_dimension,system>;

// for each (UNIT,NAME,SYM):
struct base_unit_info<abstract::{UNIT}_unit_tag> { static std::string name(), symbol(); } // "[{NAME}]", "[{SYM}]"
}
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
