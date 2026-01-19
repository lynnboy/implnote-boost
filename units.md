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
    ctor<YY>(const self<Unit,YY>& src); self& operator= <YY>(const self<Unit,YY)& src>);
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
### Unit Systems

#### SI System

```c++
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
