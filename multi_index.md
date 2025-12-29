# Boost.MultiIndex

* lib: `boost/libs/multi_index`
* repo: `boostorg/multi_index`
* commit: `e2b50ef`, 2025-09-19

------
#### 

```c++
```

------
#### Details

```c++
struct detail::any_container_view <CIt> {
    ctor<Cont>(const Cont& x) : px{&x}, pt{vtable_for<Cont>()}{}
    const void* container() const { return px; }
    CIt begin() const { return pt->begin(px); } CIt end() const { return pt->end(px); }
private: struct vtable { CIt (*begin)(const void*); CIt (*end)(const void*); };
    static CIt begin_for<Cont>(const void* px) { return ((const Cont*)px)->begin(); }
    static CIt end_for<Cont>(const void* px) { return ((const Cont*)px)->end(); }
    vtable* vtable_for<Cont>() { static vtable v={&begin_for<Cont>, &end_for<Cont>}; return &v; }
    const void* px; vtable* pt;
};

struct detail::archive_constructed<T> : noncopyable {
    ctor<Archive>(Archive& ar, unsigned ver)
    { core::load_construct_data_adl(ar,&get(),ver); try{ar>>get();}catch(...){(&get())->~T(); throw;} }
    ctor<Archive>(const char* name, Archive& ar, unsigned ver)
    { core::load_construct_data_adl(ar,&get(),ver); try{ar>>core::make_nvp(name,get());}catch(...){(&get())->~T(); throw;} }
    ~dtor() { (&get())->~T(); }
    T& get() { return *(T*)(&space); }
private: aligned_storage<sizeof(T),alingment_of<T>::value>::type space;
};

struct detail::auto_space<T,Alloc=std::allocator<T>> : noncopyable {
    using allocator=rebind_alloc_for<Alloc,T>::type; using alloc_traits = allocator_traits<allocator>;
    using pointer = alloc_traits::pointer; using size_type = alloc_traits::size_type;
    explicit ctor(const Alloc& al={}, size_type n=1); ~dtor();
    Alloc get_allocator() const { return al_; }
    pointer data() const { return data_; }
    void swap(self& x) { if constexpr (alloc_traits::propagate_on_container_swap::value) adl_swap(al_,x.al_); std::swap(n_,x.n_); std::swap(data_,x.data_); }
private: allocator al_; size_type n_; pointer data_;
};
void detail::swap<T,A>(auto_space<T,A>& x, auto_space<T,A>& y);

struct detail::bad_archive_exception : std::runtime_error;

struct detail::index_applier {
    struct apply<ISpecMeta,SuperMeta> { using index_specifier = ISpecMeta::type; using type = index_specifier::index_class<SuperMeta>::type; };
};
struct detail::nth_layer<n,Value,ISpecList,Alloc> {
    constexpr static int length = mpl::size<ISpecList>::value;
    using type = mpl::eval_if_c<n==length, mpl::identity<index_base<Value,ISpecList,Alloc>>,
                                           mpl::apply2<index_applier, at_c<ISpecList,n>, nth_layer<n+1,Value,ISpecList,Alloc>>>::type;
};
struct detail::multi_index_base_type<Value,ISpecList,Alloc> : nth_layer<0,Value,ISpecList,Alloc>{};

struct bidir_node_iterator<Node> : public bidirectional_iterator_helper<self, Node::value_type,Node::difference_type, const Node::value_type*, const Node::value_type&> {
    using node_type = Node; using node_base_type = Node::base_type;
    ctor(){}  explicit ctor(Node* node_): node{node_}{}
    const Node::value_type& operator*() const { return node->value(); }
    self& operator++() { Node::increment(node); return *this; }  self& operator--() { Node::decrement(node); return *this; }
    void serialize<Archive>(Archive& ar, unsigned ver) { core::split_member(ar, *this, ver); }
    void save<Archive>(Archive& ar, unsigned) const; void load<Archive>(Archive& ar, unsigned); // make_nvp("pointer")
    Node* get_node() const { return node; }
private: Node* node;
};
bool detail::operator==<Node>(const bidir_node_iterator<Node>& x, const bidir_node_iterator<Node>& y);

class detail::bucket_array_base<_=true> : noncopyable {
protected:
    static const size_t sizes[PP_SEQ_SIZE(BA_SIZES)]; // 28
    static size_t size_index(size_t n);
    static size_t position(size_t hash, size_t size_index_);
private: static const size_t sizes_length = 28;
};
class detail::bucket_array<Alloc> : bucket_array_base<> {
    using base_node_impl_type = hashed_index_base_node_impl<rebind_alloc_for<Alloc,char>::type>;
public: using <base>_pointer = base_node_impl_type::<base>_pointer;
    ctor(const Alloc& al, pointer end_, size_t size_);
    size_t size() const; size_t position(size_t hash) const;
    base_pointer begin() const; base_pointer end() const; base_pointer at(size_t n) const;
    void clear(pointer end_);
    void swap(self& x); void swap<BoolConstant>(self& x, BoolConstant swap_allocators);
private: using auto_space_type = auto_space<base_node_impl_type,Alloc>; using auto_space_size_type = auto_space_type::size_type;
    size_t size_index_; auto_space_type spc;
};
void detail::swap<A>(bucket_array<A>& x, bucket_array<A>& y);
void detail::load_construct_data<Ar,Alloc>(Ar&,bucket_array<Alloc>*, unsigned) { throw_exception(bad_archive_exception{}); }

struct detail::cons_stdtuple_ctor_terminal { using result_type=tuples::null_type; static result_type create<StdTuple>(const StdTuple&); };
struct detail::cons_stdtuple_ctor_normal<StdTuple,n> { using result_type=cons_stdtuple<StdTuple,n>; static result_type create(const StdTuple& t); };
struct detail::cons_stdtuple_ctor<StdTuple,n=0> :
    mpl::if_c<(n < std::tuple_size_v<StdTuple>), cons_stdtuple_ctor_normal<StdTuple,n>, cons_stdtuple_ctor_terminal>::type{};
struct detail::cons_stdtuple<StdTuple,n> {
    using head_type=std::tuple_element_t<n,StdTuple>;
    using tail_ctor=cons_stdtuple_ctor<StdTuple,n+1>; using tail_type = tail_ctor::result_type;
    const head_type& get_head() const{ return std::get<n>(t);} tail_type get_tail() const{ return tail_ctor::create(t); }
    const StdTuple& t;
};
cons_stdtuple_ctor<StdTuple>::result detail::make_cons_stdtuple<StdTuple>(const StdTuple& t);

struct detail::converter<MICont,Idx> {
    static <const> Idx& index(<const> MICont& x) { return x;}
    static Idx::<const>_iterator const_iterator(<const> MICont& x, MICont::final_node_type* node) { return x.Idx::make_iterator(node); }
};

struct detail::copy_map_entry<Node> { Node *first, *second; bool operator<(const self& x) const; };
struct detail::copy_map_value_copier { const Value& operator() <Value> (Value& x) const { return x; } };
struct detail::copy_map_value_mover { Value&& operator() <Value> (Value& x) const { return move(x); } };
class detail::copy_map<Node,Alloc> : noncopyable {
    using allocator_type=rebind_alloc_for<Alloc,Node>::type;
    using alloc_traits = allocator_traits<allocator_type>; using pointer = alloc_traits::pointer;
public: using const_iterator = const copy_map_entry<Node>*; using size_type = alloc_traits::size_type;
    ctor(const Alloc& al, size_type size, Node* header_org, Node* header_cpy); ~dtor();
    const_iterator begin() const;  const_iterator end() const;
    void copy_clone(Node* node);  void move_clone(Node* node);
    Node* find(Node* node) const;
    void release();
private: allocator_type al_; size_type size_; auto_space<copy_map_entry<Node,Alloc>> spc;
    size_type n; Node *header_org_, *header_cpy_; bool released;
    pointer allocate(); void deallocate(Node* node);
    void clone<ValueAccess>(Node* node, ValueAccess access);
};

struct detail::do_not_copy_elements_tag{};
struct detail::duplicates_iterator<Node,Pred> {
    using value_type=Node::value_type; using difference_type=Node::difference_type;
    using pointer=const Node::value_type*; using reference=const Node::value_type&;
    using iterator_category=std::forward_iterator_tag;
    ctor(<Node* node_>, Node* end_, Pred pred_);
    reference operator*() const; pointer operator->() const;
    duplicates_iterator& operator++(); duplicates_iterator operator++(int);
    Node* get_node() const;
private: Node *node, *begin_chunk, *end; Pred pred;
    void sync(); void advance();
};
bool detail::operator{==|!=}<N,P>(const duplicates_iterator<N,P>& x, const duplicates_iterator<N,P>& y);

struct detail::has_tag<Tag> { struct apply<Idx> : mpl::contains<Idx::tag_list,Tag>{}; };

struct detail::index_args_default_hash<KeyFromValue> { using type = hash<KeyFromValue::result_type>; };
struct detail::index_args_default_pred<KeyFromValue> { using type = std::equal_to<KeyFromValue::result_type>; };
struct detail::hashed_index_args<A1,A2,A3,A4> { using full_form = is_tag<A1>;
    using tag_list_type = mpl::if_<full_form,A1,tag<>>::type;
    using key_from_value_type = mpl::if_<full_form,A2,A1>::type;
    using supplied_hash_type = mpl::if_<full_form,A3,A2>::type;
    using hash_type = mpl::eval_if<is_na<supplied_hash_type>, index_args_default_hash<key_from_value_type>, identity<supplied_hash_type>>::type;
    using supplied_pred_type = mpl::if_<full_form,A4,A3>::type;
    using pred_type = mpl::eval_if<is_na<supplied_pred_type>, index_args_default_pred<key_from_value_type>, identity<supplied_pred_type>>::type;
};

struct detail::hashed_index_global_iterator_tag{}; struct detail::hashed_index_local_iterator_tag{};
struct detail::hashed_index_iterator<Node,BucketArray,IdxCat,ItCat> : forward_iterator_helper<self, Node::value_type, Node::difference_type, const Node::value_type*, const Node::value_type&> {
    using node_type = Node; using node_base_type = Node::base_type;
    ctor(){}  explicit ctor(Node* node_): node{node_}{}
    const Node::value_type& operator*() const { return node->value(); }
    self& operator++() { Node::increment(node); return *this; }
    void serialize<Archive>(Archive& ar, unsigned ver) { core::split_member(ar, *this, ver); }
    void save<Archive>(Archive& ar, unsigned) const; void load<Archive>(Archive& ar, unsigned); // make_nvp("pointer")
    Node* get_node() const { return node; }
private: Node* node;
};
bool detail::operator==<N,B,IdxCat,ItCat>(const hashed_index_iterator<N,B,IdxCat,ItCat>& x, const hashed_index_iterator<N,B,IdxCat,ItCat>& y);

struct detail::hashed_index_base_node_impl<Alloc> {
    using base_allocator=rebind_alloc_for<Alloc,self>::type;
    using node_allocator=rebind_alloc_for<Alloc,hashed_index_node_impl<Alloc>>::type;
    using {base|node}_alloc_traits=allocator_traits<{base|node}_allocator>;
    using <const>_base_pointer = base_alloc_traits::<const>_pointer;
    using <const>_pointer = node_alloc_traits::<const>_pointer;
    using difference_type = node_alloc_traits::difference_type;
    pointer& prior(); pointer prior() const;
private: pointer prior_;
};
struct detail::hashed_index_node_impl<Alloc> : hashed_index_base_node_impl<Alloc> {
    base_pointer& next(); base_pointer next() const;
    static pointer pointer_from(base_pointer x); static base_pointer base_pointer_from(pointer x);
private: base_pointer next_;
};
struct detail::default_assigner { void operator()<T>(T& x, const& val) { x=val; } };
```

------
### Configuration


------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Bind

* `<boost/bind/bind.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`
* `<boost/limits.hpp>`

#### Boost.ContainerHash

* `<boost/container_hash/hash_fwd.hpp>`

#### Boost.Core

* `<boost/core/addressof.hpp>`
* `<boost/core/enable_if.hpp>`
* `<boost/core/no_exceptions_support.hpp>`
* `<boost/core/noncopyable.hpp>`
* `<boost/core/ref.hpp>`
* `<boost/core/serialization.hpp>`

#### Boost.Integer

* `<boost/integer/common_factor_rt.hpp>`

#### Boost.Iterator

* `<boost/iterator/reverse_iterator.hpp>`

#### Boost.Move

* `<boost/move/core.hpp>`
* `<boost/move/utility_core.hpp>`
* `<boost/move/utility.hpp>`

#### Boost.MPL

* `<boost/mpl/*.hpp>`

#### Boost.Pool

* `<boost/pool/pool.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.SmartPtr

* `<boost/intrusive_ptr.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Tuple

* `<boost/tuple/tuple.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

#### Boost.Utility

* `<boost/call_traits.hpp>`
* `<boost/operators.hpp>`
* `<boost/utility/base_from_member.hpp>`

------
### Standard Facilities
