# Boost.Bimap

* lib: `boost/libs/bimap`
* repo: `boostorg/bimap`
* commit: `f64de6d`, 2025-01-28

------
#### Commons

```c++
```

bimap, list_of, multiset_of, set_of, unconstrained_set_of, unordered_multiset_of, unordered_set_of, vector_of
views/: {list|unconstrained|vector}_{map|set}_view, <unordered>_<multi>{map|set}_view
tags/: tagged
tags/support/: apply_to_value_type, default_tagged, is_tagged, overwrite_tagged, tag_of, value_type_of
support/: {data|iterator|key|map|value}_type_by, lambda, map_by
relation/: member_at, mutant_relation, pair_layout, structured_pair, symmetrical_base
relation/detail/: access_builder, metadata_access_builder, mutant, static_access_builder, to_mutable_relation_functor
relation/support/: data_extractor, get_pair_functor, get, is_tag_of_member_at, member_with_tag, opposite_tag, pair_by, pair_type_by, value_type_of
property_map/: <unordered>_set_support
detail/: bimap_core, concept_tags, generate_{index|relation|view}_binder, is_et_type, manage_additional_parameters, manage_bimap_key,
	{map|set}_view_{base|iterator}, modifier_adaptor, non_unique_views_helper, user_interface_config
detail/debug/static_error, detail/test/check_metadata
container_adaptor/: <<ordered|unordered>_associative|sequence>_container_adaptor, {list_<map>|vector_<map>|<unordered>_<multi>{map|set}}_adaptor
container_adaptor/detail/: comparison_adaptor, functor_bag, identity_converters, key_extractor, non_unique_container_helper
container_adaptor/support/: iterator_facade_converters

-----
### Configuration

-----
### Dependencies

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.ContainerHash

* `<boost/functional/hash.hpp>`
* `<boost/functional/hash/hash.hpp>`

#### Boost.Core

* `<boost/core/allocator_access.hpp>`
* `<boost/core/enable_if.hpp>`
* `<boost/core/serialization.hpp>`
* `<boost/utility/addressof.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_adaptor.hpp>`
* `<boost/iterator/iterator_traits.hpp>`

#### Boost.Lambda

* `<boost/lambda/lambda.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.MultiIndex

* `<boost/multi_index/*.hpp>`
* `<boost/multi_index_container.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/cat.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

#### Boost.Utility

* `<boost/call_traits.hpp>`
* `<boost/operators.hpp>`

------
### Standard Facilities
