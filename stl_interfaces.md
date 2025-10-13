# Boost.STLInterfaces

* lib: `boost/libs/stl_interfaces`
* repo: `boostorg/stl_interfaces`
* commit: `9bf4c92`, 2025-07-14

------
### Iterator Interface

* Header `<boost/stl_interfaces/iterator_interfaces.hpp>`

```c++
enum class element_layout: bool { discontiguous, contiguous };

using v1::detail::void_t<...T> = void;
using v1::detail::iter_different_t<It> = std::iterator_traits<It>::difference_type;

struct access { static constexpr auto base<D> (D<const>& d) noexcept -> decltype(d.base_reference()) { return d.base_reference(); } };
struct proxy_arrow_result<T> requires std::is_object_v<T> {
    constexpr ctor(T const& v) noexcept (noexcept(T(v))) : value_(v) {}
    constexpr ctor(T && v) noexcept (noexcept(T(std::move(v)))) : value_(std::move(v)) {}
    constexpr T<const>* operator->() <const> noexcept { return &value_; }
private: T value_;
};

auto detail::make_pointer<Ptr,Ref,T> (T&& v)
    requires std::is_pointer_v<Ptr> && std::is_reference_v<Ref>
{ return std::addressof(v); }
auto detail::make_pointer<Ptr,Ref,T> (T&& v)
    requires !std::is_pointer_v<Ptr> && !std::is_same<Ptr,void> && std::is_reference_v<Ref>
{ return Ptr(std::forward<T>(v)); }

struct detail::pointer<Ptr,ItConcept> { using type = Ptr; } // primary
struct detail::pointer<Ptr,std::output_iterator_tag> { using type = void; }
using detail::pointer_t<Ptr,ItConcept> = pointer<Ptr,ItConcept>::type;

using detail::interoperable<T,U> = std::bool_constant<std::is_convertible_v<T,U> || std::is_convertible_v<U,T>>;

// common_eq
using detail::common_t<T,U> = std::conditional_t<std::is_convertible_v<T,U>,U,T>;
using detail::use_base<T> = decltype(access::base(std::declval<T&>()));
using detail::void_t<...T> = void;
struct detail::detector<Void,Template<...>,...Args> : std::false_type {}; // detect idiom, primary
struct detail::detector<void_t<Template<Args...>>,Template,Args...> : std::true_type {}; // SFINAE
struct detail::common_eq<T,U,useBase=detector<void,use_base,T>::value> // primary
{ static constexpr auto call(T t, U u) { return common_t<T,U>(t).derived() == common_t<T,U>(u).derived(); } };
struct detail::common_eq<T,U,true> // compare by base
{ static constexpr auto call(T t, U u) { return access::base(t) == access::base(u); } };
constexpr auto detail::common_diff<T,U> (T t, U u) noexcept(...) -> decltype(...)
{ return common_t<T,U>(t) - common_t<T,U>(u); }

namespace v1 { // enable_if version
struct iterator_interface<Derived, IterConcept, ValT, Ref = ValT&, Ptr = ValT*, Diff = std::ptrdiff_t>
    requires is_class_v<Derived> && is_same_v<Derived,remove_cv_t<Derived>>;
constexpr auto operator== <It1,It2>(It1 lhs, It2 rhs) noexcept -> decltype(...); // and !=, <, <=, >, >=
constexpr auto operator+ <D> (D it, D::difference_type n) requires derived_iter<D> && requires {it += n;}
constexpr auto operator+ <D> (D::difference_type n, D it) requires derived_iter<D> && requires {it += n;}
constexpr auto operator- <D1,D2> (D1 it1, D2 it2) requires derived_iter<D1> && derived_iter<D2> requires { access::base(it1) - access::base(it2); }
constexpr auto operator- <D> (D it, D::difference_type n) requires derived_iter<D> && requires {it += -n;}
using proxy_iterator_interface<Derived, IterConcept, ValT, Ref=ValT, Diff=std::ptrdiff_t>
    = iterator_interface<Derived, IterConcept, ValT, Ref, proxy_arrow_result<Ref>, Diff>;
}

concept v2::detail::base_3way<D,D2=D> = requires (D d, D2 d2) { access::base(d) <=> access:base(d2); };
concept v2::detail::base_eq<D,D2=D> = requires (D d, D2 d2) { access::base(d) == access:base(d2); };
concept v2::detail::iter_sub<D,D2=D> = requires (D d, D2 d2) {
    typename D::difference_type;
    {d-d2} -> std::convertible_to<D::difference_type>;
};

constexpr auto v2::detail::category_tag<ItConcept,Ref> () {
    if constexpr (std::is_base_of_v<std::forward_iterator_tag, ItConcept>) {
        if constexpr (!std::is_reference_v<Ref>) return std::input_iterator_tag{}; // not writable
        else if constexpr (std::is_base_of_v<std::random_access_iterator_tag, ItConcept>) return std::random_access_iterator_tag{};
        else if constexpr (std::is_base_of_v<std::bidirectional_iterator_tag, ItConcept>) return std::bidirectional_iterator_tag{};
        else return std::forward_iterator_tag{};
    } else return 0; // for no tag
}
struct v2::detail::iterator_category_base <ItConcept,Ref,ItCategory=decltype(category_tag<ItConcept,Ref>())>
{ using iterator_category = ItCategory; };
struct v2::detail::iterator_category_base <ItConcept,Ref,int> {};

namespace v2 { // concept without deducing this version
struct iterator_interface<D, IterConcept, ValT, Ref=ValT&, Ptr=ValT*, Diff=std::ptrdiff_t>
    requires is_class_v<D> && same_as<D,remove_cv_t<D>>;
constexpr auto operator+ <D> (D it, D::difference_type n) requires derived_iter<D> && requires {it += n;}
constexpr auto operator+ <D> (D::difference_type n, D it) requires derived_iter<D> && requires {it += n;}
constexpr auto operator- <D1,D2> (D1 it1, D2 it2) requires derived_iter<D1> && derived_iter<D2> requires { access::base(it1) - access::base(it2); }
constexpr auto operator- <D> (D it, D::difference_type n) requires derived_iter<D> && requires {it += -n;}
constexpr auto operator<=> <D1,D2> (D1 it1, D2 it2) requires derived_iter<D1> && derived_iter<D2> && base_3way<D1,D2> && iter_sub<D1,D2>;
// also <, <=, >, >=, ==, !=
using proxy_iterator_interface<Derived, IterConcept, ValT, Ref=ValT, Diff=std::ptrdiff_t>
    = iterator_interface<Derived, IterConcept, ValT, Ref, proxy_arrow_result<Ref>, Diff>;
}

namespace v3 { // concept with deducing this version
struct iterator_interface<IterConcept, ValT, Ref=ValT&, Ptr=ValT*, Diff=std::ptrdiff_t>
    : v2::detail::iterator_category_base<IterConcept,Ref> {
    using iterator_concept = IterConcept;
    using value_type = std::remove_const_t<ValT>;
    using reference = Ref; using pointer = pointer_t<Ptr,IterConcept>; using difference_type = Diff;

    constexpr decltype(auto) operator* (this auto&& self) requires requires{*access::base(self);}
    { return *access::base(self); }
    constexpr auto           operator-> (this auto&& self)
        requires (!std::same_as<pointer,void>) && std::is_reference_v<reference> && requires{*self;}
    { return make_pointer<pointer,reference>(*self); }
    constexpr decltype(auto) operator[] (this auto const& self, difference_type n) requires requires{self+n;}
    { auto r = self; r = r+n; return *r; }
    constexpr decltype(auto) operator++ (this auto& self)
        requires requires{++access::base(self);} && !requires{ self+=difference_type(1);}
    { ++access::base(self); return self; }
    constexpr decltype(auto) operator++ (this auto& self) requires requires{self+=difference_type(1);}
    { return self += difference_type(1); }
    constexpr auto           operator++(this auto& self, int) requires requires{++self;}
    {   if constexpr (std::is_same_v<ItConcept, std::input_iterator_tag>) ++self;
        else { auto r=self; ++self; return r; }  }
    constexpr decltype(auto) operator+= (this auto& self, difference_type n) requires requires{access::base(self)+=n;}
    { access::base(self) += n; return self; }
    constexpr decltype(auto) operator-- (this auto& self)
        requires requires{--access::base(self);} && !requires{ self+=difference_type(1);}
    { --access::base(self); return self; }
    constexpr decltype(auto) operator-- (this auto& self) requires requires{self+=-difference_type(1);}
    { return self += -difference_type(1); }
    constexpr auto           operator--(this auto& self, int) requires requires{--self;}
    { auto r=self; --self; return r; }
    constexpr decltype(auto) operator-= (this auto& self, difference_type n) requires requires{self+=-n;}
    { return self += -n; }
};
void detail::derived_iterator<ItC,V,R,P,D> (iterator_interface<ItC,V,R,P,D> const&);
concept detail::derived_iter<D> = requires(D d){derived_iterator(d);};
constexpr auto operator+ <D> (D it, D::difference_type n) requires derived_iter<D> && requires {it += n;} { return it += n; }
constexpr auto operator+ <D> (D::difference_type n, D it) requires derived_iter<D> && requires {it += n;} { return it += n; }
constexpr auto operator- <D1,D2> (D1 it1, D2 it2) requires derived_iter<D1> && derived_iter<D2> requires { access::base(it1) - access::base(it2); }
{ return access::base(it1) - access::base(it2); }
constexpr auto operator- <D> (D it, D::difference_type n) requires derived_iter<D> && requires {it += -n;} { return it += -n; }
constexpr auto operator<=> <D1,D2> (D1 it1, D2 it2)
    requires derived_iter<D1> && derived_iter<D2> && base_3way<D1,D2> && iter_sub<D1,D2>
{   if constexpr (base_3way<D1,D2>) return access::base(it1) <=> access::base(it2);
    else { using diff_t = D1::difference_type; diff_t const d = it1 - it2;
        return d < diff_t(0) ? std::strong_ordering::less : diff_t(0) < d ? std::strong_ordering::greater : std::strong_ordering::equal;
} }
constexpr bool operator< <D1,D2> (D1 it1, D2 it2) requires derived_iter<D1> && derived_iter<D2> && iter_sub<D1,D2>
{ return (it1-it2) < D1::difference_type(0); }
// also <=, >, >=
constexpr bool operator== <D1,D2> (D1 it1, D2 it2)
    requires derived_iter<D1> && derived_iter<D2> && interoperable<D1,D2>::value && (base_eq<D1,D2> || iter_sub<D1,D2>)
{   if constexpr (base_eq<D1,D2>) return access::base(it1) == access::base(it2);
    else return (it1-it2) == D1::difference_type(0); }
constexpr auto operator!= <D1,D2> (D1 it1, D2 it2) -> decltype(!(it1==it2)) requires derived_iter<D1> && derived_iter<D2>
{ return !(it1-it2); }

using proxy_iterator_interface<IterConcept, ValT, Ref=ValT, Diff=std::ptrdiff_t>
    = iterator_interface<IterConcept, ValT, Ref, proxy_arrow_result<Ref>, Diff>;
}

#define BOOST_STL_INTERFACES_STATIC_ASSERT_CONCEPT(type, concept_name) // check `type` with `concept_name`
#define BOOST_STL_INTERFACES_STATIC_ASSERT_ITERATOR_TRAITS(                    \
    iter, category, concept, value_type, reference, pointer, difference_type) // check with std::is_same for trait types
```

* Only a subset of operations a required for derived class:
  - Input: `*i`, `i==i2`, `++i`
  - Output: `*i`, `++i`
  - Forward: `*i`, `i==i2`, `++i`
  - Bidirectional: `*i`, `i==i2`, `++i`, `--i`
  - Random access / Contiguous: `*i`, `i-i2`, `i += n`
* Proxy iterator: reference type is not actually a reference. `proxy_arrow_result` wraps a cached value and provides pointer-like `->` to access it.

------
### View Interface

* Header `<boost/stl_interfaces/view_interfaces.hpp>`

```c++
namespace v1 {
struct view_interfaces<Derived,contiguity=element_layout::discontiguous>
    requires is_class_v<Derived> && is_same_v<Derived, remove_cv_t<Derived>>
{};
}

using v2::view_interfaces<D,contiguity=element_layout::discontiguous> = std::ranges::view_interfaces<D>;
using v3::view_interfaces<D,contiguity=element_layout::discontiguous> = std::ranges::view_interfaces<D>;
```

* Just same as `std::range::view_interfaces` (C++20).
* Requires `begin` and `end`.
* Requires `empty` for forwarding range.

------
### Reverse Iterator

* Header `<boost/stl_interfaces/reverse_iterator.hpp>`

```c++
namespace v1 {
struct reverse_iterator<BidiIter> : iterator_interface<reverse_iterator,...>;
{};
}

using v2::reverse_iterator<BidiIter> = std::reverse_iterator<BidiIter>;
using v3::reverse_iterator<BidiIter> = std::reverse_iterator<BidiIter>;
auto make_reverse_iterator<BidiIter>(BidiIter it) { return reverse_iterator<BidiIter>(it); }
```

* Just same as `std::reverse_iterator`.

------
### Sequence Container Interface

* Header `<boost/stl_interfaces/sequence_container_interface.hpp>`

```c++
struct detail::n_iter<T,SizeT> : iterator_interface<std::random_access_iterator_tag, T> {
    ctor(T const& x, SizeT n); // ctor, init
    T const& operator*() const; // deref
    constexpr ptrdiff_t operator-(self other) const noexcept; // diff
    self& operator+= (ptrdiff_t offset);
private: T const* x_{nullptr}; SizeT n_{0}; // ptr and distance
};
constexpr auto detail::make_n_iter<T,SizeT>(T const& x, SizeT) noexcept(noexcept(n_iter(x,n)));
constexpr auto detail::make_n_iter_end<T,SizeT>(T const& x, SizeT n) noexcept(noexcept(n_iter(x,n)));
size_t detail::fake_capacity<Cont>(Cont const&) { return SIZE_MAX; } // default maximum
size_t detail::fake_capacity<Cont>(Cont const& c) requires{{c.capacity()}->size_t;} { return c.capacity(); }

namespace v1 {
struct sequence_container_interface<Derived,contiguity=element_layout::discontiguous>;
}
namespace v2 {
using detail::container_size_t<T> = T::size_type;
concept detail::range_insert<T,I> = requires (T t, std::ranges::iterator_t<T> t_it, I it) { t.insert<I>(t_it, it, it); };
using detail::n_iter_t<T> = n_iter<std::ranges::range_value_t<T>, container_size_t<T>>; // used for repeated insert

struct sequence_container_interface<D,contiguity=element_layout::discontiguous>
    requires std::is_class_v<D> && std::same_as<D, std::remove_cv_t<D>> {
private:
    constexpr D <const>& derived() <const> noexcept; constexpr D& mutable_derived() const noexcept;
    static constexpr void clear_impl(D& d) noexcept { if constexpr (requires{d.clear();}) d.clear(); }
public: // using std::ranges::begin, std::ranges::end, std::ranges::prev, std::ranges::next, std::ranges::equal, std::ranges::iterator_t, std::ranges::sentinel_t
    constexpr bool empty() const { begin(derived()) == end(derived()); }
    constexpr auto data() <const> requires std::contiguous_iterator<iterator_t<<const>D>>
    { return std::to_address(begin(derived())); }
    constexpr container_size_t<C> size <C=D> () const requires std::sized_sentinel_for<sentinel_t<const C>, iterator_t<const C>>
    { return end(derived()) - begin(derived()); }

    constexpr decltype(auto) front() <const> { return *begin(derived()); }
    constexpr void push_front <C=D> (range_value_t<C> {const&|&&} x) requires requires{...} { derived().emplace_front({x|std::move(x)}); }
    constexpr void pop_front() noexcept requires(...){d.emplace_front(x);d.erase(it);} { return derived().erase(begin(derived())); }

    constexpr decltype(auto) back() <const>
        requires std::ranges::bidirectional_range<<const>D> && std::ranges::common_range<<const>D>
    { return *std::ranges::prev(end(derived())); }
    constexpr void push_back <C=D> (range_value_t<C> {const&|&&} x)
        requires bidirectional_range<C> && common_range<C> && requires{...}
    { derived().emplace_back({x|std::move(x)}); }
    constexpr void pop_back() noexcept
        requires bidirectional_range<D> && common_range<D> && requires(...){d.emplace_back(std::move(x));d.erase(it);}
    { return derived().erase(prev(begin(derived()))); }

    constexpr decltype(auto) operator[] <C=D> (container_size_t<C> n)<const> requires std::ranges::random_access_range<C>;
    constexpr decltype(auto) at <C=D> (container_size_t<C> n)<const> requires std::ranges::random_access_range<C>;

    constexpr auto {begin|end}() const { return D::const_iterator(mutable_derived().{begin|end}()); }
    constexpr auto {cbegin|cend}() const { return derived().{begin|end}(); }
    constexpr auto {rbegin|rend}() <const>
        requires std::ranges::bidirectional_range<<const>D> && std::ranges::common_range<<const>D>
    { return std::reverse_iterator({begin|end}(derived())); }
    constexpr auto {crbegin|crend}() const
        requires std::ranges::bidirectional_range<const D> && std::ranges::common_range<const D>
    { return std::reverse_iterator({begin|end}(derived())); }

    constexpr auto insert <C=D> (iterator_t<const C> it, range_value_t<C>{const&|&&} x)
        requires(...) { return derived().emplace(it, {x|std::move(x)}); }
    constexpr auto insert <C=D> (iterator_t<const C> it, container_size_t<C> n, const range_value_t<C>& x)
        requires range_insert<C,n_iter_t<C>> { return derived().insert(it,make_n_iter(x,n),make_n_iter_end(x,n)); }
    constexpr auto insert <C=D> (iterator_t<const C> it, std::initializer_list<range_value_t<C>> il)
        requires range_insert<C,decltype(il.begin())> { return derived().insert(it,il.begin(),il.end()); }

    constexpr void erase <C=D> (C::const_iterator it) requires{...} { derived().erase(it, next(it)); }

    constexpr void assign <It,C=D> (It first, It last) requires std::input_iterator<It> && requires{...}
    {
        auto out = derived().begin(); auto const out_last = derived().end();
        for (; out!=out_last && first!=last; ++first,++out) *out = *first;
        if (out != out_last) derived().erase(out, out_last); // shrink space
        if (first != last) derived().insert(derived().end(), first, last); // append remaining
    }
    constexpr void assign <C=D> (container_size_t<C> n, const range_value_t<C>& x) requires{...}
    {
        if (fake_capacity(derived()) < n) derived().swap(C(n, x)); // swap with temp
        else {
            auto const min_size = std::min(n, derived().size());
            auto const fill_end = std::fill_n(derived().begin(), min_size, x);
            if (min_size < derived().size()) derived().erase(fill_end, derived().end());// shrink space
            else derived().insert(derived().begin(), make_n_iter(x,n-min_size), make_n_iter_end(x,n-min_size));
        }
    }
    constexpr void assign <C=D> (std::initializer_list<range_value_t<C>> il) requires{...} { derived().assign(il.begin(), il.end()); }
    constexpr decltype(auto) operator= <C=D> (std::initializer_list<range_value_t<C>> il) requires{...} { derived().assign(il.begin(), il.end()); return *this; }

    constexpr void clear() noexcept requires(...) { derived().erase(begin(derived()), end(derived())); }

    friend constexpr void swap(D& lhs, D& rhs) requires requires{...} { return lhs.swap(rhs); }

    friend constexpr bool operator==(const D& lhs, const D& rhs)
        requires sized_range<const D> && requires{...} { return lhs.size() == rhs.size() && equal(lhs,rhs); }
    friend constexpr auto operator<=>(const D& lhs, const D& rhs) -> std::compare_three_way_result_t<range_reference_t<const D>>
        requires std::three_way_comparable<range_reference_t<const D>>
    { return std::lexicographical_compare_three_way(lhs.begin(),lhs.end(), rhs.begin(),rhs.end()); }
};
}

using v3::sequence_container_interface<D,cont> = v2::sequence_container_interface<D,cont>; // TODO: mot impl by deducing this
```

* Requires derived class provide `c.begin()`, `c.end()` (non const version) and member types.
* Provides `push_front`/`push_back` if has `emplace_front`/`emplace_back`
* Provides `pop_front`/`pop_back` if also has single `erase`
* Provides `insert` if has `emplace`
* Provides repeated `insert` and `insert` for init-list if provides `insert` has iter-pair.
* Provides single `erase` if has iter-pair `erase`.
* Provides iter-pair `assign`, repeated `assign` and init-list `assign` if has iter-pair `erase` and iter-pair `insert`
* Provides `clear` if has iter-pair `erase`
* Provides hidden friend `swap` if has member `swap`
* provides hidden friend `==`, `<=>`, `!=`, `<`, `<=`, `>`, `>=`

------
### View Closures and Adaptors

* Header `<boost/stl_interfaces/view_adaptor.hpp>`

```c++
// pipeline
concept detail::pipeable_ = std::derived_from<T,pipeable_base> && std::is_object_v<T> && std::copy_constructible<T>;
struct detail::pipeable_base
{ friend constexpr auto operator| <pipeable_ T, pipeable_ U> (T&& t, U&& u) { return view_pipeline<T,U>{(T&&)t,(U&&)u}; }};
struct detail::pipeable<D> : pipeable_base
{ friend constexpr auto operator| <R> (R&& r, D{&|const&|&&}d) -> decltype(...) { return ((D&&)d)((R&&)r); } };
struct detail::view_pipeline<pipeable_ T, pipeable_ U> : pipeable<view_pipeline<T,U>> {
    T left_; U right_;
    ctor(); constexpr ctor(T&& t, U&& t);
    constexpr decltype(auto) operator() <R> (R&& r) {&|const&|&&}
        requires std::ranges::viewable_range<R> && std::invocable<T[&|const&],R> && std::invocable<U[&|const&],std::invoke_result_t<T[&|const&],R>>
    { return right_(<std::move>(left((R&&) r))); }
};
// closure
struct detail::box<i,T> { T value_; };
struct detail::view_closure_impl<Indices,Func,...T>; // primary
struct detail::view_closure_impl<std::index_sequence<i...>,Func,T...> : box<i,T>... {
    ctor(); constexpr explicit ctor(Func,T&&... x); // init base boxes
    constexpr auto operator() <R> (R&& r) {&|const&|&&}
        requires std::ranges::input_range<R> && ranges::viewable_range<R> && invocable<Func,R,T[&|const&]...> && ranges::view<invoke_result_t<Func,R,T[&|const&]...>>
    { return Func{}((R&&)r, (box<i,T>{&|const&|&&})(*this).value_...); }
};
struct detail::view_closure<std::semiregular Func, std::copy_constructible...T>
    : pipeable< vlew_closure<Func,T...>, view_closure_impl<std::index_sequence_for<T...>,Func,T...> >
{ ctor(); constexpr explicit ctor(Func f, T&&...x); };

using range_adaptor_closure<D> = std::ranges::range_adaptor_closure<D>;

struct closure<F> : range_adaptor_closure<closure<F>> {
    constexpr ctor(F f) : f_{f}{}; // init
    constexpr decltype(auto) operator() <T> (T&& t) {const&|&&} requires std::invocable<F{const&|&&},T>
    { return <std::move>(f_)((T&&) t); }
private: [[no_unique_address]] F f_;
};

struct adaptor<F> {
    constexpr ctor(F f) : f_{f}{}; // init
    constexpr auto operator() <...Args> (Args&&...args) const {
        if constexpr (std::is_invocable_v<F const&, Args...>) return f_((Args&&)args...);
        else return closure{bind_back(f_, (Args&&)args...)};
    }
private: [[no_unique_address]] F f_;
};
```

* `range_adaptor_closure` is same as `std::ranges::range_adaptor_closure` (C++23).

------
### Configuration

* `BOOST_STL_INTERFACES_USE_CONCEPTS`: use concepts & constraints instead of SFINAE. Use `v2` instead of `v1`.
    * `BOOST_STL_INTERFACES_DISABLE_CONCEPTS`: forcely disable concept/constraints/`v2`.
* `BOOST_STL_INTERFACES_USE_DEDUCED_THIS`: use deducing this. Use `v3` instead of `v2`.
    * `BOOST_STL_INTERFACES_DISABLE_DEDUCED_THIS`: forcely disable `v3`.

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_detected.hpp>`

------
### Standard Facilities
