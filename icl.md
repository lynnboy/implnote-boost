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
