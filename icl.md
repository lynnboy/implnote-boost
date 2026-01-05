# Boost.ICL

* lib: `boost/libs/icl`
* repo: `boostorg/icl`
* commit: `e2b50ef`, 2025-09-19

------
#### Concepts

##### Naming

* Element set: `std::set`
* Element map: `icl::map`
* Interval set: `interval_set`, `separate_interval_set`, `split_interval_set`
* Interval map: `interval_map`, `split_interval_map`

##### Aspects

* Fundamental
* Segmental

##### Sets and Maps

* Empty set/map: default constructed
* Subset relation: `bool T::within(const T& t1, const T& t2) const`
* Equality: `bool is_element_equal(const T& t1, const T& t2)`
* Set union: `+=`, `+`
* Set difference: `-=`, `-`
* Set intersection: `&=`, `&`

##### Addability, Subtractability, Aggregate on overlap

* `Combine`: `inplace_plus`, `inplace_minus`, `inplace_et`, `inplace_caret`, `inplace_star`,
    `inplace_slash`, `inplace_max`, `inplace_min`, `inplace_identity`, `inplace_erasure`
* `inverse` on `inplace_xx`: `plus`<->`minus`, `et`<->`caret`, `star`<->`slash`,
    `max`<->`min`, `identity`<->`erasure`, Functor<->`erasure`

#### Semantics

##### Orderings & Equivalences

* Lexicographical: `<` (strict weak ordering) `Compare`, induced: `==`
* Subset ordering, element equality: `contained_in`, `is_element_equal`

##### Sets

* union: `+`, `+=`, `|`, `|=`, associativity, neutrality, commutativity
* intersection: `&`, `&=`: associativity, commutativity
* difference: `-`, `-=`: right neutrality, inversion
* distributivity: distributivity, right distributivity
* demorgan's
* symmetric difference

##### Maps

* Collectors: Maps of Sets
* Quantifiers: Maps of Numbers

#### Interfaces

`D`=`DomainT`, `C`=`CodomainT`, `T`=`Traits`, `CP`=`Compare`, `CB`=`Combine`, `Sn`=`Section`, `I`=`IntervalT`, `A`=`Alloc`,
`SZ`=`size<D>`, `DF`=`difference<D>`, `XL`=`exclusive_less<I<D,CP>>`, `INV`=`inverse<CB>`, `(T,U)`=`std::pair<T,U>`,
`ES`=element set type, `IS`=interval set type, `S`=set type, `EM`=element map type (`icl::map`), `IM`=interval map type, `M`=map type,
`SE`=`S::element_type`, `SS`=`IS::segment_type`, `ME`=`M::element_type`, `MS`=`IM::segment_type`,
`DT`=discrete type (integral, etc), `CT`=continuous type (floating-point etc.)

##### Class templates

* Intervals
 * static: `right_open_interval`, `left_open_interval`, `open_interval`, `closed_interval`
 * dynamic: `discrete_interval`, `continuous_interval`
 * default type:
  - if `USE_STATIC_BOUNDED_INTERVALS`, `right_open_interval`
  - else: `discrete_interval` or `continuous_interval`
* Interval Sets: `interval_set`, `separate_interval_set`, `split_interval_set`
 * Template Parameters: `D`, `C=std::less`, `I=interval<D,C>::type`, `A=std::allocator`
* Map: `icl::map`
 * Template Parameters: `D`, `C`, `T=identity_absorber`, `CP=std::less`, `CB=inplace_plus`, `Sn=icl::inplace_et`, `A=std::allocator`
* Interval Maps: `interval_map`, `split_interval_map`
 * Template Parameters: `D`, `C`, `T=identity_absorber`, `CP=std::less`, `CB=inplace_plus`, `Sn=icl::inplace_et`, `I=interval<D,C>::type`, `A=std::allocator`

##### Concepts

```c++
concept Domain<D,C> = regular<D> && strict_weak_ordering<D,C> && (is_incrementable<D> || has_unit_element<D>);
concept is_integral<D> = is_incrementable<D> && is_decrementable<D>;

concept Interval<I<D,C>> = is_exclusive_less_comparable<I<D,C>>;

concept Codomain<C,CB,S> = regular<C> && combinable<C,CB> && combinable<C,inverse<CB>> && intersectable<C,S>;
```

##### Associated types

* Data (fundamental):
 * `domain_type`: `D`
 * `codomain_type`: `C` for maps, otherwise `D`
 * `element_type`: `(D,C)` for maps, otherwise `D`
 * `segment_type`: `i<D,CP>` for intervals and interval sets, `(i<D,CP>,C)` for interval maps
 * `size_type`: `SZ<D>`
 * `difference_type`: `DF<D>` for intervals and interval sets/maps, `SZ<D>` for element sets/maps
* Data (segmental):
 * `key_type`: `D` for intervals and element sets/maps, `i<D,CP>` for interval sets/maps
 * `data_type`: `D` for intervals and element sets, `i<D,CP>` for interval sets, `C` for maps
 * `value_type`: `D` for intervals and element sets, `i<D,CP>` for interval sets, `(i<D,CP>,C)` for interval maps, `(D,C)` for element maps
 * `interval_type`: `i<D,CP>` for intervals and interval sets/maps, not provided for element sets/maps
 * `allocator_type`: `a<value_type>` for sets and maps, not provided for intervals
* Ordering (fundamental):
 * `domain_compare`: `CP<D>`
* Ordering (segmental):
 * `key_compare`: `CP<D>` for intervals and element sets/maps, `XL` for interval sets/maps
 * `interval_compare`: `XL` for interval sets/maps; otherwise not provided
* Aggregation (fundamental) (maps only):
 * `codomain_combine`: `CB<C>`
 * `inverse_codomain_combine`: `INV<CB<C>>`
 * `codomain_intersect`: `Sn<C>`
 * `inverse_codomain_intersect`: `INV<Sn<C>>`

##### Functions for intervals, interval sets, interval maps, element sets, and element maps

* Special members:
 * def-ctor: provided by all
 * value ctor: interval sets/maps: from `elemnt_type`, `segment_type` and copy-ctor; other: copy-ctor
 * value assign: copy-assign provided by all
 * swap: all sets/maps
* Containedness:
 * `T::empty`: all sets/maps
 * `is_empty`: all
 * `contains` and `within`: for all supported (`element_type`, `segment_type`, etc.)
* Equivalences and Ordering:
 * relational ops: all
 * `is_element_equal`, `_less`, `_greater`: all sets/maps
 * `is_distinct_equal`: maps
* Size:
 * `T::size`: all sets/maps
 * `size`: all
 * `cardinality`: all
 * `length`: intervals and interval sets/maps
 * `interactive_size`: all sets/maps
 * `interval_count`: interval sets/maps
* Selection:
 * `T::find`: interval sets/maps: provided for `element_type` and `segment_type`, element sets/maps: provided 2 overloads
 * `find`: interval sets/maps: provided for `element_type` and `segment_type`
 * `[]`: element maps
 * `()`: maps
* Range:
 * `I hull(const T&)`: interval sets/maps
 * `T hull(const T&, const T&)`: intervals
 * `lower` and `upper`: intervals and interval sets/maps
 * `first` and `last`: intervals and interval sets/maps
* Addition:
 * `T::add(const P&)`: interval sets/maps: provided for `element_type` and `segment_type`, element maps provided for `element_type`
 * `add(T&, const P&)`: interval sets/maps: provided for `element_type` and `segment_type`, element sets/maps provided for `element_type`
 * `T::add(It p, const P&)`: interval sets/maps: provided for `segment_type`, element maps provided for `element_type`
 * `add(T&, It p, const P&)`: interval sets/maps: provided for `segment_type`, element sets/maps provided for `element_type`
 * `+=`, `+`, `|=`, `|`: all sets/maps for all supported types
* Subtraction:
 * `T::subtract(const P&)`: interval sets/maps: provided for `element_type` and `segment_type`, element maps provided for `element_type`
 * `subtract(T&, const P&)`: interval sets/maps: provided for `element_type` and `segment_type`, element sets/maps provided for `element_type`
 * `-=`, `-`, all sets/maps
 * `left_subtract(T, const T&)`, `right_subtract`: intervals
* Insertion:
 * `T::insert(const P&)`, `insert(T&, const P&)`: interval sets/maps: provided for `element_type` and `segment_type`, element sets/maps provided for `element_type`
 * `T::insert(It p, const P&)`, `insert(T&, It p, const P&)`: interval sets/maps: provided for `segment_type`, element sets/maps provided for `element_type`
 * `insert`, all sets/maps
 * `T::set`, `set_at`: interval maps: for `element_type` and `segment_type`, element maps: provided
* Erasure:
 * `T::clear`, `clear`: all sets/maps
 * `T::erase`, `erase`: all sets/maps for all supported types
 * `T::erase` for iter/iter-pair: all sets/maps
* Intersection:
 * `add_intersection`: interval sets/maps for all supported types
 * `&=`: all sets/maps for all supported types
 * `&`, `intersects`, `disjoint`: all types for all supported types
* Symmetric difference:
 * `T::flip`, `flip`, `^=`, `^`: sets/maps
* Iteration:
 * `T::begin`, `T::end`, `T::rbegin`, `T::rend`: all maps/sets
 * `T::lower_bound`, `T::upper_bound`, `T::equal_range`: all maps/sets
* Element iteration:
 * `elements_begin`, `elements_end`, `elements_rbegin`, `elements_rend`: interval sets/maps
* Streaming:
 * `<<`: all

##### Functions for interval types

* Construction:
 * `singleton`: all
 * `construct(p1, p2)`: all, ro and lo intervals for both `DT` and `CT` args
 * `construct(p1, p2, interval_bounds)`: dynamic intervals
 * `hull(p1, p2)`, `span(p1, p2)`: all, ro and lo intervals for both `DT` and `CT` args
 * `right_open`, `left_open`, `closed`, `open`: dynamic intervals
* Orderings:
 * `exclusive_less`, `lower_less`, `lower_equal`, `lower_less_equal`, `upper_less`, `upper_equal`, `upper_less_equal`: all
* Misc:
 * `touches`: all
 * `inner_complement`: all
 * `distance`: all

#### Customization for intervals

Template `interval_traits`:
* types: `interval_type`, `domain_type`, `domain_compare`
* functions: `construct`, `lower`, `upper`
* meta-function: `interval_bound_type<interval_type>`

------
### Type Traits

```c++
struct absorbs_identities<T>; // default is false
struct difference<T>; // default is T
struct size<T>; // default is size_t
struct is_continuous_interval<T>; // default is false
struct is_discrete_interval<T>; // default is false
struct is_interval_container<T>; // default is false
struct is_interval_joiner<T>; // default is false
struct is_interval_separator<T>; // default is false
struct is_interval_splitter<T>; // default is false
struct is_map<T>; // default is false
struct is_set<T>; // default is is_std_set<T>
struct is_total<T>; // default is false

struct no_type{};
struct property<T> { using argument_type=T; using result_type=bool; };
struct member_property<T> : property<T>{ using mptr = bool (T::*)()const;
    ctor(mptr pred): m_pred{pred}{}
    bool operator()(const T& x)const { return (x.*m_pred)(); }
    mptr m_pred;
};
struct relation<L,R> { using first_argument_type=L; using second_argument_type=R; using result_type=bool; };

struct interval_bound_type<T>; // default is interval_bounds::undefined
struct is_interval<T>; // interval_bound_type<T> < undefined
struct has_static_bounds<T>; // interval_bound_type<T> < dynamic
struct has_dynamic_bounds<T>; // interval_bound_type<T> == dynamic
struct has_asymmetric_bounds<T>; // b = interval_bound_type<T>; value = (b==static_left_open||b==static_right_open)
struct has_symmetric_bounds<T>; // b = interval_bound_type<T>; value = (b==static_closed||b==static_open)
struct is_discrete_static<T>; // has_static_bounds<T> && is_discrete<interval_traits<T>::domain_type>
struct is_continuous_static<T>; // has_static_bounds<T> && is_continuous<interval_traits<T>::domain_type> && has_asymmetric_bounds<T>
struct is_static_right_open<T>; // interval_bound_type<T> == static_right_open
struct is_static_left_open<T>; // interval_bound_type<T> == static_left_open
struct is_static_open<T>; // interval_bound_type<T> == static_open
struct is_static_closed<T>; // interval_bound_type<T> == static_closed
struct is_discrete_static_closed<T>; // is_static_closed<T> && is_discrete<interval_traits<T>::domain_type>
struct is_discrete_static_open<T>; // is_static_open<T> && is_discrete<interval_traits<T>::domain_type>
struct is_continuous_right_open<T>; // is_static_right_open<T> && is_continuous<interval_traits<T>::domain_type>
struct is_continuous_left_open<T>; // is_static_left_open<T> && is_continuous<interval_traits<T>::domain_type>
struct is_singelizable<T>; // has_dynamic_bounds<T> || is_discrete<interval_traits<T>::domain_type>

struct has_domain_type<T>; // detects T::domain_type
struct domain_type_of<T>; // has_domain_type<T> ? T::domain_type : no_type

struct has_codomain_type<T>; // detects T::codomain_type
struct codomain_type_of<T>; // has_codomain_type<T> ? T::codomain_type : is_std_set<T> ? T::value_type : no_type

struct has_difference_type<T>; // detects T::difference_type
struct is_subtraction_closed<T>; // is_numeric<T> || (has_rep_type<T> && !has_difference_type<T>)
struct has_difference<T>; // is_subtraction_closed<T> || is_pointer<T> || has_difference_type<T>
struct difference_type_of<T>; // has_difference_type<T> ? T::difference_type : has_difference<T> ? (T or ptrdiff_t if T is ptr) : no_type

struct has_element_type<T>; // detects T::element_type
struct element_type_of<T>; // has_element_type<T> ? T::element_type : no_type

struct has_value_type<T>; // detects T::value_type
struct value_type_of<T>; // has_value_type<T> ? T::value_type : no_type

struct has_key_type<T>; // detects T::key_type
struct key_type_of<T>; // has_key_type<T> ? T::key_type : no_type

struct has_interval_type<T>; // detects T:: interval_type
struct interval_type_of<T>; // has_interval_type<T> ? T::interval_type : no_type

struct has_key_object_type<T>; // detects T::key_object_type
struct key_container_type_of<T>; // has_key_object_type<T> ? T::key_objec_type : (is_set<T>||is_map<T>) ? T : no_type;
struct is_strict_key_container_of<Key,Obj>; // is_map<Obj> && is_same<Key,key_container_type_of<Obj>>
struct is_key_container_of<Key,Obj>; // is_strict_key_container_of<Key,Obj> || ( (is_set<Obj>||is_map<Obj> && is_same<Obj,Key>) )

struct has_rep_type<T>; // detects T::rep_type
struct represents<Rep,T>; // has_rep_type<T> && is_same<T::rep,Rep>
struct rep_type_of<T>; // has_rep_type<T> ? T::rep : no_type

struct has_segment_type<T>; // detects T::segment_type
struct segment_type_of<T>; // has_segment_type<T> ? T::segment_type : no_type

struct has_size_type<T>; // detects T::size_type
struct size_type_of<T>; // has_size_type<T> ? T::size_type : has_difference_type<T> ? T::difference_type : has_rep_type<T> ? T : size_t

struct has_inverse<T>; // is_signed<T> || is_floating_point<T>
struct adds_inversely<T,Combiner>; // has_inverse && is_negative<Combiner>
struct has_set_semantics<T>; // is_set<T> || (is_map<T> && has_set_semantics<codomain_type_of<T>>)
struct is_associative_element_container<T>; // is_element_set<T> || is_element_map<T>
struct is_asymmetric_interval<T>; // is_interval<T> && has_static_bounds<T> && has_asymmetric_bounds<T>
struct is_continuous_asymmetric<T>; // is_asymmetric_interval<T> && is_continuous<domain_type_of<interval_traits<T>>>
struct is_discrete_asymmetric<T>; // is_asymmetric_interval<T> && !is_continuous<domain_type_of<interval_traits<T>>>
struct is_overloadable<T>; // is_same<T,T::overloadable_type>
struct is_codomain_equal<L,R>; // is_same<L::codomain_type, R::codomain_type>
struct is_key_compare_equal<L,R>; // is_same<L::key_compare, R::key_compare>
struct is_codomain_type_equal<L,R>; // is_key_compare_equal<L,R> && is_codomain_equal<L,R>
struct is_concept_compatible<IsConcept,L,R>; // IsConcept<L> && IsConcept<R> && is_codomain_type_equal<L,R>
struct is_concept_combinable<LConc,RConc,L,R>; // LConc<L> && RConc<R> && is_key_compare_equal<L,R>
struct is_intra_combinable<L,R>; // is_concept_compatible<is_interval_set,L,R> || is_concept_compatible<is_interval_map,L,R>
struct is_cross_combinable<L,R>; // is_concept_combinable<is_interval_set,is_interval_map,L,R> || is_concept_combinable<is_interval_map,is_interval_set,L,R>
struct is_inter_combinable<L,R>; // is_intra_combinable<L,R> || is_cross_combinable<L,R>
struct is_fragment_of<Frag,T>; // true for T::element_type and T::segment_type, otherwise false
struct is_key_of<Key,T>; // true for T::domain_type and T::interval_type, otherwise false
struct is_container<T>; // detect value_type, iterator, size_type, reference
struct is_std_set<T>; // is_container<T> && has_key_type<T> && is_same<key_type_of<T>,value_type_of<T>> && !has_segment_type<T>
struct is_continuous<T>; // !is_discrete<T>
struct is_discrete<T>; // is_incrementable<T> && ( (!has_rep_type<T> && is_non_floating_point<T>) || (has_rep_type<T> && is_discrete<rep_type_of<T>>) )
struct is_element_map<T>; // is_map<T> && !is_interval_container<T>
struct is_element_set<T>; // (is_set<T> && !is_interval_container<T>) || is_std_set<T>
struct is_element_container<T>; // is_element_set<T> || is_element_map<T>
struct is_icl_container<T>; // is_element_container<T> || is_interval_container<T>
struct is_incrementing<Domain,Compare>; // true
struct is_incrementing<Domain,std::greater<Domain>>; // false
struct is_interval_map<T>; // is_interval_container<T> && is_map<T>
struct is_interval_set<T>; // is_interval_container<T> && !is_interval_map<T>

struct is_fixed_numeric<T>; // 0 < std::numeric_limits<T>::digits
struct is_std_numeric<T>; // std::numeric_limits<T>::is_specialized
struct is_std_integral<T>; // std::numeric_limits<T>::is_integer
struct is_numeric<T>; // is_std_numeric<T> || is_integral<T> || is_std_integral<T>
struct is_numeric<std::complex<T>>; // true
struct numeric_minimum<T,Comp,_=false> {
    static bool is_less_than(T){return true;}
    static bool is_less_than_or(T,bool){return true;}
};
struct numeric_minimum<T,std::less<T>,true> {
    static bool is_less_than(T v){return std::less<T>()(std::numeric_limits<T>::min(), v);}
    static bool is_less_than_or(T v,bool cond){return cond||is_less_than(v);}
};
struct numeric_minimum<T,std::greater<T>,true> {
    static bool is_less_than(T v){return std::greater<T>()(std::numeric_limits<T>::max(), v);}
    static bool is_less_than_or(T v,bool cond){return cond||is_less_than(v);}
};
struct is_non_floating_point<T>; // !is_floating_point<T>

struct is_interval_set_derivative<T,Assoc>; // is_interval_container<T> if Assoc is T::domain_type or T::interval_type, otherwise false
struct is_interval_map_derivative<T,Assoc>; // is_interval_container<T> if Assoc is T::domain_mapping_type, T::interval_mapping_type or T::value_type, otherwise false
struct is_intra_derivative<T,Assoc>; // (is_interval_set<T> && is_interval_set_derivative<T,Assoc>) || (is_interval_map<T>&&is_interval_map_derivative<T,Assoc>)
struct is_cross_derivative<T,Assoc>; // is_interval_map<T> && is_interval_set_derivative<T,Assoc>
struct is_inter_derivative<T,Assoc>; // is_intra_derivative<T,Assoc> || is_cross_derivative<T,Assoc>
struct is_interval_set_right_combinable<Guide,Companion>; // is_interval_set<Guide> && (is_interval_set_derivative<Guide,Companion> || is_concept_compatible<is_interval_set,Guide,Companion>)
struct is_interval_map_right_intra_combinable<Guide,Companion>; // is_interval_map<Guide> && (is_interval_map_derivative<Guide,Companion> || is_concept_compatible<is_interval_map,Guide,Companion>)
struct is_interval_map_right_cross_combinable<Guide,Companion>; // is_interval_map<Guide> && (is_cross_derivative<Guide,Companion> || is_concept_combinable<is_interval_map,is_interval_set,Guide,Companion>)
struct is_interval_map_right_inter_combinable<Guide,Companion>; // is_interval_map_right_intra_combinable<Guide,Companion> || is_interval_map_right_cross_combinable<Guide,Companion>
struct is_right_intra_combinable<Guide,Companion>; // is_interval_set_right_combinable<Guide,Companion> || is_interval_map_right_intra_combinable<Guide,Companion>
struct is_right_inter_combinable<Guide,Companion>; // is_interval_set_right_combinable<Guide,Companion> || is_interval_map_right_inter_combinable<Guide,Companion>
struct combines_right_to_interval_set<Guide,ISet>; // is_concept_combinable<is_interval_container,is_interval_set,Guide,ISet>
struct combines_right_to_interval_map<Guide,IMap>; // is_concept_compatible<is_interval_map,GuideT,IMap>
struct combines_right_to_interval_container<Guide,ICont>; // combines_right_to_interval_set<Guide,ICont> || combines_right_to_interval_map<Guide,ICont>
struct unknown_fineness<T>; // 0
struct known_fineness<T>; // T::fineness;
struct segmentational_fineness<T>; // is_interval_container<T> ? known_fineness<T> : unknown_fineness<T>
struct is_interval_set_companion<Guide,Companion>; // combines_right_to_interval_set<Guide,Companion> || is_interval_set_derivative<Guide,Companion>
struct is_interval_map_companion<Guide,Companion>; // combines_right_to_interval_map<Guide,Companion> || is_interval_map_derivative<Guide,Companion>
struct is_coarser_interval_set_companion<Guide,Companion>; // is_interval_set_companion<Guide,Companion> && (segmentational_fineness<Guide> > segmentational_fineness<Cmopanion>)
struct is_coarser_interval_map_companion<Guide,Companion>; // is_interval_map_companion<Guide,Companion> && (segmentational_fineness<Guide> > segmentational_fineness<Cmopanion>)
struct is_binary_interval_set_combinable<Guide,Companion>; // is_interval_set<Guide> && is_coarser_interval_set_companion<Guide,Companion>
struct is_binary_interval_map_combinable<Guide,Companion>; // is_interval_map<Guide> && is_coarser_interval_map_companion<Guide,Companion>
struct is_binary_intra_combinable<Guide,Companion>; // is_binary_interval_set_combinable<Guide,Companion> || is_binary_interval_map_combinable<Guide,Companion>
struct is_binary_cross_combinable<Guide,Companion>; // is_interval_map<Guide> && (is_coarser_interval_map_companion<Guide,Companion> || is_interval_set_companion<Guide,Companion>)
struct is_binary_inter_combinable<Guide,Companion>; // (is_interval_map<Guide> && is_binary_cross_combinable<Guide,Companion>) || (is_interval_set<Guide> && is_binary_intra_combinable<Guide,Companion>)

struct is_concept_equivalent<IsConcept,L,R>; // IsConcept<L> && IsConcept<R>
struct has_same_concept<IsConcept,L,R>; // IsConcept<L> && is_concept_equivalent<IsConcept,L,R>

struct has_std_infinity<T>; // is_numeric<T> && numeric_limits<T>::has_infinity
struct has_max_infinity<T>; // is_numeric<T> && !numeric_limits<T>::has_infinity
struct numeric_infinity<T> { static T value(); }; // has_std_infinity<T> ? std::numeric_limits<T>::infinity : has_max_infinity<T> ? std::numeric_limits<T>::max() ? T{};
struct infinity<T> { static T value(); }; // is_numeric<T> ? numeric_infinity<T>::value() : has_rep_type<T> ? numeric_infinity<T::rep>::value() :
    // has_size_type<T> ? numeric_infinity<T::size_type>::value() : has_difference_type<T> ? numeric_infinity<T::difference_type>::value() : identity_element<T>::value()
struct infinity<std::string>; // std::string{}

static Inc succ<Inc>(Inc x) { return ++x; }
static Dec pred<Dec>(Dec x) { return ++x; }
struct successor<Domain,Comp> { static Domain apply(Domain v); }; // ++ if increasing, otherwise --
struct predecessor<Domain,Comp> { static Domain apply(Domain v); }; // -- if increasing, otherwise ++

struct identity_element<T> { static T value(); T operator()()const { return value(); }; };
std::string unary_template_to_string<identity_element>::apply() { return "0"; }

struct interval_type_default<D,CP=std::less> {
    using type = is_discrete<D> ? discrete_interval<D,CP> : continuous_interval<D,CP>;
}; // if defined USE_STATIC_BOUNDED_INTERVALS, use static interval types

struct to_string<T>{ static std::string apply(const T& v); }; // stringstream convert

struct type_to_string<T> { static std::string apply(); } // spec for integers & floating-points, std::string, and U<T>, B<T1,T2>

struct unit_element<T> { static T value(); }; // true/1/1.0/" "

struct value_size<T> { static size_t apply(const T& v); }; // abs(v) for int/double, otherwise interative_size(v)
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.ConceptCheck

* `<boost/concept/assert.hpp>`
* `<boost/concept/detail/concept_def.hpp>`, `<boost/concept/detail/concept_undef.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/pragma_message.hpp>`
* `<boost/detail/workaround.hpp>`

#### Boost.Container

* `<boost/container/map.hpp>`
* `<boost/container/set.hpp>`

#### Boost.Core

* `<boost/core/ignore_unused.hpp>`
* `<boost/utility/enable_if.hpp>`

#### Boost.DateTime

* `<boost/date_time/gregorian/gregorian.hpp>`
* `<boost/date_time/posix_time/posix_time.hpp>`

#### Boost.Detail

* `<boost/detail/is_incrementable.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_facade.hpp>`
* `<boost/next_prior.hpp>`

#### Boost.Move

* `<boost/move/move.hpp>`

#### Boost.MPL

* `<boost/mpl/and.hpp>`
* `<boost/mpl/bool.hpp>`
* `<boost/mpl/has_xxx.hpp>`
* `<boost/mpl/if.hpp>`
* `<boost/mpl/not.hpp>`
* `<boost/mpl/or.hpp>`

#### Boost.Range

* `<boost/range/iterator_range.hpp>`

#### Boost.Rational

* `<boost/rational/rational.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits.hpp>`
* `<boost/type_traits/is_const.hpp>`
* `<boost/type_traits/is_floating_point.hpp>`
* `<boost/type_traits/is_pointer.hpp>`
* `<boost/type_traits/is_same.hpp>`
* `<boost/type_traits/is_signed.hpp>`
* `<boost/type_traits/remove_const.hpp>`

#### Boost.Utility

* `<boost/call_traits.hpp>`

------
### Standard Facilities
