# Boost.MultiIndex

* lib: `boost/libs/multi_index`
* repo: `boostorg/multi_index`
* commit: `e2b50ef`, 2025-09-19

------
#### Index

```c++
struct indexed_by<...T> : mpl::vector<T...>{};

struct detail::sequenced_index<SuperMeta,TagList> : SuperMeta::type {
protected: using index_node_type = sequenced_index_node<base::index_node_type>;
private: using node_impl_type = index_node_type::impl_type;
public: using value_type = index_node_type::value_type; using ctor_args = tuples::null_type;
    using allocator_type = base::final_allocator_type; using alloc_traits = allocator_traits<allocator_type>;
    using <const>_reference = <const> value_type&;
    using iterator = bidir_node_iterator<index_node_type>;, using const_iterator = iterator;
    using <const>_pointer = alloc_traits::<const>_pointer;
    using size_type = alloc_traits::size_type; using difference_type = alloc_traits::difference_type;
    using <const>_reverse_iterator = reverse_iterator<<const>_iterator>;
    using node_type = base::final_node_handle_type;
    using insert_return_type = insert_return_type<iterator,node_type>;
    using tag_list = TagList;
protected: using ctor_args_list = tuples::cons<ctor_args, base::ctor_args_list>;
    using index_type_list = mpl::push_front<base::index_type_list, sequenced_index>::type;
    using <const>_iterator_type_list = mpl::push_front<base::<const>_iterator_type_list, <const>_iterator>::type;
private: using value_param_type = call_traits<value_type>::param_type;
public: self& operator=(const self& x); self& operator=(std::initializer_list<value_type> list);
    void assign<InIt>(InIt first, InIt last); void assign(std::initializer_list<value_type> list);
    void assign(size_type n, value_param_type value);
  allocator_type get_allocator() const noexcept;
  <const>_iterator {begin|end}() <const> noexcept; const_iterator c{begin|end}() const noexcept;
  <const>_reverse_iterator r{begin|end}() <const> noexcept; const_reverse_iterator cr{begin|end}() const noexcept;
  <const>_iterator iterator_to(const value_type& x) <const>;
  bool empty() const noexcept; size_type size() const noexcept; size_type max_size() const noexcept;
  void resize(size_type n, <value_param_type x>);
  const_reference {front|back}() const;
  std::pair<iterator,bool> emplace_{front|back}<...Args>(Args&&... args);
  std::pair<iterator,bool> push_{front|back}(value_type {const&|&&} x);
  void pop_{front|back}();
  std::pair<iterator,bool> emplace<...Args>(iterator p, Args&&... args);
  std::pair<iterator,bool> insert(iterator p, value_type {const&|&&} x);
  void insert(iterator p, size_type n, value_param_type x);
  void insert<InIt>(iterator p, InIt first, InIt last); void insert(iterator p, std::initializer_list<value_type> list);
  insert_return_type insert(const_iterator p, node_type&& nh);
  node_type extract(const_iterator p);
  iterator erase(iterator p, <iterator last>);
  bool replace(iterator p, value_type {const&|&&} x);
  bool modify<Modifier,[Rollback]>(iterator p, Modifier mod, <Rollback back_>);
  void swap(self& x);
  void clear() noexcept;
  void splice<Idx>(iterator p, Idx{&|&&} x, <Idx::iterator i>, <Idx::iterator last>);
  void remove(value_param_type value); void remove_if<Pred>(Pred pred);
  void unique(); void unique<BPred>(BPred binary_pred);
  void merge(self& x, <Comp comp>); void merge<Comp>(self& x, Comp comp);
  void sort(); void sort<Comp>(Comp comp);
  void reverse() noexcept;
  void relocate(iterator p, iterator i, <iterator last>);
  void rearrange<InIt>(InIt first);
protected: ctor(const ctor_args_list& args_list, const allocator_type& al);
  ctor(const self& x); ctor(const self& x, do_not_copy_elements_tag); ~dtor();
  <const>_iterator make_iterator(index_node_type* node)<const>;
  void copy_(const self& x, const copy_map_type& map);
  final_node_type* insert_<Variant>(value_param_type v, <index_node_type* p>, final_node_type*& x, Variant variant);
  void extract_<Dst>(index_node_type* x, Dst dst);
  void delete_all_nodes_(); void clear_();
  void swap_<BoolConstant>(self& x, BoolConstant swap_allocators); void swap_elements_(self& x);
  bool replace_<Variant>(value_param_type v, index_node_type* x, Variant variant);
  bool modify_(index_node_type* x); bool modify_rollback_(index_node_type* x);
  bool check_rollback_(index_node_type* x) const;
  void save_<Ar>(Ar& ar, unsigned ver, const index_saver_type& sm) const;
  void load_<Ar>(Ar& ar, unsigned ver, const index_loader_type& sm);
  bool invariant_()const; void check_invariant_()const;
};
bool detail::operator{==|!=|<|>|<=|>=}<SM1,TL1,SM2,TL2>(const sequenced_index<SM1,TL1>& x, const sequenced_index<SM1,TL1>& y);
bool detail::swap<SM,TL>(sequenced_index<SM,TL>& x, sequenced_index<SM,TL>& y);

struct sequenced<TagList=tag<>> {
    struct node_class<Super> { using type=sequenced_index_node<Super>; };
    struct index_class<SuperMeta> { using type = sequenced_index<SuperMeta,TagList::type>; };
};

struct detail::random_access_index<SuperMeta,TagList> : SuperMeta::type {
protected: using index_node_type = random_access_index_node<base::index_node_type>;
private: using node_impl_type = index_node_type::impl_type;
    using ptr_array = random_access_index_ptr_array<base::final_allocator_type>;
    using node_impl_ptr_pointer = ptr_array::pointer;
public: using value_type = index_node_type::value_type; using ctor_args = tuples::null_type;
    using allocator_type = base::final_allocator_type; using alloc_traits = allocator_traits<allocator_type>;
    using <const>_reference = <const> value_type&;
    using iterator = rnd_node_iterator<index_node_type>;, using const_iterator = iterator;
    using <const>_pointer = alloc_traits::<const>_pointer;
    using size_type = alloc_traits::size_type; using difference_type = alloc_traits::difference_type;
    using <const>_reverse_iterator = reverse_iterator<<const>_iterator>;
    using node_type = base::final_node_handle_type;
    using insert_return_type = insert_return_type<iterator,node_type>;
    using tag_list = TagList;
protected: using ctor_args_list = tuples::cons<ctor_args, base::ctor_args_list>;
    using index_type_list = mpl::push_front<base::index_type_list, random_access_index>::type;
    using <const>_iterator_type_list = mpl::push_front<base::<const>_iterator_type_list, <const>_iterator>::type;
private: using value_param_type = call_traits<value_type>::param_type;
public: self& operator=(const self& x); self& operator=(std::initializer_list<value_type> list);
    void assign<InIt>(InIt first, InIt last); void assign(std::initializer_list<value_type> list);
    void assign(size_type n, value_param_type value);
  allocator_type get_allocator() const noexcept;
  <const>_iterator {begin|end}() <const> noexcept; const_iterator c{begin|end}() const noexcept;
  <const>_reverse_iterator r{begin|end}() <const> noexcept; const_reverse_iterator cr{begin|end}() const noexcept;
  <const>_iterator iterator_to(const value_type& x) <const>;
  bool empty() const noexcept; size_type size() const noexcept;
  size_type max_size() const noexcept; size_type capacity() const noexcept;
  void reserve(size_type n); void shrink_to_fit();
  void resize(size_type n, <value_param_type x>);
  const_reference operator[](size_type n) const; const_reference at(size_type n) const;
  const_reference {front|back}() const;
  std::pair<iterator,bool> emplace_{front|back}<...Args>(Args&&... args);
  std::pair<iterator,bool> push_{front|back}(value_type {const&|&&} x);
  void pop_{front|back}();
  std::pair<iterator,bool> emplace<...Args>(iterator p, Args&&... args);
  std::pair<iterator,bool> insert(iterator p, value_type {const&|&&} x);
  void insert(iterator p, size_type n, value_param_type x);
  void insert<InIt>(iterator p, InIt first, InIt last); void insert(iterator p, std::initializer_list<value_type> list);
  insert_return_type insert(const_iterator p, node_type&& nh);
  node_type extract(const_iterator p);
  iterator erase(iterator p, <iterator last>);
  bool replace(iterator p, value_type {const&|&&} x);
  bool modify<Modifier,[Rollback]>(iterator p, Modifier mod, <Rollback back_>);
  void swap(self& x);
  void clear() noexcept;
  void splice<Idx>(iterator p, Idx{&|&&} x, <Idx::iterator i>, <Idx::iterator last>);
  void remove(value_param_type value);
  void remove_if<Pred>(Pred pred);
  void unique(); void unique<BPred>(BPred binary_pred);
  void merge(self& x, <Comp comp>); void merge<Comp>(self& x, Comp comp);
  void sort(); void sort<Comp>(Comp comp);
  void reverse() noexcept;
  void relocate(iterator p, iterator i, <iterator last>);
  void rearrange<InIt>(InIt first);
protected: ctor(const ctor_args_list& args_list, const allocator_type& al);
  ctor(const self& x); ctor(const self& x, do_not_copy_elements_tag); ~dtor();
  <const>_iterator make_iterator(index_node_type* node)<const>;
  void copy_(const self& x, const copy_map_type& map);
  final_node_type* insert_<Variant>(value_param_type v, <index_node_type* p>, final_node_type*& x, Variant variant);
  void extract_<Dst>(index_node_type* x, Dst dst);
  void delete_all_nodes_(); void clear_();
  void swap_<BoolConstant>(self& x, BoolConstant swap_allocators); void swap_elements_(self& x);
  bool replace_<Variant>(value_param_type v, index_node_type* x, Variant variant);
  bool modify_(index_node_type* x); bool modify_rollback_(index_node_type* x);
  bool check_rollback_(index_node_type* x) const;
  void save_<Ar>(Ar& ar, unsigned ver, const index_saver_type& sm) const;
  void load_<Ar>(Ar& ar, unsigned ver, const index_loader_type& sm);
  bool invariant_()const; void check_invariant_()const;
private: ptr_array ptrs;
};
bool detail::operator{==|!=|<|>|<=|>=}<SM1,TL1,SM2,TL2>(const random_access_index<SM1,TL1>& x, const random_access_index<SM1,TL1>& y);
bool detail::swap<SM,TL>(random_access_index<SM,TL>& x, random_access_index<SM,TL>& y);

class detail::hashed_index<KeyFromValue,Hash,Pred,SuperMeta,TagList,Cat> : SM::type {
protected: using index_node_type = hashed_index_node<base::index_node_type>;
private: using node_alg=index_node_type::node_alg<Cat>::type;
  using node_impl_type=index_node_type::impl_type; using node_impl_<base>_pointer=node_impl_type::<base>_pointer;
  using bucket_array_type = bucket_array<base::final_allocator_type>;
public: using key_type=KeyFromValue::result_type; using value_type=index_node_type::value_type;
  using key_from_value=KeyFromValue; using hasher=Hash; using key_equal=Pred;
  using allocator_type = base::final_allocator_type; using alloc_traits=allocator_traits<allocator_type>;
  using <const>_pointer = alloc_traits::<const>_pointer; using <const>_reference = <const> value_type&;
  using size_type = alloc_traits::size_type; using difference_type = alloc_traits::difference_type;
  using ctor_args = tuple<size_type,key_from_value,hasher,key_equal>;
  using iterator = hashed_index_iterator<index_node_type, bucket_array_type, Cat, hashed_index_global_iterator_tag>;
  using local_iterator = hashed_index_iterator<index_node_type, bucket_array_type, Cat, hashed_index_local_iterator_tag>;
  using const_<local>_iterator = <local>_iterator;
  using node_type = base::final_node_handle_type;
  using insert_return_type = insert_return_type<iterator,node_type>;
  using tag_list = TagList;
protected: using ctor_args_list = tuples::cons<ctor_args,base::ctor_args_list>;
  using index_type_list = mpl::push_front<base::index_type_list, hashed_index>::type;
  using <const>_iterator_type_list = mpl::push_front<base::<const>_iterator_type_list, <const>_iterator>::type;
private: using value_param_type = call_traits<value_type>::param_type; using key_param_type = call_traits<key_type>::param_type;
public: self& operator=(const self& x); self& operator=(std::initializer_list<value_type> list);
  allocator_type get_allocator() const noexcept;
  bool empty() const noexcept; size_type size() const noexcept; size_type max_size() const noexcept;
  <const>_iterator {begin|end}() <const> noexcept; const_iterator c{begin|end}() const noexcept;
  <const>_iterator iterator_to(const value_type& x) <const>;
  std::pair<iterator,bool> emplace<...Args>(Args&&...args);
  iterator emplace_hint<...Args>(iterator p, Args&&...args);
  std::pair<iterator,bool> insert(value_type {const&|&&} x);
  iterator insert(iterator p, value_type {const&|&&} x);
  void insert<InIt>(InIt first, InIt last); void insert(std::initializer_list<value_type> list);
  insert_return_type insert(node_type&& nh); iterator insert(const_iterator p, node_type&& nh);
  node_type extract(const_iterator p); node_type extract(key_param_type x);
  iterator erase(iterator p, <iterator last>); size_type erase(key_param_type k);
  bool replace(iterator p, value_type {const&|&&} x);
  bool modify<Modifier,[Rollback]>(iterator p, Modifier mod, <Rollback back_>);
  bool modify_key<Modifier,[Rollback]>(iterator p, Modifier mod, <Rollback back_>);
  void clear() noexcept;
  void swap(self& x);
  void merge<Idx>(Idx{&|&&} x, <Idx::iterator i>, <Idx::iterator last>);
  key_from_value key_extractor() const; hasher hash_function() const; key_equal key_eq() const;

  iterator find<Key2,[Hash2,Pred2]>(const Key2& k, <const Hash2& hash, const Pred2& eq>) const;
  size_type count<Key2,[Hash2,Pred2]>(const Key2& k, <const Hash2& hash, const Pred2& eq>) const;
  bool contains<Key2,[Hash2,Pred2]>(const Key2& k, <const Hash2& hash, const Pred2& eq>) const;
  std::pair<iterator,iterator> equal_range<Key2,[Hash2,Pred2]>(const Key2& k, <const Hash2& hash, const Pred2& eq>) const;
  size_type bucket_count() const noexcept; size_type max_bucket_count() const noexcept;
  size_type bucket_size(size_type n) const; size_type bucket(key_param_type k) const;
  <const>_local_iterator {begin|end}(size_type n)<const>;   const_local_iterator c{begin|end}(size_type n) const;
  <const>_local_iterator local_iterator_to(const value_type& x) <const>;
  float load_factor() const noexcept;
  float max_load_factor() const noexcept; void max_load_factor(float z);
  void rehash(size_type n); void reserve(size_type n);
protected: ctor(const ctor_args_list& args_list, const allocator_type& al);
  ctor(const self& x); ctor(const self& x, do_not_copy_elements_tag); ~dtor();
  <const>_iterator make_iterator(index_node_type* node)<const>;
  <const>_local_iterator make_local_iterator(index_node_type* node)<const>;
  void copy_(const self& x, const copy_map_type& map);
  final_node_type* insert_<Variant>(value_param_type v, <index_node_type* p>, final_node_type*& x, Variant variant);
  void extract_<Dst>(index_node_type* x, Dst dst);
  void delete_all_nodes_(); void clear_();
  void swap_<BoolConstant>(self& x, BoolConstant swap_allocators); void swap_elements_(self& x);
  bool replace_<Variant>(value_param_type v, index_node_type* x, Variant variant);
  bool modify_(index_node_type* x); bool modify_rollback_(index_node_type* x);
  bool check_rollback_(index_node_type* x) const;
  bool equals(const self& x) const;
  void save_<Ar>(Ar& ar, unsigned ver, const index_saver_type& sm) const;
  void load_<Ar>(Ar& ar, unsigned ver, const index_loader_type& sm);
  bool invariant_()const; void check_invariant_()const;
private: key_from_value key; hasher hash_; key_equal eq_; bucket_array_type buckets; float mlf; size_type max_load;
};
bool operator{==|!=}<KFV,H,P,SM,TL,Cat>(const hashed_index<KFV,H,P,SM,TL,Cat>& x, const hashed_index<KFV,H,P,SM,TL,Cat>& y);
bool swap<KFV,H,P,SM,TL,Cat>(hashed_index<KFV,H,P,SM,TL,Cat>& x, hashed_index<KFV,H,P,SM,TL,Cat>& y);

struct hashed_<non>_unique<A1,A2,A3,A4> {
  using index_args = hashed_index_args<A1,A2,A3,A4>;
  using tag_list_type=index_args::tag_list_type::type; using key_from_value_type = index_args::key_from_value_type;
  using hash_type = index_args::hash_type; using pred_type = index_args::pred_type;
  struct node_class<Super> { using type=hashed_index_node<Super>; };
  struct index_class<SuperMeta> { using type = hashed_index<key_from_value_type,hash_type,pred_type,SuperMeta,tag_list_type,hashed_<non>_unique_tag>; };
};

struct detail::null_augment_policy {
    struct augmented_interface<OrdIdxImpl> { using type=OrdIdxImpl; };
    struct augmented_node<OrdIdxNodeImpl> { using type=OrdIdxNodeImpl; }
    static void {add|remove|copy|rotate_left|rotate_right}<Pointer>(Pointer,Pointer){}
    static bool invariant<Pointer>(Pointer){return true;}
};
struct ordered_<non>_unique<A1,A2=na,A3=na> {
    using index_args=ordered_index_args<A1,A2,A3>;
    using tag_list_type = index_args::tag_list_type::type;
    using key_from_value_type = index_args::key_from_value_type; using compare_type = index_args::compare_type;
    struct node_class<Super> { using type=ordered_index_node<null_augment_policy,Super>; };
    struct index_class<SuperMeta> { using type=ordered_index<key_from_value_type,compare_type,SuperMeta,tag_list_type,ordered_<non>_unique_tag,null_augment_policy>; };
};
struct detail::ranked_node<OrdIdxNodeImpl> : OrdIdxNodeImpl { using size_type = OrdIdxNodeImpl::size_type; size_type size; };
struct detail::ranked_index<OrdIdxImpl> : OrdIdxImpl {
    size_type count<Key2,[Comp2]>(const Key2& x, <const Comp2& comp>) const;
    iterator nth(size_type n) const;
    size_type rank(iterator p) const;
    size_type {find|lower_bound|upper_bound}_rank<Key2,[Comp2]>(const Key2& x, <const Comp2& comp>) const;
    std::pair<size_type,size_type> equal_range_rank<Key2,[Comp2]>(const Key2& x, <const Comp2& comp>) const;
    std::pair<size_type,size_type> range_rank<LB,UB>(LB lower, UB upper) const;
protected: ctor(const self& x); ctor(const self& x, do_not_copy_elements_tag);
    ctor(const ctor_args_list& args_list, const allocator_type& al);
};
struct detail::rank_policy {
    struct augmented_node<OrdIdxNodeImpl> { using type=ranked_node<OrdIdxNodeImpl>; }
    struct augmented_interface<OrdIdxImpl> { using type=ranked_index<OrdIdxImpl>; };
    static void {add|remove|copy|rotate_left|rotate_right}<Pointer>(Pointer,Pointer){}
    static bool invariant<Pointer>(Pointer){return true;}
};
struct ranked_<non>_unique<A1,A2=na,A3=na> {
    using index_args=ordered_index_args<A1,A2,A3>;
    using tag_list_type = index_args::tag_list_type::type;
    using key_from_value_type = index_args::key_from_value_type; using compare_type = index_args::compare_type;
    struct node_class<Super> { using type=ordered_index_node<null_augment_policy,Super>; };
    struct index_class<SuperMeta> { using type=ordered_index<key_from_value_type,compare_type,SuperMeta,tag_list_type,ordered_<non>_unique_tag,null_augment_policy>; };
};
```

##### Key Extractors

```c++
namespace detail{
// and less, greater, hash
struct key_equal_to<KeyFromValue> { using type=std::equal_to<KeyFromValue::result_type>; };
struct key_equal_to<tuples::null_type> { using type=tuples::null_type; };
struct nth_composite_key_equal_to<CompositeKey,n> { using type=key_equal_to<nth_key_from_value<CompositeKey,N>::type>::type; };
struct generic_operator_equal{bool operator<T,Q>(const T& x, const Q& y) const{return x==y;}};
using generic_operator_equal_tuple = tuple<generic_operator_equal,...>; // 10 elements
// and compare_x_x
struct equal_ckey_ckey_{terminal|normal}<KeyCons1,Value1,KeyCons2,Value2,EqualCons> {static bool compare(const KeyCons1&,const Value1&, const KeyCons2&, const Value2&, const EqualCons&);};
struct equal_ckey_ckey<KeyCons1,Value1,KeyCons2,Value2,EqualCons> : mpl::if_<..., equal_ckey_ckey_terminal<__>,equal_ckey_ckey_normal<__>>::type{};
struct equal_ckey_cval_{terminal|normal}<KeyCons,Value,ValCons,,CompareCons>
{static bool compare(const KeyCons&, const Value&, const ValCons&, const CompareCons&);
 static bool compare(const ValCons&, const KeyCons&, const Value&, const CompareCons&);};
struct equal_ckey_cval<KeyCons,Value,ValCons,,CompareCons> : mpl::if_<..., equal_ckey_cval_terminal<__>,equal_ckey_cval_normal<__>>::type{};

struct hash_ckey_{terminal|normal}<KeyCons,Value,HashCons> {static size_t hash(const KeyCons&,const Value&, const HashCons&, size_t carry);};
struct hash_ckey<KeyCons,Value,HashCons> : mpl::if_<..., hash_ckey_terminal<__>,hash_ckey_normal<__>>::type{};
struct hash_cval_{terminal|normal}<ValCons,HashCons> {static size_t hash(const ValCons&, const HashCons&, size_t carry);};
struct hash_cval<ValCons,HashCons> : mpl::if_<..., hash_ckey_terminal<__>,hash_ckey_normal<__>>::type{};
}

struct composite_key_result<CompositeKey> {
  using composite_key_type=CompositeKey; using value_type = CompositeKey::value_type;
  const composite_key_type& composite_key; const value_type& value;
};
struct composite_key<Value,...KeyFromValue> : private tuple<KeyFromValue...> {
  using key_extractor_tuple=base; using value_type=Value; using result_type=composite_key_result<composite_key>;
  ctor(KeyFromValue...k); ctor(const key_extractor_tuple& x);
  <const> key_extractor_tuple& key_extractors() <const>;
  result_type operator()<ChainedPtr>(const ChainedPtr& x) const;
  result_type operator()(const value_type& x) const;
  result_type operator()(const reference_wrapper<<const> value_type>& x) const;
};
bool operator{==|<}<CompKey1,CompKey2>(const composite_key_result<CompKey1>& x, const composite_key_result<CompKey2>& y);
bool operator{==|<}<CompKey,...Value>(const composite_key_result<CompKey>& x, const tuple<Value...>& y);
bool operator{==|<}<CompKey,...Value>(const tuple<Value...>& x, const composite_key_result<CompKey>& y);
// and !=, >, <=, >=

// and compare key_comps
struct composite_key_equal_to<...Pred> : private tuple<Pred...> {
  using key_eq_tuple=base;
  ctor(const Pred&...k); ctor(const key_eq_tuple& x);
  <const> key_eq_tuple& key_eqs() <const>;
  bool operator()<CompKey1,CompKey2>(const composite_key_result<CompKey1>& x, const composite_key_result<CompKey1>& y) const;
  bool operator()<CompKey,...Value>(const tuple<Value...>& x, const composite_key_result<CompKey>& y) const;
  bool operator()<CompKey,...Value>(const composite_key_result<CompKey>& x, const tuple<Value...>& y) const;
};
struct composite_key_hash<...Hash> : private tuple<Hash...> {
  using key_hasher_tuple=base;
  ctor(const Hash&...k); ctor(const key_hasher_tuple& x);
  <const> key_hasher_tuple& key_hash_functions() <const>;
  size_t operator()<CompKey>(const composite_key_result<CompKey>& x) const;
  size_t operator()<...Value>(const tuple<Value...>& x) const;
};
// and compare, hash
struct composite_key_result_equal_to<CompKeyRes> : composite_key_equal_to<nth_composite_key_equal_to<CompKeyRes::composite_key_type,n>::type...>{using base::operator();};

struct detail::const_identity_base<Type> { using result_type=Type;
  Type& operator()<ChainedPtr>(const ChainedPtr& x) const;
  Type& operator()(Type& x) const;
  Type& operator()(const reference_wrapper<Type>& x)const;
  Type& operator()(const reference_wrapper<remove_const<Type>>& x)const;
};
struct detail::non_const_identity_base<Type> { using result_type=Type;
  const Type& operator()<ChainedPtr>(const ChainedPtr& x) const;
  <const> Type& operator()(<const> Type& x) const;
  <const> Type& operator()(const reference_wrapper<<const> Type>& x)const;
};
struct identity<Type>: mpl::if_c<is_const_v<Type>, const_identity_base<Type>, non_const_identity_base<Type>>::type {};

struct detail::<non>_<const>_ref_global_fun_base<Value,Type,Type(*p)(Value)> {
  using result_type = remove_reference_t<Type>;
  Type operator()<ChainedPtr>(const ChainedPtr& x) const;
  bool operator()(Value x) const;
  bool operator()(const reference_wrapper<remove_reference_t<Value>>& x) const;
};
struct global_fun<Value,Type,Type(*p)(Value)> : ...{}; // chose between const_ref, non_const_ref, non_ref

struct detail::<non>_const_member_base<Class,Type,Type Class::*mp> {
    using result_type = Type;
    Type& operator()<ChainedPtr>(const ChainedPtr& x) const;
    Type& operator()(const Class& x) const;
    Type& operator()(const reference_wrapper<<const> Class>& x) const;
};
struct member<Class,Type,Type Class::*mp> : mpl::if_c<is_const_v<Type>, const_member_base<__>, non_const_member_base<__>>::type{};
struct detail::<non>_const_member_offset_base<Class,Type,offset> {
    using result_type = Type;
    Type& operator()<ChainedPtr>(const ChainedPtr& x) const;
    Type& operator()(const Class& x) const;
    Type& operator()(const reference_wrapper<<const> Class>& x) const;
};
struct member_offset<Class,Type,offset> : mpl::if_c<is_const_v<Type>, const_member_offset_base<__>, non_const_offset_base<__>>::type{};

struct detail::<const>_mem_fun_impl<Class,Type,MFPtr,mp> {
    using result_type = remove_reference_t<Type>;
    Type operator()<ChainedPtr>(const ChainedPtr& x) const;
    Type operator()(const Class& x) const;
    Type operator()(const reference_wrapper<<const> Class>& x) const;
};
struct const_mem_fun<Class,Type,Type (Class::*mp)()const> : const_mem_fun_impl<Class,Type,Type(Class::*)()const,mp>{};
struct cv_mem_fun<Class,Type,Type (Class::*mp)()const volatile> : const_mem_fun_impl<Class,Type,Type(Class::*)()const  volatile,mp>{};
struct mem_fun<Class,Type,Type (Class::*mp)()> : const_mem_fun_impl<Class,Type,Type(Class::*)(),mp>{};
struct volatile_mem_fun<Class,Type,Type (Class::*mp)()volatile> : const_mem_fun_impl<Class,Type,Type(Class::*)()volatile,mp>{};
struct cref_mem_fun<Class,Type,Type (Class::*mp)()const&> : const_mem_fun_impl<Class,Type,Type(Class::*)()const&,mp>{};
struct cvref_mem_fun<Class,Type,Type (Class::*mp)()const volatile&> : const_mem_fun_impl<Class,Type,Type(Class::*)()const volatile&,mp>{};
struct ref_mem_fun<Class,Type,Type (Class::*mp)()&> : const_mem_fun_impl<Class,Type,Type(Class::*)()&,mp>{};
struct vref_mem_fun<Class,Type,Type (Class::*mp)()volatile&> : const_mem_fun_impl<Class,Type,Type(Class::*)()volatile&,mp>{};

struct detail::typed_key_impl<Type Class::*, mptr> requires !is_function_v<Type> { using value_type=Class; using type = member<Class,Type,mptr>; };
struct detail::typed_key_impl<Type (Class::*)()<const><volatile><&>, mfptr> { using value_type=Class; using type = <c|v><ref>_mem_fun<Class,Type,mfptr>; };
struct detail::typed_key_impl<Type(*)(Value),fptr> { using value_type=Value; using type=global_fun<Value,Type,fptr>; };

struct detail::remove_noexcept<T>{using type=T;};
struct detail::remove_noexcept<R(C::*)(Args...)<const><volatile><&|&&>noexcept>{using type=R(C::*)(Args...)<const><volatile><&|&&>;};
struct detail::remove_noexcept<R(C::*)(Args...,...)<const><volatile><&|&&>noexcept>{using type=R(C::*)(Args...,...)<const><volatile><&|&&>;};
struct detail::remove_noexcept<R(*)(Args...)noexcept>{using type=R(*)(Args...);};
struct detail::remove_noexcept<R(*)(Args...,...)noexcept>{using type=R(*)(Args...,...);};
using detail::remove_noexcept_t<T> = remove_noexcept_t<T>;
struct detail::key_impl<key> : typed_key_impl<remove_noexcept_t<decltype(key)>,key>{};
struct detail::least_generic<T0,Ts...> { using type=T0; };
struct detail::least_generic<T0,T1,Ts...> { using type=least_generic<std::conditional_t<is_convertible_v<const T0&,const T1&>,T0,T1>,Ts...>::type; };
struct detail::key_impl<keys...> {
    using value_type = least_generic<std::decay_t<key_impl<keys>::value_type>...>::type;
    using type = composite_key<value_type, key_impl<keys>::type...>;
};
struct detail::composite_key_size<_=composite_key<void,void>>;
struct detail::composite_key_size<composite_key<Args...>> { static constexpr auto value=sizeof...(Args)-1; };
struct detail::limited_size_key_impl<...keys> { using type = key_impl<keys...>::type; };
using key<...keys> = limited_size_key_impl<keys...>::type;
```

#### Multi Index Container

```c++
struct detail::unequal_alloc_move_ctor_tag{};

class multi_index_container<Value,ISpecList=indexed_by<ordered_unique<identity<Value>>>,
    Alloc=std::allocator<Value>,  _alloc=rebind_alloc_for<Alloc,multi_index_node_type<Value,ISpecList,Alloc>::type>::type>
    : private base_from_member<_alloc>, header_holder<allocator_traits<_alloc>::pointer, self>, multi_index_base_type<__>
{ using node_allocator=rebind_alloc_for<Alloc,base::index_node_type>::type;
    using node_alloc_traits=allocator_traits<node_allocator>;
    using bfm_allocator=base_from_member<node_allocator>;
    using bfm_header = header_holder<node_pointer, self>;
    using index_specifier_type_list = ISpecList;
    using allocator_type = base::final_allocator_type;
    explicit ctor(<const allocator_type& al>); ctor(const {self&|&&} x, <const allocator_type& al>); ~dtor();
    explicit ctor(const ctor_args_list& args_list, const allocator_type& al={});
    ctor<InIt>(InIt first, InIt last, const ctor_args_list& args_list={}, const allocator_type& al={});
    ctor(std::initializer_list<Value> list, const ctor_args_list& args_list={}, const allocator_type& al={});
    self& operator=(self {const&|&&} x); self& operator=(std::initializer_list<Value> list);
    allocator_type get_allocator() const noexcept;
    struct nth_index<n> { using type = mpl::at_c<index_type_list,n>::type; };
    <const> nth_index<n>::type& get() <const> noexcept;
    struct index<Tag> { using iter=mpl::find_if<index_type_list,has_tag<Tag>>::type; using type=mpl::deref<iter>::type; };
    <const> index<Tag>::type& get() <const> noexcept;
    struct nth_index_<const>_iterator<n> { using type = nth_index<n>::type::<const>_iterator; };
    nth_index_<const>_iterator<n>::type project<n,It>(It it) <const>;
    struct index_<const>_iterator<Tag> { using index<Tag>::type::<const>_iterator; };
    index_<const>_iterator<Tag>::type project<Tag,It>(It it) <const>;
protected:
    ctor(self& x, const allocator_type& al, unequal_alloc_move_ctor_tag);
    ctor(const self& x, do_not_copy_elements_tag);
    void copy_construct_from(const self& x);
    final_node_type* header() const;
    final_node_type* allocate_node(); void deallocate_node(final_node_type* x);
    void construct_value(final_node_type* x, Value {const&|&&} v); void destroy_value(final_node_type* x);
    bool empty_() const; size_type size_() const; size_type max_size_() const;
    std::pair<final_node_type*,bool> insert_<Variant>(const Value& v, <final_node_type* p>, Variant variant);
    std::pair<final_node_type*,bool> insert_<rv>_(const Value& v, <final_node_type* p>);
    std::pair<final_node_type*,bool> insert_ref_<T>(T& t, <final_node_type* p>);
    std::pair<final_node_type*,bool> insert_ref_(value {const&|&&} x, <final_node_type* p>);
    std::pair<final_node_type*,bool> insert_nh_(final_node_handle_type& nh, <final_node_type* p>);
    std::pair<final_node_type*,bool> transfer_<Idx>(Idx& x, final_node_type* n);
    std::pair<final_node_type*,bool> emplace_<...Args>(Args&&...args);
    std::pair<final_node_type*,bool> emplace_hint_<...Args>(final_node_type* p, Args&&...args);
    final_node_handle_type extract_(final_node_type* x);
    void extract_for_transfer_<Dst>(final_node_type* x, Dst dst);
    void erase_(final_node_type* x); void delete_node_(final_node_type* x);
    void delete_all_node_(); void clear_();
    void transfer_range_<Idx>(Idx& x, Idx::iterator first, Idx::iterator last);
    void swap_(self& x); void swap_elements(self& x);
    bool replace_<rv>_(const Value& k, final_node_type* x);
    bool modify_<Modifier,[Rollback]>(Modifier& mod, <Rollback& back_>, final_node_type* x);
    void serialize<Ar>(Ar& ar, unsigned ver);
    void save<Ar>(Ar& ar, unsigned) const; void load<Ar>(Ar& ar, unsigned); 
    bool invariant_() const; void check_invariant_() const;
private: size_type node_count;
};

struct nth_index<MICont,n> { using type = mpl::at_c<MICont::index_type_list,n>::type; };
<const> nth_index<multi_index_container<Value,ISpecList,Alloc>,n>::type& get<n,Value,ISpecList,Alloc>(<const> multi_index_container<Value,ISpecList,Alloc>& m) noexcept;

struct index<MICont,Tag> { using iter=mpl::find_if<index_type_list,has_tag<Tag>>::type; using type=mpl::deref<iter>::type; };
<const> index<multi_index_container<Value,ISpecList,Alloc>,Tag>::type& get<Tag,Value,ISpecList,Alloc>(<const> multi_index_container<Value,ISpecList,Alloc>& m) noexcept;

struct nth_index_<const>_iterator<MICont,n> { using type = nth_index<MICont,n>::type::<const>_iterator; };
nth_index_<const>_iterator<multi_index_container<Value,ISpecList,Alloc>,n>::type& project<n,It,Value,ISpecList,Alloc>(<const> multi_index_container<Value,ISpecList,Alloc>& m, It it);

struct index_<const>_iterator<MICont,Tag> { using type = index<MICont,Tag>::type::<const>_iterator; };
index_<const>_iterator<multi_index_container<Value,ISpecList,Alloc>,Tag>::type& project<Tag,It,Value,ISpecList,Alloc>(<const> multi_index_container<Value,ISpecList,Alloc>& m, It it);

bool operator{==|!=|<|>|<=|>=} <V1,ISL1,A1,V2,ISL2,A2> (const multi_index_container<V1,ISL1,A1>& x, const multi_index_container<V1,ISL1,A1>& y);
void swap<V,ISL,A> (multi_index_container<V,ISL,A>& x, multi_index_container<V,ISL,A>& y);
```

------
#### Details

```c++
namespace detail;
struct any_container_view <CIt> {
    ctor<Cont>(const Cont& x) : px{&x}, pt{vtable_for<Cont>()}{}
    const void* container() const { return px; }
    CIt begin() const { return pt->begin(px); } CIt end() const { return pt->end(px); }
private: struct vtable { CIt (*begin)(const void*); CIt (*end)(const void*); };
    static CIt begin_for<Cont>(const void* px) { return ((const Cont*)px)->begin(); }
    static CIt end_for<Cont>(const void* px) { return ((const Cont*)px)->end(); }
    vtable* vtable_for<Cont>() { static vtable v={&begin_for<Cont>, &end_for<Cont>}; return &v; }
    const void* px; vtable* pt;
};

struct archive_constructed<T> : noncopyable {
    ctor<Archive>(Archive& ar, unsigned ver)
    { core::load_construct_data_adl(ar,&get(),ver); try{ar>>get();}catch(...){(&get())->~T(); throw;} }
    ctor<Archive>(const char* name, Archive& ar, unsigned ver)
    { core::load_construct_data_adl(ar,&get(),ver); try{ar>>core::make_nvp(name,get());}catch(...){(&get())->~T(); throw;} }
    ~dtor() { (&get())->~T(); }
    T& get() { return *(T*)(&space); }
private: aligned_storage<sizeof(T),alingment_of<T>::value>::type space;
};

struct auto_space<T,Alloc=std::allocator<T>> : noncopyable {
    using allocator=rebind_alloc_for<Alloc,T>::type; using alloc_traits = allocator_traits<allocator>;
    using pointer = alloc_traits::pointer; using size_type = alloc_traits::size_type;
    explicit ctor(const Alloc& al={}, size_type n=1); ~dtor();
    Alloc get_allocator() const { return al_; }
    pointer data() const { return data_; }
    void swap(self& x) { if constexpr (alloc_traits::propagate_on_container_swap::value) adl_swap(al_,x.al_); std::swap(n_,x.n_); std::swap(data_,x.data_); }
private: allocator al_; size_type n_; pointer data_;
};
void swap<T,A>(auto_space<T,A>& x, auto_space<T,A>& y);

struct bad_archive_exception : std::runtime_error;

struct index_applier {
    struct apply<ISpecMeta,SuperMeta> { using index_specifier = ISpecMeta::type; using type = index_specifier::index_class<SuperMeta>::type; };
};
struct nth_layer<n,Value,ISpecList,Alloc> {
    constexpr static int length = mpl::size<ISpecList>::value;
    using type = mpl::eval_if_c<n==length, mpl::identity<index_base<Value,ISpecList,Alloc>>,
                                           mpl::apply2<index_applier, at_c<ISpecList,n>, nth_layer<n+1,Value,ISpecList,Alloc>>>::type;
};
struct multi_index_base_type<Value,ISpecList,Alloc> : nth_layer<0,Value,ISpecList,Alloc>{};

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
bool operator==<Node>(const bidir_node_iterator<Node>& x, const bidir_node_iterator<Node>& y);

class bucket_array_base<_=true> : noncopyable {
protected:
    static const size_t sizes[PP_SEQ_SIZE(BA_SIZES)]; // 28
    static size_t size_index(size_t n);
    static size_t position(size_t hash, size_t size_index_);
private: static const size_t sizes_length = 28;
};
class bucket_array<Alloc> : bucket_array_base<> {
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
void swap<A>(bucket_array<A>& x, bucket_array<A>& y);
void load_construct_data<Ar,Alloc>(Ar&,bucket_array<Alloc>*, unsigned) { throw_exception(bad_archive_exception{}); }

struct cons_stdtuple_ctor_terminal { using result_type=tuples::null_type; static result_type create<StdTuple>(const StdTuple&); };
struct cons_stdtuple_ctor_normal<StdTuple,n> { using result_type=cons_stdtuple<StdTuple,n>; static result_type create(const StdTuple& t); };
struct cons_stdtuple_ctor<StdTuple,n=0> :
    mpl::if_c<(n < std::tuple_size_v<StdTuple>), cons_stdtuple_ctor_normal<StdTuple,n>, cons_stdtuple_ctor_terminal>::type{};
struct cons_stdtuple<StdTuple,n> {
    using head_type=std::tuple_element_t<n,StdTuple>;
    using tail_ctor=cons_stdtuple_ctor<StdTuple,n+1>; using tail_type = tail_ctor::result_type;
    const head_type& get_head() const{ return std::get<n>(t);} tail_type get_tail() const{ return tail_ctor::create(t); }
    const StdTuple& t;
};
cons_stdtuple_ctor<StdTuple>::result make_cons_stdtuple<StdTuple>(const StdTuple& t);

struct converter<MICont,Idx> {
    static <const> Idx& index(<const> MICont& x) { return x;}
    static Idx::<const>_iterator const_iterator(<const> MICont& x, MICont::final_node_type* node) { return x.Idx::make_iterator(node); }
};

struct copy_map_entry<Node> { Node *first, *second; bool operator<(const self& x) const; };
struct copy_map_value_copier { const Value& operator() <Value> (Value& x) const { return x; } };
struct copy_map_value_mover { Value&& operator() <Value> (Value& x) const { return move(x); } };
class copy_map<Node,Alloc> : noncopyable {
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

struct do_not_copy_elements_tag{};
struct duplicates_iterator<Node,Pred> {
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
bool operator{==|!=}<N,P>(const duplicates_iterator<N,P>& x, const duplicates_iterator<N,P>& y);

struct has_tag<Tag> { struct apply<Idx> : mpl::contains<Idx::tag_list,Tag>{}; };

struct index_args_default_hash<KeyFromValue> { using type = hash<KeyFromValue::result_type>; };
struct index_args_default_pred<KeyFromValue> { using type = std::equal_to<KeyFromValue::result_type>; };
struct hashed_index_args<A1,A2,A3,A4> { using full_form = is_tag<A1>;
    using tag_list_type = mpl::if_<full_form,A1,tag<>>::type;
    using key_from_value_type = mpl::if_<full_form,A2,A1>::type;
    using supplied_hash_type = mpl::if_<full_form,A3,A2>::type;
    using hash_type = mpl::eval_if<is_na<supplied_hash_type>, index_args_default_hash<key_from_value_type>, identity<supplied_hash_type>>::type;
    using supplied_pred_type = mpl::if_<full_form,A4,A3>::type;
    using pred_type = mpl::eval_if<is_na<supplied_pred_type>, index_args_default_pred<key_from_value_type>, identity<supplied_pred_type>>::type;
};

struct hashed_index_global_iterator_tag{}; struct hashed_index_local_iterator_tag{};
struct hashed_index_iterator<Node,BucketArray,IdxCat,ItCat> : forward_iterator_helper<self, Node::value_type, Node::difference_type, const Node::value_type*, const Node::value_type&> {
    using node_type = Node; using node_base_type = Node::base_type;
    ctor(){}  explicit ctor(Node* node_): node{node_}{}
    const Node::value_type& operator*() const { return node->value(); }
    self& operator++() { Node::increment(node); return *this; }
    void serialize<Archive>(Archive& ar, unsigned ver) { core::split_member(ar, *this, ver); }
    void save<Archive>(Archive& ar, unsigned) const; void load<Archive>(Archive& ar, unsigned); // make_nvp("pointer")
    Node* get_node() const { return node; }
private: Node* node;
};
bool operator==<N,B,IdxCat,ItCat>(const hashed_index_iterator<N,B,IdxCat,ItCat>& x, const hashed_index_iterator<N,B,IdxCat,ItCat>& y);

struct hashed_index_base_node_impl<Alloc> {
    using base_allocator=rebind_alloc_for<Alloc,self>::type;
    using node_allocator=rebind_alloc_for<Alloc,hashed_index_node_impl<Alloc>>::type;
    using {base|node}_alloc_traits=allocator_traits<{base|node}_allocator>;
    using <const>_base_pointer = base_alloc_traits::<const>_pointer;
    using <const>_pointer = node_alloc_traits::<const>_pointer;
    using difference_type = node_alloc_traits::difference_type;
    pointer& prior(); pointer prior() const;
private: pointer prior_;
};
struct hashed_index_node_impl<Alloc> : hashed_index_base_node_impl<Alloc> {
    base_pointer& next(); base_pointer next() const;
    static pointer pointer_from(base_pointer x); static base_pointer base_pointer_from(pointer x);
private: base_pointer next_;
};
struct default_assigner { void operator()<T>(T& x, const& val) { x=val; } };
struct unlink_undo_assigner<Node> {
  using <base>_pointer = Node::<base>_pointer;
  void operator()(<base>_pointer& x, <base>_pointer& val);
  void operator()(); // undo
  struct <base>_pointer_track { <base>_pointer* x; <base>_pointer val; };
  pointer_track pointer_trakcs[3]; int pointer_track_count;
  base_pointer_track base_pointer_trakcs[2]; int base_pointer_track_count;
};

struct hashed_unique_tag{}; struct hashed_non_unique_tag{};
struct hashed_index_node_alg<Node,hashed_unique_tag> {
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
struct hashed_index_node_alg<Node,hashed_non_unique_tag> {
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
struct hashed_index_node_trampoline<Super> : hashed_index_node_impl<rebind_alloc_for<Super::allocator_type,char>::type> {
  using impl_type = base;
};
struct hashed_index_node<Super> : Super, hashed_index_node_trampoline<Super> {
  using <const>_impl_<base>_pointer = base::<const>_<base>_poniter;
  struct node_alg<Category> { using type = hashed_index_node_alg<impl_type,Category>; };
  <const>_impl_pointer impl() <const>;
  static <const> self* from_impl(<const>_impl_pointer x);
  static void increment_<local> <Category>(self*& x);
};

struct header_holder<NodeTypePtr,Final> : noncopyable {
  ctor(); ~dtor(); // allocate_node/deallocate_node
  NodeTypePtr member;
};

struct index_access_sequence_terminal{ctor(void*){}};
struct index_access_sequence_normal<MICont,n> {
  MICont* p; nth_index<MICont,n>::type& get(); index_access_sequence<MICont,n+1> next();
};
struct index_access_sequence_base<MICont,n>
  : mpl::if_c<(n<mpl::size<MICont::index_type_list>::type::value),
    index_access_sequence_normal<MICont,n>, index_access_sequence_terminal> {};
struct index_access_sequence<MICont,n> : index_access_sequence_base<MICont,n>::type {};

struct lvalue_tag{}; struct rvalue_tag{}; struct emplaced_tag{};

class index_base<Value,ISpecList,Alloc> {
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

struct index_loader<Node,FinalNode,Alloc> : noncopyable {
  ctor(const Alloc& al, size_t size);
  void add<Ar>(Node* node, Ar& ar, unsigned);
  void add_track<Ar>(Node* node, Ar& ar, unsigned);
  void load<Rearranger,Ar>(Rearranger r, Ar& ar, unsigned) const;
private: auto_space<Node*,Alloc> spc; size_t size_,n; mutable bool sorted;
};
struct index_matcher::entry {
  ctor(void* node_, size_t pos_=0) : node{node_}, pos{pos_}{}
  void* node; size_t pos; entry* previous; bool ordered;
  struct less_by_node { bool operator()(const entry& x, const entry& y) const; };// node
  size_t pile_top; entry* pile_top_entry;
  struct less_by_pile_top { bool operator()(const entry& x, const entry& y) const; }; // pile_top
};
struct index_matcher::algorithm_base<Alloc> : noncopyable {
protected: ctor(const Alloc& al, size_t size);
  void add(void* node);
  void begin_algorithm() const;
  void add_node_to_algorithm(void* node) const;
  void finish_algorithm() const;
  bool is_ordered(void* node) const;
private: auto_space<entry,Alloc> spc; size_t size_,n_; mutable bool sorted; mutable size_t num_piles;
};
struct index_matcher::algorithm<Node,Alloc> : private algorithm_base<Alloc> {
  ctor(const Alloc& al, size_t size);
  void add(Node* node);
  void execute<IdxIt>(IdxIt first, IdxIt last) const;
  bool is_ordered(Node* node) const;
};
struct index_saver<Node,Alloc> : noncopyable {
  ctor(const Alloc& al, size_t size);
  void add<Ar>(Node* node, Ar& ar, unsigned);
  void add_track<Ar>(Node* node, Ar& ar, unsigned);
  void save<IdxIt,Ar>(IdxIt first, IdxIt last, Ar& ar, unsigned) const;
private: index_matcher::algorithm<Node,Alloc> alg;
};

struct pod_value_holder<Value> { using space = aligned_storage<sizeof(Value),alingment_of<Value>::value>::type; };
struct index_node_base<Value,Alloc>: private pod_value_holder<Value> {
  using base_type = self; using value_type = Value; using allocator_type = Alloc;
  <const> value_type = value() <const>;
  static self* from_value(const value_type* p);
private: void serialize<Ar>(Ar&, unsigned);
};
Node* node_from_value<Node,Value>(const Value* p);
void load_construct_data<Ar,Value,Alloc>(Ar&, index_node_base<Value,Alloc>*, unsigned) { throw_exception(bad_archive_exception()); };

struct invalidate_iterators
{ using iterator=void; self& get() {return *this;} self& next() { return *this;} };

struct is_index_list<T> { constexpr static bool value=mpl::is_sequence<T>::value && !mpl::empty<T>::value; };
struct is_transparent<F,A1,A2>: mpl_true{};
struct is_transparent_class<F,A1,A2>; struct is_transparent_function<F,A1,A2>;
struct is_transparent<F,A1,A2> requires (is_class<F>&&!is_final<F>): is_transparent_class<F,A1,A2>{};
struct is_transparent<F,A1,A2> requires (is_function<remove_pointer_t<F>>): is_transparent_function<remove_pointer_t<F>,A1,A2>{};

struct iter_adaptor_access {
    static C::reference dereference<C>(const C& x) { return x.dereference(); }
    static bool equal<C>(const C& x, const C& y) { return x.equal(y); }
    static void increment<C>(C& x) { x.increment(); }
    static void decrement<C>(C& x) { x.decrement(); }
    static void advance<C>(C& x, C::difference_type n) { x.advance(n); }
    static C::difference_type distance_to<C>(const C& x, const C& y) { return x.distance_to(y); }
};
struct forward_iter_adaptor_base<D,Base> : forward_iterator_helper<D,Base::value_type,Base::difference_type,Base::pointer,Base::reference> {
    reference operator*() const;
    friend bool operator==(const D& x, const D& y);
    D& operator++();
};
struct iter_adaptor_selector<std::forward_iterator_tag> { struct apply<D,B> { using type = forward_iter_adaptor_base<D,B>; }; };
struct bidirectional_iter_adaptor_base<D,Base> : bidirectional_iterator_helper<D,Base::value_type,Base::difference_type,Base::pointer,Base::reference> {
    reference operator*() const;
    friend bool operator==(const D& x, const D& y);
    D& operator++(); D& operator++();
};
struct iter_adaptor_selector<std::bidirectional_iterator_tag> { struct apply<D,B> { using type = bidirectional_iter_adaptor_base<D,B>; }; };
struct random_access_iter_adaptor_base<D,Base> : random_access_iterator_helper<D,Base::value_type,Base::difference_type,Base::pointer,Base::reference> {
    reference operator*() const;
    friend bool operator==(const D& x, const D& y);     friend bool operator<(const D& x, const D& y);
    D& operator++(); D& operator++();
    D& operator+=(difference_type n); D& operator-=(difference_type n);
    friend difference_type operator-(const D& x, const D& y);
};
struct iter_adaptor_selector<std::random_access_iterator_tag> { struct apply<D,B> { using type = random_access_iter_adaptor_base<D,B>; }; };
struct iter_adaptor_base<D,Base> {
    using type=mpl::apply<iter_adaptor_selector<Base::iterator_category>,D,Base>::type;
};
struct iter_adaptor<D,Base> : public iter_adaptor_base<D,Base>::type {
protected: ctor(); explicit ctor(const Base& b_);
    <const> Base& base_reference() <const>;
private: Base b;
};

struct modify_key_adaptor<Fun,Value,KeyFromValue> { void operator()(Value& x); private: Fun f; KeyFromValue kfv; };

struct duplicate_tag_mark{};
struct duplicate_tag_marker { struct apply<MplSet,Tag>{using type = mpl::s_item<if_<has_key<MplSet,Tag>,duplicate_tag_mark,Tag>::type,MplSet>; }; };
struct no_duplicate_tags<TagList> { using aux=mpl::fold<TagList,set0<>,duplicate_tag_marker>; bool value=!mpl::has_key<aux,duplicate_tag_mark>::value; };
struct duplicate_tag_list_marker { struct apply<MplSet,Idx>: mpl::fold<Idx::tag_list,MplSet,duplicate_tag_marker>{}; };
struct no_duplicate_tags_in_index_list<IndexList> { using aux=mpl::fold<IndexList,set0<>,duplicate_tag_list_marker>::type; bool value=!mpl::has_key<aux,duplicate_tag_mark>::value; };

struct node_handle<Node,Alloc> { // not copyable
    using value_type=Node::value_type; using allocator_type=Alloc; using alloc_traits = allocator_traits<Alloc>;
    ctor() noexcept; ctor(self&& x) noexcept; ~dtor; self& operator=(self&& x);
    explicit operator bool() const noexcept;
    [[nodiscard]] bool empty() const noecxept;
    void swap(self& x); friend void swap(self& x, self& y);
private: ctor(Node* node_, const allocator_type& al);
    void release_node();
    <const> allocator_type* allocator_ptr() <const>;
    void move_construct_allocator(self&& x);
    void move_assign_allocator(self&& x);
    void destroy_allocator();
    void delete_node();
    Node* node; aligned_storage<sizeof(Alloc),alingment_of<Alloc>::value>::type space;
};
struct insert_return_type<It,NodeHandle> {
    ctor(It pos, bool ins, NodeHandle&& node_); ctor(self&& x); self& operator=(self&& x); // no copy
    It position; bool inserted; NodeHandle node;
};

struct index_node_applier { struct apply<ISpecIt,Super>{ using type=mpl::deref<ISpecIt>::type::node_class<Super>::type; }; };
struct multi_index_node_type<Value,ISpecList,Alloc> { using type=mpl::reverse_iter_fold<ISpecList,index_node_base<Value,Alloc>,mpl::bind<index_node_applier,_2,_1>>::type; };

struct index_args_default_compare<KeyFromValue> { using type = std::less<KeyFromValue::result_type>; };
struct ordered_index_args<A1,A2,A3> { using full_form=is_tag<A1>;
    using tag_list_type = mpl::if_<full_form,A1,tag<>>::type;
    using key_from_value_type = mpl::if_<full_form,A2,A1>::type;
    using supplied_compare_type = mpl::if_<full_form,A3,A2>::type;
    using compare_type = mpl::eval_if<is_na<supplied_compare_type>, index_args_default_compare<key_from_value_type>,identity<supplied_compare_type>>::type;
};

struct ordered_unique_tag{}; struct ordered_non_unique_tag{};
class ordered_index_impl<KeyFromValue,Compare,SuperMeta,TagList,Category,AugmentPolicy> : SuperMeta::type {
protected: using index_node_type=ordered_index_node<AugmentPolicy,base::index_node_type>;
    using node_impl_type=index_node_type::impl_type; using node_impl_pointer=node_impl_type::pointer;
public: using key_type=KeyFromValue::result_type; using value_type=index_node_type::value_type;
    using key_from_value=KeyFromValue; using key_compare=Compare; using value_compare=value_comparison<value_type,KeyFromValue,Compare>;
    using ctor_args=tuple<key_from_value,key_compare>;
    using allocator_type = base::final_allocator_type; using alloc_traits=allocator_traits<allocator_type>;
    using <const>_reference = <const> value_type&;
    using iterator=bidir_node_iterator<index_node_type>; using const_iterator=iterator;
    using size_type=alloc_traits::size_type; using difference_type=alloc_traits::difference_type;
    using <const>_pointer=alloc_traits::<const>_pointer;
    using <const>_reverse_iterator=reverse_iterator<<const>_iterator>;
    using node_type = base::final_node_handle_type;
    using insert_return_type = insert_return_type<iterator,node_type>;
    using tag_list=TagList;
protected: using ctor_args_list=tuples::cons<ctor_args,base::ctor_args_list>;
    using index_type_list = mpl::push_front<base::index_type_list,ordered_index<__>>::type;
    using <const>_iterator_type_list = mpl::push_front<base::<const>_iterator_type_list, <const>_itreator>:type;
    using value_param_type = call_traits<value_type>::param_type; using key_param_type = call_traits<key_type>::param_type;
    using pair_return_type = std::pair<iterator,bool>;
public:
    allocator_type get_allocator() const noexcept;
    <const>_iterator {begin|end} <const> noexcept; const_iterator c{begin|end} const noexcept;
    <const>_reverse_iterator r{begin|end} <const> noexcept;
    <const>_iterator iterator_to(const value_type& x) <const>;
    bool empty() const noexcept; size_type size() const noexcept; size_type max_size() const noexcept;

    std::pair<iterator,bool> emplace<...Args>(Args&&... args);
    iterator emplace_hint<...Args>(iterator position, Args&&... args);
    std::pair<iterator,bool> insert(value_type {const&|&&} x);
    iterator insert(iterator p, value_type {const&|&&} x);
    void insert<InIt>(InIt first, InIt last);
    void insert(std::initializer_list<value_type> list);
    insert_return_type insert(node_type&& nh);
    iterator insert(const_iterator p, node_type&& nh);
    node_type extract(const_iterator p); node_type extract(key_param_type x);
    iterator erase(iterator p); size_type erase(key_param_type x);
    iterator erase(iterator first, iterator last);
    bool replace(iterator p, value_type {const&|&&} x);
    bool modify<Modifier,[Rollback]>(iterator p, Modifier mod, <Rollback back_>);
    bool modify_key<Modifier,[Rollback]>(iterator p, Modifier mod, <Rollback back_>);
    void swap(ordered_index<__>& x);
    void clear() noexcept;
    void merge<Idx>(Idx {&|&&} x, <Idx::iterator i>, <Idx::iterator last>);

    key_from_value key_extractor() const;
    key_compare key_comp() const; value_compare value_comp() const;
    iterator find<Key2,[Comp2]>(const Key2& x, <const Comp2& comp>) const;
    size_type count<Key2,[Comp2]>(const Key2& x, <const Comp2& comp>) const;
    bool contains<Key2,[Comp2]>(const Key2& x, <const Comp2& comp>) const;
    iterator {lower|upper}_bound<Key2,[Comp2]>(const Key2& x, <const Comp2& comp>) const;
    std::pair<iterator,iterator> equal_range<Key2,[Comp2]>(const Key2& x, <const Comp2& comp>) const;
    std::pair<iterator,iterator> range<LB,UB>(LB lower, UB upper) const;
protected: ctor(const ctor_args_list& args_list, const allocator_type& al);
    ctor(const self& x); ctor(const self& x, do_not_copy_elements_tag{}); ~dtor();
    void copy_(const self& x, const copy_map_type& map);
    final_node_type* insert_<Variant>(value_param_type v, <index_node_type* p>, final_node_type*& x, Variant variant);
    void extract_<Dst>(index_node_type* x, Dst dst);
    void delete_all_nodes_();
    void clear_();
    void seap_<BoolConstant>(self& x, BoolConstant swap_allocators);
    void swap_elements_(self& x);
    bool replace_<Variant>(value_param_type v, index_node_type* x, Variant variant);
    bool modify_(index_node_type* x); bool modify_rollback_(index_node_type* x);
    bool check_rool_back_(index_node_type* x) const;
    void save_<Ar>(Ar& ar, unsigned ver, const index_saver_type& sm) const;
    void load_<Ar>(Ar& ar, unsigned ver, const index_loader_type& sm);
    bool invariant_() const; void check_invariant_() const;
    index_node_type* header() const; index_node_type* root() const;
    index_node_type* {left|right}most() const;
    key_from_value key; key_compare comp_;
};
struct ordered_index<KeyFromValue,Compare,SuperMeta,TagList,Category,AugmentPolicy>
    : AugmentPolicy::augmented_interface<ordered_index_impl<__>>::type {
    self& operator=(const self& x); self& operator=(std::initializer_list<value_type> list);
protected: ctor(const ctor_args_list& args_list, const allocator_type& al);
    ctor(const ordered_index& x, <do_not_copy_elements_tag>);
};
bool operator{==|!=|<|>|>=|<=}<KFV1,C1,SM1,TL1,Cat1,AP1,KFV2,C2,SM2,TL2,Cat2,AP2>
    (const ordered_index<KFV1,C1,SM1,TL1,Cat1,AP1>& x, const ordered_index<KFV2,C2,SM2,TL2,Cat2,AP2>& y);
void swap<KFV,C,SM,TL,Cat,AP>(const ordered_index<KFV,C,SM,TL,Cat,AP>& x, const ordered_index<KFV,C,SM,TL,Cat,AP>& y);

enum ordered_index_color{red,black};
enum ordered_index_side{to_left,to_right};
struct ordred_index_node_traits<AugmentPolicy,Alloc> {
    using allocator = rebind_alloc_for<Alloc,ordred_index_node_impl<AugmentPolicy,Alloc>>::type;
    using alloc_traits = allocator_traits<allocator>;
    using <const>_pointer = alloc_traits::<const>_pointer;
    using difference_type = alloc_traits::difference_type; using size_type = alloc_traits::size_type;
};
struct ordered_index_node_std_base<AugmentPolicy,Alloc> {
    using node_traits = ordred_index_node_traits<AugmentPolicy,Alloc>;
    using node_allocator = node_traits::allocator; using <const>_pointer = node_traits::<const>_pointer;
    using difference_type = node_traits::difference_type; using size_type = node_traits::size_type;
    using color_ref = ordered_index_color&; using parent_ref& = pointer&;
    ordered_index_color& color(); ordered_index_color color() const;
    pointer& {parent|left|right}(); parent {parent|left|rigth}() const;
private: ordered_index_color color_; pointer parent_, left_, right_;
};
struct ordered_index_node_compressed_base<AugmentPolicy,Alloc> {
    using node_traits = ordred_index_node_traits<AugmentPolicy,Alloc>;
    using <const>_pointer = <const> ordered_index_node_impl<AugmentPolicy,Alloc>*;
    using difference_type = node_traits::difference_type; using size_type = node_traits::size_type;
    struct color_ref {
        ctor(uintptr_type* r_); ctor(const self& x);
        self& operator=(ordered_index_color c); self& operator=(const self& x);
        operator ordered_index_color() const;
    private: uintptr_type* r;
    };
    struct parent_ref {
        ctor(uintptr_type* r_); ctor(const self& x);
        self& operator=(pointer c); self& operator=(const self& x);
        operator pointer() const;   pointer operator->()const;
    private: uintptr_type* r;
    };
    color_ref color();  ordered_index_color color() const;
    parent_ref parent(); pointer parent() const;
    pointer& {left|right}(); pointer {left|right}() const;
private: uintptr_type parentcolor_; pointer left_, right_;
};
struct ordered_index_node_impl_base<AP,Al>
    : AugmentPolicy::augmented_node<mpl::if_c<..., ordered_index_node_std_base<AP,Al>, ordered_index_node_compressed_base<AP,Al>>::type>::type {};
struct ordered_index_node_impl<AP,Al> : ordered_index_node_impl_base<AP,Al> {
    static void increment(pointer& x); static void decrement(pointer& x);
    static void rotate_{left|right}(pointer x, parent_ref root);
    static pointer minimum(pointer x); static pointer maximum(pointer x);
    static void rebalance(pointer x, parent_ref root);
    static void link(pointer x, ordered_index_side side, pointer p, pointer header);
    static pointer rebalance_for_extract(pointer z, parent_ref root, pointer& leftmost, pointer& rightmost);
    static void restore(pointer x, pointer p, pointer header);
    static size_t black_count(pointer node, pointer root); // invariant check
};
struct ordered_index_node_trampoline<AP,Super> : ordered_index_node_impl<AP,rebind_alloc_for<Super::allocator_type,char>::type> { using impl_type = base; }
struct ordered_index_node<AP,Super> : Super, ordered_index_node_trampoline<AP,Super> {
    using impl_color_ref = base::color_ref; using impl_parent_ref = base::parent_ref;
    using <const>_impl_pointer = base::<const>_pointer;
    using difference_type = base::difference_type; using size_type = base::size_type;
    impl_color_ref color(); ordered_index_color color() const;
    impl_parent_ref parent(); impl_pointer parent() const;
    impl_pointer& {left|right}(); impl_pointer {left|right}() const;
    <const>_impl_pointer impl() <const>;
    static <const> ordered_index_node* from_impl(<const>_impl_pointer x);
    static void increment(ordered_index_node*& x); static void decrement(ordered_index_node*& x);
};

Node* ordered_index_{find|lower_bound|upper_bound}<Node,KeyFromValue,Key2,Comp2>
    (Node* top, Node* y, const KeyFromValue& key, const Key2& x, const Comp& comp);
std::pair<Node*,Node*> ordered_index_equal_range<Node,KeyFromValue,Key2,Comp2>
    (Node* top, Node* y, const KeyFromValue& key, const Key2& x, const Comp& comp);

struct promotes_1st_arg<F,A1,A2> : mpl::and_<not_<is_transparent<F,A1,A2>>,is_convertible<const A1,A2>,is_transparent<F,A2,A2>>{};
struct promotes_2nd_arg<F,A1,A2> : mpl::and_<not_<is_transparent<F,A1,A2>>,is_convertible<const A2,A1>,is_transparent<F,A1,A1>>{};
RawPointer raw_ptr<RawPointer,Pointer>(Pointer const& p);

class random_access_index_loader_base<Alloc> : noncopyable {
protected: using node_impl_type = random_access_index_node_impl<rebind_alloc_for<Alloc,char>::type>;
    using node_impl_pointer = node_impl_type::pointer; using ptr_array = random_access_index_ptr_array<Alloc>;
    ctor(const Alloc& al, ptr_array& ptrs_); ~dtor();
    void rearrange(node_impl_pointer p, node_impl_pointer x);
private: Alloc al; ptr_array& ptrs; node_impl_pointer header; auto_space<node_impl_pointer,Alloc> prev_spc; bool preprocessed;
};
class random_access_index_loader<Node,Alloc> : private random_access_index_loader_base<Alloc> {
public: ctor(const Alloc& al_, ptr_array& ptrs_);
    void rearrange(Node* p, Node* x);
};
struct random_access_index_node_impl<Alloc> {
    using node_allocator = rebind_alloc_for<Alloc,self>::type;
    using node_alloc_traits = allocator_traits<node_allocator>;
    using <const>_pointer = node_alloc_traits::<const>_pointer;
    using difference_type = node_alloc_traits::difference_type;
    using ptr_allocator = rebind_alloc_for<Alloc,pointer>::type;
    using ptr_alloc_traits = allocator_traits<ptr_allocator>;
    using ptr_pointer = ptr_alloc_traits::pointer;
    ptr_pointer& up(); ptr_pointer up() const;
    static void increment(pointer& x); static void decrement(pointer& x);
    static void advance(pointer& x, difference_type n); static difference_type distance(pointer x, pointer y);
    static void relocate(ptr_pointer pos, ptr_pointer x, <ptr_pointer last>);
    static void extract(ptr_pointer x, ptr_pointer pend);
    static void transfer(ptr_pointer pbegin0, ptr_pointer pend0, ptr_pointer pbegin1);
    static void reverse(ptr_pointer pbegin, ptr_pointer pend);
    static ptr_pointer gather_nulls(ptr_pointer pbegin, ptr_pointer pend, ptr_pointer x);
};
struct random_access_index_node_trampoline<Super> : random_access_index_node_impl<rebind_alloc_for<Super::allocator_type,char>::type>{ using impl_type=base; };
struct random_access_index_node<Super> : Super, random_access_index_node_trampoline<Super> {
    using <const>_impl_pointer = base::<const>_pointer;
    using difference_type = base::difference_type; impl_ptr_pointer = base::ptr_pointer;
    impl_ptr_pointer& up(); impl_ptr_pointer up() const;
    <const>_impl_pointer impl() const;
    static <const> random_access_index_node* from_impl(<const>_impl_pointer x);
    static void increment(random_access_index_node*& x); static void decrement(random_access_index_node*& x);
    static void advance(random_access_index_node*& x, difference_type n);
    static difference_type distance(random_access_index_node* x, random_access_index_node* y);
};
Node* random_access_index_remove<Node,Alloc,Pred>(random_access_index_ptr_array<Alloc>& ptrs, Pred pred);
Node* random_access_index_unique<Node,Alloc,BPred>(random_access_index_ptr_array<Alloc>& ptrs, BPred binary_pred);
void random_access_index_inplace_merge<Node,Alloc,Comp>(const Alloc& al, random_access_index_ptr_array<Alloc>& ptrs, Node::impl_ptr_pointer first1, Comp comp);
struct random_access_index_sort_compare<Node,Comp> {
    using first_argument_type = Node::impl_pointer; using second_argument_type = Node::impl_pointer; using result_type = bool;
    bool operator()(Node::impl_pointer x, Node::impl_pointer y) const;
private: Comp comp;
};
void random_access_index_sort<Node,Alloc,Comp>(const Alloc& al, random_access_index_ptr_array<Alloc>& ptrs, Comp comp);
struct random_access_index_ptr_array<Alloc> : noncopyable {
    using node_impl_type = random_access_index_node_impl<rebind_alloc_for<Alloc,char>::type>;
    using value_type = node_impl_type::pointer;
    using value_allocator = rebind_alloc_for<Alloc,value_type>::type; using alloc_traits = allocator_traits<value_allocator>;
    using pointer = alloc_traits::pointer; using size_type = alloc_traits::size_type;
    ctor(const Alloc& al, value_type end_, size_type sz);
    size_type size() const; size_type capacity() const;
    void room_for_one(); void reserve(size_type c); void shrink_to_fit();
    pointer {begin|end}() const; pointer at(size_type n) const;
    void push_back(value_type x); void erase(value_type x); void clear();
    void swap(self& x); void swap<BoolConstant>(self& x, BoolConstant swap_allocators);
private: size_type size_, capacity_; auto_space<value_type,Alloc> spc;
};
void swap<Alloc>(random_access_index_ptr_array<Alloc>& x, random_access_index_ptr_array<Alloc>& y);

struct rnd_node_iterator<Node> : public random_access_iterator_helper<rnd_node_iterator<Node>,
    Node::value_type, Node::difference_type, const Node::value_type*, const Node::value_type&> {
    ctor(); explicit ctor(Node* node_);
    const Node::value_type& operator*() const;
    rnd_node_iterator& operator{++|--}();
    rnd_node_iterator& operator{+=|-=}(Node::difference_type n);
    void serialize<Ar>(Ar& ar, unsigned ver);
    void save<Ar>(Ar& ar, unsigned) const; void load<Ar>(Ar& ar, unsigned); 
    Node* get_node() const;
private: Node* node;
};
bool operator{==|<}<Node>(const rnd_node_iterator<Node>& x, const rnd_node_iterator<Node>& y);
Node::difference_type operator-<Node>(const rnd_node_iterator<Node>& x, const rnd_node_iterator<Node>& y);

struct ranked_node_size_type<Pointer>{ using type = pointer_traits<Pointer>::element_type::size_type; };
ranked_node_size_type<Pointer>::type ranked_node_size<Poniter>(Pointer x);
Pointer ranked_index_nth<Pointer>(ranked_node_size_type<Pointer>::type n, Pointer end_);
ranked_node_size_type<Pointer>::type ranked_index_rank<Pointer>(Pointer x, Pointer end_);
Node::size_type ranked_index_{find|lower_bound|upper_bound}_rank<Node,KeyFromValue,Key2,Comp2>
    (Node* top, Node* y, const KeyFromValue& key, const Key2& x, const Comp& comp);
std::pair<Node::size_type,Node::size_type> ranked_index_equal_range_rank<Node,KeyFromValue,Key2,Comp2>
    (Node* top, Node* y, const KeyFromValue& key, const Key2& x, const Comp& comp);

struct scope_guard_impl_base { // no copy assign
    ctor(); void dismiss() const; void touch() const{}
protected: ~dtor(){}; ctor(const self& other);
    static void safe_execute<J>(J& j);
    mutable bool dismissed_;
};
using scope_guard = const scope_guard_impl_base&;
struct null_guard : scope_guard_impl_base { ctor<...Ts>(const Ts&...);};
struct null_guard_return<cond,T> { using type = mpl::if_c<cond,T,null_guard>::type; };
struct scope_guard_implN<F,...P> : public scope_guard_impl_base {
    ctor(F fun, P ...p); ~dtor(); void execute() { fun_(p...); }
protected: F fun_; const P...p;
};
scope_guard_implN<F,P...> make_guard<F,...P>(F fun, P...p);
null_guard_return<cond,scope_guard_implN<F,P...>> make_guard_if_c<cond,F,...P>(F fun, P...p);
null_guard_return<Cond::value,scope_guard_implN<F,P...>> make_guard_if<Cond,F,...P>(F fun, P...p);
struct obj_scope_guard_implN<Obj,MemFun,...P> : public scope_guard_impl_base {
    ctor(Obj& obj, MemFun mem_fun, P...p); ~dtor(); void execute();
protected: Obj& obj_; MemFun mem_fun_; const P...p;
};
obj_scope_guard_implN<Obj,MemFun,P...> make_guard<Obj,MemFun,...P>(Obj& obj, MemFun mem_fun, P...p);
null_guard_return<cond,obj_scope_guard_implN<Obj,MemFun,P...>> make_obj_guard_if_c<cond,Obj,MemFun,...P>(Obj& obj, MemFun mem_fun, P...p);
null_guard_return<Cond::value,obj_scope_guard_implN<Obj,MemFun,P...>> make_obj_guard_if<Cond,Obj,MemFun,...P>(Obj& obj, MemFun mem_fun, P...p);

struct scoped_bilock<Mutex> : noncopyable {
    ctor(Mutex& mutex1, Mutex& mutex2); ~dtor();
private: using sl = Mutex::scoped_lock;
    bool mutex_eq; aligned_storage<sizeof(sl), alingment_of<sl>::value>::type lock1, lock2;
};

struct sequenced_index_node_impl<Alloc> {
    using node_allocator = rebind_alloc_for<Alloc,self>::type; using alloc_traits = allocator_traits<node_allocator>;
    using <const>_pointer = alloc_traits::<const>_pointer; using difference_type = alloc_traits::difference_type;
    pointer& {prior|next}(); pointer {prior|next}() const;
    static void increment(pointer& x); static void decrement(pointer& x);
    static void link(pointer x, pointer header); static void unlink(pointer x);
    static void relink(pointer p, pointer x, <pointer y>);
    static void reverse(pointer header);
    static void swap(pointer x, pointer y);
private: pointer prior_, next_;
};
struct sequenced_index_node_trampoline<Super> : sequenced_index_node_impl<rebind_alloc_for<Super::allocator_type,char>::type>{using impl_type=base;};
struct sequenced_index_node<Super> : Super, sequenced_index_node_trampoline<Super> {
    using <const>_impl_pointer = base::<const>_pointer; using difference_type = base::difference_type;
    impl_pointer& {prior|next}(); impl_pointer {prior|next}() const;
    <const>_impl_pointer impl() const;
    static <const> sequenced_index_node* from_impl(<const>_impl_pointer x);
    static void increment(sequenced_index_node*& x); static void decrement(sequenced_index_node*& x);
};

void sequenced_index_remove<SeqIdx,Pred>(SeqIdx& x, Pred pred);
void sequenced_index_unique<SeqIdx,BPred>(SeqIdx& x, BPred binary_pred);
void sequenced_index_merge<SeqIdx,Comp>(SeqIdx& x, SeqIdx& y, Comp comp);
void sequenced_index_collate<Node,Comp>(Node::impl_type* x, Node::impl_type* y, Comp comp);
static constexpr size_t sequenced_index_sort_max_fill = std::numeric_limits<size_t>::digits+1;
void sequenced_index_sort<Node,Comp>(Node* header, Comp comp);

struct serialization_version<T> {
    ctor(); self& operator=(unsigned x); operator unsigned int() const;
private: unsigned int value; // = ver
    void serialize<Ar>(Ar& ar, unsigned ver);
    void save<Ar>(Ar&, unsigned) const{}; void load<Ar>(Ar&, unsigned ver);
};
struct serialization::version<serialization_version<T>> { int value=version<T>::value; };

struct uintptr_candidates<n> {using type=unsigned int}; // -1: uint, 0: uint, 1: ushort, 2: ulong, 3: ulonglong, 4: uint64
struct uintptr_aux { int index=/**/; bool has_uintptr_type=(index>=0); using type=uintptr_candidates<index>::type; };
using has_uintptr_type = mpl::bool_<uintptr_aux::has_uintptr_type>;
using uintptr_type=uintptr_aux::type;

class unbounded_helper{ctor();ctor(const self); friend self unbounded(self); };
using unbounded_type = unbounded_helper(*)(unbounded_helper);
unbounded_helper unbounded(unbounded_helper) { return{}; }
struct non_unbounded_tag{}; struct lower_unbounded_tag{}; struct upper_unbounded_tag{}; struct both_unbounded_tag{};

struct value_comparison<Value,KeyFromValue,Compare> {
    bool operator()(call_traits<Value>::param_type x, call_traits<Value>::param_type y) const;
private; KeyFromValue key; Compare comp;
};
```

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
