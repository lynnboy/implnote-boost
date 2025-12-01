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
```

accumulators/: collector, count, fraction, mean, ostrem, sum, weighted_mean, weighted_sum

accumulators, algorithm, histogram, indexed, literals, make_histogram, make_profile, multi_index, ostream, sample, serialization, unsafe_access, weight

------
### Implementation Details

```c++
struct detail::priority<n> : priority<n-1>{}; struct detail::priority<0>{};
struct detail::relaxed_equal { constexpr bool operator()<T,U>(const T& t, const U& u) const noexcept{...} };
using detail::replace_type<T,From,To> = std::conditional_t<is_same_v<T,From>, To, T>;
using detail::replace_default<T,Default> = replace_type<T,use_default,Default>;
using detail::replace_cstring<T> = replace_type<T,const char*,std::string>;

// detect
using detail::void_t<...> = void;
struct detail::detect_base{ static T&& val<T>(); static T& ref<T>(); static T const& cref<T>(); };
using detail::has_method_reset<T> = requires(T& t) {t.reset(0);};
using detail::has_method_push_back<T> = requires(T& t) {&T::push_back;};
using detail::is_indexable<T> = requires(T& t) {t[0];};
using detail::is_transform<T,U=T> = requires(T& t, U& u) {t.inverse(t.forward(u));};
using detail::is_indexable_container<T> = requires(T& t) {t[0]; t.size(), std::begin(t); std::end(t);};
using detail::is_vector_like<T> = requires(T& t) {t[0]; t.size(); t.resize(0); std::begin(t); std::end(t);};
using detail::is_array_like<T> = requires(T& t) {t[0]; t.size(); std::tuple_size<T>::value; std::begin(t); std::end(t);};
using detail::is_map_like<T> = requires(T& t) {typename T::key_type; typename T::mapped_type; std::begin(t); std::end(t);};
using detail::is_axis<T> = requires(T& t) {t.size(); &T::index;};
using detail::is_iterable<T> = requires(T& t) {std::begin(t); std::end(t);};
using detail::is_iterator<T> = requires(T& t) {typename std::iterator_traits<T>::iterator_category;};
using detail::is_streamable<T> = requires(T& t, std::ostream& os) {os<<t;};
using detail::is_allocator<T> = requires(T& t) {T::allocate; T::deallocate;};
using detail::has_operator_preincrement<T> = requires(T& t) {++t;};
using detail::has_operator_equal<T,U=T> = requires(const T& t, U& u) {t == u;};
using detail::has_operator_radd<T,U=T> = requires(T& t, U& u) {t += u;};
using detail::has_operator_rsub<T,U=T> = requires(T& t, U& u) {t -= u;};
using detail::has_operator_rmul<T,U=T> = requires(T& t, U& u) {t *= u;};
using detail::has_operator_rdiv<T,U=T> = requires(T& t, U& u) {t /= u;};
using detail::has_method_eq<T,U=T> = requires(const T& t, U& u) {t.operator==(u);};
using detail::has_threading_support<T> = requires(T& t) {T::has_threading_support;};
using detail::is_explicitly_convertible<T,U=T> = requires(T& t, U& u) {static_cast<U>(t);};
using detail::is_complete<T> = requires(T& t) {sizeof(T);};
using detail::is_storage<T> = mp_and<is_indexable_container<T>, has_method_reset<T>, has_threading_support<T>>;
using detail::is_adaptible<T> = mp_and<mp_not<is_storage<T>>, mp_or<is_vector_like<T>, is_array_like<T>, is_map_like<T>>>;
using detail::is_tuple<T> = ...; using detail::is_variant<T> = ...;
using detail::is_axis_variant<T> = ...;
using detail::is_any_axis<T> = mp_or<is_axis<T>, is_axis_variant<T>>;
using detail::is_sequence_of_axis<T> = mp_and<is_iterable<T>, is_axis<mp_first<T>>>;
using detail::is_sequence_of_axis_variant = mp_and<is_iterable, is_axis_variant<mp_first<T>>>;
using detail::is_sequence_of_any_axis = mp_and<is_iterable, is_any_axis<mp_first<T>>>;
struct detail::requires_storage<T>{}; requires_storage_or_adaptible; requires_iterator; requires_iterable; requires_axis; requires_any_axis;
    requires_sequence_of_axis; requires_sequence_of_axis_variant; requires_sequence_of_any_axis; requires_axes; requires_transform, requires_allocator

// iterator_adaptor
struct detail::operator_arrow_dispatch_t<Ref> {
    struct pointer{ Ref m_ref; Ref* operator->() noexcept { return std::addressof(m_ref); } } using result_type = pointer;
    static result_type apply(Ref const& x) noexcept { return {x}; } };
struct detail::operator_arrow_dispatch_t<T&> { using result_type = T*;
    static result_type apply(Ref const& x) noexcept { return std::addressof(x); } };
struct detail::get_difference_type<T> = ...;
class detail::iterator_adaptor<Derived,Base,Ref=std::remove_pointer_t<Base>&,Value=std::decay_t<Ref>> {
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

constexpr decltype(auto) detail::static_if_c<b,...Ts>(Ts&&...ts) noexcept;
constexpr decltype(auto) detail::static_if<Bool,...Ts>(Ts&&...ts) noexcept;

constexpr T* detail::ptr_cast<T,U>(U*);
T detail::try_cast<T,E,U>(U&& u) noexcept(...);

std::string detail::type_name<T>();

using detail::args_type<FuncPtr> = std::tuple<...>; // tuple of parameter type list
using detail::arg_type<T,size_t n=0> = std::tuple_element_t<n,args_type<T>>;

using detail::convert_integer<T,U> = std::conditional_t<is_integral_v<decay_t<T>>, U,T>;

struct detail::counting_streambuf<Ch,Tr=std::char_traits<Ch>> : std::basic_streambuf<Ch,Tr> {
    std::streamsize* p_count;
    ctor(std::streamsize& c) : p_count{&c}{}
    std::streamsize xsputn(const char_type*, std::streamsize n) override { *p_count += n; return n; }
    int_type overflow(int_type ch) override { ++*p_count; return ch; }
};
struct detail::count_guard<Ch,Tr> {
    using bos = std::basic_ostream<Ch,Tr>; using bsb = std::basic_streambuf<Ch,Tr>;
    counting_streambuf<Ch,Tr> csb; bos* p_os; bsb* p_rdbuf;
    ctor(bos& os, std::streamsize& s) : csb{s}, p_os{&os}, p_rdbuf{os.rdbuf(&csb)} {}
    ctor(self&& o); self& operator=(self&& o); // move
    ~dtor() { if (p_os) p_os->rdbuf(p_rdbuf); }
};
count_guard<Ch,Tr> detail::make_count_guard<Ch,Tr>(std::basic_ostream<Ch,Tr>& os, std::streamsize& s) { return {os, s}; }

struct detail::variant_proxy<Variant> {
    Variant& variant;
    void serialize<Archive>(Archive& ar, unsigned) {...}; // "which", "value"
};

using detail::has_array_optimization<T>;
struct detail::array_wrapper<T> { using pointer = T*;
    pointer ptr; size_t size;
    void serialize<Archive>(Archive ar, unsigned);
};
auto detail::make_array_wrapper<T>(T* t, size_t s) { return array_wrapper<T>{t, s}; }

struct detail::mirrored<T,U>{ friend bool operator<(const U& a, const T& b) noexcept{return b > a;} }; // all 6 comparasions, inversed impl
struct detail::mirrored<T,void> { friend bool operator< <U> (const U& a, const T& b) noexcept requires !is_same_v<T,U> { return b > a;} }; // all 6 comparasions, inversed impl
struct detail::mirrored<T,T> { friend bool operator>(const T& a, const T& b) noexcept { return b.operator<(a); } };
struct detail::equality<T,U> { friend bool operator!=(const T& a, const U& b) noexcept { return !a.operator==(b); } };
struct detail::equality<T,void> { friend bool operator!=(const T& a, const U& b) noexcept requires !is_same_v<T,U> { return !(a==b); } };
using detail::totally_ordered<T,...Ts> = ...;
using detail::partially_ordered<T,...Ts> = ...;

auto detail::make_unsigned<T>(const T& t) noexcept;
using detail::number_category<T> = ...;
struct detail::safe_equal { bool operator()<T,U>(const T& t, const U& u) const noexcept; };
struct detail::safe_less { bool operator()<T,U>(const T& t, const U& u) const noexcept; };
struct detail::safe_greater { bool operator()<T,U>(const T& t, const U& u) const noexcept; };

using detail::is_unsigned_integral<T> = mp_bool<is_integral_v<T>, is_unsigned_v<T>>;
bool detail::safe_increment<T>(T& t);
bool detail::safe_radd<T,U>(T& t, const U& u);
struct detail::large_int<Alloc> : totally_ordered<self,self>, partially_ordered<self,void> {
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
```

algorithm/: empty, project, reduce, sum
detail/: accumulator_traits, argument_traits, atomic_number, axes, chunk_vector, common_type, debug, erf_inv, fill_n, fill, ignore_deprecation_warning_beginend, index_translator, large_int, limits, linearize, make_default, mutex_base, nonmember_container_access, normal, optional_index, reduce_command, relaxed_tuple_size, square, static_vector, term_info, tuple_slice
utility/: binomial_proportion_interval, clopper_pearson_interval, jeffreys_interval, waid_interval, wilson_interval

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
