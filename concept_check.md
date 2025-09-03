# Boost.ConceptCheck

* lib: `boost/libs/concept_check`
* repo: `boostorg/concept_check`
* commit: `37c9bdd`, 2022-11-10

------
### Concept Check

Header `<boost/concept_check.hpp>`

#### Concept Assertion

Header `<boost/concept/assert.hpp>`

* Macro `BOOST_CONCEPT_ASSERT((Model))`.
* Force instantiation of `Model::~Model()`.

#### Concept Requirement

Header `<boost/concept/requires.hpp>`

* Macro `BOOST_CONCEPT_REQUIRES(((Model))((Model2)),(type))`
* Asserts on every Model, then results in the `type`.
* Can be used as function return type, similar to `enable_if`.

#### Concept Modeling

Header `<boost/concept/usage.hpp>`

* Macro `BOOST_CONCEPT_USAGE(Model)`, used in concept class with class name as `Model`.
* Expands to assertion of instantiation of destructor, then the head of destructor `~Model()`.
* A concept is a template class
  * Inherit other concept classes as `requires` (refinement)
  * Add `BOOST_CONCEPT_ASSERT` as needed.
  * Add a `BOOST_CONCEPT_USAGE(Concept)`, or define destructor, to assert for valid expressions.

------
### Predefined Concepts

Header `<boost/concept_check.hpp>`

* `Integer`, `SignedInteger`, `UnsignedInteger`
* `DefaultConstructible`, `Assignable`, `CopyConstructible`, `SGIAssignable`, `Convertible`
* `EqualityComparable`, `LessThanComparable`, `Comparable`
* `EqualOp`, `NotEqualOp`, `LessThanOp`, `LessEqualOp`, `GreaterThanOp`, `GreaterEqualOp`
* `PlusOp`, `TimesOp`, `DivideOp`, `SubtractOp`, `ModOp`
* `Generator`, `UnaryFunction`, `BinaryFunction`, `UnaryPredicate`, `[Const_]BinaryPredicate`
* `AdaptableGenerator`, `AdaptableUnaryFunction`, `AdaptableBinaryFunction`, `AdaptablePredicate`, `AdaptableBinaryPredicate`
* `InputIterator`, `OutputIterator`, `[Mutable_]ForwardIterator`, `[Mutable_]BidirectionalIterator`, `[Mutable_]RandomAccessIterator`
* `[Mutable_]Container`, `[Mutable_]ForwardContainer`, `[Mutable_]ReversibleContainer`, `[Mutable_]RandomAccessContainer`
* `Sequence`, `FrontInsertionSequence`, `BackInsertionSequence`
* `AssociativeContainer`, `UniqueAssociativeContainer`, `MultipleAssociativeContainer`, `SimpleAssociativeContainer`, `PairAssociativeContainer`, `SortedAssociativeContainer`
* `Collection`

------
### Predefined Concept Archetypes

Archetypes are used to check agains generic code obey its documented concept requirements.
Naming convention is `concept_name_archetype`.

* `null_archetype` and `boolean_archetype`
* `default_constructible`, `assignable`, `copy_constructible`, `sgi_assignable`, `convertible_to`, `convertible_from`
* `equality_comparable`, `equality_comparable2_{first|second}`, `less_than_comparable`, `comparable`
* `equal_op_{first|second}`, `not_equal_op_{first|second}`, `less_than_op_{first|second}`, `less_equal_op_{first|second}`, `greater_than_op_{first|second}`, `greater_equal_op_{first|second}`
* `addable`, `subtractable`, `multipliable`, `dividable`, `modable`
* `plus_op_{first|second}`, `time_op_{first|second}`, `divide_op_{first|second}`, `subtract_op_{first|second}`, `mod_op_{first|second}`
* `generator`, `void_generator`, `unary_function`, `binary_function`, `unary_predicate`, `binary_predicate`
* `input_iterator`, `input_iterator_archetype_no_proxy`, `output_iterator`, `input_output_iterator`
* `[mutable_]forward_iterator`, `[mutable_]bidirectional_iterator`, `[mutable_]random_access_itertor`

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.Core

* `<boost/utility/enable_if.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_same.hpp>`, `<boost/type_traits/is_void.hpp>`
* `<boost/type_traits/conversion_traits.hpp>`
* `<boost/type_traits/integral_constant.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/seq/for_each.hpp>`
* `<boost/preprocessor/seq/for_each_i.hpp>`
* `<boost/preprocessor/seq/enum.hpp>`
* `<boost/preprocessor/cat.hpp>`
* `<boost/preprocessor/comma_if.hpp>`

------
### Standard Facilities

* Proposals:
  * N4361 - Concept Lite TS
