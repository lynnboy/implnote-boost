# Boost.Iterator

* lib: `boost/libs/iterator`
* repo: `boostorg/iterator`
* commit: `2484943`, 2025-08-26

------
### Iterator Concepts & Checking

Header `<boost/iterator/iterator_categories.hpp>`

```c++
// categories
struct no_traversal_tag {};
struct incrementable_traversal_tag : no_traversal_tag {}; // output tag
struct single_pass_traversal_tag : incrementable_traversal_tag {}; // input tag
struct forward_traversal_tag : single_pass_traversal_tag {};
struct bidirectional_traversal_tag : forward_traversal_tag {};
struct random_access_traversal_tag : bidirectional_traversal_tag {};
using iterator_category_to_triversal_t<Cat> = /* */; // accept both category and traversal tag types
struct iterator_category_to_triversal<Cat> { using type = iterator_category_to_triversal_t<Cat>; };
using iterator_traversal_t<I> = iterator_category_to_triversal_t<iterator_traits<I>::iterator_category>;
struct iterator_triversal<I> { using type = iterator_traversal_t<I>; };
using pure_traversal_tag_t<Trav> = /* */; // only accept traversal tag types
struct pure_traversal_tag<Trav> { using type = pure_traversal_tag_t<Trav>; };
using pure_iterator_traversal_t<I> = pure_traversal_tag_t<iterator_traversal_t<I>>;
struct pure_iterator_traversal<I> { using type = pure_iterator_traversal_t<I>; };
```

Header `<boost/iterator/iterator_concepts.hpp>` and `<boost/iterator/iterator_archetypes.hpp>`

```c++
// concepts
concept ReadableIterator<I> = Assignable<I> && CopyConstructible<I> &&
  requires(I i) { { *i } -> convertible_to<value_type>; };
concept WritableIterator<I> = CopyConstructible<I> &&
  requires(I i, value_type o) { { *i = o }; };
concept SwappableIterator<I> =
  requires(I a, I b) { { iter_swap(a, b) }; };
concept LvalueIterator<I> =
  requires(I i) { { const_cast<value_type&>(*i) }; };

concept IncrementableIterator<I> = Assignable<I> && CopyConstructible<I> &&
  convertible_to<traversal_category, incrementable_traversal_tag> &&
  requires(I i) { { ++i; }; { i++}; };
concept SinglePassIterator<I> = IncrementableIterator<I> && EqualityComparable<I> &&
  convertible_to<traversal_category, single_pass_traversal_tag>;
concept ForwardTraversal<I> = DefaultConstructible<I> && SinglePassIterator<I> &&
  is_integral_v<difference_type> && numeric_limits<difference_type>::is_signed &&
  convertible_to<traversal_category, forward_traversal_tag>;
concept BidirectionalTraversal<I> = ForwardTraversal<I> &&
  convertible_to<traversal_category, bidirectional_traversal_tag> &&
  requires(I i) { { --r }; { r-- }; }
concept RandomAccessTraversal<I> = BidirectionalTraversal<I> &&
  convertible_to<traversal_category, random_access_traversal_tag> &&
  requires(I i, I j, iterator_traits<I>::difference_type n) {
    { i += n }; { i = i + n }; { i = n + i };
    { i -= n }; { i = i - n }; { n = i - j };
  };

concept InteroperableIterators<I,CI> = SinglePassIterator<I> && SinglePassIterator<CI> &&
  requires(I x, CI y) { { y = x } } &&
  requires(I const& i1, CI const& i2)
  { {i1 == i2} -> convertible_to<bool>; /* and !=, and inversed order */ } &&
  same_as<traversal_category_t<I>, traversal_category_t<CI>> &&
  same_as<traversal_category_t<I>, single_pass_traversal_tag> || // both are single-pass
  (same_as<traversal_category_t<I>, random_access_traversal_tag> && // both are randomaccess, and then:
  { {i1 < i2} -> convertible_to<bool>; /* and >, <=, >=, and inversed order */
    {i1 - i2} -> convertible_to<difference_type_t<CI>>; /* and inversed order */ });

// archetypes
enum { readable_iterator_bit = 1, writable_iterator_bit = 2, swappable_iterator_bit = 4, lvalue_iterator_bit = 8 };
using readable_iterator_t = integral_constant<uint, 1>, writable_iterator_t = 2, swappable_iterator_t = 4, lvalue_iterator_t = 8;
using readable_writable_iterator_t = 3, readable_lvalue_iterator_t = 9, writable_lvalue_iterator_t = 10;
template<D,B> using has_access = bool_constant<D::value&B::value == B::value>;

struct undefined<class>;

struct access_archetype<Value,AccessCategory>;
struct traversal_archetype<Derived,Value,AccessCategory,TraversalCategory>;
struct iterator_archetype<Value,AccessCategory,TraversalCategory>;
struct iterator_archetype_impl<AccessCategory>; // specialized for xxx_iterator_t
struct traversal_archetype_base<Value,AccessCategory,TraversalCategory>;
```

------
#### Testing

Header `<boost/iterator/new_iterator_tests.hpp>`, testing of iterator types against iterator/traversal concepts.

------
### Iterator Primitives

#### Iterator Traits Meta Functions

Header `<boost/iterator/iterator_traits.hpp>`

```c++
using iterator_value_t<T> = iterator_value<T>::type = std::iterator_traits<T>::value_type;
using iterator_reference_t<T> = iterator_reference<T>::type = std::iterator_traits<T>::reference;
using iterator_pointer_t<T> = iterator_pointer<T>::type = std::iterator_traits<T>::pointer;
using iterator_difference_t<T> = iterator_difference<T>::type = std::iterator_traits<T>::difference_type;
using iterator_category_t<T> = iterator_category<T>::type = std::iterator_traits<T>::iterator_category;
```

Provide MPL meta functions for traits.

Header `<boost/iterator/interoperable.hpp>`
```c++
using is_interoperable<A, B>::type = (is_convertible_v<A, B> || is_convertible_v<B, A>);
```

Header `<boost/iterator/is_iterator.hpp>`, `<boost/iterator/is_lvalue_iterator.hpp>`, `<boost/iterator/is_readable_iterator.hpp>`
```c++
struct is_iterator<T> : (has_iterator_category<T> ? true_type : // has iterator_category, or complete object pointer
                            (is_pointer<T> && is_complete<remove_pointer<T> && !is_function<remove_pointer_<T>>>));
struct is_readable_iterator<T> : /* `decltype(*it)` -> `value_type const&` */;
struct is_lvalue_iterator<T> : /* `decltype(*it)` is `T&` and -> `value_type const&` */;
struct is_non_const_lvalue_iterator<T> : /* `decltype(*it)` is `T&` and -> `value_type&` */;
```

Header `<boost/iterator/min_category.hpp>`
```c++
using min_category_t<...Cats> = min_category<...Cats>::type = /**/; // select most base type (order by is_convertible<>)
```

------
#### Unified Pointee Type (type of `*e`)

Header `<boost/pointee.hpp>`, `<boost/indirect_reference.hpp>`

```c++
using smart_ptr_pointee<P>::type = P::element_type;
using iterator_pointee<I>::type = is_const<decltype(*declval<I>())> ? const I::value_type : I::value_type;
struct pointee<P> : (is_incrementable<P> ? smart_ptr_pointee<P> : iterator_pointee<P>);
using pointee_t<P> = pointee<P>::type;

using smart_ptr_reference<P>::type = pointee_t<P>&;
struct indirect_reference<P> : (is_incrementable<P> ? iterator_reference<P> : smart_ptr_reference<P>>);
using indirect_reference_t<P> = indirect_reference<P>::type;
```

Detect smart pointer by checking of `++`. `const` adjusted.

------
#### Iterator Algorithms

Header `<boost/iterator/advance.hpp>` and `<boost/iterator/distance.hpp>`:

```c++
constexpr void advance<InputIter, Dist>(InputIter& it, Dist n) {
  using tag = iterator_traversal<InputIter>::type;
  if constexpr (is_same_v<tag, incrementable_traversal_tag>)
    while (n-- > 0) ++it;
  else if constexpr (is_same_v<tag, bidirectional_traversal_tag>)
    if (n >= 0) while (n-- > 0) ++it; else while (n++ < 0) --it;
  else if constexpr (is_same_v<tag, random_access_traversal_tag>)
    it += n;
}
constexpr auto distance<SinglePassIter>(SinglePassIter f, SinglePassIter l) -> iterator_difference<SinglePassIter>::type {
  using tag = iterator_traversal<InpSinglePassIterutIter>::type;
  if constexpr (is_same_v<tag, single_pass_traversal_tag>)
  { n = 0; while (f != l) { ++f; ++n; } return n; }
  else if constexpr (is_same_v<tag, random_access_traversal_tag>)
    return l - f;
}
```

Both functions prevents from ADL.

Header `<boost/next_prior.hpp>`

```c++
template <class T> T next(T x) { return ++x; }
template <class T> T prior(T x) { return --x; }
template <class T, class Dist> T next(T x, Dist n) {
  if constexpr (is_iterator<T>::value) { advance(x, n), return x; }
  else if constexpr (has_plus_assign<T, Dist>::value) { x += n; return x; }
  else if constexpr (has_plus<T, Dist>::value) { return x + n; }
  else // no impl
}
template <class T, class Dist> T prior(T x, Dist n) {
  if constexpr (is_iterator<T>::value)
  { reverse_iterator<T> rx{x}; advance(rx, n); return rx.base(); }
  else if constexpr (has_minus_assign<T, Dist>::value) { x -= n; return x; }
  else if constexpr (has_minus<T, Dist>::value) { return x - n; }
  else // no impl
}
```

`is_iterator` means having trait type `iterator_category` or being a pointer.

------
#### Misc

Header `<boost/iterator/enable_if_convertible.hpp>`
```c++
struct enable_if_convertible<F,T> : std::enable_if<is_convertible_v<F,T>, struct enable_type> {};
using enable_if_convertible_t<F,T> = enable_if_convertible<F,T>::type;
```

Header `<boost/iterator/detail/if_default.hpp>` and `<boost/iterator/detail/eval_if_default.hpp>`
```c++
using if_default_t<T,D,ND=T> = if_default<T,D,ND=T>::type = T == use_default ? D : ND;
using eval_if_default_t<T,D,ND=type_identity<T>> = eval_if_default<T,D,ND=type_identity<T>>::type = T == use_default ? D::type : ND::type;
```

------
### Iterator Facade

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
struct detail::operator_brackets_proxy<Iter> {
  Iter m_iter;
  operator Iter::reference () const { return *m_iter; }
  auto& operator= <T>(T && val) { *m_iter = std::move(val); return *this; } // enabled if assignable
  Iter::reference operator->() const { return *m_iter; }
  auto operator* <Ref=Iter::reference, Res = /*deref type of Ref*/>() const -> Ref { return **m_iter; }
};
using operator_bracket_result<Iter,Value,Ref> =
  !(is_copy_constructible<Value> && is_const<Value>) ? operator_brackets_proxy<Iter> : Value;

class iterator_core_access { // named iterator API
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

using detail::facade_iterator_category<CatOrTrav, Val, Ref>::type =
  is_iterator_category<CatOrTrav> ? CatOrTrav : ({
    using category = is_reference<Ref> ?
      { case CatOrTrav -> random_access_traversal_tag: random_access_iterator_tag,
        case CatOrTrav -> bidirectional_traversal_tag: bidirectional_iterator_tag,
        case CatOrTrav -> forward_traversal_tag: forward_iterator_tag,
        case CatOrTrav -> single_pass_traversal_tag && Ref -> Val: input_iterator_tag,
        default: CatOrTrav }
      : (CatOrTrav -> single_pass_traversal_tag && Ref -> Val) ? input_iterator_tag
      : CatOrTrav;
    if (CatOrTrav == iterator_category_to_triversal_t<category>) return category;
    else return (struct iterator_category_with_traversal: CatOrTrav, category {});
  });
struct detail::iterator_facade_types<V,TC,R,D> {
  using value_type = remove_const<V>;
  using pointer = V*;
  using iterator_category = facade_iterator_category<TC,V,R>::type;
};
struct detail::iterator_facade_base<I,V,TC,R,D,bool isBidi, bool isRandom>;
struct detail::iterator_facade_base<I,V,TC,R,D,false,false> : iterator_facade_types<V,TC,R,D> { // base case
  using reference = Ref;
  using difference_type = Diff;
  using pointer = operator_arrow_dispatch<Ref,iterator_facade_types::pointer>::result_type;
  using iterator_category = iterator_facade_types::iterator_category;
  reference operator*() const { return iterator_core_access::dereference(derived()); }
  pointer operator->() const { return operator_arrow_dispatch<>::apply(*derived()); }
  I& operator++() { iterator_core_access::increment(derived()); return derived(); }
  protected: I [const]& derived() [const] { return *static_cast<I [const]*>(this); }
};
struct detail::iterator_facade_base<I,V,TC,R,D,true,false> : iterator_facade_base<I,V,TC,R,D,false,false> { // bidi
  I& operator--() { iterator_core_access::decrement(derived()); return derived(); }
  I operator--(int) { I tmp{derived()}; --*this; return tmp; }
};
struct detail::iterator_facade_base<I,V,TC,R,D,true,true> : iterator_facade_base<I,V,TC,R,D,true,false> { // rand acc
  operator_brackets_proxy<I> operator[](difference_type n) const {
    return operator_brackets_proxy{derived() + n};
  };
  I& operator+=(difference_type n) { iterator_core_access::advance(derived(), n); return derived(); }
  I& operator-=(difference_type n) { iterator_core_access::advance(derived(), -n); return derived(); }
  I operator-(difference_type n) { I result{derived}; return result -= n; };
};

struct iterator_facade<Iter,Value,CategoryOrTraversal,Ref=Value&,Diff=ptrdiff_t> // main API
  : iterator_facade_base<Iter,Value,CategoryOrTraversal,Ref,Diff,
     is_at_least<bidirectional_traversal_tag>, is_at_least<random_access_traversal_tag>> {
  using iterator_facade_ = iterator_facade<Iter,Value,CategoryOrTraversal,Ref,Diff>; // self type
};

postfix_increment_result<I,V,R,TC>::type operator++(iterator_facade<I,V,TC,R,D>& i, int)
  { postfix_increment_result::type tmp{*static_cast<I*>(&i)}; ++i; return tmp; }

// operators, enabled according to categories.  D is derived class of iterator_facade
template<D1,V1,TC1,R1,Diff1,D2,V2,TC2,R2,Diff2> // enable_if_interoperable
bool operator {==|!=} (iterator_facade<D1,V1,TC1,Diff1> const& i1, iterator_facade<...2> const& i2)
{ return [!]iterator_core_access::equal(i1, i2); }
template<D1,V1,TC1,R1,Diff1,D2,V2,TC2,R2,Diff2> // enable_if_interoperable_and_random_access_traversal
bool operator {<|>|<=|=>} (iterator_facade<D1,V1,TC1,Diff1> const& i1, iterator_facade<...2> const& i2)
{ return iterator_core_access::distance_from(i1, i2) < 0; } // > 0, <= 0, >= 0
template<D1,V1,TC1,R1,Diff1,D2,V2,TC2,R2,Diff2> // enable_if_interoperable_and_random_access_traversal
common_type_t<Diff1,Diff2> operator - (iterator_facade<D1,V1,TC1,Diff1> const& i1, iterator_facade<...2> const& i2)
{ return iterator_core_access::distance_from(i1, i2); }
template<D,V,TC,R,Diff> // enable_if is_traversal_at_least<TC, random_access_traversal_tag>
D operator + (iterator_facade<D,V,TC,R,Diff> const& i, D::Difference_type n) // and inversed args
{ D tmp{static_cast<D const&>(i)}; return tmp += n; }
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
using iterator_adaptor_base<Derived,Base,Value,Traversal,Ref,Diff> =
  iterator_facade<Derived,
    Value == use_default ? Ref == use_default ? iterator_value_t<Base> : remove_reference_t<Ref> : Value,
    Traversal == use_default ? iterator_traversal_t<Base> : Traversal,
    Ref == use_default ? Value == use_default ? iterator_reference_t<Base> : add_lvalue_reference_t<Value> : Ref,
    Diff == use_default ? iterator_difference_t<Base> : Diff>;

struct iterator_adaptor<Derived,Base,V=use_default,Tr=use_default,Ref=use_default,Diff=use_default>
  : iterator_adaptor_base<Derived,Base,V,Tr,Ref,Diff> {
  using base_type = Base;
  using iterator_adaptor_ = iterator_adaptor<Derived,Base,V,Tr,Ref,Diff>;
  iterator_adaptor() = default;
  explicit iterator_adaptor(Base const& iter) : m_iterator(iter) { }
  Base const& base() const { return m_iterator; }
  Base [const]& base_reference() [const] { return m_iterator; }
private:
  Base m_iterator; // adapted iterator
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

Adapts an iterator-like type into `iterator_facade`.

------
### Iterators and Iterator Adaptors

#### Counting Iterator

Header `<boost/iterator/counting_iterator.hpp>`

```c++
class counting_iterator<Incrementable,CatOrTrav=use_default,Diff=use_default>
counting_iterator<Inc> make_counting_iterator<Inc>(Inc x);
```

Use an incrementable type (such as int) to serve as iterator, like a generator.
If not `use_default`, treat numeric `Incrementable` as random_access, otherwise treate it as iterator.

#### Filter Iterator

Header `<boost/iterator/filter_iterator.hpp>`

```c++
class filter_iterator<Pred,Iter>;
filter_iterator<Pred,Iter> make_filter_iterator<Pred,Iter>(Pred f, Iter x, Iter end=Iter());
filter_iterator<Pred,Iter> make_filter_iterator<Pred,Iter>(Iter x, Iter end=Iter());
```

Apply filter predicate on base iterator. Random access falls into bidirectional.

#### Function Input Iterator

Header `<boost/iterator/function_input_iterator.hpp>`

```c++
class function_input_iterator<Function,Input> : iterator_facade<...> {
  Function* m_f; // ptr to fun obj or fun, or `Function f` when Function is function pointer
  Input m_state;
  optional<result_of_t<Function()>> m_value;
  void increment() { if (m_value) m_value.reset(); else (*m_f)(); ++m_state; }
  reference dereference() const { if (!m_value) m_value = (*m_f)(); return m_value.get(); }
  bool equal(const& other) const { return m_f == other.m_f && m_state == other.m_state; }
};
function_input_iterator<F,I> make_function_input_iterator<F,I>(F& f, I state);
function_input_iterator<F*,I>make_function_input_iterator<F,I>(F* f, I state);
struct infinite;
```

* Single pass, generator.
* `value` cache result of one step call.
* `state` remembers the increment step count, thus serve as the `end` condition for a range.
* `infinite` compares false, thus infinite generator.

#### Function Output Iterator

Header `<boost/iterator/function_output_iterator.hpp>`

```c++
class function_output_iterator<UnaryFunc> {
  using iterator_category = output_iterator_tag;
  using value_type = pointer = reference = void;
  using difference_type = std::ptrdiff_t;
  UnaryFunc m_f;
  struct proxy { // disable copy/move-assign, defaulted copy-ctor
    UnaryFunc& m_f;
    proxy const& operator= <T> (T&& value) const { m_f(forwad<T&&>(value)); return *this; } // forwarding for &&, T != proxy
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
reverse_iterator<BidiIter> make_reverse_iterator<BidiIter>(BidiIter x);
```

Same as `std::reverse_iterator`.

#### Shared Container Iterator

Header `<boost/iterator/shared_container_iterator.hpp>`

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
public:  // ctor, etc
  using reference = mpl::transform<IterTuple, iterator_reference<_1>>::type;
  reference dereference() const { return fusion::transform(m_iterator_tuple, *_1); }
  // other members likewisely, fusion::foreach on each iterator
  IterTuple const& get_iterator_tuple() const { return m_iterator_tuple; }
};
zip_iterator<IterTuple> make_zip_iterator<Itertuple>(IterTuple t);
```

Support fusion tuple and `std::pair`.

#### Generator Iterator

Header `<boost/iterator/generator_iterator.hpp>`

Superseded by function input iterator.

------
### Dependency

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>`, `<boost/concept/detail/concept_[un]def.hpp>` - for iterator concepts
* `<boost/concept_archetype.hpp>` - for archetypes and tests

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`
* `<boost/config/header_deprecated.hpp>`

#### Boost.Core

* `<boost/core/empty_value.hpp>`
* `<boost/core/use_default.hpp>`
* `<boost/core/lightweight_test.hpp>` - for `iterator_tests.hpp` and `new_iterator_tests.hpp`

#### Boost.Detail

* `<boost/detail/is_incrementable.hpp>`
* `<boost/detail/numeric_traits.hpp>` - for `counting_iterator`

#### Boost.Fusion

* `<boost/fusion/*.hpp>` - For `zip_iterator`

#### Boost.MP11

* `boost/mp11/utility.hpp>`
* `boost/mp11/list.hpp>`

#### Boost.MPL

* `<boost/mpl/arg_fwd.hpp>`

#### Boost.Optional

* `<boost/optional/optional.hpp>` - for `function_input_iterator`

#### Boost.TypeTraits

* `<boost/type_traits/has_plus.hpp>`, `<boost/type_traits/has_plus_assign.hpp>`,
  `<boost/type_traits/has_minus.hpp>`, `<boost/type_traits/has_minus_assign.hpp>` - for `next_prior.hpp`
* `<boost/type_traits/is_complete.hpp>` - for `is_iterator.hpp` and `next_prior.hpp`
* `<boost/type_traits/conjunction.hpp>`, `<boost/type_traits/disjunction.hpp>`,
  `<boost/type_traits/negation.hpp>`, `<boost/type_traits/type_identity.hpp>` - when stdlib traits are not available.

#### Boost.Utility

* `<boost/operators.hpp>` - for concept archetypes and `int_iterator`

------
### Standard Facilities

* Standard Library: `<ranges>`, range views (C++20). `<generator>` (C++23).
