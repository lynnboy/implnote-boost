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
struct detail::unlink_undo_assigner<Node> {
  using <base>_pointer = Node::<base>_pointer;
  void operator()(<base>_pointer& x, <base>_pointer& val);
  void operator()(); // undo
  struct <base>_pointer_track { <base>_pointer* x; <base>_pointer val; };
  pointer_track pointer_trakcs[3]; int pointer_track_count;
  base_pointer_track base_pointer_trakcs[2]; int base_pointer_track_count;
};

struct detail::hashed_unique_tag{}; struct detail::hashed_non_unique_tag{};
struct detail::hashed_index_node_alg<Node,hashed_unique_tag> {
  using <const>_base_pointer = Node:<const>_base_pointer; using <const>_pointer=Node::<const>_pointer;
  static bool is_first_of_bucket(pointer x);
  static pointer after_<local>(pointer x); static pointer next_to_inspect(pointer x);
  static void link(pointer x, base_pointer buc, pointer end); static void unlink(pointer x);
  using unlink_undo = unlink_undo_assigner<Node>;
  static void unlink<Assigner>(pointer x, Assigner& assign);
  static void append(pointer x, pointer end);
  static bool unlink_last(pointer end);
private: static bool is_last_of_bucket(pointer x);
};
struct detail::hashed_index_node_alg<Node,hashed_non_unique_tag> {
  using <const>_base_pointer = Node:<const>_base_pointer; using <const>_pointer=Node::<const>_pointer;
  static bool is_first_of_bucket(pointer x); static bool is_first_of_group(pointer x);
  static pointer after_<local>(pointer x); static pointer next_to_inspect(pointer x);
  static void link(pointer x, base_pointer buc, pointer end); static void unlink(pointer x);
  using unlink_undo = unlink_undo_assigner<Node>;
  static void unlink<Assigner>(pointer x, Assigner& assign);
  static void link_range(pointer first, pointer last, base_pointer buc, pointer cend);
  static void append_range(pointer first, pointer last, pointer cend);
  static bool unlink_last_group(pointer end);
  static bool unlink_range(pointer first, pointer last);
private: static bool is_last_of_bucket(pointer x);
  static void {left|right}_unlink<Assigner>(pointer x, Assigner& assign);
  static void {left|right}_unlink_{first|last}_of_bucket<A>(pointer, A& assign);
  static void {right|left}_unlink_{first|last}_of_group<A>(pointer, A& assign);
  static void unlink_{last_but_one|second}_of_group<A>(pointer, A& assign);
};
struct detail::hashed_index_node_trampoline<Super> : hashed_index_node_impl<rebind_alloc_for<Super::allocator_type,char>::type> {
  using impl_type = base;
};
struct detail::hashed_index_node<Super> : Super, hashed_index_node_trampoline<Super> {
  using <const>_impl_<base>_pointer = base::<const>_<base>_poniter;
  struct node_alg<Category> { using type = hashed_index_node_alg<impl_type,Category>; };
  <const>_impl_pointer impl() <const>;
  static <const> self* from_impl(<const>_impl_pointer x);
  static void increment_<local> <Category>(self*& x);
};

struct detail::header_holder<NodeTypePtr,Final> : noncopyable {
  ctor(); ~dtor(); // allocate_node/deallocate_node
  NodeTypePtr member;
};

struct detail::index_access_sequence_terminal{ctor(void*){}};
struct detail::index_access_sequence_normal<MICont,n> {
  MICont* p; nth_index<MICont,n>::type& get(); index_access_sequence<MICont,n+1> next();
};
struct detail::index_access_sequence_base<MICont,n>
  : mpl::if_c<(n<mpl::size<MICont::index_type_list>::type::value),
    index_access_sequence_normal<MICont,n>, index_access_sequence_terminal> {};
struct detail::index_access_sequence<MICont,n> : index_access_sequence_base<MICont,n>::type {};

struct detail::lvalue_tag{}; struct detail::rvalue_tag{}; struct detail::emplaced_tag{};

class detail::index_base<Value,ISpecList,Alloc> {
protected:
  using index_node_type=index_node_base<Value,Alloc>;
  using final_node_type=multi_index_base_type<Value,ISpecList,Alloc>::type;
  using final_type = multi_index_container<Value,ISpecList,Alloc>;
  using ctor_args_list = null_type;
  using final_allocator_type = rebind_alloc_for<Alloc,Alloc::value_type>::type;
  using final_node_handle_type = node_handle<final_node_type, final_allocator_type>;
  using index_type_list=mpl::vector0<>; using <const>_iterator_type_list=mpl::vector0<>;
  using copy_map_type = copy_map<final_node_type,final_allocator_type>;
  using index_saver_type = index_saver<index_node_type,final_allocator_type>;
  using index_loader_type = index_loader<index_node_type, final_node_type, final_allocator_type>;
private:
  using value_type=Value; using alloc_traits=allocator_traits<Alloc>; using size_type = alloc_traits::size_type;
protected:
  explicit ctor(const ctor_args_list&, const Alloc&);
  ctor(const self& do_not_copy_elements_tag);
  void copy_(const self&, const copy_map_type&);
  final_node_type* insert_(const value_type& v, <index_node_type*>, final_node_type*& x, {lvalue|rvalue|emplaced}_tag);;
  final_node_type* insert_<MICont>(const value_type&, final_node_type*& x, MICont* p);
  void extract_<Dst>(index_node_type* Dst){}
  void clear_(){}
  void swap_<BoolConstant>(self&,BoolConstant){}
  void swap_elements_(self&){}
  bool replace_(const value_type& v, index_node_type* x, {lvalue|rvalue}_tag);
  bool modify_<rollback>_(index_node_type*){return true;}
  bool check_rollback_(index_node_type*) const {return true;}
  void save_<Ar>(Ar&, unsigned, const index_saver_type&)const{}
  void load_<Ar>(Ar&, unsigned, const index_loader_type&){}
  bool invariant_() const {return true;}

  <const> final_type& final()<const> {return *(<const>final_type*)this; }
  static Idx::final_type& final<Idx>(Idx& x) { return (Idx::final_type&)x; }
  final_node_type* final_header() const {return final().header();}
  bool final_empty_()const{return final().empty_();}
  size_type final_size_()const{return final().size_();}
  size_type final_max_size_()const{return final().max_size_();}
  std::pair<final_node_type*,bool> final_insert_<rv>_(const value_type& x, <final_node_type* position>);
  std::pair<final_node_type*,bool> final_insert_ref_<T>(<const>T & x, <final_node_type* position>);
  std::pair<final_node_type*,bool> final_insert_nh_(final_node_handle_type& nh, <final_node_type* position>);
  std::pair<final_node_type*,bool> final_transfer_<Idx>(Idx& x, final_node_type* position);
  std::pair<final_node_type*,bool> final_transfer_<Idx>(Idx& x, final_node_type* position);
  std::pair<final_node_type*,bool> final_emplace_<...Args>(Args&&...args);
  std::pair<final_node_type*,bool> final_emplace_hint_<...Args>(final_node_type* position, Args&&...args);
  final_node_handle_type final_extract_(final_node_type* x);
  void final_extract_for_transfer_<Dst>(final_node_type* x, Dst dst);
  void final_erase_(final_node_type* x);
  void final_delete_node_(final_node_type* x);
  void final_delete_all_nodes_();
  void final_clear_();
  void final_transfer_range_<Idx>(Idx& x, Idx::iterator first, Idx::iterator last);
  void final_swap_(final_type& x);
  bool final_replace_<rv>_(const value_type& k, final_node_type* x);
  bool final_modify_<Modifier,[Rollback]>(Modifier& mod, <Rollback& back>, final_node_type* x);
  void final_check_invariant_() const;
};

struct detail::index_loader<Node,FinalNode,Alloc> : noncopyable {
  ctor(const Alloc& al, size_t size);
  void add<Ar>(Node* node, Ar& ar, unsigned);
  void add_track<Ar>(Node* node, Ar& ar, unsigned);
  void load<Rearranger,Ar>(Rearranger r, Ar& ar, unsigned) const;
private: auto_space<Node*,Alloc> spc; size_t size_,n; mutable bool sorted;
};
struct detail::index_matcher::entry {
  ctor(void* node_, size_t pos_=0) : node{node_}, pos{pos_}{}
  void* node; size_t pos; entry* previous; bool ordered;
  struct less_by_node { bool operator()(const entry& x, const entry& y) const; };// node
  size_t pile_top; entry* pile_top_entry;
  struct less_by_pile_top { bool operator()(const entry& x, const entry& y) const; }; // pile_top
};
struct detail::index_matcher::algorithm_base<Alloc> : noncopyable {
protected: ctor(const Alloc& al, size_t size);
  void add(void* node);
  void begin_algorithm() const;
  void add_node_to_algorithm(void* node) const;
  void finish_algorithm() const;
  bool is_ordered(void* node) const;
private: auto_space<entry,Alloc> spc; size_t size_,n_; mutable bool sorted; mutable size_t num_piles;
};
struct detail::index_matcher::algorithm<Node,Alloc> : private algorithm_base<Alloc> {
  ctor(const Alloc& al, size_t size);
  void add(Node* node);
  void execute<IdxIt>(IdxIt first, IdxIt last) const;
  bool is_ordered(Node* node) const;
};
struct detail::index_saver<Node,Alloc> : noncopyable {
  ctor(const Alloc& al, size_t size);
  void add<Ar>(Node* node, Ar& ar, unsigned);
  void add_track<Ar>(Node* node, Ar& ar, unsigned);
  void save<IdxIt,Ar>(IdxIt first, IdxIt last, Ar& ar, unsigned) const;
private: index_matcher::algorithm<Node,Alloc> alg;
};

struct detail::pod_value_holder<Value> { using space = aligned_storage<sizeof(Value),alingment_of<Value>::value>::type; };
struct index_node_base<Value,Alloc>: private pod_value_holder<Value> {
  using base_type = self; using value_type = Value; using allocator_type = Alloc;
  <const> value_type = value() <const>;
  static self* from_value(const value_type* p);
private: void serialize<Ar>(Ar&, unsigned);
};
Node* node_from_value<Node,Value>(const Value* p);
void detail::load_construct_data<Ar,Value,Alloc>(Ar&, index_node_base<Value,Alloc>*, unsigned) { throw_exception(bad_archive_exception()); };

struct detail::invalidate_iterators
{ using iterator=void; self& get() {return *this;} self& next() { return *this;} };

struct detail::is_index_list<T> { constexpr static bool value=mpl::is_sequence<T>::value && !mpl::empty<T>::value; };
struct detail::is_transparent<F,A1,A2>: mpl_true{};
struct detail::is_transparent_class<F,A1,A2>; struct detail::is_transparent_function<F,A1,A2>;
struct detail::is_transparent<F,A1,A2> requires (is_class<F>&&!is_final<F>): is_transparent_class<F,A1,A2>{};
struct detail::is_transparent<F,A1,A2> requires (is_function<remove_pointer_t<F>>): is_transparent_function<remove_pointer_t<F>,A1,A2>{};
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
