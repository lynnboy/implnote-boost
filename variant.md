# Boost.Variant

* lib: `boost/libs/variant`
* repo: `boostorg/variant`
* commit: `9738f57`, 2025-09-13

------
#### `variant`

* Header `<boost/variant.hpp>`, `<boost/variant/variant_fwd.hpp>`

```c++
struct detail::void_;
struct detail::convert_void<T> { using type = T; };
struct detail::convert_void<void_> { using type = mpl::na; };

class detail::backup_holder<T> { T* backup_;
public: ~dtor() noexcept { delete backup_; } explicit ctor(T* backup) noexcept : backup_{backup} {} // no copy-ctor
    self& operator=(const self& rhs) { *backup_ = rhs.get(); return *this; } // deep copy
    self& operator=(const T& rhs) { *backup_ = rhs; return *this; }
    void swap(self& rhs) noexcept; friend void swap(self&, self&) noexcept; // swap backup_
    <const>T& get() <const> noexcept { return *backup_; }
};
T detail::forced_return <T> () { /* ensure T can be returned */ }
void detail::move_swap<T>(T& lhs, T& rhs) { using std::swap; swap(lhs,rhs); }

struct detail::make_initializer_node { // MPL-based recursion
    struct apply<BaseIndexPair,It> { using base = BaseIndexPair::first; using index = BaseIndexPair::second;
        struct initializer_node : public base {
            using rec_T = mpl::deref<It>::type; using T = unwrap_recursive<rec_T>::type;
            using base::initialize;
            static int initialize(void* dest, if_<is_reference_v<T>,T,const T&>::type);
            static int initialize(void* dest, T&&) requires !is_reference_v<T>;
        };
        using type = mpl::pair<initializer_node, mpl::next<index>::type>;
    };
};
struct detail::initializer_root { static void initialize(); };
// MPL fold `make_initializer_node` over a type list
#define INITIALIZER_T(mpl_seq) mpl::iter_fold<mpl_seq, mpl::pair<initializer_root,mpl::int_<0>>, mpl::protect<make_initializer_node>>::type::first

struct detail::make_variant_list<...T> { using type = mpl::list<T...>::type; };

struct detail::over_sequence<Types> { using type = Types; };
struct detail::is_over_sequence<T> : false_ {};
struct detail::is_over_sequence<over_sequence<Types>> : true_ {};

<const>T& detail::cast_storage<T>(<const>void* st) { return *(<const>T*)(st); }

struct detail::apply_visitor_unrolled {}; // tag
struct detail::visitation_impl_step<It,End> { using type = mpl::deref<It>::type; using next = visitation_impl_step<mpl::next<It>::type, End>; };
struct detail::visitation_impl_step<End,End> { using type = apply_visitor_unrolled; using next = self; }; // rec end
Visitor::result_type detail::visitation_impl_invoke <Visitor,VoidPtrCV,T,NoBackup> (int internal_which, Visitor& visitor, VoidPtrCV storage, T*, NoBackup, int) {
    if constexpr (mpl::or_<NoBackup,is_nothrow_move_constructible<T>,has_nothrow_copy<T>>::value || internal_which)
        return visitor.internal_visit(cast_storage<T>(storage), 1L); // call `long` version
    else return visitor.internal_visit(cast_storage<backup_holder<T>>(storage), 1L); // call `long` version
}
Visitor::result_type detail::visitation_impl_invoke <Visitor,VoidPtrCV,NoBackup> (int, Visitor&, VoidPtrCV, apply_visitor_unrolled*, NoBackup, long) {return forced_return<Visitor::result_type>();} // unreachable

#define BOOST_VARIANT_VISITATION_UNROLLING_LIMIT 20
Visitor::result_type detail:;visitation_impl <Which,Step0,Visitor,VoidPtrCV,NoBackup>
    (int internal_which, int logical_which, Visitor& visitor, VoidPtrCV storage, false_, NoBackupFlag nb, Which*=nullptr,Step0*=nullptr)
{
    using T0 = step0::type; using step1 = step0::next; ...// upto UNROLLING_LIMIT-1 (T19, step20)
    switch (logical_which) { // expand for T0 ... T19
        case Which::value + 0: return visitation_impl_invoke(internal_which, visitor, storage, (T0*)0, nb, 1L); // upto 20
    }
    using next_which = mpl::int_<Which::value + UNROLLING_LIMIT>; using next_step = step20;
    // try next batch
    return visitation_impl(internal_which, logical_which, visitor, storage, is_same<next_type,apply_visitor_unrolled>{}, nb, (next_which*)0, (next_step*)0);
}

struct detail::is_static_visitor_tag {};
using detail::is_static_visitor<T> = is_base_of<is_static_visitor_tag,T>;
struct static_visitor<R=void> : public is_static_visitor_tag { using result_type = R; }; // base class usage only

std::tuple<std::tuple_element<1+i,Tuple>::type...> detail::tuple_tail_impl <...i,Tuple> (const Tuple& tup, index_sequence<i...>) { return std::make_tuple(std::get<1+i>(tup)...); }
std::tuple<Tail...> detail::tuple_tail <Head,...Tail> (const std::tuple<Head,Tail...>& tup) { return tuple_tail_impl(tup, make_index_sequence<sizeof...(Tail)>()); }
struct detail::MoveableWrapper<T,doMove> { T& v; };
MoveableWrapper<Tp,doMove> detail::wrap <Tp,doMove=!is_lvalue_reference_v<Tp>> (Tp& t) { return MoveableWrapper{t}; }
T  detail::unwrap <Tp,true>  (MoveableWrapper<Tp, true>& w) { return std::move(w.v); }
T& detail::unwrap <Tp,false> (MoveableWrapper<Tp,false>& w) { return w.v; }

struct detail::has_result_type<T> { static constexpr bool value = requires{ typename T::result_type; }; };
{Visitor::result_type|decltype(auto)} apply_visitor<Visitor,Visitable> (Visitor&& visitor, Visitable&& visitable) { return std::forward<Visitable>(visitable).apply_visitor(visitor); }
Visitor::result_type apply_visitor<Visitor,V1,V2> (<const>Visitor& visitor, V1&& v1, V2&& v2)
{ return apply_visitor([&,auto&&v2=std::forward<V2>(v2)]{return apply_visitor(visitor,std::forward<V2>(v2));}, std::forward<V1>(v1)); }

class detail::one_by_one_visitor_and_vaule_referrer<Visitor,Visitibles,...Values> {
    Visitor& visitor_; std::tuple<Values...> values_; Visitables visitables_;
public: ctor(Visitor& visitor, Visitables visitables, std::tuple<Values...> values) noexcept; // init
    decltype(auto) do_call <Tuple,...i> (Tuple t, index_sequence<i...>) const { return visitor_(unwrap(std::get<i>(t))...); }
    decltype(auto) operator() <Value> ()(Value&& value) const { // recursion
        if constexpr (std::is_same_v<Visitables,std::tuple<>>)
            return do_call(std::tuple_cat(values_, std::make_tuple(wrap(value))), make_index_sequence<sizeof...(Values) + 1>());
        else return apply_visitor(self{visitor_,tuple_tail(visitables_),std::tuple_cat(values_, std::make_tuple(wrap(value)))}, unwrap(std::get<0>(visitables_)));
    }
};
{Visitor::result_type|decltype(auto)} apply_visitor <Visitor,T1,T2,T3,...TN> (<const>Visitor& visitor, T1&& v1, T2&& v2, T3&& v3, TN&&...vn)
{ return apply_visitor(one_by_one_visitor_and_vaule_referrer{visitor, std::make_tuple(wrap(v2),wrap(v3),wrap(vn)...), std::tuple<>{}}, std::forward<T1>(v1)); }

auto apply_visitor <Visitor> (Visitor& visitor) { return [&](auto...v) { return apply_visitor(visitor, v...); }; }

size_t hash_value <...T> (variant<T...>const& val)
{ size_t seed = apply_visitor([](auto const& v){return boost::hash<decltype(v)>{}(v);},val);
    hash_combine(seed, val.which()); return seed; }
struct std::hash<variant<T...>> { size_t operator()(const variant<T...>& val) const { return hash_value(val); } };

struct detail::max_value<Seq,F> { using type = mpl::deref<mpl::max_element<mpl::transform1<Seq,F>::type>::type>::type; };
struct detail::add_alignment { using apply<State,Item> = mpl::size_t<static_lcm<State::value,alignment_of_v<Item>>::value>; };
struct detail::no_failback_type;
struct detail::find_fallback_type_pred { using apply<It> = mpl::not_<has_nothrow_constructor<deref<It>::type>>; }; // search nothrow constructible
struct detail::find_fallback_type<Types> {
    using res1 = mpl::iter_fold_if<Types,int_<0>,protect<next<>>,protect<find_fallback_type_pred>>::type; // enumerate result, find nothrow constructible
    using res2 = mpl::iter_fold_if<iterator_range<res1::second,end<Types>>, res1::first, protect<next<>>, protect<not_same_as<blank>>>::type; // find non-blank
    using type = mpl::eval_if<is_same<res2::second,end<Types>>, if_<is_same<res1::second,end<Types>>, pair<no_fallback_type,no_fallback_type>, res1>, identity<res2>>::type
};

struct detail::is_variant_move_noexcept_constructible<Types> { using type = is_same<mpl::find_if<Types,not_<is_nothrow_move_constructible<_1>>>,end<Types>>; };
struct detail::is_variant_move_noexcept_assignable<Types> { using type = is_same<mpl::find_if<Types,not_<is_nothrow_move_assignable<_1>>>,end<Types>>; };
struct detail::is_constructible_ext<T1,T2> : mpl::or_<is_constructible<T1,T2>,is_constructible<T1,add_lvalue_reference_t<T2>>> {};
struct detail::is_variant_constructible_from<T,Types> : mpl::not_<is_same<find_if<Types,is_constructible_ext<_1,T>>::type,end<Types>::type>> {};
struct detail::is_variant_constructible_from<variant<T...>,Types> : is_same<find_if<variant<T...>::recursive_enabled_types, not_<is_variant_constructible_from<_1,Types>>>::type, end<variant<T...>::recursive_enabled_types>::type> {};
struct detail::is_variant_constructible_from<variant<T...>[const]{&|&&}>,Types> : is_variant_constructible_from<variant<T...>,Types> {};

struct detail::make_storage<Types,NeverUseBackup> {
    using types = mpl::eval_if<NeverUseBackup, identity<Types>, push_front<Types,backup_holder<void*>>>::type;
    using max_size = max_value<types, sizeof_<_1>>::type; using max_alignment = mpl::fold<types,size_t<1>,add_alignment>::type;
    using type = aligned_storage<max_size::value, max_alignment::value>;
};
struct detail::destroyer : static_visitor<> { void internal_visit<T>(T& v,int) const noexcept { v.~T(); } };
class detail::known_get<T> : public static_visitor<T&> {
    T& operator()(T& v) const noexcept { return v; }
    T& operator() <U>(U&) const { return forced_return<T&>(); } // unreachable
};
class detail::copy_into : public static_visitor<> { void* storage_;
public: explicit ctor(void* storage) noexcept : storage_(storage){}
    void internal_visit <T>(backup_holder<T> <const>& v, long) const { new(storage_) T(v.get()); }
    void internal_visit <T> (const T& v, int) const { new(storage_) T(v); }
};
class detail::move_into : public static_visitor<> { void* storage_;
public: explicit ctor(void* storage) noexcept : storage_(storage){}
    void internal_visit <T>(backup_holder<T> & v, long) const { new(storage_) T(detail::move(v.get())); }
    void internal_visit <T> (const T& v, int) const { new(storage_) T(detail::move(v)); }
};
class detail::assign_storage : public static_visitor<> { void* storage_;
public: explicit ctor(void* storage) noexcept : storage_(storage){}
    void internal_visit <T>(backup_holder<T> <const>& v, long) const { v.get() = ((const backup_holder<T>*)storage_)->get(); }
    void internal_visit <T> (const T& v, int) const { v = *(const T*)storage_; }
};
class detail::move_storage : public static_visitor<> { void* storage_;
public: explicit ctor(void* storage) noexcept : storage_(storage){}
    void internal_visit <T>(backup_holder<T> <const>& v, long) const { v.get() = detail::move(((const backup_holder<T>*)storage_)->get()); }
    void internal_visit <T> (const T& v, int) const { v = detail::move(*(const T*)storage_); }
};
class detail::direct_assigner : public static_visitor<bool> { const T& rhs_;
public: explicit ctor(const T& rhs) noexcept : rhs_(rhs){}
    bool operator()(T& lhs) { lhs = rhs_; return true; }
    bool operator() <U> (U&) noexcept { return false; }
};
class detail::direct_mover : public static_visitor<bool> { T& rhs_;
public: explicit ctor(const T& rhs) noexcept : rhs_(rhs){}
    bool operator()(T& lhs) { lhs = detail::move(rhs_); return true; }
    bool operator() <U> (U&) noexcept { return false; }
};
class detail::backup_assigner<V> : public static_visitor<> {
    V& lhs_; int rhs_which; const void* rhs_content_; void (*copy_rhs_content_)(void*, const void*);
    static void construct_impl<RhsT> (void* addr, const void* obj) { new(addr)RhsT(*(const RhsT*)(obj)); }
public: ctor<RhsT> (V& lhs, int rhs_which, const RhsT& rhs_content)
    : lhs_(lhs), rhs_which_(rhs_which), rhs_content_(&rhs_content), copy_rhs_content_(&construct_impl<RhsT>) {};
    void internal_visit <LhsT> (LhsT& lhs_content, int) {
        if constexpr (is_nothrow_move_constructible_v<LhsT>) {
            LhsT backup_lhs_content{detail::move(lhs_content)}; lhs_content.~dtor();
            try { copy_rhs_content_(lhs_.storage_.address(), rhs_content_); }
            catch(...) { new(lhs_.storage_.address()) LhsT(detail::move(backup_lhs_content)); throw; }
            lhs_.indicate_which(rhs_which_);
        } else if constexpr (is_backup_holder<LhsT>) {
            LhsT backup_lhs_content{0}; backup_lhs_content.swap(lhs_content); lhs_content.~dtor();
            try { copy_rhs_content_(lhs_.storage_.address(), rhs_content_); }
            catch(...) { (new(lhs_.storage_.address()) LhsT(0))->swap(backup_lhs_content); throw; }
            lhs_.indicate_which(rhs_which_);
        } else {
            LhsT* backup_lhs_ptr = new LhsT(lhs_content); lhs_content.~dtor();
            try { copy_rhs_content_(lhs_.storage_.address(), rhs_content_); }
            catch(...) { new(lhs_.storage_.address()) backup_holder<LhsT>(backup_lhs_ptr); lhs_.indicate_backup_which(lhs_.which()); throw; }
            lhs_.indicate_which(rhs_which_);
            delete backup_lhs_ptr;
        }
    }
};
class detail::swap_with<V> : public static_visitor<> { V& toswap_;
public: explicit ctor(V& toswap) noexcept : toswap_(toswap) {}
    void operator() <T> (T& v) const { T& other = toswap_.apply_visitor(known_get<T>{}); move_swap(v, other); }
};
struct detail::reflect : public static_visitor<boost::typeindex::type_info&> {
    const type_info& operator() (const T&) const noexcept { return typeindex::type_id<T>().type_info(); }
};
class detail::comparer<V,Comp> : public static_visitor<bool> { const V& lhs_;
public: explicit ctor(const V& lhs) noexcept : lhs_(lhs) {}
    bool operator() <T> (T& rhs_content) const { const T& lhs_content = lhs_.apply_visitor(known_get<T>{}); return Comp(lhs_content, rhs_content); }
};

struct detail::equal_comp { bool operator() <T> (const T& l, const T& r) const { return l == r; } };
struct detail::less_comp { bool operator() <T> (const T& l, const T& r) const { return l < r; } };
class detail::invoke_visitor<V,doMove> { V& visitor_;
public: using result_type = V::result_type;
    explicit ctor(V& visitor) noexcept: visitor_(visitor) {}

    result_type internal_visit <T> (T&& v, int) { if constexpr(doMove) return visitor_(std::move(v)); else return visitor_(v); }
    result_type internal_visit <T> (recursive_wrapper<T> <const>& v, long) { return internal_visit(operand.get(),1L); }
    result_type internal_visit <T> (reference_content<T> <const>& v, long) { return internal_visit(operand.get(),1L); }
    result_type internal_visit <T> (backup_holder<T> <const>& v, long) { return internal_visit(operand.get(),1L); }
};

class variant<T0_,...T> {
    using unwrapped_T0_ = mpl::eval_if<is_recursive_flag<T0_>, T0_, identity<T0_>>::type;
    using specified_types = mpl::eval_if<is_over_sequence<unwrapped_T0_>, unwrapped_T0_, make_variant_list<unwrapped_T0_, T...>>::type;
public:
    using recursive_enabled_types = mpl::eval_if<is_recursive_flag<T0_>, transform<specified_types,protect<quoted_enable_recursive<self>>>, identity<specified_types>>::type;
    using types = mpl::transform<recursive_enabled_types, unwrap_recursive<_1>>::type;
private: using internal_types = mpl::transform<recursive_enabled_types, protect<make_reference_content<>>>::type;
    using internal_T0 = mpl::front<internal_types>::type;
    using fb_res = find_fallback_type<internal_types>::type;
    using fallback_type_index_ = fb_res::first; using fallback_type_ = fb_res::second;
    using has_fallback_type_ = not_<is_same<fallback_type_,no_fallback_type>>; using never_uses_backup_flag = has_fallback_type;
    using storage_t = make_storage<internal_types, never_uses_backup_flag>;
    using which_t = int;

    which_t which_; storage_t storage_;

    void indicate_which(int arg) noexcept { which_ = arg; } void indicate_backup_which(int arg) noexcept { which_ = -(arg+1); }
    bool using_backup() const noexcept { return which_ < 0; }
    struct initializer : INITIALIZER_T(recursive_enabled_types, recursive_enabled_T) {};
    void destroy_content() noexcept { internal_apply_visitor(destroyer{}); }

public: int which() const noexcept { if (using_backup()) return -(which_+1); return which_; }
    ~dtor() noexcept { destroy_content(); }
    ctor() noexcept(has_nothrow_constructor<internal_T0>::value) { new(storage_.address()) internal_T0(); indicate_which(0); }
    ctor(self const& o) { o.internal_apply_visitor(copy_into{storage_.address()}); indicate_which(o.which()); }
    ctor(self && o) noexcept(is_variant_move_noexcept_constructible::value)
    { o.internal_apply_visitor(move_into{storage_.address()}); indicate_which(o.which()); }

private: class convert_copy_into : public static_visitor<int> { void* storage_;
    public: explicit ctor(void* storage) : storage_(storage) {}
        int internal_visit<T> (T& v, int) const { return initializer::initialize(storage_, v); } // base case
        // and overloads for `reference_content`, `backup_holder`, and `recursive_wrapper` and `const` version of them, call `get()`
    };
    class convert_move_into : public static_visitor<int> { void* storage_;
    public: explicit ctor(void* storage) : storage_(storage) {}
        int internal_visit<T> (T& v, int) const { return initializer::initialize(storage_, detail::move(v)); } // base case
        // and overloads for `reference_content`, `backup_holder`, and `recursive_wrapper` and `const` version of them, call `get()`
    };
    void convert_construct<T>(T{&|&&} v, int, false_) { indicate_which(initializer:initialize(storage_.address(), <detail::move>(v))); }
    void convert_construct<Vis>(Vis{&|&&} v, int, true_) { indicate_which(v.internal_apply_visitor({convert_copy_into|convert_move_into}{})); }
    void convert_construct_variant<V>(V{&|&&} v) {
        using is_foreign_variant = is_same< mpl::find_if<types, is_same<add_const<_1>, const V>>::type, end<types>:type >::type;
        convert_construct(<detail::move>(v), 1, is_foreign_variant{});
    }
    void convert_construct <...U> (variant<U...> {<const>&|&&} v, long)
        requires is_same_v<variant<U...>, self> || is_variant_constructible_from<variant<U...>&, internal_types>
        { convert_construct_variant(<detail::move>(v)); }
public: ctor <T> (T {<const>&|&&} v) requires (!is_same_v<T,self> && is_variant_constructible_from<const T&, internal_types>) || is_same_v<T,recursive_variant_>
    { convert_construct(<detail::move>(v), 1L); }

private: class assigner : public static_visitor<> { protected: variant& lhs_; const int rhs_which_;
        void construct_fallback() {
            new(lhs_.storage_.address()) fallback_type_;
            lhs_.indicate_which(fallback_type_index_::value);
        }
        public: ctor(variant& lhs, int rhs_which) noexcept : lhs_(lhs), rhs_which_(rhs_which) {}
        void internal_visit <RhsT> (const RhsT& rhs_content, int) const {
            if constexpr (has_nothrow_copy<RhsT>::value) {
                lhs_.destroy_content();
                new(lhs_.storage_.address()) RhsT(rhs_content);
                lhs_.indicate_which(rhs_which_);
            } else if constexpr (is_nothrow_move_constructible<RhsT>::value) {
                RhsT temp{rhs_content};
                lhs_.destroy_content();
                new(lhs_.storage_.address()) RhsT(detail::move(temp));
                lhs_.indicate_which(rhs_which_);
            } else if constexpr (has_fallback_type_::value) {
                lhs_.destroy_content();
                try { new(lhs_.storage_.address()) RhsT(rhs_content); }
                catch (...) { construct_fallback(); throw; }
                lhs_.indicate_which(rhs_which_);
            } else {
                lhs_.internal_apply_visitor(backup_assigner<variant>{lhs_, rhs_which_, rhs_content});
            }
        }
    };
    class move_assigner : public assigner {
        public: using base::ctor;
        void internal_visit <RhsT> (const RhsT& rhs_content, int) const {
            if constexpr (is_nothrow_move_constructible<RhsT>::value) {
                lhs_.destroy_content();
                new(lhs_.storage_.address()) RhsT(detail::move(temp));
                lhs_.indicate_which(rhs_which_);
            } else if constexpr (has_nothrow_copy<RhsT>::value) {
                lhs_.destroy_content();
                new(lhs_.storage_.address()) RhsT(rhs_content);
                lhs_.indicate_which(rhs_which_);
            } else if constexpr (has_fallback_type_::value) {
                lhs_.destroy_content();
                try { new(lhs_.storage_.address()) RhsT(detail::move(rhs_content)); }
                catch (...) { construct_fallback(); throw; }
                lhs_.indicate_which(rhs_which_);
            } else {
                lhs_.internal_apply_visitor(backup_assigner<variant>{lhs_, rhs_which_, rhs_content});
            }
        }
    };
    void variant_assign(self {const&|&&} rhs) {
        if (which_ == rhs.which_) internal_apply_visitor({assign_storage|move_storage}{rhs.storage_.address()});
        else internal_apply_visitor({assigner|move_assigner}{*this, rhs.which()});
    }
    void assign <T> (const T& rhs)
    { if (apply_visitor(direct_assigner<T>{rhs})==false) { variant tmp{rhs}; variant_assign(detail::move(temp)); } }
    void move_assign <T> (T&& rhs)
    { if (apply_visitor(direct_mover<T>{rhs})==false) { variant tmp{detail::move(rhs)}; variant_assign(detail::move(temp)); } }
public: self& operator= <T> (T&& rhs) requires is_rvalue_reference_v<T&&> && !is_const_v<T> && is_variant_constructible_from<T&&, internal_types>::value
    { move_assign(detail::move(rhs)); return *this; }
    self& operator= <T> (const T& rhs) requires is_same_v<T,self> || is_variant_constructible_from<const T&, internal_types>::value
    { assign(rhs); return *this; }
    self& operator=(const self& rhs) { variant_assign(rhs); return *this; }
    self& operator=(self&& rhs) noexcept(is_variant_move_noexcept_constructible::value && is_variant_move_noexcept_assignable::value)
    { variant_assign(detail::move(rhs)); return *this; }
    void swap(self& rhs) {
        if (which() == rhs.which()) apply_visitor(swap_with<self>{rhs});
        else { self tmp{detail::move(rhs)}; rhs = detail::move(*this); *this = detail::move(tmp); }
    }

    bool empty() const noexcept { return false; }
    const typeindex::type_info& type() const { return apply_visitor(reflect{}); }
    void operator==<U>(const U&) const = delete; // also delete <, !=, >, <=, >=
    bool operator==(const self& rhs) const {
        if (which() != rhs.which()) return false;
        return rhs.apply_visitor(comparer<self, equal_comp>{*this});
    } // also !=
    bool operator< (const self& rhs) const {
        if (which() != rhs.which()) return which() < rhs.which();
        return rhs.apply_visitor(comparer<self, less_comp>{*this});
    } // also >, <=, >=

private: Visitor::result_type internal_apply_visitor <Visitor> (Visitor& visitor)<const> {
        using first_which = mpl::int_<0>; using first_step = visitation_impl_step<begin<internal_types>::type, end<internal_types>::type>;
        return visitation_impl(which_, which(), visitor, storage_.address(), false_{}, never_uses_backup_flag{}, (first_which*)0, (first_step**)0);
    }
public: Visitor::result_type apply_visitor<Visitor> (Visitor& visitor) <const>{&|&&} {
        return internal_apply_visitor(invoke_visitor<Visitor,{false|true}>{});
    }
};

struct make_variant_over<Types> {
    using copied_sequence_t = mpl::insert_range<list<>, end<list<>>::type, Types>::type;
    using type = variant<over_sequence<copied_sequence_t>>;
}

void swap<T...> (variant<...T>& lhs, variant<...T>& rhs) { lhs.swap(rhs); }

class detail::printer<OS> : public static_visitor<> { OS& out_;
public: explicit ctor(OS& out) : out_(out) {}
    void operator() <T> (const T& v) const { out_ << v; }
};
std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr,...U> (std::basic_ostream<Ch,Tr>& out, const variant<U...>& rhs)
{ rhs.apply_visitor(printer{out}); return out; }
```

------
#### `make_recursive_variant`

```c++
struct detail::substitute<T,Dest,Src,Arity=mpl::int_<mpl::template_arity<T>::value>> { using type = T; };
struct detail::substitute<[cv]Src,Dest,Src,int_<-1>> { using type = <cv> Dest; };
struct detail::substitute<T*[cv],Dest,Src,int_<-1>> { using type = substitute<T,Dest,Src>::type* <cv>; };
struct detail::substitute<T&,Dest,Src,int_<-1>> { using type = substitute<T,Dest,Src>::type&; };
struct detail::substitute<F<Ts...>,Dest,Src,Arity> { using type = F<substitute<Ts,Dest,Src>::type...>; };
struct detail::substitute<R(*)(A...),Dest,Src,int_<-1>>
{ using r = substitute<R,Dest,Src>::type; using type = r (*)(substitute<A,Dest,Src>::type...); };

class recursive_wrapper<T> {
    T* p_;
    void assign(const T&) { get() = rhs; }
public: using type = T;
    ~dtor() { checked_delete(p_); } ctor():p_(new T){}
    ctor(self{const &|&&} v):p_(new T{<detail::move>(v.get())}) {}
    ctor(T{const&|&&} v):p_(new T{<detail::move>(v)}) {}
    self& operator=(self{const&|&&} rhs) { assign(<detail::move>(rhs.get())); return *this; }
    self& operator=(T{const&|&&} rhs) { assign(<detail::move>(rhs)); return *this; }
    void swap(self& v) noexcept { T* tmp = v.p_; v.p_ = p_; p_ = tmp; }
    <const>T& get() <const> { return *get_pointer(); }
    <const>T* get_pointer() <const> { return p_; }
};
void swap<T> (recursive_wrapper<T>& lhs, recursive_wrapper<T>& rhs) noexcept { lhs.swap(rhs); }

struct is_constructible<recursive_wrapper<T>,U> : false_type {}; // type_traits spec
struct is_constructible<recursive_wrapper<T>,[const]T[&]> : true_type {}; // type_traits spec
struct is_constructible<recursive_wrapper<T>,[const]recursive_wrapper<T>[&]> : true_type {}; // type_traits spec
struct is_nothrow_move_constructible<recursive_wrapper<T>> : false_type {}; // wrapper makes dynamic alloc
struct is_recursive_wrapper<T> : false_ {};
struct is_recursive_wrapper<recursive_wrapper<T>> : true_ {};
struct unwrap_recursive<T> { using type = T; };
struct unwrap_recursive<recursive_wrapper<T>> { using type = T; };

struct detail::recursive_flag<T> { using type = T; };
struct detail::is_recursive_flag<T> : false_ {};
struct detail::is_recursive_flag<recursive_flag<T>> : true_ {};
struct detail::enable_recursive<T,RecV,NoWrapper=false_> {
    using t_ = substitute<T,RecV,recursive_variant_>::type;
    using type = mpl::if_<or_<NoWrapper, is_same<t_,T>, is_reference<t_>, is_pointer<t_>>, t_, recursive_wrapper<t_>>::type;
};
struct detail::quoted_enable_recursive<RecV,NoWrapper=false_> { struct apply<T> : enable_recursive<T,RecV,NoWrapper>{}; };

struct detail::substitute<variant<recursive_flag<T0>,T...>, RecV, recursive_variant_, Arity>
{ using type = variant<recursive_flag<T0>,T...>; };
struct detail::substitute<variant<over_sequence<T0>,T...>, RecV, recursive_variant_, Arity>
{   using initial_types = T0; using types = mpl::transform<initial_types, protect<quoted_enable_recursive<RecV,true_>>>::type;
    using type = mpl::if_<equal<initial_types,types,is_same<_1,_2>>, variant<over_sequence<T0>,T...>, variant<over_sequence<types>>>::type; };
struct detail::substitute<variant<T0,T...>, RecV, recursive_variant_, Arity>
{ using type = variant<enable_recursive<T0,RecV,true_>::type, enable_recursive<T,RecV,true_>::type...>; };

struct recursive_variant_{};
struct make_recursive_variant<T0,...T> { using type = variant<recursive_flag<T0>,T...>; };
struct make_recursive_variant_over<Types> { using type = make_recursive_variant<over_sequence<Types>>::type; };
```

------
#### `get`, `visitor_ptr`

```c++
struct detail::variant_element_functor<Elem,T> : mpl::or_<is_same<Elem,T>, is_same<Elem,recursive_wrapper<T>, is_same<Elem,T&>>> {};
struct detail::element_iterator_impl<Types,T> : mpl::find_if<Types, or_<variant_element_functor<_1,T>, variant_element_functor<_1, remove_cv_t<T>>>> {};
struct detail::element_iterator<Variant,T> : element_iterator_impl<Variant::types, remove_reference_t<T>> {};
struct detail::holds_element<Variant,T> : mpl::not_<is_same< element_iterator<Variant,T>::type, end<Variant::Types>::type >> {};

class bad_get : public std::exception { /* what() = "failed value get using boost::get" */ };

struct detail::get_visitor<T> { using pointer = T*; using reference = T&; using result_type = pointer;
    pointer operator()(reference v) const noexcept { return addressof(v); }
    pointer operator() <U> (const U&) const noexcept { return nullptr; }
};

U<const>* relaxed_get<U,...T> (variant<T...> <const>* v) noexcept
{ if (!v) return nullptr; return v->apply_visitor(get_visitor<<const>U>{}); }
U<const>& relaxed_get<U,...T> (variant<T...> <const>& v)
{ U<const>* res = relaxed_get<<const>U>(addressof(v)); if (!res) throw_exception(bad_get{}); return *res; }
U&& relaxed_get<U,...T> (variant<T...> && v)
{ U* res = relaxed_get<U>(addressof(v)); if (!res) throw_exception(bad_get{}); return std::move(*res); }

U<const>* strict_get<U,...T> (variant<T...> <const>* v) noexcept { /*assert*/ return relaxed_get<U>(v); }
U<const>& strict_get<U,...T> (variant<T...> <const>& v) { /*assert*/ return relaxed_get<U>(v); }
U&& strict_get<U,...T> (variant<T...> && v) { /*assert*/ return relaxed_get<U>(detail::move(v)); }

U<const>* get<U,...T> (variant<T...> <const>* v) noexcept { /*assert*/ return strict_get<U>(v); } // or relaxed_get if USE_RELAxED_GET_BY_DEFAULT
U<const>& get<U,...T> (variant<T...> <const>& v) { /*assert*/ return strict_get<U>(v); } // or relaxed_get if USE_RELAxED_GET_BY_DEFAULT
U&& get<U,...T> (variant<T...> && v) { /*assert*/ return strict_get<U>(detail::move(v)); } // or relaxed_get if USE_RELAxED_GET_BY_DEFAULT

class bad_polymorphic_get : public bad_get { /* what() = "failed value get using boost::polymorphic_get" */ };

struct detail::element_polymorphic_iterator_impl<Types,T> : mpl::find_if<Types, or_<variant_element_functor<_1,T>, variant_element_functor<_1, remove_cv_t<T>, is_base_of<T,_1>>>> {};
struct detail::holds_element_polymorphic<Variant,T> : mpl::not_<is_same< element_polymorphic_iterator_impl<Variant::types,remove_reference_t<T>>::type, end<Variant::Types>::type >> {};

struct detail::get_polymorphic_visitor<Base> { using pointer = Base*; using reference = Base&; using result_type = pointer;
    pointer operator() <U> (U& v) const noexcept { using base_t = remove_reference_t<Base>;
        if constexpr ((is_base_of_v<base_t,U> && (is_const_v<base_t> || !is_const_v<U>)) || is_same_v<base_t,U> || is_same_v<remove_cv_t<base_t>,U>)
            return addressof(v);
        else return nullptr;
    }
};

U<const>* polymorphic_relaxed_get<U,...T> (variant<T...> <const>* v) noexcept
{ if (!v) return nullptr; return v->apply_visitor(get_polymorphic_visitor<<const>U>{}); }
U<const>& polymorphic_relaxed_get<U,...T> (variant<T...> <const>& v)
{ U<const>* res = polymorphic_relaxed_get<<const>U>(addressof(v)); if (!res) throw_exception(bad_get{}); return *res; }
U&& polymorphic_relaxed_get<U,...T> (variant<T...> && v)
{ U* res = polymorphic_relaxed_get<U>(addressof(v)); if (!res) throw_exception(bad_get{}); return std::move(*res); }

U<const>* polymorphic_strict_get<U,...T> (variant<T...> <const>* v) noexcept { /*assert*/ return polymorphic_relaxed_get<U>(v); }
U<const>& polymorphic_strict_get<U,...T> (variant<T...> <const>& v) { /*assert*/ return polymorphic_relaxed_get<U>(v); }
U&& polymorphic_strict_get<U,...T> (variant<T...> && v) { /*assert*/ return polymorphic_relaxed_get<U>(detail::move(v)); }

U<const>* polymorphic_get<U,...T> (variant<T...> <const>* v) noexcept { /*assert*/ return polymorphic_strict_get<U>(v); } // or polymorphic_relaxed_get if USE_RELAxED_GET_BY_DEFAULT
U<const>& polymorphic_get<U,...T> (variant<T...> <const>& v) { /*assert*/ return polymorphic_strict_get<U>(v); } // or polymorphic_relaxed_get if USE_RELAxED_GET_BY_DEFAULT
U&& polymorphic_get<U,...T> (variant<T...> && v) { /*assert*/ return polymorphic_strict_get<U>(detail::move(v)); } // or polymorphic_relaxed_get if USE_RELAxED_GET_BY_DEFAULT

struct bad_visit : public std::exception { /* what() = "failed visitation using boost::apply_visitor" */ };

class visitor_ptr_t <T,R> : public static_visitor<R> {
    using visitor_t = R(*)(T); visitor_t visitor_;
    using argument_fwd_type = mpl::eval_if<is_reference<T>, identity<T>, const T&>::type;
public: using result_type = R;
    explicit ctor(visitor_t visitor) noexcept : visitor_(visitor) {}
    result_type operator() <U> (const U&) const { throw_exception(bad_visit{}); }
    result_type operator() (argument_fwd_type v) const { return visitor_(v); }
};
visitor_ptr_t<T,R> visitor_ptr<R,T> (R (*v)(T)) { return {v}; } // factory
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`
* `<boost/assert/source_location.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`

#### Boost.ContainerHash

* `<boost/functional/hash_fwd.hpp>`

#### Boost.Core

* `<boost/core/addressof.hpp>`
* `<boost/core/checked_delete.hpp>`
* `<boost/core/enable_if.hpp>`
* `<boost/core/no_exceptions_support.hpp>`

#### Boost.Detail

* `<boost/blank_fwd.hpp>`, `<boost/blank.hpp>`
* `<boost/detail/reference_content.hpp>`

#### Boost.Integer

* `<boost/integer/common_factor_ct.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeIndex

* `<boost/type_index.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

#### Boost.Utility

* `<boost/utility/call_traits.hpp>`
* `<boost/utility/declval.hpp>`

------
### Standard Facilities

Library: `<variant>` (C++11), `<bind>` (C++11), `mem_fn` (C++11)
