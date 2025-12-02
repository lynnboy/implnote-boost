# Boost.Histogram

* lib: `boost/libs/histogram`
* repo: `boostorg/histogram`
* commit: `3afe76f`, 2025-06-13

------
### Concepts

```c++
concept Axis<A,V,Alloc,Meta> = copy_constructible<A> && copy_assignable<A> && nothrow_move_assignable<A>
    && requires(A a, V v) { {a.size()}->index_type; {a.index(v)}->index_type; {a.get_allocator()}->Alloc;}
    && requires(A a, A b, V v, index_type i, index_type j, unsigned n, std::basic_ostream<Ch,Tr>& os, Archive ar) { // optional
        {a.update(v)} -> std::pair<index_type,index_type>; A{a,i,j,n};
        {a.options()}->unsigned; {a.inclusive()}->bool; {a.metadata()}->M&;
        {a==b}->bool; {a!=b}->bool; {os << a} -> std::basic_ostream<Ch,Tr>&; a.serialize(ar, n); };
concept DiscreteAxis<A,V,It,RIt> = Axis<A,V> = requires(A a, V v, index_type i) {
    {a.ordered()}->bool; {a.value(i)}->V; {a.bin(i)}->V;
    {s.begin()}->It; {s.end()}->It; {s.rbegin()}->RIt; {s.rend()}->RIt; };
concept IntervalAxis<A,V,B,It,RIt> = Axis<A,V> = requires(A a, V v, index_type i, real_index_type j) {
    {a.value(i)}->V; {a.bin(i)}->B;
    {s.begin()}->It; {s.end()}->It; {s.rbegin()}->RIt; {s.rend()}->RIt; };
concept Transform<T, floating_point X, floating_point Y> = default_constructible<T> && copy_constructible<T> && copy_assignable<T> &&
    requires(T t, X x, Y y) { {t.forward(x)} ->Y; {t.inverse(y)} ->X; {t==u} ->bool; }
    && requires(T t, Archive ar) { t.serialize(ar, n); }; // optional
concept Storage<S,Alloc> = default_constructible<S> && copy_constructible<S> && copy_assignable<S>
    && requires(S s, S t, S const cs, size_t i, size_t n) { typename S::value_type; typename S::<const>_reference; typename S::<const>_iterator;
        {S::has_threading_support}->bool;
        {s.size()}->size_t; s.resize(n);
        {s.{begin|end}()} -> S::iterator; {cs.{begin|end}()} -> S::<const>_iterator;
        {s[i]}->S::reference; {cs[i]}->S::const_reference;
        {s==t} ->bool; {s.get_allocator()} ->Alloc; }
    && requires(double x, Archive ar) { {S*=x}->S&; s.serialize(ar, n); }; // optional
concept Accumulator<A> = default_constructible<A> && copy_constructible<A> && copy_assignable<A>
    && requires(A a, A b, Ts...ts) { a(ts...); /*or*/ ++a; {a==b}->bool; {a!=b}->bool; }
    && requires(A a, A b, weight_type w, Ts...ts, basic_ostream<Ch,Tr>& os, Archive ar) {
        a+=w; /*or*/ a(w,ts...); {a+=b}->A&; {a*=x}->A&;
        {os<<a}->basic_ostream<Ch,Tr>&; a.serialize(ar, n); };
```

------
### Axis

```c++
namespace axis;

using index_type = int; using real_index_type = double;
struct null_type { void serialize<Archive>(Archive&, unsigned){} };
using empty_type = null_type;

// options
struct option::bitset<Bits> : std::integral_constant<unsigned, Bits> { static constexpr auto test<B>(bitset<B>); };
constexpr auto option::operator|<B1,B2>(bitset<B1>, bitset<B2>) { return bitset<(B1|B2)>{}; } // and &
constexpr auto option::operator-<B1,B2>(bitset<B1>, bitset<B2>) { return bitset<(B1 & ~B2)>{}; }
using option::bit<Pos> = bitset<(1<<Pos)>;
using option::none_t = bitset<0>; constexpr none_t none{};
using option::underflow_t = bit<0>; constexpr underflow_t underflow{};
using option::overflow_t = bit<1>; constexpr overflow_t overflow{};
using option::circular_t = bit<2>; constexpr circular_t circular{};
using option::growth_t = bit<3>; constexpr growth_t growth{};

using traits::value_type<Axis> = ...;
using traits::is_continuous<Axis> = ...;
using traits::is_reducible<Axis> = ...;
using traits::get_options<Axis> = ...;
using traits::is_inclusive<Axis> = ...;
using traits::is_ordered<Axis> = ...;
constexpr unsigned traits::options<Axis>(const Axis& axis) noexcept;
unsigned traits::options<...Ts>(const variant<Ts...>& axis) noexcept;
constexpr unsigned traits::inclusive<Axis>(const Axis& axis) noexcept;
bool traits::inclusive<...Ts>(const variant<Ts...>& axis) noexcept;
constexpr unsigned traits::ordered<Axis>(const Axis& axis) noexcept;
bool traits::ordered<...Ts>(const variant<Ts...>& axis) noexcept;
constexpr unsigned traits::continuous<Axis>(const Axis& axis) noexcept;
bool traits::continuous<...Ts>(const variant<Ts...>& axis) noexcept;
index_type traits::extent<Axis>(const Axis& axis) noexcept;
decltype(auto) traits::metadata<Axis>(Axis&& axis) noexcept;
decltype(auto) traits::value<Axis>(const Axis& axis, real_index_type index);
Result traits::value_as<Result,Axis>(const Axis& axis, real_index_type index);
index_type traits::index<Axis,U>(const Axis& axis, const U& value) noexcept;
index_type traits::index<...Ts,U>(const variant<Ts...>& axis, const U& value);
constexpr unsigned traits::rank<Axis>(const Axis& axis);
unsigned traits::rank<...Ts>(const variant<Ts...>& axis);
std::pair<index_type,index_type> traits::update<Axis,U>(Axis& axis, const U& value) noexcept;
std::pair<index_type,index_type> traits::update<...Ts,U>(variant<Ts...>& axis, const U& value);
decltype(auto) traits::width<Axis>(const Axis& axis, index_type index);
Result traits::width_as<Result,Axis>(const Axis& axis, index_type index);

class metadata_base<Metadata,empty> {
    mutable metadata_type data_;
protected: using metadata_type = Metadata;
    ctor()=default; ctor(const self&)=default; self& operator(const self&)=default;
    ctor(self&& o) noexcept; ctor(metadata_type&& o) noexcept; self& operator=(self&&) noexcept;
public: metadata_type& metadata() <const> noexcept { return data_; }
};
class metadata_base<Metadata,true> {
protected: using metadata_type = Metadata;
    ctor()=default; ctor(metadata_type&&){} self& operator=(self&&) noexcept { return *this; }
public: metadata_type& metadata() <const> noexcept { static metadata_type data; return data; }
};
using metadata_base_t<Metadata,Detail=replace_default<Metadata,std::string>>
    = metadata_base<Detail,(std::is_empty_v<Detail> && std::is_final_v<Detail>)>;

// axis intereval_view
class interval_view<Axis> {
    const Aixs& axis_; const index_type idx_;
public: ctor(const Axis& axis, index_type idx); ctor(Axis&& axis, index_type idx)=delete;
    decltype(auto) lower() const noexcept { return axis_.value(idx_); }
    decltype(auto) upper() const noexcept { return axis_.value(idx_+1); }
    decltype(auto) center() const noexcept { return axis_.value(idx_+0.5); }
    decltype(auto) width() const noexcept { return upper()-lower(); }
    bool operator==<BinType>(const BinType& rhs) const noexcept; // and !=
};
// axis iterator
class iterator<Axis> : public iterator_adaptor<self, index_type, decltype(declval<Axis>().bin(0))> {
    const Axis& axis_;
public: ctor(const Axis& axis, index_type idx);
    reference operator*() const { return axis_.bin(this->base()); }
};
struct iterator_mixin<Derived> {
    using const_iterator=iterator<Derived>; using const_reverse_iterator = std::reverse_iterator<const_iterator>;
    const_iterator {begin|end}() const noexcept { return {(const Derived*)this, {0|size()}}; }
    const_reverse_iterator {rbegin|rend}() const noexcept;
};

// polymorphic axis bin
class polymorphic_bin<Real> { using value_type = Real;
public: ctor(value_type lower, value_type upper): lower_or_value_{lower}, upper_{upper} {}
    operator const value_type& () const noexcept { return lower_or_value_; }
    value_type lower() const noexcept { return lower_or_value_; }
    value_type upper() const noexcept { return upper_; }
    value_type center() const noexcept { return 0.5 * (lower()+upper()); }
    value_type width() const noexcept { return upper()-lower(); }
    bool operator==<BinType>(const BinType& rhs) const noexcept; // and !=
    bool is_discrete() const noexcept { return lower_or_value_ == upper_; }
};

// axis regularity and transforms
namespace transform {
struct id {
    static T forward<T>(T&& x) noexcept { return std::forward<T>(x); }
    static T inverse<T>(T&& x) noexcept { return std::forward<T>(x); }
    void serialize<Archive>(Archive&, unsigned){}
};
struct log {
    static T forward<T>(T x) { return std::log(x); }
    static T inverse<T>(T x) { return std::exp(x); }
    void serialize<Archive>(Archive&, unsigned){}
};
struct sqrt {
    static T forward<T>(T x) { return std::sqrt(x); }
    static T inverse<T>(T x) { return x * x; }
    void serialize<Archive>(Archive&, unsigned){}
};
struct pow {
    double power=1;
    ctor()=default; explicit ctor(double p) : power{p} {}
    auto forward<T>(T x) const { return std::pow(x, power); }
    auto inverse<T>(T x) const { return std::pow(x, 1.0/power); }
    bool operator==(const pow& o) const noexcept { return power == o.power; }
    void serialize<Archive>(Archive&, unsigned){ ar & make_nvp("power", power); }
};
}
struct step_type<T> { T value; };
step_type<T> step<T>(T t) { return {t}; }

// boolean axis
struct boolean<Meta> : iterator_mixin<self>, metadata_base_t<Meta> { using value_type = bool;
    explicit ctor(metadata_type meta={}) noexcept;
    index_type index(value_type x) const noexcept { return {x}; }
    value_type value(index_type i) const noexcept { return {i}; }
    value_type bin(index_type i) const noexcept { return value(i); }
    index_type size() const noexcept { return 2; }
    static constexpr bool inclusive() noexcept { return true; }
    static constexpr unsigned options() noexcept { return option::none_t::value; }
    bool operator==<M>(const self<M>& o) const noexcept { return relaxed_equal{}(metadata(), o.metadata()); }
    bool operator!=<M>(const self<M>& o) const noexcept { return !operator==(o); }
    void serialize<Archive>(Archive& ar, unsigned) { ar & make_nvp("meta", metadata()); }
};
boolean()-> boolean<null_type>; boolean<M>(M) -> boolean<replace_cstring<decay_t<M>>>

// category axis
class category<Value,Meta,Options,Alloc> : public iterator_mixin<self>, public metadata_base_t<Meta> {
    using options_type = replace_default<Options,overflow_t>; using vector_type = std::vector<Value,Alloc>;
    vector_type vec_;
public: constexpr ctor()=default; explicit ctor(Alloc alloc);
    ctor<It>(It b, It e, metadata_type meta={}, options_type options={}, Alloc alloc={}) requires is_iterator<It>;
    ctor<It,A>(It b, It e, metadata_type meta, A alloc) requires is_iterator<It> && is_allocator<A>;
    ctor<C>(const C& c, metadata_type meta={}, options_type options={}, Alloc alloc={}) requires is_iterable<C>;
    ctor<C,A>(const C& c, metadata_type meta, A alloc) requires is_iterable<C> && is_allocator<A>;
    ctor<U>(std::initializer_list<U> il, metadata_type meta={}, options_type options={}, Alloc alloc={});
    ctor<U,A>(std::initializer_list<U> il, metadata_type meta, A alloc) requires is_allocator<A>;
    ctor(const self& src, index_type b, index_type e, unsigned merge);

    index_type index(const value_type& x) const noexcept
    { return (index_type)std::distance(vec_.begin(), std::find(vec_.begin(), vec_.end(), x)); }
    std::pair<index_type, index_type> update(const value_type& x) { const auto i = index(x);
        if (i<size()) return {i,0}; vec_.emplace_back(x); return {i, -1}; }
    auto value(index_type idx) const ->std::conditional_t<is_scalar_v<value_type>, value_type, const value_type&>
    { if (idx < 0 || idx >= size()) throw_exception(std::out_of_range("...")); return vec_[idx]; }
    decltype(auto) bin(index_type idx) const { return value(idx); }
    index_type size() const noexcept { return {vec_size()}; }
    static constexpr unsigned options() noexcept { return options_type::value; }
    static constexpr bool inclusive() noexcept { return options() & (overflow|growth); }
    static constexpr bool ordered() noexcept { return false; }
    bool operator== <V,M,O,A>(const self<V,M,O,A>& o) const noexcept; // relaxed_equal on two ranges and metadata
    bool operator!= <V,M,O,A>(const self<V,M,O,A>& o) const noexcept;
    Alloc get_allocator() const { return vec_.get_allocator(); }
    void serialize<Archive>(Archive& ar, unsigned) { ar & make_nvp("seq", vec_) & make_nvp("meta", metadata()); }
};
category<T>(std::initializer_list<T>) -> category<replace_cstring<std::decay_t<T>>, null_type>;
category<T,M>(std::initializer_list<T>,M) -> category<replace_cstring<std::decay_t<T>>, replace_cstring<decay_t<M>>>;
category<T,M,B>(std::initializer_list<T>,M,option::bitset<B> const&) -> category<replace_cstring<std::decay_t<T>>, replace_cstring<decay_t<M>>, option::bitset<B>>;

// integer axis
class integer<Value,Meta,Options> : public iterator_mixin<self>, public metadata_base_t<Meta> {
    using options_type = replace_default<Options,decltype(underflow|overflow)>;
    using local_index_type = conditional_t<is_integral_v<value_type>, index_type, real_index_type>;
    index_type size_{0}; value_type min_{0};
public: constexpr ctor()=default;
    ctor(value_type start, value_type stop, metadata_type meta={}, options_type options={});
    ctor(const self& src, index_type begin, index_type end, unsigned merge);

    index_type index(const value_type& x) const noexcept;
    auto update(const value_type& x) noexcept;
    value_type value(index_type idx) const noexcept;
    decltype(auto) bin(index_type idx) const noexcept;
    index_type size() const noexcept { return size_; }
    static constexpr unsigned options() noexcept { return options_type::value; }
    static constexpr bool inclusive() noexcept;
    static constexpr bool ordered() noexcept { return false; }
    bool operator== <V,M,O,A>(const self<V,M,O,A>& o) const noexcept;
    bool operator!= <V,M,O,A>(const self<V,M,O,A>& o) const noexcept;
    void serialize<Archive>(Archive& ar, unsigned) { ar & make_nvp("size", size_) & make_nvp("meta", metadata()) & make_nvp("min", min_); }
};
integer<T>(T,T) -> integer<convert_integer<T,index_type>,nulltype>;
integer<T,M>(T,T,M) -> integer<convert_integer<T,index_type>,replace_cstring<decay_t<M>>>;
integer<T,M,B>(T,T,M,const option::bitset<B>&) -> integer<convert_integer<T,index_type>,replace_cstring<decay_t<M>>, option::bitset<B>>;

// regular axis
class regular<Value,Trans,Meta,Options> : public iterator_mixin<self>, protected replace_default<Trans,transform::id>, public metadata_base_t<Meta> {
    using options_type = replace_default<Options,decltype(underflow|overflow)>; using transform_type = replace_default<Trans,transform::id>;
    using unit_type = get_unit_type<value_type>; using internal_value_type = get_scale_type<value_type>;
    index_type size_{0}; internal_value_type min_{0}, delta_{1};
public: constexpr ctor()=default;
    explicit ctor(<transform_type trans>, unsigned n, value_type start, value_type stop, metadata_type meta={}, options_type options={});
    explicit ctor<T>(<transform_type trans>, step_type<T> step, value_type start, value_type stop, metadata_type meta={}, options_type options={});
    ctor(const self& src, index_type begin, index_type end, unsigned merge);

    const transform_type& transform() const noexcept { return *this; }
    index_type index(value_type x) const noexcept;
    std::pair<index_type, index_type> update(value_type x) noexcept;
    value_type value(real_index_type i) const noexcept;
    decltype(auto) bin(index_type idx) const noexcept;
    index_type size() const noexcept { return size_; }
    static constexpr unsigned options() noexcept { return options_type::value; }
    bool operator== <V,T,M,O>(const self<V,T,M,O>& o) const noexcept;
    bool operator!= <V,T,M,O>(const self<V,T,M,O>& o) const noexcept;
    void serialize<Archive>(Archive& ar, unsigned) {
        ar & make_nvp("transform", (transform_type&)*this) & make_nvp("size", size_)
            & make_nvp("meta", metadata()) & make_nvp("min", min_) & make_nvp("delta", delta_); }
};
regular<[Tr],T>(Tr,unsigned,T,T)->regular<convert_integer<T,double>, {transform::id|Tr}, null_type>;
regular<[Tr],T,M>(Tr,unsigned,T,T,M)->regular<convert_integer<T,double>, {transform::id|Tr}, replace_cstring<std::decay_t<M>>>;
regular<[Tr],T,M,B>(Tr,unsigned,T,T,M,const option::bitset<B>&)->regular<convert_integer<T,double>, {transform::id|Tr}, replace_cstring<std::decay_t<M>>, option::bitset<B>>;

using circular<Value=default,Meta=use_default,Options=use_default> =
    regular<Value,transform::id, Meta, decltype(replace_default<Options,overflow_t>{}|circular)>;

// variable sized axis
class variable<Value,Meta,Options,Alloc> : public iterator_mixin<self>, public metadata_base_t<Meta> {
    using options_type = replace_default<Options,decltype(underflow|overflow)>; using transform_type = replace_default<Trans,transform::id>;
    using value_type = Value; using allocator_type = Alloc; using vector_type = std::vector<Value,allocator_type>;
    vector_type vec_;
public: constexpr ctor()=default; explicit ctor(allocator_type alloc): vec_{alloc}{}
    ctor<It>(It b, It e, metadata_type meta={}, options_type options={}, Alloc alloc={}) requires is_iterator<It>;
    ctor<It,A>(It b, It e, metadata_type meta, A alloc) requires is_iterator<It> && is_allocator<A>;
    ctor<C>(const C& c, metadata_type meta={}, options_type options={}, Alloc alloc={}) requires is_iterable<C>;
    ctor<C,A>(const C& c, metadata_type meta, A alloc) requires is_iterable<C> && is_allocator<A>;
    ctor<U>(std::initializer_list<U> il, metadata_type meta={}, options_type options={}, Alloc alloc={});
    ctor<U,A>(std::initializer_list<U> il, metadata_type meta, A alloc) requires is_allocator<A>;
    ctor(const self& src, index_type b, index_type e, unsigned merge);

    index_type index(const value_type& x) const noexcept;
    std::pair<index_type, index_type> update(const value_type& x);
    value_type value(real_index_type i) const noexcept;
    auto bin(index_type idx) const { return interval_view<self>(*this, idx); }
    index_type size() const noexcept { return {vec_size() - 1}; }
    static constexpr unsigned options() noexcept { return options_type::value; }
    bool operator== <V,M,O,A>(const self<V,M,O,A>& o) const noexcept;
    bool operator!= <V,M,O,A>(const self<V,M,O,A>& o) const noexcept;
    Alloc get_allocator() const { return vec_.get_allocator(); }
    void serialize<Archive>(Archive& ar, unsigned) { ar & make_nvp("seq", vec_) & make_nvp("meta", metadata()); }
};
variable<T>(std::initializer_list<T>) -> category<replace_cstring<std::decay_t<T>>, null_type>;
variable<T,M>(std::initializer_list<T>,M) -> category<replace_cstring<std::decay_t<T>>, replace_cstring<decay_t<M>>>;
variable<T,M,B>(std::initializer_list<T>,M,option::bitset<B> const&) -> category<replace_cstring<std::decay_t<T>>, replace_cstring<decay_t<M>>, option::bitset<B>>;
variable<Cont>(Cont,M) -> variable<convert_integer<decay_t<decltype(*begin(declval<Cont&>()))>, double>, null_type> requires is_iterable<Cont>;
variable<Cont,M>(Cont,M) -> variable<convert_integer<decay_t<decltype(*begin(declval<Cont&>()))>, double>, replace_cstring<decay_t<M>>>;
variable<Cont,M,B>(Cont,M,const option::bitset<B>&) -> variable<convert_integer<decay_t<decltype(*begin(declval<Cont&>()))>, double>, replace_cstring<decay_t<M>>, option::bitset<B>>;

// polymorphic axis
class variant<...Ts> : public iterator_mixin<self> {
    using impl_type = variant2::variant<Ts...>;
    using is_bounded_type<T> = mp_contains<self, decay_t<T>>;
    using metadata_type = remove_cv_ref_t<decltype(traits::metadata(declval<remove_pointer_t<mp_first<self>>>()))>;
public: ctor()=default; ctor(self{const &|&&})=default; self& operator=(self{const &|&&})=default;
    ctor<T>(T&& t) requires is_bounded_type<T> : impl{std::forward<T>(t)} {}
    self& operator= <T> (T&& t) requires is_bounded_type<T> { impl = std::forward<T>(t); return *this; }
    ctor<...Us>(const self<Us...>& u) { this->operator=(u); }
    self& operator=<...Us> (const self<Us...>& u) { visit([this](const auto& u){...}, u); return *this; };

    index_type size() const { return visit([](const auto&a){return a.size();}, *this); }
    unsigned options() const { return visit([](const auto&a){return traits::options(a);}, *this); }
    bool inclusive() const { return visit([](const auto&a){return traits::inclusive(a);}, *this); }
    bool ordered() const { return visit([](const auto&a){return traits::ordered(a);}, *this); }
    bool continuous() const { return visit([](const auto&a){return traits::continuous(a);}, *this); }
    metadata_type& metadata() <const> { return visit([](<const>auto&a){...}, *this); }
    index_type index<U>(const U& u) const { return visit([&u](const auto&a){return traits::index(a,u);}, *this); }
    double value(real_index_type idx) const { return visit([idx](const auto&a){return traits::value_as<double>(a,idx);}, *this); }
    auto bin(index_type idx) const { return visit([idx](const auto&a){...;}, *this); }
    void serialize<Archive>(Archive& ar, unsigned) { variant_proxy<self> p{*this}; ar & make_nvp("variant", p); }
};
class variant<>{};
decltype(auto) visit<Visitor,...Us>(Visitor&& vis, variant<Us...>{const&|&||&&} var);
auto get_if<T,...Us>(variant<Us...> <const>* v);
decltype(auto) get<T,...Us>(variant<Us...>{const&|&|&&} v);
decltype(auto) visit<Visitor,T>(Visitor&& vis, T&& var) { return std::forward<Visitor>(vis)(std::forward<T>(var)); }
decltype(auto) get<T,U>(U&& u) { return std::forward<U>(u); }
auto get_if<T,U>(<const> U* u) { return is_same_v<T,decay_t<U>> ? (<const>T*)u : nullptr; }
bool operator{==|!=} <...Us,...Vs>(const variant<Us...>& u, const variant<Vs...>& vs) noexcept;
bool operator{==|!=} <...Us,T>(const variant<Us...>& u, const T& t) noexcept;
bool operator{==|!=} <T,...Us>(const T& t, const variant<Us...>& u) noexcept;

// output of axis
class std::basic_ostream<Ts...>& operator<< <...Ts> (std::basic_ostream<Ts...>& os, const null_type&);
class std::basic_ostream<Ts...,U>& operator<< <...Ts> (std::basic_ostream<Ts...>& os, const interval_view<U>& i); // [<i.lower()>, <i.upper()>]
class std::basic_ostream<Ts...,U>& operator<< <...Ts> (std::basic_ostream<Ts...>& os, const polymorphic_bin<U>& i); // [<i.lower()>, <i.upper()>]
class std::basic_ostream<Ts...>& transform::operator<< <...Ts> (std::basic_ostream<Ts...>& os, const id&);
class std::basic_ostream<Ts...>& transform::operator<< <...Ts> (std::basic_ostream<Ts...>& os, const log&); // transform::log{}
class std::basic_ostream<Ts...>& transform::operator<< <...Ts> (std::basic_ostream<Ts...>& os, const sqrt&); // transform::sqrt{}
class std::basic_ostream<Ts...>& transform::operator<< <...Ts> (std::basic_ostream<Ts...>& os, const pow&); // transform::pow{}
class std::basic_ostream<Ts...>& operator<< <...Ts,...Us> (std::basic_ostream<Ts...>& os, const regular<Us...>& a); // regular([transform], size, value(0), value(size), metadata, options)
class std::basic_ostream<Ts...>& operator<< <...Ts,...Us> (std::basic_ostream<Ts...>& os, const integer<Us...>& a); // integer(value(0), value(size), metadata, options)
class std::basic_ostream<Ts...>& operator<< <...Ts,...Us> (std::basic_ostream<Ts...>& os, const variable<Us...>& a); // variable(value(0), ..., value(size), metadata, options)
class std::basic_ostream<Ts...>& operator<< <...Ts,...Us> (std::basic_ostream<Ts...>& os, const category<Us...>& a); // category(value(0), ..., value(size), metadata, options)
class std::basic_ostream<Ts...>& operator<< <...Ts,M> (std::basic_ostream<Ts...>& os, const boolean<M>& a); // boolean(metadata)
class std::basic_ostream<Ts...>& operator<< <...Ts,...Us> (std::basic_ostream<Ts...>& os, const variant<Us...>& v);
```

------
### Storage

```c++
struct detail::is_large_int<T>;
using detail::next_type<L,T> = mp_at_c<L,mp_find<L,T>::value+1>;
class detail::construct_guard<Alloc> { Alloc& a_; pointer p_; size_t n_; // no copy
public: ~dtor() { if (p_){ a_.deallocate(p_, n_); } } void release() { p_ = pointer{}; } };
void* detail::buffer_create<Alloc>(Alloc& a, size_t n);
auto detail::buffer_create<Alloc,It>(Alloc& a, size_t n, It iter);
void detail::buffer_destroy<Alloc>(Alloc& a, pointer p, size_t n);

class unlimited_storage<Alloc> { using U8=uint8_t; using U16=uint16_t; using U32=uint32_t; using U64=uint64_t;
    struct incrementor {
        void operator()<T>(T* tp, buffer_type& b, size_t i);
        void operator()(large_int* tp, buffer_type& b, size_t i);
        void operator()(double* tp, buffer_type& b, size_t i);
    };
    struct adder {
        void operator()<U>(double* tp, buffer_type&, size_t i, const U& x);
        void operator()(large_int* tp, buffer_type&, size_t i, const large_int& x);
        void operator()<T,U>(T* tp, buffer_type& b, size_t i, const U& x);
    };
    struct multiplier {
        void operator()<T>(T* tp, buffer_type& b, <size_t i>, double x);
        void operator()(double* tp, buffer_type& b, <size_t i>, double x);
    };
    class iterator_impl<Value,Reference> : public iterator_adaptor<self, size_t, Reference, Value> {
        mutable buffer_type* buffer_ = nullptr;
    public: ctor()=default; ctor<V,R>(const self<V,R>& it); ctor(buffer_type* b, size_t i) noexcept;
        Reference operator*() const noexcept { return {*buffer_, base()}; }
    };
    mutable buffer_type buffer_;
public:
    static constexpr bool has_threading_support = false;
    using allocator_type = Alloc; using value_type = double; using large_int = large_int<allocator_traits<allocator_type>::rebind_alloc<U64>>;
    struct buffer_type { using types = mp_list<U8,U16,U32,U64,large_int,double>;
        allocator_type alloc; size_t size{0}; unsigned type{0}; mutable void* ptr{nullptr};
        static constexpr unsigned type_index<T>() noexcept { return (unsigned)mp_find<types,T>::value; }
        decltype(auto) visit<F,...Ts>(F&& f, Ts&&...ts) const;
        ctor(const allocator_type& a={}); ~dtor() noexcept { destroy(); }
        ctor(self {const&|&&} o) noexcept; self& operator=(self {const&|&&} o) noexcept;
        void destroy() noexcept;
        void make<T,[U]>(size_t n, <U iter>);
    };
    class const_reference : partially_ordered<self, self, void> {
    protected: buffer_type& bref_; size_t idx_;
    public: ctor(buffer_type& b, size_t i) noexcept; ctor(const self&) noexcept=default; // no copy/move assign
        operator double() const noexcept;
        bool operator{<|==} (const self& o) const noexcept;
        bool operator{<|>|==} <U> (const U& o) const noexcept requires is_arithmetic_v<U> || is_same_v<large_int>;
    };
    class reference : public const_reference, public partially_ordered<self, self, void> {
    public: ctor(buffer_type& b, size_t i) noexcept; ctor(const self&) noexcept=default;
        self& operator=(const self& x); self& operator=(const const_reference& x); // assign through
        self& operator=<U> (const U& o) requires is_arithmetic_v<U> || is_same_v<large_int>;
        bool operator{<|==} (const self& o) const noexcept;
        bool operator{<|>|==} <U> (const U& o) const noexcept requires is_arithmetic_v<U> || is_same_v<large_int>;
        self& operator+=(const const_reference& x);
        self& operator+=<U>(const U& x) const noexcept requires is_arithmetic_v<U> || is_same_v<large_int>;
        self& operator-=(double x);
        self& operator*=(double x); self& operator/=(double x);
        self& operator++();
    };
    using <const>_iterator = iterator_impl<value_type<const>, <const>_reference>;
    explicit ctor(const allocator_type& a={}); ctor(self {const&|&&})=default; self& operator=(self {const&|&&})=default;
    explicit ctor<Iterable>(const Iterable& s) requires is_iterable<Iterable>;
    self& operator=<Iterable>(const Iterable& s) requires is_iterable<Iterable>;
    allocator_type get_allocator() const;
    void reset(size_t n);
    size_t size() const noexcept;
    <const>_reference operator[](size_t i) <const> noexcept;
    bool operator==(self const& x) const noexcept;
    bool operator==<Iterable>(Iterable const& iterable) const;
    self& operator*=(double x);
    <const>_iterator {begin|end}() <const> noexcept;
    ctor<T>(size_t s, const T* p, const allocator_type& a={});
    void serialize<Archive>(Archive& ar, unsigned); // "type", "size", "buffer"
};

struct detail::vector_impl<T> : T {
    static constexpr bool has_threading_support = is_thread_safe<T::value_type>::value;
    ctor(const allocator_type& a={}); // and defaulted copy/move-ctor/assign
    explicit ctor(T {const&|&&} t);
    explicit ctor<iterable U>(const U& u, const allocator_type& a={}); self& operator=<iterable U>(const U& u);
    void reset(size_t n);
    void serialize<Archive>(Archive& ar, unsigned); // "vector"
};
struct detail::array_impl<T> : T { size_t size_{0};
    static constexpr bool has_threading_support = is_thread_safe<T::value_type>::value;
    ctor(const allocator_type& a={}); // and copy/move-ctor/assign
    explicit ctor(T {const&|&&} t);
    explicit ctor<iterable U>(const U& u); self& operator=<iterable U>(const U& u);
    void reset(size_t n);
    <const>_iterator end() <const> noexcept { return begin() + size_; }
    size_t size() const noexcept { return size_; }
    void serialize<Archive>(Archive& ar, unsigned); // "size", "array"
};
struct detail::map_impl<T> : T {
    size_t size_{0};
    using value_type = T::mapped_type; using const_reference = const value_type&;
    static constexpr bool has_threading_support = false;
    struct reference {
        map_impl* map; size_t idx;
        ctor(map_impl* m, size_t i); // copy-ctor and copy-assign
        operator const_reference() const noexcept;
        self& operator=(const_reference u);
        self& operator+=<U,V=value_type>(const U& u) requires has_operator_radd<V,U>::value;
        self& operator-=<U,V=value_type>(const U& u) requires has_operator_rsub<V,U>::value;
        self& operator*=<U,V=value_type>(const U& u) requires has_operator_rmul<V,U>::value;
        self& operator/=<U,V=value_type>(const U& u) requires has_operator_rdiv<V,U>::value;
        self operator++<V=value_type>() requires has_operator_preincrement<V>::value;
        value_type operator++<V=value_type>(int) requires has_operator_preincrement<V>::value;
        bool operator{==|!=}<U>(const U& rhs) const requires has_operator_equal<value_type,U>::value;
        friend std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr>(std::basic_ostream<Ch,Tr>& os, self x) { return os << (const_reference)x; }
        auto operator() <...Ts> (const Ts&...args) ->decltype(...) { return (*map)[idx](args...); }
    };
    struct iterator_t<Value,Reference,MapPtr> : iterator_adaptor<self, size_t, Reference> {
        MapPtr map_=nullptr;
        ctor()=default;
        ctor<V,R,M>(const self<VmR,M>& it) noexcept requires is_convertible_v<M,MapPtr>;
        ctor(MapPtr m, size_t i);
        bool equal<V,R,M>(const self<V,R,M>& rhs) const noexcept;
        Reference operator*() const { return (*map_)[base()]; }
    };
    using <const>_iterator = iterator_t<<const> value_type, <const>_reference, <const> map_impl*>;
    ctor(const allocator_traits& a={}); // and defaulted copy/move-ctor/assign
    ctor(T {const&|&&} t);
    explicit ctor<iterable U>(const U& u, const allocator_type& a={});
    self& operator=<iterable U>(const U& u);
    void reset(size_t n);
    <const>_reference operator[](size_t i) <const> noexcept;
    <const>_iterator {begin|end}() <const> noexcept;
    size_t size() const noexcept;
    void serialize<Archive>(Archive& ar, unsigned); // "size", "array"
};
using detail::storage_adaptor_impl<T> = mp_cond<is_vector_like<T>,vecotr_impl<T>, is_array_like<T>,array_impl<T>, is_map_like<T>,map_impl<T>, true_type,struct ERROR_type<T>>;
struct storage_adaptor<T> : storage_adaptor_impl<T> {
    using base::ctor; using base::operator=;
    bool operator==<iterable U>(const U& u) const; // safe_equal over this and u
    void serialize<Archive>(Archive& ar, unsigned); // "impl" on base
};
```

------
### Accumulators

```c++
namespace accumulators;
struct is_thread_safe<T>;

class collector<Cont> {
    container_type container_;
public: using container_type = Cont; // and value_type, allocator_type, const_reference, const_pointer, <const>_iterator, size_type from Cont
    explicit ctor<...Args>(Args&&...args) requires is_constructible_v<Cont,Args...>;
    explicit ctor<T,...Args>(std::initializer_list<T> il, Args&&...args) requires is_constructible_v<Cont,...>;
    void operator()(const_reference x) { container_.push_back(x); }
    self& operator+=<C>(const self<C>& rhs);
    bool operator{==|!=}<iterable C>(const C& rhs) const noexcept;
    size_type size() const noexcept; size_type count() const noexcept;
    const const_iterator {begin|end}() const noexcept;
    const_reference operator[](size_type idx) const noexcept;
    const_pointer data() const noexcept;
    allocator_type get_allocator() const;
    void serialize<Archive>(Archive& ar, unsigned); // "container" for container_
};

class count<T, thread_safe> {
    using internal_type = conditional_t<thread_safe, atomic_number<T>, T>;
    internal_type value_{};
public: using value_type = T; using const_reference = const value_type&;
    ctor() noexcept=default; ctor(const_reference value) noexcept;
    ctor<T,b>(const self<T,b>& c) noexcept;
    self& operator++() noexcept;
    self& operator{+=|*=|/=}(const self& s) noexcept;
    self& operator{+=|*=}(const_reference value) noexcept;
    self operator{*|/}(const count& rhs) const noexcept;
    bool operator{==|!=|<|>|<=|>=}(const self& rhs) const noexcept;
    value_type value() const noexcept;
    explicit operator value_type() const noexcept;
    void serialize<Archive>(Archive& ar, unsigned); // "value" for value()
    static constexpr bool thread_safe() noexcept;
    friend bool operator{==|!=|<|>|<=|>=}(const_reference x, const self& rhs) noexcept;
};
struct std::common_type<count<T,b1>,count<U,b2>> { using type = count<common_type_t<T,U>, b1||b2>; };

class fraction<T> {
    value_type succ_{}, fail{};
public: using value_type = T; using const_reference = const value_type&;
    using real_type = std::conditional_t<is_floating_point_v<value_type>, value_type, double>;
    using interval_type = wilson_interval<real_type>::interval_type;
    ctor() noexcept=default; ctor(const_reference successes, const_reference failures) noexcept;
    ctor<T>(const self<T>& c) noexcept;
    void operator()(bool x) noexcept { if (x) ++succ_; else ++fail_; }
    self& operator+=(const self& s) noexcept;
    const_reference successes() const noexcept; const_reference failures() const noexcept;
    value_type count() const noexcept { return succ_ + fail_; }
    real_type value() const noexcept { return (real_type)succ_ / count(); }
    real_type variance() const noexcept { const real_type p = value(); return p*(1-p)/count(); }
    interval_type confidence_level() const noexcept { return wilson_interval<real_type>{}((real_type)successes(), (real_type)failures()); }
    bool operator{==|!=}(const self& rhs) const noexcept;
    void serialize<Archive>(Archive& ar, unsigned); // "value" for value()
};
struct std::common_type<fraction<T>,fraction<U>> { using type = fraction<common_type_t<T,U>>; };

class mean<T> {
    value_type sum_{}, mean_{}, sum_of_deltas_squared_{};
public: using value_type = T; using const_reference = const value_type&;
    ctor() noexcept=default; ctor(const_reference n, const_reference mean, const_reference variance) noexcept;
    ctor<T>(const self<T>& c) noexcept;
    void operator()(const_reference x) noexcept
    { sum_+=1; const auto delta=x-mean_; mean_+=delta/sum_; sum_of_deltas_squared_+=delta*(x-mean_); }
    void operator()(const weight_type<value_type>& w, const_reference x) noexcept
    { sum_+=w.value; const auto delta=x-mean_; mean_+=w.value*delta/sum_; sum_of_deltas_squared_+=w.value*delta*(x-mean_); }
    self& operator+=(const self& s) noexcept {
        if (rhs.sum_==0) return *this;
        const auto n1=sum_, mu1=mean_, n2=rhs.sum_, mu2=rhs.mean_;
        sum_+=rhs.sum_; mean_ = (n1*mu1 + n2*mu2)/sum_; sum_of_deltas_squared += rhs.sum_of_deltas_squared + n1*square(mean_-mu1) + n2*square(mean_-mu2);
        return *this;
    }
    self& operator*=(const_reference s) noexcept { mean_*=s; sum_of_deltas_squared_*=s*s; return *this; }
    bool operator{==|!=}(const self& rhs) const noexcept;
    const_reference count() const noexcept { return sum_; }
    const_reference value() const noexcept { return mean_; }
    value_type variance() const noexcept { return sum_of_deltas_squared_/(sum_-1); }
    void serialize<Archive>(Archive& ar, unsigned); // "sum", "mean", "sum_of_deltas_squared"
};
struct std::common_type<mean<T>,mean<U>> { using type = mean<common_type_t<T,U>>; };

class weighted_mean<T> {
    value_type sum_of_weights_{}, sum_of_weights_squared_{}, weighted_mean_{}, sum_of_weighted_deltas_squared_{};
public: using value_type = T; using const_reference = const value_type&;
    ctor() noexcept=default; ctor(const_reference wsum, const_reference wsum2, const_reference mean, const_reference variance);
    ctor<T>(const self<T>& c) noexcept;
    void operator()(const_reference x)
    void operator()(const weight_type<value_type>& w, const_reference x) noexcept
    { sum_of_weights_+=w.value; sum_of_weights_squared_ += w.value*w.value; const auto delta=x-weighted_mean_;
        weighted_mean_+=w.value*delta/sum_of_weights_; sum_of_weighted_deltas_squared_+=w.value*delta*(x-weighted_mean_); }
    self& operator+=(const self& s) noexcept {
        if (rhs.sum_of_weights_==0) return *this;
        const auto n1=sum_of_weights_, mu1=weighted_mean_, n2=rhs.sum_of_weights_, mu2=rhs.weighted_mean_;
        sum_of_weights_+=rhs.sum_of_weights_; weighted_mean_ = (n1*mu1 + n2*mu2)/sum_of_weights_;
        sum_of_weighted_deltas_squared_ += rhs.sum_of_weighted_deltas_squared_ + n1*square(weighted_mean_-mu1) + n2*square(weighted_mean_-mu2);
        return *this;
    }
    self& operator*=(const_reference s) noexcept { weighted_mean_*=s; sum_of_weighted_deltas_squared_*=s*s; return *this; }
    bool operator{==|!=}(const self& rhs) const noexcept;
    const_reference sum_of_weights() const noexcept; const_reference sum_of_weights_squared() const noexcept;
    value_type count() const noexcept { return square(sum_of_weights_) / sum_of_weights_squared_; }
    const_reference value() const noexcept { return weighted_mean_; }
    value_type variance() const noexcept { return sum_of_weighted_deltas_squared_ / (sum_of_weights_ - sum_of_weights_squared_/sum_of_weights_); }
    void serialize<Archive>(Archive& ar, unsigned); // "sum_of_weights", "sum_of_weights_squared", "weighted_mean", "sum_of_weighted_deltas_squared"
};
struct std::common_type<weighted_mean<T>,weighted_mean<U>> { using type = weighted_mean<common_type_t<T,U>>; };

class sum<T> {
    value_type large_{}, small_{};
public: using value_type = T; using const_reference = const value_type&;
    ctor() noexcept=default; ctor(const_reference large_part, const_reference small_part) noexcept;
    ctor<T>(const self<T>& s) noexcept;
    self& operator++() noexcept;
    self& operator+=(const_reference s) noexcept { volatile value_type l; value_type s;
        if (abs(large_)>=abs(value)) l=large_,s=value; else l=value,s=large_;
        large_+=value; l-=large_; l+=s; small_+=l;
        return *this;
    }
    self& operator*=(const_reference s) noexcept { large_*=s; small_*=s; return *this; }
    self& operator{+=|*=|/=}(const self& s) noexcept;
    self operator{*|/}(const self& rhs) const noexcept;
    bool operator{==|!=|<|>|<=|>=}(const self& rhs) const noexcept;
    const_reference value() const noexcept { return large_+small_; }
    const_reference large_part() const noexcept; const_reference small_part() const noexcept;
    explicit operator value_type() const noexcept { return value(); }
    void serialize<Archive>(Archive& ar, unsigned); // "large", "small"
};
struct std::common_type<sum<T>,sum<U>> { using type = sum<common_type_t<T,U>>; };

class weighted_sum<T> {
    value_type sum_of_weights_{}, sum_of_weights_squared_{};
public: using value_type = T; using const_reference = const value_type&;
    ctor() noexcept=default; ctor(const_reference value, const_reference variance=value) noexcept;
    ctor<T>(const self<T>& s) noexcept;
    self& operator++();
    self& operator+=(const weight_type<value_type>& w) noexcept { sum_of_weights_+=w.value; sum_of_weights_squared_+=square(w.value); return *this; }
    self& operator{+=|/=}(const self& s);
    bool operator{==|!=}(const self& rhs) const noexcept;
    const_reference value() const noexcept;
    const_reference variance() const noexcept;
    explicit operator const_reference() const;
    void serialize<Archive>(Archive& ar, unsigned); // "sum_of_weights", "sum_of_weights_squared"
};
struct std::common_type<weighted_sum<T>,weighted_sum<U>> { using type = weighted_sum<common_type_t<T,U>>; };
struct std::common_type<weighted_sum<T>,U> { using type = weighted_sum<common_type_t<T,U>>; };
struct std::common_type<T,weighted_sum<U>> { using type = weighted_sum<common_type_t<T,U>>; };

std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,U,b> (std::basic_ostream<Ch,Tr>& os, const count<U,b>& x); // just value
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,U> (std::basic_ostream<Ch,Tr>& os, const sum<U>& x); // sum(<large_part> + <small_part>)
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,U> (std::basic_ostream<Ch,Tr>& os, const weighted_sum<U>& x); // weighted_sum(<value>, <variance>)
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,U> (std::basic_ostream<Ch,Tr>& os, const mean<U>& x); // mean(<count>, <value>, <variance>)
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,U> (std::basic_ostream<Ch,Tr>& os, const weighted_mean<U>& x); // weighted_mean(<sum_of_weights>, <value>, <variance>)
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,U> (std::basic_ostream<Ch,Tr>& os, const fraction<U>& x); // fraction(<successes>, <failures>)
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,U> (std::basic_ostream<Ch,Tr>& os, const collector<U>& x); // collector{...}
```

------
### Algorithms

```c++
namespace algorithm;

auto empty<A,S>(const histogram<A,S>& h, coverage cov)

auto project<A,S,n,...Ns>(const histogram<A,S>& h, std::integral_constant<unsigned,n>, Ns...);
auto project<A,S,iterable Cont>(const histogram<A,S>& h, const Cont& c);

reduce_command shrink(unsigned iaxis, double lower, double upper)
{ return {.iaxis=iaxis, .range=values, .begin={lower}, .end={upper}, .merge=1, .crop=false}; }
reduce_command shrink(double lower, double upper) { return shrink(unset, lower, upper); };
reduce_command crop(unsigned iaxis, double lower, double upper) { auto r = shrink(iaxis, lower, upper); r.crop=true; return r; }
reduce_command crop(double lower, double upper) { return crop(unset, lower, upper); };
enum class slice_mode { shrink, crop };
reduce_command slice(unsigned iaxis, index_type begin, index_type end, slice_mode mode=shrink)
{ return {.iaxis=iaxis, .range=indices, .begin={lower}, .end={upper}, .merge=1, .crop=mode==crop}; }
reduce_command slice(index_type begin, index_type end, slice_mode mode=shrink) { return slice(unset, begin, end mode); }
reduce_command rebin(unsigned iaxis, unsigned merge)
{ return {.iaxis=iaxis, .range=indices, .merge=merge, .crop=mode==crop}; }
reduce_command rebin(unsigned merge) { return rebin(unset, merge); }
reduce_command shrink_and_rebin(unsigned iaxis, double lower, double upper, unsigned merge) { auto r = shrink(iaxis, lower, upper); r.merge=rebin(merge).merge; return r; }
reduce_command shrink_and_rebin(double lower, double upper, unsigned merge) { return shrink_and_rebin(unset, lower, upper, merge); }
reduce_command crop_and_rebin(unsigned iaxis, double lower, double upper, unsigned merge) { auto r = crop(iaxis, lower, upper); r.merge=rebin(merge).merge; return r; }
reduce_command crop_and_rebin(double lower, double upper, unsigned merge) { return crop_and_rebin(unset, lower, upper, merge); }
reduce_command slice_and_rebin(unsigned iaxis, index_type begin, index_type end, unsigned merge, slice_mode mode=shrink)
{ auto r = slice(iaxis, begin, end, mode); r.merge=rebin(merge).merge; return r; }
reduce_command slice_and_rebin(index_type begin, index_type end, unsigned merge, slice_mode mode=shrink) { return slice_and_rebin(unset, begin, end, merge, mode); }
Hist reduce<Hist,iterable Cont>(const Hist& hist, const Cont& options);
Hist reduce<Hist,...Ts>(const Hist& hist, const reduce_command& opt, const Ts&... opts) { return reduce(hist, std::initializer_list{opt, opts...});}
auto sum<A,S>(const histogram<A,S>& hist, const coverage cov=all);
```

histogram, literals, make_histogram, make_profile, multi_index, ostream, sample, serialization

------
### Interval Utilities

```c++
struct binomial_proportion_interval<T> { // base class
    using value_type = T; using interval_type = std::pair<value_type, value_type>;
    virtual interval_type operator()(value_type successes, value_type failures) const noexcept=0;
    interval_type operator()<T>(const fraction<T>& fraction) const noexcept { return operator()(fraction.successes(), fraction.failures()); }
};
class deviation { double d_;
public: explicit ctor(double d) d_{d} { if (d<=0) throw_exception(std::invalid_argument{"..."}); }
    explicit operator T() const noexcept requires floating_point<T>;
    operator confidence_level() const noexcept { return {std::fma(2.0, normal_cdf(d_), -1.0)}; }
    friend derivation operator*(derivation d, double z) noexcept;
    friend derivation operator*(double z, derivation d) noexcept;
    friend bool operator{==|!=}(derivation a, deviation b) noexcept;
};
class confidence_level { double cl_;
public: explicit ctor(double cl) : cl_{cl} { if (cl<=0 || cl>=1) throw_exception(std::invalid_argument{""}); }
    explicit operator T() const noexcept requires floating_point<T>;
    operator deviation() const noexcept { return {normal_ppf(std::fma(0.5, cl_, 0.5))}; }
    friend bool operator{==|!=}(derivation a, deviation b) noexcept;
};

class clopper_pearson_interval<T> : public binomial_proportion_interval<T> {
    value_type alpha_half_;
public: explicit ctor(confidence_level cl=deviation{1}) noexcept : alpha_half_{(value_type)(0.5 - 0.5*(double)cl)}{}
    using base::operator();
    interval_type operator()(value_type successes, value_type failures) const noexcept override {
        const value_type one{1.0}, zero{0.0}, total{successes+failures};
        if (successes==0) return {zero, one-std::pow(alpha_half_, one/total)};
        if (failures==0) return {std::pow(alpha_half_, one/total), one};
        math::beta_distrubution<value_type> beta_a{successes,failures+1}, beta_b{successes+1,failures};
        return {math::quantile(beta_a, alpha_half_), math::quantile(beta_b, one-alpha_half_)};
    }
};

class jeffreys_interval<T> : public binomial_proportion_interval<T> {
    value_type alpha_half_;
public: explicit ctor(confidence_level cl=deviation{1}) noexcept : alpha_half_{(value_type)(0.5 - 0.5*(double)cl)}{}
    using base::operator();
    interval_type operator()(value_type successes, value_type failures) const noexcept override {
        const value_type half{0.5}, one{1.0}, zero{0.0}, total{successes+failures};
        if (successes==0) return {zero, one-std::pow(alpha_half_, one/total)};
        if (failures==0) return {std::pow(alpha_half_, one/total), one};
        math::beta_distrubution<value_type> beta{successes+half, failures+half};
        return {successes==1 ? zero : math::quantile(beta, alpha_half_), failures==1 ? one : math::quantile(beta, one-alpha_half_)};
    }
};

class wald_interval<T> : public binomial_proportion_interval<T> {
    value_type z_;
public: explicit ctor(derivation d=deviation{1}) noexcept : z_{(value_type)d}{}
    using base::operator();
    interval_type operator()(value_type successes, value_type failures) const noexcept override {
        const value_type total_inv=1/(successes+failures), a=successes*total_inv, b=(z_*total_inv)*std::sqrt(successes*failures*total_inv);
        return {a-b, a+b};
    }
};

class wilson_interval<T> : public binomial_proportion_interval<T> {
    value_type z_;
public: explicit ctor(derivation d=deviation{1}) noexcept : z_{(value_type)d}{}
    using base::operator();
    interval_type operator()(value_type successes, value_type failures) const noexcept override {
        const value_type half{0.5}, quarter{0.25}, zsq{z_*z_}, total={successes+failures};
        const value_type minv=1/(total+zsq), t1=(successes+half*zsq)*minv, t2=z_*minv*std::sqrt(successes*failures/total + quarter*zsq);
        return {t1-t2, t1+t2};
    }
};
```

------
### Common Parts

```c++
struct weight_type<T> { T value; operator self<U>() const; };
auto weight<T>(T&& t) noexcept { return weight_type<T>{std::forward<T>(t)}; }

enum class coverage { inner, all };
class indexed_range<Hist> {
    using histogram_type = Hist;
    static constexpr unsigned buffer_size = buffer_size<std::decay_t<Hist>::axes_type>::value;
    iterator begin_, end_;
    auto make_range(Hist& hist, coverage cov);
public: using value_iterator = std::conditional_t<is_const_v<Hist>, Hist::const_iterator, Hist::iterator>;
    using value_reference = std::iterator_traits<value_iterator>::reference;
    using value_type = std::iterator_traits<value_iterator>::value_type;
    class accessor : mirrored<self,void> {
        iterator& iter_;
        ctor(iterator& i) noexcept;
        ctor(const self&)=default;
    public: class index_view { using index_pointer = const iterator::index_data*;
            index_pointer begin_, end_;
            ctor(index_pointer b, index_pointer e);
        public: using const_reference = const axis::index_type&;
            class const_iterator : public iterator_adaptor<self, index_pointer, const_reference> {
                explicit ctor(index_pointer i) noexcept;
            public: const_reference operator*() const noexcept { return base()->idx; }
            };
            const_iterator {begin|end}() const noexcept;
            size_t size() const noexcept;
            const_reference operator[](unsigned d) const noexcept;
            const_reference at(unsigned d) const;
        };
        self& operator=(const self& o);
        self& operator=<T>(const T& x);
        value_reference get() const noexcept;
        value_reference operator*() const noexcept;
        value_iterator operator->() const noexcept;
        axis::index_type index(unsigned d=0) const noexcept;
        index_view indices() const noexcept;
        decltype(auto) bin<n=0>(std::integral_constant<unsigned,n>={}) const;
        decltype(auto) bin(unsigned d) const;
        double density() const;
        bool operator{<|>|==|!=|<=|>=}(const self& o) noexcept;
        bool operator{<|>|==|!=|<=|>=} <U> (const U& o) const noexcept;
        operator value_type() const noexcept;
    };
    class iterator {
        struct pointer_proxy { reference ref_; reference* operator->() noexcept; };
        struct index_data { axis::index_type idx, begin, end; size_t begin_skip, end_skip; };
        struct indices_t : private std::array<index_data, buffer_size> {
            Hist* hist_;
            using <const>_pointer = <const> index_data*;
            ctor(Hist* h) noexcept;
            using base::operator[];
            unsigned size() const noexcept;
            <const>_pointer {begin|end}() <const> noexcept;
        };
        value_iterator iter_; indices_t indices_;
        ctor(value_iterator i, Hist& h);
    public: using pointer = pointer_proxy; using difference_type = ptrdiff_t; using iterator_category = forward_iterator_tag;
        reference operator*() noexcept;
        pointer operator->() noexcept { return pointer_proxy{operator*()}; }
        iterator& operator++(); iterator operator++(int);
        bool operator{==|!=}(const iterator& x) const noexcept;
        bool operator{==|!=}(const value_iterator& x) const noexcept;
        size_t offset() const noexcept;
    };
    ctor(Hist& hist, coverage cov);
    ctor<iterable Cont>(Hist& hist, Cont&& range);
    iterator {begin|end}() noexcept;
};

auto indexed<Hist>(Hist&& hist, coverage cov=inner)
{ return indexed_range<std::remove_reference_t<Hist>>{std::forward<Hist>(hist), cov}; }
auto indexed<Hist,iterable Cont>(Hist&& hist, Cont&& range)
{ return indexed_range<std::remove_reference_t<Hist>>{std::forward<Hist>(hist), std::forward<Cont>(range)}; }

struct unsafe_access {
    static <const> auto& axes<Hist>(<const> Hist& hist);
    static decltype(auto) axis<Hist,i=0>(Hist& hist, std::integral_constant<unsigned,i>={});
    static decltype(auto) axis<Hist>(Hist& hist, unsigned i);
    static <const> auto& storage<Hist>(<const> Hist& hist);
    static <const> auto& offset<Hist>(<const> Hist& hist);
    static constexpr auto& unlimited_storage_buffer<Alloc>(unlimited_storage<Alloc>& storage);
    static constexpr auto& storage_adaptor_impl<T>(storage_adaptor<T>& storage);
};
```

------
### Implementation Details

```c++
namespace detail;

struct priority<n> : priority<n-1>{}; struct detail::priority<0>{};
struct relaxed_equal { constexpr bool operator()<T,U>(const T& t, const U& u) const noexcept{...} };
using replace_type<T,From,To> = std::conditional_t<is_same_v<T,From>, To, T>;
using replace_default<T,Default> = replace_type<T,use_default,Default>;
using replace_cstring<T> = replace_type<T,const char*,std::string>;

// detect
using void_t<...> = void;
struct detect_base{ static T&& val<T>(); static T& ref<T>(); static T const& cref<T>(); };
using has_method_reset<T> = requires(T& t) {t.reset(0);};
using has_method_push_back<T> = requires(T& t) {&T::push_back;};
using is_indexable<T> = requires(T& t) {t[0];};
using is_transform<T,U=T> = requires(T& t, U& u) {t.inverse(t.forward(u));};
using is_indexable_container<T> = requires(T& t) {t[0]; t.size(), std::begin(t); std::end(t);};
using is_vector_like<T> = requires(T& t) {t[0]; t.size(); t.resize(0); std::begin(t); std::end(t);};
using is_array_like<T> = requires(T& t) {t[0]; t.size(); std::tuple_size<T>::value; std::begin(t); std::end(t);};
using is_map_like<T> = requires(T& t) {typename T::key_type; typename T::mapped_type; std::begin(t); std::end(t);};
using is_axis<T> = requires(T& t) {t.size(); &T::index;};
using is_iterable<T> = requires(T& t) {std::begin(t); std::end(t);};
using is_iterator<T> = requires(T& t) {typename std::iterator_traits<T>::iterator_category;};
using is_streamable<T> = requires(T& t, std::ostream& os) {os<<t;};
using is_allocator<T> = requires(T& t) {T::allocate; T::deallocate;};
using has_operator_preincrement<T> = requires(T& t) {++t;};
using has_operator_equal<T,U=T> = requires(const T& t, U& u) {t == u;};
using has_operator_radd<T,U=T> = requires(T& t, U& u) {t += u;};
using has_operator_rsub<T,U=T> = requires(T& t, U& u) {t -= u;};
using has_operator_rmul<T,U=T> = requires(T& t, U& u) {t *= u;};
using has_operator_rdiv<T,U=T> = requires(T& t, U& u) {t /= u;};
using has_method_eq<T,U=T> = requires(const T& t, U& u) {t.operator==(u);};
using has_threading_support<T> = requires(T& t) {T::has_threading_support;};
using is_explicitly_convertible<T,U=T> = requires(T& t, U& u) {static_cast<U>(t);};
using is_complete<T> = requires(T& t) {sizeof(T);};
using is_storage<T> = mp_and<is_indexable_container<T>, has_method_reset<T>, has_threading_support<T>>;
using is_adaptible<T> = mp_and<mp_not<is_storage<T>>, mp_or<is_vector_like<T>, is_array_like<T>, is_map_like<T>>>;
using is_tuple<T> = ...; using detail::is_variant<T> = ...;
using is_axis_variant<T> = ...;
using is_any_axis<T> = mp_or<is_axis<T>, is_axis_variant<T>>;
using is_sequence_of_axis<T> = mp_and<is_iterable<T>, is_axis<mp_first<T>>>;
using is_sequence_of_axis_variant = mp_and<is_iterable, is_axis_variant<mp_first<T>>>;
using is_sequence_of_any_axis = mp_and<is_iterable, is_any_axis<mp_first<T>>>;
struct requires_storage<T>{}; requires_storage_or_adaptible; requires_iterator; requires_iterable; requires_axis; requires_any_axis;
    requires_sequence_of_axis; requires_sequence_of_axis_variant; requires_sequence_of_any_axis; requires_axes; requires_transform, requires_allocator

// iterator_adaptor
struct operator_arrow_dispatch_t<Ref> {
    struct pointer{ Ref m_ref; Ref* operator->() noexcept { return std::addressof(m_ref); } } using result_type = pointer;
    static result_type apply(Ref const& x) noexcept { return {x}; } };
struct operator_arrow_dispatch_t<T&> { using result_type = T*;
    static result_type apply(Ref const& x) noexcept { return std::addressof(x); } };
struct get_difference_type<T> = ...;
class iterator_adaptor<Derived,Base,Ref=std::remove_pointer_t<Base>&,Value=std::decay_t<Ref>> {
    <const> Derived& derived() <const> noexcept; 
    base_type iter_;
public: using base_type = Base; using reference = Ref; using value_type = Value; using pointer = operator_arrow_dispatch_t::result_type;
    using difference_type = get_difference_type<Base>; using iterator_category = std::random_access_iterator_tag;
    ctor(); explicit ctor(base_type const& iter);
    decltype(auto) operator*() const noexcept;
    Derived& operator+=(difference_type); difference_type operator-<...Ts>(const self<Ts...>& x) const noexcept;
    bool operator==<...Ts>(const self<Ts...>&) const noexcept; // and !=, <, >, >=, <=
    reference operator[](difference_type) const; pointer operator->() const noexcept;
    Derived& operator-=(difference_type);
    Derived& operator++(); Derived& operator--(); Derived operator++(int); derived operator--(int);
    Derived operator+(difference_type) const; Derived operator-(difference_type);
    friend Derived operator+(difference_type, const Derived&);
    Base const& base() const noexcept;
protected: using iterator_adaptor_ = self;
};

constexpr decltype(auto) static_if_c<b,...Ts>(Ts&&...ts) noexcept;
constexpr decltype(auto) static_if<Bool,...Ts>(Ts&&...ts) noexcept;

constexpr T* ptr_cast<T,U>(U*);
T try_cast<T,E,U>(U&& u) noexcept(...);

std::string type_name<T>();

using args_type<FuncPtr> = std::tuple<...>; // tuple of parameter type list
using arg_type<T,size_t n=0> = std::tuple_element_t<n,args_type<T>>;

using convert_integer<T,U> = std::conditional_t<is_integral_v<decay_t<T>>, U,T>;

struct counting_streambuf<Ch,Tr=std::char_traits<Ch>> : std::basic_streambuf<Ch,Tr> {
    std::streamsize* p_count;
    ctor(std::streamsize& c) : p_count{&c}{}
    std::streamsize xsputn(const char_type*, std::streamsize n) override { *p_count += n; return n; }
    int_type overflow(int_type ch) override { ++*p_count; return ch; }
};
struct count_guard<Ch,Tr> {
    using bos = std::basic_ostream<Ch,Tr>; using bsb = std::basic_streambuf<Ch,Tr>;
    counting_streambuf<Ch,Tr> csb; bos* p_os; bsb* p_rdbuf;
    ctor(bos& os, std::streamsize& s) : csb{s}, p_os{&os}, p_rdbuf{os.rdbuf(&csb)} {}
    ctor(self&& o); self& operator=(self&& o); // move
    ~dtor() { if (p_os) p_os->rdbuf(p_rdbuf); }
};
count_guard<Ch,Tr> make_count_guard<Ch,Tr>(std::basic_ostream<Ch,Tr>& os, std::streamsize& s) { return {os, s}; }

struct variant_proxy<Variant> {
    Variant& variant;
    void serialize<Archive>(Archive& ar, unsigned) {...}; // "which", "value"
};

using has_array_optimization<T>;
struct array_wrapper<T> { using pointer = T*;
    pointer ptr; size_t size;
    void serialize<Archive>(Archive ar, unsigned);
};
auto make_array_wrapper<T>(T* t, size_t s) { return array_wrapper<T>{t, s}; }

struct mirrored<T,U>{ friend bool operator<(const U& a, const T& b) noexcept{return b > a;} }; // all 6 comparasions, inversed impl
struct mirrored<T,void> { friend bool operator< <U> (const U& a, const T& b) noexcept requires !is_same_v<T,U> { return b > a;} }; // all 6 comparasions, inversed impl
struct mirrored<T,T> { friend bool operator>(const T& a, const T& b) noexcept { return b.operator<(a); } };
struct equality<T,U> { friend bool operator!=(const T& a, const U& b) noexcept { return !a.operator==(b); } };
struct equality<T,void> { friend bool operator!=(const T& a, const U& b) noexcept requires !is_same_v<T,U> { return !(a==b); } };
using totally_ordered<T,...Ts> = ...;
using partially_ordered<T,...Ts> = ...;

auto make_unsigned<T>(const T& t) noexcept;
using number_category<T> = ...;
struct safe_equal { bool operator()<T,U>(const T& t, const U& u) const noexcept; };
struct safe_less { bool operator()<T,U>(const T& t, const U& u) const noexcept; };
struct safe_greater { bool operator()<T,U>(const T& t, const U& u) const noexcept; };

using is_unsigned_integral<T> = mp_bool<is_integral_v<T>, is_unsigned_v<T>>;
bool safe_increment<T>(T& t);
bool safe_radd<T,U>(T& t, const U& u);
struct large_int<Alloc> : totally_ordered<self,self>, partially_ordered<self,void> {
    std::vector<uint64_t, Alloc> data;
    explicit ctor(std::uint64_t v=0, const Alloc& a={}): data{1,v,a}{}
    ctor(self{const&|&&})=default; self& operator=(self{const&|&&})=default;
    self& operator=(uint64_t o);
    self& operator++();
    self& operator+=(const self& o); self& operator+=(uint64_t o);
    explicit operator double() const noexcept;
    bool operator{<|==}(const self& o) const noexcept;
    bool operator{<|>|==} <{integral|floating_point} U>(const U& o) const noexcept;
    bool operator{<|>|==} <U>(const U& o) const noexcept requires (!is_arithmetic_v<U> && is_convertible_v<U,double>);
    uint64_t& maybe_extend(size_t i);
    static void add_remainder(uint64_t& d, uint64_t o) noexcept;
    void serialize<Archive>(Archive& ar, unsigned);
};

struct atomic_number<T> : std::atomic<T> {
    using base::ctor;
    ctor() noexcept=default; ctor(const self& o) noexcept; self& operator=(const self& o) noexcept;
    self& operator++() noexcept;
    self& operator{+=|*=|/=}(const T& x) noexcept;
};

double normal_cdf(double x) noexcept { return std::fma(0.5, std::erf(x/std::sqrt(2)), 0.5); }
double normal_ppf(double p) noexcept { return std::sqrt(2) * erf_inv(2 * (p-0.5)); }

T square<T>(T t) { return t*t; }

T make_default<T>(const T& t);

constexpr auto data<C>(<const> C& c) -> decltype(c.data()) { return c.data(); }
constexpr T* data<T,n>(T (&aray)[n]) noexcept { return array; }
constexpr const E* data<E>(std::initializer_list<E> il) noexcept { return il.begin(); }
constexpr <const> E* data<E>(<const> std::valarray<E>& v) noexcept { return std::begin(v); }
constexpr auto size<C>(const C& c) -> decltype(c.size()) { return c.size(); }
constexpr size_t size<T,n>(const T(&)[n]) { return n; }

constexpr auto invalid_index = ~(size_t)0;
struct optional_index {
    size_t value;
    self& operator=(size_t x) noexcept;
    self& operator+=(intptr_t x) noexcept;
    self& operator+=(const self& x) noexcept;
    operator size_t() const noexcept;
    friend bool operator<=(size_t x, self idx) noexcept;
};
constexpr bool is_valid(size_t) noexcept { return true; }
bool is_valid(const optional_index x) noexcept { return x.value!=invalid_index; }

using dynamic_size = std::integral_constant<size_t, -1>;
constexpr dynamic_size relaxed_tuple_size<T>(const T&) noexcept;
constexpr sd::integral_constant<size_t,sizeof...(Ts)> relaxed_tuple_size<...Ts>(const std::tuple<Ts...>&) noexcept;
using relaxed_tuple_size_t<T> = decltype(relaxed_tuple_size(std::declval<T>()));

class static_vector<T,n> {
    size_type size_=0; element_type data_[n];
public: using element_type = T; using size_type = size_t;
    using <const>_reference = <const>T&; using <const>_point=<const>T*; using <const>_iterator = <const>_pointer;
    ctor()=default; explicit ctor(size_t s) noexcept;
    ctor(size_t s, const T& value) noexcept(...);
    ctor(std::initializer_list<T> il) noexcept(...);
    <const>_reference at(size_type pos) <const> noexcept;
    <const>_reference operator[](size_type pos) <const> noexcept;
    <const>_reference {front|back}() <const> noexcept;
    <const>_pointer data() <const> noexcept;
    <const>_iterator {begin|end} <const> noexcept;
    const_iterator {cbegin|cend}() const noexcept;
    constexpr size_type max_size() const noexcept;
    size_type size() const noexcept;
    bool empty() const noexcept;
    void fill(const_reference value) noexcept(...);
    void swap(self& other) noexcept(...);
};
bool operator{==|!=}<T,n>(const static_vector<T,n>& a, const static_vector<T,n>& b) noexcept;
void std::swap<T,n>(const static_vector<T,n>& a, const static_vector<T,n>& b) noexcept(...);

void for_each_axis<T,Unary>(T&& t, Unary&& p);
struct axis_merger { T operator()<T,U>(const T& a, const U& u); };
auto make_empty_dynamic_axes<T>(const T& axes) { return make_default(axes); }
auto make_empty_dynamic_axes<...Ts>(const std::tuple<Ts...>&);
auto axes_transform<...Ts,Functor>(const std::tuple<Ts...>& old_axes, Functor&& f);
T axes_transform<T,Functor>(const T& old_axes, Functor&& f);
std::tuple<Ts...> axes_transform<...Ts,Binary>(const std::tuple<Ts...>& lhs, const std::tuple<Ts...>& rhs, Binary&& bin);
T axes_transform<T,Binary>(const T& lhs, const T& rhs, Binary&& bin);
unsigned axes_rank<T>(const T& axes);
constexpr unsigned axes_rank<...Ts>(const std::tuple<Ts...>&);
void throw_if_axes_is_too_large<T>(const T& axes);
void throw_if_axes_is_too_large<...Ts>(const std::tuple<Ts...>&);
decltype(auto) axis_get<n,...Ts>(<const> std::tuple<Ts...>& axes);
decltype(auto) axis_get<n,T>(<const> T& axes);
decltype(auto) axis_get<Ts>(<const> std::tuple<Ts...>& axes, unsigned i);
decltype(auto) axis_get<T>(<const> T& axes, unsigned i);
bool axes_equal<T,U>(const T& u, const U& u) noexcept;
std::tuple<Us...> axes_assign<...Ts,...Us>(std::tuple<Ts...>&, const std::tuple<Us...>&) requires(/*not same*/);
void axes_assign<...Ts>(std::tuple<Ts...>& t, const std::tuple<Ts...>& u);
void axes_assign<...Ts,U>(std::tuple<Ts...>& t, const U& u);
void axes_assign<T,...Us>(T& t, const std::tuple<Us...>& u);
void axes_assign<T,U>(T& t, const U& u);
void axes_serialize<Archive,T>(Archive& ar, T& axes) { ar & make_nvp("axes", axes); }
void axes_serialize<Archive,...Ts>(Archive& ar, std::tuple<Ts...>& axes); // "axes", with each "item"
size_t bincount<T>(const T& axes);
size_t offset<T>(const T& axes);
auto make_stack_buffer<T,A>(const A& a, <const T& t>);
using has_underflow<T>;
using is_growing<T>;
using is_not_inclusive<T>;
using axis_types<T>;
using has_special_axis<Trait<T>,Axes>;
using has_growing_axis<Axes>;
using has_non_inclusive_axis<Axes>;
constexpr size_t type_score<T>();
using type_less<T,U>
using value_types<Axes>;

struct reduce_command {
    static constexpr unsigned unset = (unsigned)-1;
    unsigned iaxis = unset;
    enum class range_t : char { none, indices, values } range = none;
    union { index_type index; double value; } begin{0}, end{0};
    unsigned merge=0;
    bool crop=false, is_ordered=true, use_underflow_bin=true, use_overflow_bin=true;
};

void normalize_reduce_commands(span<reduce_command> out, span<const reduce_command> in);
```

detail/: accumulator_traits, argument_traits, chunk_vector, common_type, debug,
    erf_inv, fill_n, fill, ignore_deprecation_warning_beginend, index_translator,
    large_int, limits, linearize, mutex_base, term_info, tuple_slice

------
### Dependencies

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/workaround.hpp>`
* `<boost/config/pragma_message.hpp>`

#### Boost.Core

* `<boost/core/alloc_construct.hpp>`
* `<boost/core/empty_value.hpp>`
* `<boost/core/exchange.hpp>`
* `<boost/core/make_span.hpp>`
* `<boost/core/nvp.hpp>`
* `<boost/core/span.hpp>`
* `<boost/core/typeinfo.hpp>`
* `<boost/core/use_default.hpp>`
* `<boost/type.hpp>`

#### Boost.Math

* `<boost/math/distributions/beta.hpp>`

#### Boost.MP11

* `<boost/mp11/*.hpp>`

#### Boost.Serialization

* `<boost/serialization/*.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Variant2

* `<boost/variant2/variant.hpp>`

------
### Standard Facilities
