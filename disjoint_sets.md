# Boost.DisjointSets

* lib: `boost/libs/disjoint_sets`
* repo: `boostorg/disjoint_sets`
* commit: `b322618c`, 2017-01-12

------
### disjoint_sets

#### Header

* `<boost/pending/disjoint_sets.hpp>`

#### Class `boost::disjoint_sets`

```c++
template <class RankPA, class ParentPA,
  class FindCompress = find_with_full_path_compression>
class disjoint_sets;
```

##### Template parameters

* `RankPA`: type of property accessor on element for rank value.
* `ParentPA`: type of property accessor on element for parent value.
* `FindCompress`: functor for the key operation of disjoint sets.
  * `find_with_path_halving`
  * `find_with_full_path_compression`

##### Constructors

* `disjoint_sets(RankPA r, ParentPA p)`
* `disjoint_sets(const disjoint_sets&)`

##### Methods

* `make_set(Element x)`
* `link(Element x, Element y)`
* `union_set(Element x, Element y)`
* `find_set(Element x)`
* `count_sets(first, last) -> std::size_t`
* `normalize_sets(first, last)`
* `compress_sets(first, last)`

------
#### Class `boost::disjoint_sets_with_storage`

```c++
template <class ID = identity_property_map,
  class InverseID = identity_property_map,
  class FindCompress = find_with_full_path_compression>
class disjoint_sets_with_storage;
```

##### Template parameters

* `ID`: type to get index from vertex value.
* `ParentPA`: type to get vertex from index.
* `FindCompress`: functor for the key operation of disjoint sets.

##### Constructors

* `disjoint_sets_with_storage(n = 0, id = ID(), inv = InverseID())`

##### Methods

Same as class `disjoint_sets`.

------
### Dependency

#### Boost.Graph

* `<boost/graph/properties.hpp>`, which depends on Boost::PropertyMap.
