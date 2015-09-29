# Boost.ForEach

* lib: `boost/libs/foreach`
* repo: `boostorg/foreach`
* commit: `68645f9c`, 2015-1-22

------
### for each solution

#### Header

* `<boost/foreach.hpp>`

#### API

* Macro `BOOST_FOREACH(var, col)`.
* Macro `BOOST_REVERSE_FOREACH(var, col)`.
* `var` can be declaration or just named variable.
* `col` can be array, STL collection, Boost.Range types, and C-strings.

##### Extensibility

* Trait `boost::foreach::is_lightweight_proxy<T>` and overloadable function
  `::boost_foreach_is_lightweight_proxy(T*&, boost::foreach::tag) -> boost::mpl::bool_`
  to recognize lightweight container type.
* Trait `boost::foreach::is_noncopyable<T>`, by default detect abstract classes and
  classes derived from `boost::noncopyable`.

##### Tricks and Workarounds

* Idiom: Use overloaded functions for meta programming.
* Idiom: Use conditional expression `?:` to make unevaluated expression and get its type.
* If compiler supported, use compile-time rvalue detection for `col` expression,
  otherwise, use run-time rvalue detection.
* If the `col` is rvalue or lightweight proxy type, it is copied before being iterated.
* The `BOOST_FOREACH` have two-level `for` loop, with a flag to support `continue`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`.

#### Boost.Core

* `<boost/noncopyable.hpp>`.
* `<boost/utility/addressof.hpp>`.
* `<boost/utility/enable_if.hpp>`, for runtime rvalue detection.

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`, array, const, rvalue, etc.
* `<boost/aligned._storage.hpp>`, etc., for runtime rvalue detection.

#### Boost.MPL

* `<boost/mpl/if.hpp>`, `<boost/mpl/assert.hpp>`, etc.

#### Boost.Iterator

* `<boost/iterator/iterator_traits.hpp>`, for `iterator_reference`.

#### Boost.Range

* `<boost/range/begin.hpp>`, `<boost/range/end.hpp>`, etc. range iterator support.

------
### Standard Facilities

* Language: Range-based `for` (C++11).
