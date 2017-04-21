# Boost.Iterator

* lib: `boost/libs/iterator`
* repo: `boostorg/iterator`
* commit: `cccbd8c6`, 2017-04-08

------
### Iterator Concepts

Header `<boost/iterator/iterator_concepts.hpp>` and `<boost/iterator/iterator_archetypes.hpp>`

Header `<boost/iterator/iterator_categories.hpp>`

```c++
concept ReadableIterator<I> = Assignable<I> && CopyConstructible<I> &&
  requires(I i) { { *i } -> I::value_type &&; { i.operator->() } -> I::value_type &&; }
concept WritableIterator<I> = CopyConstructible<I> &&
  requires(I i) { { *i = o } }
concept SwappableIterator<I> = CopyConstructible<I> &&
  requires(I a, I b) { { iter_swap(a, b) } -> void; }
concept LvalueIterator<I> = (ReadableIterator<I> || WritableIterator<I>) &&
  requires(I i) { { *i } -> I::value_type &; }

concept IncrementableIterator<I> = Assignable<I> && CopyConstructible<I> &&
  requires(I i) { ++r -> I&; r++; *r++; iterator_traversal<I>::type -> incrementable_traversal_tag }
concept SinglePassIterator<I> = IncrementableIterator<I> && EqualityComparable<I> &&
  requires(I a, I b) { { a == b } -> bool; { a != b } -> bool;
    iterator_traits<I>::difference_type; iterator_traversal<I>::type -> single_pass_traversal_tag } 
concept ForwardTraversal<I> = DefaultConstructible<I> && SinglePassIterator<I> &&
  requires(I i) { iterator_traversal<I>::type -> forward_traversal_tag }
concept BidirectionalTraversal<I> = ForwardTraversal<I> &&
  requires(I i) { { --r } -> I&; { r-- }; iterator_traversal<I>::type -> bidirectional_traversal_tag }
concept RandomAccessTraversal<I> = BidirectionalTraversal<I> &&
  requires(I r, I s, iterator_traits<I>::difference_type n) {
    { r += n } -> I&; { r + n } -> I; { n + r } -> I;
    { r -= n } -> I&; { r - n } -> I; { r - s } -> iterator_traits<I>::difference_type;
    { r[n] } -> I&&; { r[n] = v } -> I&&;
    { r < s } -> bool; { r > s } -> bool; { r >= s } -> bool; { r <= s } -> bool;
    iterator_traversal<I>::type -> random_access_traversal_tag }

concept InteroperableIterators<I,J> = SinglePassIterator<I> && SinglePassIterator<J> &&
  requires(I x, J y) { { y = x } -> J; { Y{x} } -> Y; {x == y} -> bool; { x != y } -> bool; }
concept InteroperableRandomAccessIterators<I,J> =
  RandomAccessTraversal<I> && RandomAccessTraversal<J> &&
  requires(I x, J y) { { x < y } -> bool; { x > y } -> bool; { x >= y } -> bool; { x <= y } -> bool;
    { y - x } -> iterator_traits<J>::difference_type; { x - y } -> iterator_traits<J>::difference_type; }

concept InputIterator<I> = ReadableIterator<I> && SinglePassIterator<I>;
concept ConstantForwardIterator<I> = InputIterator<I> && LvalueIterator<I> && ForwardTraversal<I>;
concept ConstantBidirectionalIteartor<I> = ConstantForwardIterator<I> && BidirectionalTraversal<I>;
concept ConstantRandomAccessIterator<I> = ConstantBidirectionalIteartor<I> && RandomAccessTraversal<I>;

concept OutputIterator<I> = WritableIterator<I> && IncrementableIterator<I>;
concept MutableForwardIterator<I> = OutputIterator<I> && LvalueIterator<I> && ForwardTraversal<I>;
concept MutableBidirectionalIterator<I> = MutableForwardIterator<I> && BidirectionalTraversal<I>;
concept MutableRandomAccessIterator<I> = MutableBidirectionalIterator<I> && RandomAccessTraversal<I>;

// archetypes
struct readable_iterator_t;
struct writable_iterator_t;
struct readable_writable_iterator_t;
struct readable_lvalue_iterator_t;
struct writable_lvalue_iterator_t;
struct swappable_iterator_t;
struct lvalue_iterator_t;

struct access_archetype<Value,AccessCategory>;
struct traversal_archetype<Derived,Value,AccessCategory,TraversalCategory>;
struct iterator_archetype<Value,AccessCategory,TraversalCategory>;
struct iterator_archetype_impl<AccessCategory>;
struct traversal_archetype_base<Value,AccessCategory,TraversalCategory>;

// categories
struct no_traversal_tag;
struct incremental_traversal_tag : no_traversal_tag;;
struct single_pass_traversal_tag : incremental_traversal_tag;
struct forward_traversal_tag : single_pass_traversal_tag;
struct bidirectional_traversal_tag : forward_traversal_tag;
struct random_access_traversal_tag : bidirectional_traversal_tag;
struct iterator_category_to_triversal<Cat>;
struct iterator_triversal<Iter>;
```

------
#### Iterator Traits Meta Functions

Header `<boost/iterator/iterator_traits.hpp>`

```c++
using iterator_value<T>::type = iterator_traits<T>::value_type;
using iterator_reference<T>::type = iterator_traits<T>::reference;
using iterator_pointer<T>::type = iterator_traits<T>::pointer;
using iterator_difference<T>::type = iterator_traits<T>::difference_type;
using iterator_category<T>::type = iterator_traits<T>::iterator_category;
```

Provide MPL meta functions for traits.

------
#### Unified Pointee Type (type of `*e`)

Header `<boost/pointee.hpp>`, `<boost/indirect_reference.hpp>`

```c++
using smart_ptr_pointee<P>::type = P::element_type;
using iterator_pointee<I>::type = is_const<decltype(*declval<I>())> ? const I::value_type : I::value_type;
using pointee<P> = is_incrementable<P> ? smart_ptr_pointee<P> : iterator_pointee<P>;
using indirect_reference<P> = is_incrementable<P> ? iterator_reference<P> : pointee<P>::type &;
```

------
#### Iterator Facade

Header `<boost/iterator/iterator_facade.hpp>`

```c++
using is_interoperable<A,B> = is_convertible<A,B> || is_convertible<B,A>;

// it++ handling
struct postfix_increment_proxy<Iter> { // cached val
  mutable iterator_value<Iter>::type stored_value;
  auto& operator*() const { return stored_value; }
  .ctor(Iter const& x) : stored_value{*x} {}
};
struct writable_postfix_increment_proxy<Iter> { // both as cached val and writer
  mutable iterator_value<Iter>::type stored_value;
  Iter stored_iterator;
  .ctor(Iter const& x) : stored_value{*x}, stored_iterator{x} {}
  auto const& operator*() const { return *this; }
  operator iterator_value<Iter>::type & () const { return stored_value; }
  T [const]& operator= <T> (T [const]& x) const { *stored_iterator = x; return x; } // write
  operator Iter const& () const { return stored_iterator; }
};
using is_non_proxy_reference<Ref,Value> =
  is_convertible<remove_reference_t<Ref> const volatile*, Value const volatile*>;
using postfix_increment_result<Iter, Value, Ref, CatOrTrav> =
  is_convertible<Ref, Value const&> && !is_convertible<CatOrTrav, forward_traversal_tag> ?
  (is_non_proxy_reference<Ref,Value> ? postfix_increment_proxy<Iter>
     : writable_postfix_increment_proxy<Iter>)
  : Iter;

// it->m handling
struct operator_arrow_dispatch<Ref,Ptr> {
  struct proxy {
    Ref m_ref;
    Ref* operator->(); operator Ref*(); // addressof(m_ref)
  };
  static proxy apply(Ref const& x) { return proxy{x}; }
};
struct operator_arrow_dispatch<T&,Ptr> {
  static Ptr apply(T& x) { return addressof(x); }
};

// it[n] handling
struct operator_brackets_proxy<Iter> {
  Iter m_iter;
  operator Iter::reference () const { return *m_iter; }
  auto& operator= (Iter::value_type const& val) { *m_iter = val; return *this; }
};
using operator_bracket_result<Iter,Value,Ref> =
  !(is_copy_constructible<Value> && is_const<Value>) ? operator_brackets_proxy<Iter> : Value;

class iterator_core_access {
  // templatize the members, instiantation failure when derived class doesn't provide them
  F::reference dereference<F>(F const& f) { return f.dereference(); }
  void increment<F>(F& f) { f.increment(); }
  void decrement<F>(F& f) { f.decrement(); }
  bool equal<F1, F2>(F1 const& f1, F2 const& f2, true_) { return f1.equal(f2); }
  bool equal<F1, F2>(F1 const& f1, F2 const& f2, false_) { return f2.equal(f1); } // F2 provides
  void advance<F>(F& f, F::difference_type n) { f.advance(n); }
  F1::difference_type distance_from<F1, F2>(F1 const& f1, F2 const& f2, true_)
    { return -f1.distance_to(f2); }
  F2::difference_type distance_from<F1, F2>(F1 const& f1, F2 const& f2, false_)
    { return f2.distance_to(f1); }

  I [const]& derived<I,V,TC,R,D>(iterator_facade<I,V,TC,R,D> [const]& f)
    { return *static_cast<I [const]*>(&facade); }
};

struct iterator_facade_types<V,TC,R,D> {
  using value_type = remove_const<V>;
  using pointer = V*;
  using iterator_category = ...;
};
struct iterator_facade_base<I,V,TC,R,D,bool isBidi, bool isRandom>;
struct iterator_facade_base<I,V,TC,R,D,false,false> : iterator_facade_types<V,TC,R,D> {
  using reference = Ref;
  using difference_type = Diff;
  using pointer = operator_arrow_dispatch<Ref,iterator_facade_types::pointer>::result_type;
  reference operator*() const { iterator_core_access:dereference(derived()); }
  pointer operator->() const { operator_arrow_dispatch<>::apply(derived()); }
  I& operator++() { iterator_core_access::increment(derived()); return derived(); }
  protected: I [const]& derived() [const] { return *static_cast<I [const]*>(this); }
};
struct iterator_facade_base<I,V,TC,R,D,true,false> : iterator_facade_base<I,V,TC,R,D,false,false> {
  I& operator--() { iterator_core_access::decrement(derived()); return derived(); }
  I operator--(int) { I tmp{derived()}; --*this; return tmp; }
};
struct iterator_facade_base<I,V,TC,R,D,true,true> : iterator_facade_base<I,V,TC,R,D,true,false> {
  operator_brackets_result<I,V,reference>::type operator[](difference_type n) const {
    return operator_brackets_result{derived() + n};
  };
  I& operator+=(difference_type n) { iterator_core_access::advance(derived(), n); return derived(); }
  I& operator-=(difference_type n) { iterator_core_access::advance(derived(), -n); return derived(); }
  I operator-(difference_type n) { I result{derived}; return result -= n; };
};

struct iterator_facade<Iter,Value,CategoryOrTraversal,Ref=Value&,Diff=ptrdiff_t>
  : iterator_facade_base<Iter,Value,CategoryOrTraversal,Ref,Diff,
     is_at_least<bidirectional_traversal_tag>, is_at_least<random_access_traversal_tag>> {
  using iterator_facade_ = iterator_facade<Iter,Value,CategoryOrTraversal,Ref,Diff>;
};

postfix_increment_result<I,V,R,TC> operator++(iterator_facade<I,V,TC,R,D>& i, int)
  { postfix_increment_result tmp{*static_cast<I*>(&i)}; ++i; return tmp; }
if (iteratoperable<Dr1,Dr2>) {
  bool operator== <Dr1,V1,TC1,R1,D1,Dr2,V2,TC2,R2,D2>(
    iterator_facade<Dr1,V1,TC1,R1,D1> const& lhs,
    iterator_facade<Dr2,V2,TC2,R2,D2> const& rhs);
  // also !=, <, <=, >, >=
  ?? operator- <Dr1,V1,TC1,R1,D1,Dr2,V2,TC2,R2,D2>(
    iterator_facade<Dr1,V1,TC1,R1,D1> const& lhs,
    iterator_facade<Dr2,V2,TC2,R2,D2> const& rhs);
}
Iter operator+ <Iter,V,TC,R,D> (iterator_facade<Iter,V,TC,R,D> const&, Iter::difference_type n);
Iter operator+ <Iter,V,TC,R,D> (Iter::difference_type n, iterator_facade<Iter,V,TC,R,D> const&);
```

Derived class of `iterator_facade` should provide following (as needed):
* `reference dereference()`
* `bool equal(rhs)`
* `increment()`
* `decrement()`
* `advance(difference_type n)`
* `difference_type distance_to(rhs)`

------
### Iterator Adaptor Base

Header `<boost/iterator/iterator_adaptor.hpp>`

```c++
struct use_default;

using iterator_adaptor_base<Derived,Base,Value,Traversal,Ref,Diff> =
  iterator_facade<Derived,
    Value == use_default ? iterator_value<Base> : Value,
    Traversal == use_default ? iterator_traversal<Base> : Traversal,
    Ref == use_default ? iterator_reference<Base> : add_reference<Value>,
    Diff == use_default ? iterator_difference<Base> : Diff>;

struct iterator_adaptor<Derived,Base,V=use_default,Tr=use_default,Ref=use_default,Diff=use_default>
  : iterator_adaptor_base<Derived,Base,V,Tr,Ref,Diff> {
  using base_type = Base;
  using iterator_adaptor_ = iterator_adaptor<Derived,Base,V,Tr,Ref,Diff>;
  Base m_iterator;
  iterator_adaptor() = default;
  explicit iterator_adaptor(Base const& iter) : m_iterator(iter) { }
  Base const& base() const { return m_iterator; }
  Base [const]& base_reference() [const] { return m_iterator; }
  reference dereference() const { return *m_iterator; }
  bool equal<A,I,V,C,R,D>(iterator_adaptor<A,I,V,C,R,D> const& x) const
  { return m_iterator == x.base(); }
  void advance(difference_type n) { m_iterator += n; } // assert for random access traversal tag
  void increment() { ++m_iterator; }
  void decrement() { --m_iterator; } // assert for bidirectional traversal tag
  difference_type distance_to<A,I,V,C,R,D>(iterator_adaptor<A,I,V,C,R,D> const& y) const
  { return y.base() - m_iterator; } // assert for random access traversal tag
};
```

------
### Iterators and Iterator Adaptors

#### Counting Iterator

Header `<boost/iterator/counting_iterator.hpp>`

```c++
class counting_iterator<Incrementable,CatOrTrav=use_default,Diff=use_default>;
counting_iterator<Inc> make_counting_iterator<Inc>(Inc x);
```

Use an incrementable type (such as int) to serve as iterator, like a generator.

#### Filter Iterator

Header `<boost/iterator/filter_iterator.hpp>`

```c++
class filter_iterator<Pred,Iter>;
filter_iterator<Pred,Iter> make_filter_iterator<Pred,Iter>(Pred f, Iter x, Iter end=Iter());
filter_iterator<Pred,Iter> make_filter_iterator<Pred,Iter>(Iter x, Iter end=Iter());
```

Apply filter predicate on base iterator.

#### Function Input Iterator

Header `<boost/iterator/function_input_iterator.hpp>`

```c++
class function_input_iterator<Function,Input> : iterator_facade<...> {
  Function* f; // ptr to fun obj or fun
  Input state;
  optional<result_of_t<Function()>> value;
};
function_input_iterator<F,I> make_function_input_iterator<F,I>(F& f, I state);
function_input_iterator<F*,I>make_function_input_iterator<F,I>(F* f, I state);
struct infinite;
```

* Single pass, generator.
* `state` remembers the increment step count, thus serve as the `end` condition for a range.
* `infinite` compares false, thus infinite generator.

#### Function Output Iterator

Header `<boost/function_output_iterator.hpp>`

```c++
class function_output_iterator<UnaryFunc> {
  using iterator_category = output_iterator_tag;
  using value_type = difference_type = pointer = reference = void;
  UnaryFunc m_f;
  struct proxy {
    UnaryFunc& m_f;
    proxy& operator= <T> (T const& value) { m_f(value); return *this; }
  };
  auto operator*() { return proxy{m_f}; }
  // ++ just return *this
};
function_output_iterator<UF> make_function_output_iterator<UF>(UF const& f = UF());
```

Output iterator for sink function.

#### Indirect Iterator

Header `<boost/iterator/indirect_iterator.hpp>`

```c++
class indirect_iterator<Iter,V=use_default,C=use_default,R=use_default,D=use_default>;
indirect_iterator<Iter> make_indirect_iterator<Iter>(Iter x);
```

Perform `**x` in `dereference()`, for use in multi-level iterator scenarios.

#### Permutation Iterator

Header `<boost/iterator/permutation_iterator.hpp>`

```c++
class permutation_iterator<ElemIter, IndexIter>;
permutation<EI, II> make_permutation_iterator<EI, II>(EI e, II i);
```

Access elements from ElemIter in the order specified by the IndexIter.

#### Reverse Iterator

Header `<boost/iterator/reverse_iterator.hpp>`

```c++
class reverse_iterator<BidiIter>;
reverse_iterator<BidiIter> make_reverse_iterator<BidiIter x);
```

Same as `std::reverse_iterator`.

#### Shared Container Iterator

Header `<boost/shared_container_iterator.hpp>`

```c++
class shared_container_iterator<Container>;
shared_container_iterator<C> make_shared_container_iterator<C>(C::iterator iter, shared_ptr<C> const& c);
pair<shared_container_iterator<C>,shared_container_iterator<C>>
  make_shared_container_range(shared_ptr<C> const& container);
```

Keeps a `shared_ptr` to the container in the iterator.

#### Transform Iterator

Header `<boost/iterator/transform_iterator.hpp>`

```c++
class transform_iterator<UnaryFunc,Iter,Ref=use_default,Value=use_default>;
transform_iterator<UF,I> make_transform_iterator<UF,I>(I it, UF func=UF());
```

Perform a transform function `func` on `dereference()`.

#### Zip Iterator

Header `<boost/iterator/zip_iterator.hpp>`

```c++
class zip_iterator<IterTuple> : ... {
  IterTuple m_iterator_tuple;
  // ctor, etc
  using reference = mpl::transform<IterTuple, iterator_reference<_1>>::type;
  reference dereference() const { return fusion::transform(m_iterator_tuple, *_1); }
  // other members likewisely
};
zip_iterator<IterTuple> make_zip_iterator<Itertuple>(IterTuple t);
```

Support fusion tuple and `std::pair`.

#### Generator Iterator

Superseded by function input iterator.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>` - for iterator concepts
* `<boost/concept_archetype.hpp>` - for archetypes

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*>`

#### Boost.MPL

* `<boost/mpl/*>`

#### Boost.Utility

* `<boost/operators.hpp>` - for concept archetypes
* `<boost/utility/result_of.hpp>` - for `function_input_iterator`
* `<boost/next_prior.hpp>` - for `reverse_iterator`

#### Boost.Core

* `<boost/utility/addressof.hpp>`

#### Boost.Detail

* `<boost/detail/is_incrementable.hpp>`

#### Boost.Optional

* `<boost/optional/optional.hpp>` - for `function_input_iterator`
* `<boost/none.hpp>` - for `function_input_iterator`

#### Boost.SmartPtr

* `<boost/shared_ptr.hpp>` - for `shared_container_iterator`

#### Boost.Fusion

For `zip_iterator`
* `<boost/fusion/adapted/boost_tuple.hpp>`
* `<boost/fusion/algorithm/iteration/for_each.hpp>`
* `<boost/fusion/algorithm/transformation/transform.hpp>`
* `<boost/fusion/sequence/convert.hpp>`
* `<boost/fusion/sequence/intrinsic/at_c.hpp>`
* `<boost/fusion/sequence/comparison/equal_to.hpp>`
* `<boost/fusion/support/tag_of_fwd.hpp>`

------
### Standard Facilities

* Standard Library:
* Proposals:
