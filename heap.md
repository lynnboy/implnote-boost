# Boost.Heap

* lib: `boost/libs/heap`
* repo: `boostorg/heap`
* commit: `3b22808`, 2025-09-06

------
### API

------
#### Concepts

```c++
concept PriorityQueue<C> = ForwardContainer<C> && Assignable<C::value_type> && Container<C> && EqualityComparable<C> && Comparable<C> &&
  requires (C c, C c2, C::allocator_type a, C::value_type v, C::value_compare cmp) {
    typename C::<const>_iterator; typename C::allocator_type;
    typename C::value_compare; typename C::value_type; typename C::const_reference;
    c.swap(c2); c.clear(); a = c.get_allocator(); c.push(v); v = c.top(); c.pop; cmp = c.value_comp();
    {C::has_ordered_iterators}->bool; {C::is_mergable}->bool; {C::is_stable}->bool;
};
concept MergablePriorityQueue<C> = PriorityQueue<C> && requires(C c, C c2) { c.merge(c2); };
concept MutablePriorityQueue<C> = PriorityQueue<C> && Assignable<MutablePriorityQueue::handle_type> &&
  requires(C c, C::value_type v, C::handle_type h, C::handle_type h2) {
    {c.push(v)}->C::handle_type;
    c.update(h, v); c.increase(h, v); c.decrease(h, v); c.update(h); c.increase(h); c.decrease(h);
    {h==h2}->bool; {h!=h2}->bool; h2=h;
};
```

------
#### Heap Algorithm

```c++
struct detail::heap_merge_emulate<H1,H2> {
    struct dummy_reserver{static void reserve(H1&, size_t){}};
    struct reserver{static void reserve(H1& lhs, size_t required_size){lhs.reserve(required_size);}};
    using space_reserver = std::conditional_t<H1::has_reserve, reserver, dummy_reserver>;
    static void merge(H1& lhs, H2& rhs) {
        if (H1::constant_time_size && H2::constant_time_size)
        { if (H1::has_reserve) space_reserver::reserve(lhs, lhs.size()+rhs.size()); }
        while(!rhs.empty()) { lhs.push(rhs.top()); rhs.pop(); }
        lhs.set_stability_count(std::max(lhs.get_stability_count(), rhs.get_stability_count())); rhs.set_stability_count(0);
    }
};
struct detail::heap_merge_same_mergeable<H> { static void merge(H& lhs, H& rhs) { lhs.merge(rhs); } };
struct detail::heap_merge_same<H> {
    using heap_merger = std::conditional_t<H::is_mergable, heap_merge_same_mergeable<H>, heap_merge_emulate<H,H>>;
    static void merge(H& lhs, H& rhs) { heap_merger::merge(lhs, rhs); }
};
void heap_merge<H1,H2>(H1& lhs, H2& rhs) requires PriorityQueue<H1> && PriorityQueue<H2> && same_as<H1::value_compare,H2::value_compare> {
    using heap_merger = std::conditional_t<std::is_same_v<H1,H2>, heap_merge_same<H1>, heap_merge_emulate<H1,H2>>;
    heap_merger::merge(lhs, rhs);
}
```

------
#### Common Parts

```c++
struct allocator<T> : parameter::template_keyword<struct tag::allocator,T> {};
struct compare<T> : parameter::template_keyword<struct tag::compare,T> {};
struct stable<b> : parameter::template_keyword<struct tag::stable, std::bool_constant<b>> {};
struct mutable_<b> : parameter::template_keyword<struct tag::mutable_, std::bool_constant<b>> {};
struct constant_time_size<b> : parameter::template_keyword<struct tag::constant_time_size, std::bool_constant<b>> {};
struct store_parent_pointer<b> : parameter::template_keyword<struct tag::store_parent_pointer, std::bool_constant<b>> {};
struct arity<i> : parameter::template_keyword<struct tag::arity, std::integral_constant<int,i>> {};
struct objects_per_page<i> : parameter::template_keyword<struct tag::objects_per_page, std::integral_constant<int,i>> {};
struct stability_counter_type<T> : parameter::template_keyword<struct tag::stability_counter_type,T> {};

struct detail::has_arg<BoundArgs,TagType> {
    using type = parameter::binding<BoundArgs,TagType,void>;
    static const bool value = !std::is_void_v<type>;
};
struct detail::extract_stable<BoundArgs> {
    static const bool has_stable = has_arg<BoundArgs,tag::stable>::value;
    using stable_t = std::conditional_t<has_stable, has_arg<BoundArgs,tag::stable>::type, std::false_type>;
    static const bool value = stable_t::value;
}
struct detail::extract_mutable<BoundArgs> {
    static const bool has_mutable = has_arg<BoundArgs,tag::mutable_>::value;
    using mutable_t = std::conditional_t<has_stable, has_arg<BoundArgs,tag::mutable_>::type, std::false_type>;
    static const bool value = mutable_t::value;
}

struct detail::log2<Int> { Int operator()(Int value) { Int l=0; while((value>>l) > 1)++l; return l; } };
Int log2<Int>(Int value) { return log2<Int>{}(value); }

class detail::ordered_adaptor_iterator<Value,InternalT,Cont,Alloc,Comp,Dispatcher>
    : public iterator_facade<self,Value,forward_traversal_tag>, Dispatcher {
    struct compare_by_heap_value : Comp {
        ctor(const Cont* container, Comp const& cmp) : base{cmp}, container{container} {}
        bool operator()(size_t lhs, size_t rhs)
        { return base::operator(Dispatcher::get_internal_value(container, lhs), Dispatcher::get_internal_value(container, rhs)); }
    };
    const Cont* container{nullptr}; size_t current_index{std::numeric_limits<size_t>::max()};
    std::priority_queue<size_t, std::vector<size_t, allocator_rebind<Alloc,size_t>::type>, compare_by_heap_value> unvisited_nodes;
    bool equal(ordered_adaptor_iterator const& rhs) const
    { if (current_index!=rhs.current_index) return false; if (container != rhs.container) return false; return true; }
    void increment()
    { if (unvisited_nodes.empty()) current_index = Dispatcher::max_index(container) + 1;
      else { current_index = unvisited_nodes.top(); unvisited_nodes.pop(); discover_nodes(current_index); } }
    Value const& dereference() const { return Dispatcher::get_value(Dispatcher::get_internal_value(container, current_index)); }
    void discover_nodes(size_t index) {
        if (Dispatcher::is_leaf(container, index)) return;
        auto child_range = Dispatcher::get_child_nodes(container, index);
        for (size_t i = child_range.first; i <= child_range.second; ++i) unvisited_nodes.push(i);
    }
public: ctor() : unvisited_nodes{compare_by_heap_value{nullptr,Comp{}}} {}
    ctor(const Cont* container, Comp con& cmp) : container{container}, current_index{container->size()}, unvisited_nodes{compare_by_heap_value{nullptr,cmp}} {}
    ctor(size_t initial_index, const Cont* container, Comp con& cmp) : container{container}, current_index{initial_index}, unvisited_nodes{compare_by_heap_value{nullptr,cmp}} { discover_nodes(initial_index); }
};
struct detail::identity<T> { T<const>& operator()(T<const>& x) const noexcept { return x; } };
struct detail::dereferencer<Node> { Node* operator()<It>(It const& it) { return (Node*)(*it); } };
struct detail::pointer_to_reference<Node> { const Node* operator()<It>(It const& it) { return (const Node*)(&*it); } };
struct detail::unordered_tree_iterator_storage<Handle,Alloc,Comp> {
    std::vector<Handle, allocator_rebind<Alloc,Handle>::type> data_;
    ctor(Comp const&) {}
    void push(Handle h) { data_.push_back(h); }
    Handle const& top() { return data_.back(); }
    void pop() { data_.pop_back(); }
    bool empty() const { return data_.empty(); }
};
struct detail::ordered_tree_iterator_storage<Value,Handle,Alloc,Comp,ValueExtractor> : ValueExtractor {
    struct compare_values_by_handle : ValueExtractor, Comp {
        ctor(Comp const& cmp) : Comp{cmp} {}
        bool operator()(Handle const& lhs, Handle const& rhs) const {
            Value const& lval = ValueExtractor::operator()(lhs->value), rval = ValueExtractor::operator()(rhs->value);
            return Comp::operator()(lval, rval);
        }
    };
    std::priority_queue<Handle,std::vector<Handle, allocator_rebind<Alloc,Handle>::type>, compare_values_by_handle> data_;
    ctor(Value const& cmp) : data_{compare_values_by_handle{cmp}} {}
    void push(Handle h) { data_.push(h); }
    Handle const& top() { return data_.top(); }
    void pop() { data_.pop(); }
    bool empty() const noexcept { return data_.empty(); }
};
class detail::tree_iterator<Node,Value,Alloc=std::allocator<Node>,ValueExtractor=identity<Node::value_type>,PointerExtractor=dereferencer<Node>,
    check_null_pointer=false,ordered_iterator=false,Comp=std::less<Value>>
    : public iterator_adaptor<self, const Node*, Value, forward_traversal_tag>, ValueExtractor, PointerExtractor {
    unvisited_node_container unvisited_nodes;
    void increment() {
        if (unvisited_nodes.empty()) base::base_reference() = nullptr;
        else { const Node* next = unvisited_nodes.top(); unvisited_nodes.pop();
            discover_nodes(next); base::base_reference() = next;
        }
    }
    Value const& dereference() const { return ValueExtractor::operator()(base::base_reference()->value); }
    void discover_nodes(const Node* n) {
        for (auto it = n->children.begin(); it != n->children.end(); ++it)
        { const Node* n = PointerExtractor::operator()(it);
            if (check_null_pointer && n==nullptr) continue;
            unvisited_nodes.push(n); }
    }
public: ctor() : base{0}, unvisited_nodes{Comp{}} {}
    ctor(Comp const& cmp) : base{0}, unvisited_nodes{cmp} {}
    ctor(const Node* it, Comp const& cmp) : base{it}, unvisited_nodes{cmp} { if (it) discover_nodes(it); }
    ctor<NodePtrIt>(NodePtrIt begin, NodePtrIt end, const Node* top_node, Comp const& cmp) : base{0}, unvisited_nodes{cmp} {
        if (begin == end) return;
        self::base_reference() = top_node; discover_nodes(top_node);
        for (auto it = begin; it != end; ++it) if (current_node != top_node) unvisited_nodes.push((const Node*)(&*it));
    }
    bool operator!=(self const& rhs) const { return base::base() != rhs::base(); }
    bool operator==(self const& rhs) const { return !operator!=(rhs); }
    const Node* get_node() const { return base::base_reference(); }
};

struct detail::list_iterator_converter<Node,NodeList> {
    NodeList::const_iterator operator()(const Node* node) { return NodeList::s_iterator_to(*node); }
    Node* operator()(NodeList::const_iterator it) { return (Node*)(&*it); }
};
class detail::recursive_tree_iterator<Node,NodeIt,Value,ValueExtractor=identity<Node::value_type>,ItConverter=identity<NodeIt>>
    : public iterator_adaptor<self, NodeIt, Value const, bidirectional_traversal_tag>, ValueExtractor, ItConverter {
public: ctor() : base{0} {}  explicit ctor(NodeIt const& it) : base{it} {}
    void increment() {
        NodeIt next = base::base_reference(); const Node* n = get_node(next);
        if (n->children.empty()) {
            const Node* parent = get_node(next)->get_parent(); ++next;
            while (true) { if (parent==nullptr || next!=parent->children.end()) break;
                next = ItConverter::operator()(parent); parent = get_node(next)->get_parent(); ++next; }
        } else next = n->children.begin();
        base::base_reference() = next;
    }
    Value const& dereference() const { return ValueExtractor::operator()(get_node(base::base_reference())->value); }
    static const Node* get_node(NodeIt const& it) { return (const Node*)(&*it); }
    const Node* get_node() const { return get_node(base::base_reference()); }
};

struct detail::heap_node_base<auto_unlink=false> : intrusive::list_base_hook<std::conditional_t<auto_unlink,
    intrusive::link_mode<intrusive::auto_unlink>, intrusive::link_mode<intrusive::safe_link>>> {};
using detail::heap_node_list = intrusive::list<heap_node_base<false>>;
struct detail::nop_disposer{ void operator()<T>(T*) {assert(false);} };
bool detail::is_heap<Node,HeapBase>(const Node* n, HeapBase::value_compare const& cmp) {
    for (auto it = n->children.begin(); it!=n->children.end(); ++it) {
        Node const& this_node = (Node const&)(*it); Node const* child = (Node const*)(&this_node);
        if (cmp(HeapBase::get_value(n->value), HeapBase::get_value(child->value)) || !is_heap<Node,HeapBase>(child, cmp)) return false;
    }
    return true;
}
size_t detail::count_nodes<Node>(const Node* n) { return 1 + count_list_nodes<>(n->children); }
size_t detail::count_list_nodes<Node,List>(List const& node_list) {
    size_t ret = 0;
    for (auto it = node_list.begin(); it != node_list.end(); ++it) ret += count_nodes<>((const Node*)(&*it));
    return ret;
}
void detail::destroy_node<Node>(Node& node) { node.~dtor(); }

class detail::node_cloner<Node,NodeBase,Alloc> { Alloc& allocator;
public: ctor(Alloc& allocator) : allocator{allocator} {}
    Node* operator()(NodeBase const& node)
    { Node* ret = allocator.allocate(1); new (ret) Node((Node const&)(node), allocator); return ret; }
    Node* operator()(NodeBase const& node, Node* parent)
    { Node* ret = allocator.allocate(1); new (ret) Node((Node const&)(node), allocator, parent); return ret; }
};

struct detail::node_disposer<Node,NodeBase,Alloc> { Alloc& alloc_;
    ctor(Alloc& alloc) : alloc_{alloc} {}
    void operator()(NodeBase* base) {
        auto n = allocator_pointer<Alloc>::type(base);
        n->clear_subtree(alloc_); destroy_node(*n); alloc_.deallocate(n, 1);
    }
};

struct detail::heap_node<Value,constant_time_child_size=true> : heap_node_base<!constant_time_child_size> {
    using value_type = Value;
    using child_list = intrusive::list<node_base, intrusive::constant_time_size<constant_time_child_size>>;
    using <const>_child_iterator = child_list::<const>_iterator; using size_type = child_list::size_type;
    Value value; child_list children;
    ctor(Value const& v) : value{v} {}
    ctor<...Args>(Args&&...args) : value{std::forward<Args>(args)...} {}
    ctor(heap_node const& rhs) : value{rhs.value} {}
    ctor<Alloc>(heap_node const& rhs, Alloc& allocator) value{rhs.value}
    { children.clone_from(rhs.children, node_cloner<heap_node, node_base, Alloc>{allocator}, nop_disposer{}); }
    size_type child_count() const { return children.size(); }
    void add_child(heap_node* n) { children.push_back(*n); }
    void clear_subtree<Alloc>(Alloc& alloc) { children.clear_and_dispose(node_disposer<heap_node, node_base, Alloc>{alloc}); }
    void swap_children(heap_node* rhs) { children.swap(rhs->children); }
};

struct detail::parent_pointing_heap_node<Value> : heap_node<Value> {
    class node_cloner<Alloc> { Alloc& allocator; outer* parent_;
    public: ctor(Alloc& allocator, outer* parent) : allocator{allocator}, parent_{parent} {}
        outer* operator()(base::node_base const& node)
        { auto* ret = allocator.allocate(1); new(ret) outer{(outer const&)(node), allocator, parent}; return ret; }
    };
    self* parent;
    ctor(Value const& v) : base{v} {}
    ctor<...Args>(Args&&...args) : base{std::forward<Args>(args)...} {}
    ctor<Alloc>(self const& rhs, Alloc& allocator, self* parent) : base{rhs}, parent{parent}
    { base::children.clone_from(rhs.children, node_cloner<Alloc>{allocator, this}, node_disposer{}); }
    void update_children() { for (auto it = base::children.begin(); it!=base::children.end(); ++it) { ((self*)(&*it))->parent = this; } }
    void remove_from_parent() { parent->children.erase(heap_node_list::s_iterator_to(*this)); parent = nullptr; }
    void add_child(self* n) { n->parent = this; base::add_child(n); }
    <const> self* get_parent() <const> { return parent; }
};

struct detail::marked_heap_node<Value> : parent_pointing_heap_node<Value> {
    bool mark {false};
    ctor(Value const& v) : base{v} {}  ctor<...Args>(Args&&...args) : base{std::forward<Args>(args)...} {}
    <const> self* get_parent() <const> { return (self*)(base::parent); }
};

struct detail::cmp_by_degree<Node> {
    bool operator() <NodeBase> (NodeBase const& left, NodeBase const& right)
    { return (const Node*)(&left)->child_count() < (const Node*)(&right)->child_count(); }
};

Node* detail::find_max_child<List,Node,Cmp>(List const& list, Cmp const& cmp) {
    auto ret = (const Node*)&list.front();
    for (auto it = list.begin(); it != list.end(); ++it) if (cmp(ret->value, (const Node*)(&*it))) ret = currnet;
    return (Node*)ret;
};

bool detail::value_equality<H1,H2>(H1 const& lhs, H2 const& rhs, H1::value_type lval, H2::value_type rval)
{ auto cmp = lhs.value_comp(); return !cmp(lval, rval) && !cmp(rval, lval); }
bool detail::value_compare<H1,H2>(H1 const& lhs, H2 const& rhs, H1::value_type lval, H2::value_type rval)
{ return lhs.value_comp()(lval, rval); }

struct detail::heap_equivalence_copy {
    bool operator()<H1,H2>(H1 const& lhs, H2 const& rhs) requires PriorityQueue<H1> && PriorityQueue<H2> && is_same_v<H1::value_compare, H2::value_compare> {
        if (H1::constant_time_size && H2::constant_time_size) if (lhs.size() != rhs.size()) return false;
        if (lhs.empty() && rhs.empty) return true;
        H1 lhs_copy{lhs}; H2 rhs_copy{rhs};
        while (true) {
            if (!value_equality(lhs_copy, rhs_copy, lhs_copy.top(), rhs_copy.top())) return false;
            lhs_copy.pop(); rhs_copy.pop();
            if (lhs_copy.empty() && rhs_copy.empty()) return true;
            if (lhs_copy.empty() || rhs_copy.empty()) return false;
        }
    }
};
struct detail::heap_equivalence_iteration {
    bool operator()<H1,H2>(H1 const& lhs, H2 const& rhs) requires PriorityQueue<H1> && PriorityQueue<H2> && is_same_v<H1::value_compare, H2::value_compare> {
        if (H1::constant_time_size && H2::constant_time_size) if (lhs.size() != rhs.size()) return false;
        if (lhs.empty() && rhs.empty) return true;
        H1::ordered_iterator it1 = lhs.ordered_begin(), it1_end = lhs.ordered_end();
        H2::ordered_iterator it2 = lhs.ordered_begin(), it2_end = lhs.ordered_end();
        while (true) {
            if (!value_equality(lhs, rhs, *it1, *it2)) return false;
            ++it1; ++it2;
            if (it1 == it1_end && it2 == it2_end) return true;
            if (it1 == it1_end || it2 == it2_end) return false;
        }
    }
};
bool detail::heap_equality<H1,H2>(H1 const& lhs, H2 const& rhs) {
    using equivalence_check = std::conditional_t<H1::has_ordered_iterators && H2::has_ordered_iterators, heap_equivalence_iteration : heap_equivalence_copy>;
    return equivalence_check{}(lhs, rhs);
}

struct detail::heap_compare_iteration {
    bool operator()<H1,H2>(H1 const& lhs, H2 const& rhs) {
        auto left_size = lhs.size(); auto right_size = rhs.size();
        if (left_size < right_size) return true; if (left_size > right_size) return false;
        H1::ordered_iterator it1 = lhs.ordered_begin(), it1_end = lhs.ordered_end();
        H2::ordered_iterator it2 = rhs.ordered_begin(), it2_end = rhs.ordered_end();
        while (true) {
            if (!value_compare(lhs, rhs, *it1, *it2)) return true;
            if (!value_compare(lhs, rhs, *it2, *it1)) return false;
            ++it1; ++it2;
            if (it1 == it1_end && it2 == it2_end) return true;
            if (it1 == it1_end || it2 == it2_end) return false;
        }
    }
};
struct detail::heap_compare_copy {
    bool operator()<H1,H2>(H1 const& lhs, H2 const& rhs) {
        auto left_size = lhs.size(); auto right_size = rhs.size();
        if (left_size < right_size) return true; if (left_size > right_size) return false;
        H1 lhs_copy{lhs}; H2 rhs_copy{rhs};
        while (true) {
            if (value_compare(lhs_copy, rhs_copy, lhs_copy.top(), rhs_copy.top())) return true;
            if (value_compare(lhs_copy, rhs_copy, rhs_copy.top(), lhs_copy.top())) return false;
            lhs_copy.pop(); rhs_copy.pop();
            if (lhs_copy.empty() && rhs_copy.empty()) return false;
        }
    }
};
bool detail::heap_compare<H1,H2>(H1 const& lhs, H2 const& rhs) {
    using compare_check = std::conditional_t<H1::has_ordered_iterators && H2::has_ordered_iterators, heap_compare_iteration : heap_compare_copy>;
    return compare_check{}(lhs, rhs);
}

struct detail::size_holder<constantSize,SizeType> {
    static const bool constant_time_size = constantSize; using size_type = SizeType;
    SizeType size_;
    ctor() noexcept : size_{0} {}
    ctor(self&& rhs) noexcept : size_{rhs.size_}{ rhs.size_=0; }
    ctor(self const& rhs) noexcept : size_{rhs.size_} {}
    self& operator=(self&& rhs) noexcept { size_ = rhs.size_; rhs.size_ = 0; return *this; }
    self& operator=(self const& rhs) noexcept { size_ = rhs.size_; return *this; }
    SizeType get_size() const noexcept { return size_; } void set_size(SizeType size) noexcept { size_ = size; }
    void decrement() noexcept { --size_; } void increment() noexcept { ++size_; }
    void add(SizeType value) noexcept { size_ += value;} void sub(SizeType value) noexcept { size_ -= value; }
    void swap(self& rhs) noexcept { std::swap(size_, rhs.size_); }
};
struct detail::size_holder<false,SizeType> {}; // same members as primary, but no `size_`

struct detail::heap_base<T,Cmp,constant_time_size, StabilityCounterType=uintmax_t, stable=false> : size_holder<constant_time_size, size_t> {
    using stability_counter_type = StabilityCounterType; using value_type = T; using internal_type = T;
    using size_holder_type = size_holder<constant_time_size, size_t>; using value_compare = Cmp; using internal_compare = Cmp;
    static const bool is_stable = stable;
    Cmp cmp_;
    ctor(Cmp const& cmp={}) : cmp_{cmp} {}
    ctor(self&& rhs) noexcept(is_nothrow_move_constructible_v<Cmp>) : base{std::move(rhs)}, cmp_{std::move(rhs.cmp_)} {}
    ctor(self const& rhs) : base{rhs}, cmp_{rhs.value_comp()} {}
    self& operator=(self&& rhs) noexcept(is_nothrow_move_assignable<Cmp>)
    { value_comp_ref() = std::move(rhs.value_comp_ref()); base::operator=(std::move(rhs)); return *this; }
    self& operator=(self const& rhs) { value_comp_ref() = rhs.value_comp(); base::operator=(rhs); return *this; }
    bool operator()(internal_type const& lhs, internal_type const& rhs) const { return value_comp()(lhs, rhs); }
    internal_type make_node(T const& val) { return val; } T&& make_node(T&& val) { return std::forward<T>(val); }
    internal_type make_node<...Args>(Args&&...val) { return internal_type{std::forward<Args>(val)...}; }
    static T <const>& get_value(internal_value <const>& val) <noexcept> { return val; }
    Cmp const& value_comp() const noexcept { return cmp_; }
    Cmp const& get_internal_cmp() const noexcept { return value_comp(); }
    void swap(self& rhs) noexcept(is_nothrow_move_constructible_v<Cmp> && is_nothrow_move_assignable<Cmp>)
    { std::swap(value_comp_ref(), rhs.value_comp_ref()); base::swap(rhs); }
    stability_counter_type get_stability_count() const noexcept { return 0; }
    void set_stability_count(stability_counter_type) noexcept {}
    Cmp& value_comp_ref() { return cmp_; }
};
struct detail::heap_base<T,Cmp,constant_time_size,StabilityCounterType,true> : size_holder<constant_time_size, size_t> {
    using stability_counter_type = StabilityCounterType; using value_type = T;
    struct internal_type { T first; stability_counter_type second;
        ctor<...Args>(stability_counter_type const& cnt, Args&...args) : first{std::forward<Args>(args)...}, second{cnt}{}
        ctor(stability_counter_type const& cnt, T const& value) : first{value}, second{cnt} {}
    };
    using size_holder_type = size_holder<constant_time_size, size_t>; using value_compare = Cmp;
    struct internal_compare : Cmp { ctor(Cmp const& cmp={}): Cmp{cmp} {}
        bool operator()(internal_type const& lhs, internal_type const& rhs) const {
            if (base::operator()(lhs.first, rhs.first)) return true;
            if (base::operator()(rhs.first, lhs.first)) return false;
            return lhs.second > rhs.second;
        }
    };
    Cmp cmp_; stability_counter_type counter_{0}
    ctor(Cmp const& cmp={}) : Cmp{cmp} {}
    ctor(self&& rhs) noexcept(is_nothrow_move_constructible_v<Cmp>) : base{std::move(rhs)}, cmp_{std::move(rhs.cmp_)}, counter_{rhs.counter_} {}
    ctor(self const& rhs) : base{rhs}, cmp_{rhs.value_comp()}, counter_{rhs.counter_} {}
    self& operator=(self&& rhs) noexcept(is_nothrow_move_assignable<Cmp>)
    { value_comp_ref() = std::move(rhs.value_comp_ref()); base::operator=(std::move(rhs)); counter_=rhs.counter_; rhs.counter_=0; return *this; }
    self& operator=(self const& rhs) { value_comp_ref() = rhs.value_comp(); base::operator=(rhs); counter_=rhs.counter_; return *this; }
    bool operator()(internal_type const& lhs, internal_type const& rhs) const { return get_internal_cmp()(lhs, rhs); }
    bool operator()(T const& lhs, T const& rhs) { return value_comp()(lhs, rhs); }
    internal_type make_node(T const& val) { auto count = ++counter_;
        if (counter_ == std::numeric_limits<stability_counter_type>::max()) throw_exception(std::runtime_error{"counter overflow"});
        return internal_type{count, val}; }
    internal_type make_node<...Args>(Args&&...val) { auto count = ++counter_;
        if (counter_ == std::numeric_limits<stability_counter_type>::max()) throw_exception(std::runtime_error{"counter overflow"});
        return internal_type{count, std::forward<Args>(val)...}; }
    static T <const>& get_value(internal_value <const>& val) <noexcept> { return val.first; }
    Cmp const& value_comp() const noexcept { return cmp_; }
    internal_compare get_internal_cmp() const { return {value_comp()}; }
    void swap(self& rhs) noexcept(is_nothrow_move_constructible_v<Cmp> && is_nothrow_move_assignable<Cmp>)
    { std::swap(cmp_, rhs.cmp_); std::swap(counter_, rhs.counter_); base::swap(rhs); }
    stability_counter_type get_stability_count() const { return counter_; }
    void set_stability_count(stability_counter_type new_count) { counter_ = new_count; }
    Cmp& value_comp_ref() { return cmp_; }
};

struct detail::node_handle<NodePtr,Extractor,Ref> {
    NodePtr node_;
    explicit ctor(NodePtr n=0) : node_{n} {}
    reference operator*() const { return Extractor::get_value(node_->value); }
    bool operator==(self const& rhs) const { return node_ == rhs.node_; } // also !=
};
struct detail::value_extractor<Value,InternalT,Extractor> {
    Value const& operator()(InternalT const& data) const { return Extractor::get_value(data); }
};
class detail::stable_heap_iterator<T,ContIt,Extractor> : public iterator_adaptor<self,ContIt,T const, random_access_traversal_tag> {
    T const& dereference() const { return Extractor::get_value(*base::base()); }
public: ctor():base(0){}  explicit ctor(ContIt const& it):base{it}{}
};

struct detail::make_heap_base<T,Parspec,constant_time_size> {
    using compare_argument = parameter::binding<Parspec,tag::compare,std::less<T>>;
    using allocator_argument = parameter::binding<Parspec,tag::allocator,std::allocator<T>>;
    using stability_counter_type = parameter::binding<Parspec,tag::stability_counter_type,uintmax_t>;
    static const bool is_stable = extract_stable<Parspec>::value;
    using type = heap_base<T, compare_argument, constant_time_size, stability_counter_type, is_stable>;
};
struct detail::extract_allocator_types<Alloc> {
    using size_type = boost::allocator_size_type<Alloc>::type; // direference_type, <const>_pointer
    using <const>_reference = Alloc::value_type<const>&;
};

class detail::priority_queue_mutable_wrapper<PQueue> {
    using node_type = std::pair<value_type, size_type>;
    using object_list = std::list<node_type, allocator_rebind<allocator_type, node_type>::type>;
    using <const>_list_iterator = object_list::<const>_iterator;
    using stability_counter_type = PQueue::stability_counter_type;
    using q_type = PQueue::rebind<list_iterator, indirect_cmp, allocator_type, index_updater>::other;
    struct index_updater { static void run<It>(It& it, size_type new_index) { q_type::get_value(it)->second = new_index; } };
    struct indirect_cmp : value_compare {
        ctor(base const& cmp={}) : base{cmp} {}
        bool operator()(const_list_iterator const& lhs, rhs) const { return base::operator()(lhs->first, rhs->first); }
    };

    stability_counter_type get_stability_count() const { return q_.get_stability_count(); }
    void set_stability_count(stability_counter_type new_count) { q_.set_stability_count(new_count); }

protected:  q_type q_; object_list objects;
    ctor(value_compare const& cmp={}) : q_{cmp} {}
    ctor(self const& rhs) : q_{rhs.q_}, objects{rhs.objects} { for (auto it = objects.begin(); it!=objects.end(); ++it) q_.push(it); }
    self& operator=(self const& rhs) {
        q_ = rhs.q_; objects = rhs.objects; q_.clear();
        for (auto it = objects.begin(); it!=objects.end(); ++it) q_.push(it);
        return *this;
    }
    ctor(self&& rhs) : q_(std::move(rhs.q_)) { std::swap(objects, rhs.objects); }
    self& operator(self&& rhs) { q_ = std::move(rhs.q_); objects.clear(); std::swap(objects, rhs.objects); return *this; }

public: using value_type = PQueue::value_type; // size_type, value_compare, allocator_type, <const>_pointer, <const>_reference
    static const bool is_stable = PQueue::is_stable;
    class handle_type { list_iterator iterator;
        explicit ctor(list_iterator const& it) : iterator{it} {}
    public: ctor(){}  ctor(self& const rhs) : iterator{rhs.iterator}{}
        value_type& operator*() const { return iterator->first; }
        self& operator=(self const& rhs) { iterator = rhs.iterator; return *this; }
        bool operator==(self const& rhs) const { return iterator == rhs.iterator; } // and op!=
    };
    struct iterator_base<It> : iterator_adaptor<self, It, value_type const, bidirectional_traversal_tag> {
        ctor() : base{0}{}  explicit ctor<T>(T const& it) : base{it} {}
        value_type const& dereference() const { return base::base()->first; }
        iterator_type get_list_iterator() const { return base::base_reference(); }
    };
    using <const>_iterator = iterator_base<<const>_list_iterator>;
    using difference_type = object_list::difference_type;
    class ordered_iterator : iterator_adaptor<self, const_list_iterator, value_type const, forward_traversal_tag>, q_type::ordered_iterator_dispatcher {
        std::priority_queue<const_list_iterator, std::vector<const_list_iterator, allocator_rebind<allocator_type, const_list_iterator>::type>, indirect_cmp> unvisited_nodes;
        const priority_queue_mutable_wrapper* q_;
        void increment() {
            if (unvisited_nodes.empty()) bas::base_reference() = q_->objects.end();
            else { auto next = unvisited_nodes.top(); unvisited_nodes.pop();
                discover_nodes(next); base::base_reference() = next;
            }
        }
        value_type const& dereference() const { return base::base()->first; }
        void discover_nodes(const_list_iterator current) {
            size_type current_index = current->second; const q_type* q = &(q_->q_);
            if (base::is_leaf(q, current_index)) return;
            auto child_range = base::get_child_nodes(q, current_index);
            for (size_type i = child_range.first; i<=child_range.second; ++i) unvisited_nodes.push(q->get_value(base::get_internal_value(q, i)));
        }
    public: ctor(): base{0}, unvisited_nodes{indirect_cmp{}}, q_{nullptr} {}
        ctor(const priority_queue_mutable_wrapper* q, indirect_cmp const& cmp) : base{0}, unvisited_nodes{cmp}, q_{q} {}
        ctor(const_list_iterator it, const outer* q, indirect_cmp const& cmp)
            : base{it}, unvisited_nodes{cmp}, q_{q} { if (it != q->objects.end()) discover_nodes(it); }
        bool operator!=(ordered_iterator const& rhs) const { return base::base() != rhs.base(); } // and op==
    };
    bool empty() const { return q_.empty(); }
    size_type size() const { return q_.size(); } size_type max_size() const { return objects.max_size(); }
    void clear() { q_.clear(); objects.clear(); }
    allocator_type get_allocator() const { return q_.get_allocator(); }
    void swap(self& rhs) { objects.swap(rhs.objects); q_.swap(rhs.q_); }
    const_reference top() const { return q_.top()->first; }
    handle_type push(value_type const& v) {
        objects.push_front(std::make_pair(v,0));
        list_iterator ret = objects.begin(); q_.push(ret); return handle_type{ret};
    }
    handle_type emplace<...Args>(Args&&...args) {
        objects.push_front(std::make_pair(std::forward<Args>(args)..., 0));
        list_iterator ret = objects.begin(); q_.push(ret); return handle_type{ret};
    }
    void pop() { list_iterator q_top = q_.top(); q_.pop(); objects.erase(q_top); }
    void update(handle_type handle, const_reference v)
    { if (q_.value_comp()(v, handle.iterator->first)) decrease(handle, v); else increase(handle, v); }
    void update(handle_type handle) { q_.update(handle.iterator->second); }
    void increase(handle_type handle, const_reference v) { handle.iterator->first = v; increase(handle); }
    void increase(handle_type handle) { q_.increase(handle.iterator->second); }
    void decrease(handle_type handle, const_reference v) { handle.iterator->first = v; decrease(handle); }
    void decrease(handle_type handle) { q_.decrease(handle.iterator->second); }
    void erase(handle_type handle) { q_.erase(handle.iterator->second); objects.erase(handle.iterator); }
    <const>_iterator begin() <const> { return <const>_iterator{objects.begin()}; }
    <const>_iterator end() <const> { return <const>_iterator{objects.end()}; }
    ordered_iterator ordered_begin() const
    { if (!empty()) return ordered_iterator{q_.top(), this, indirect_cmp{q_.value_comp()}}; else return ordered_end(); }
    ordered_iterator ordered_end() const { return ordered_iterator{ objects.end(), this, indirect_cmp{q_.value_comp()}}; }
    static handle_type s_handle_from_iterator(iterator const& it) { return handle_type{it.get_list_iterator()}; }
    value_compare const& value_comp() const { return q_.value_comp(); }
    void reserve(size_type element_count) { q_.reserve(element_count); }
};
```

------
#### Priority Queue

```c++
using detail::priority_queue_signature = parameter::parameters<optional<tag::allocator>, optional<tag::compare>, optional<tag::stable>, optional<tag::stability_counter_type>>;
class priority_queue<T,A0=parameter::void_,...,A3=parameter::void_>
    : make_heap_base<T,priority_queue_signature::bind<A0,A1,A2,A3>::type, false>::type {
    using heap_base_maker = make_heap_base<T,priority_queue_signature::bind<A0,A1,A2,A3>::type, false>;
    using container_type = std::vector<internal_type, allocator_rebind<heap_base_maker::allocator_argument, internal_type>::type>;
    struct types: extract_allocator_types<heap_base_maker::allocator_argument> {
        using value_compare = heap_base_maker::compare_argument;
        using iterator = stable_heap_iterator<T, container_type::const_iterator, base>; using const_iterator = iterator;
        using allocator_type = container_type::allocator_type;
    };
    container_type q_;
public: using value_type = T;
    using size_type = types::size_type; // difference_type, value_compare, allocator_type, <const>_reference, <const>_pointer, <const>_iterator
    static const bool constant_time_size{true}, has_ordered_iterators{false}, is_mergable{false}, is_stable{heap_base_maker::is_stable}, has_reserve{true};
    explicit ctor(value_compare const& cmp={}) : base{cmp} {}
    explicit ctor(allocator_type const& alloc) : q_{alloc} {}
    ctor(self const& rhs) : base{rhs}, q_{rhs.q_}{}
    ctor(self&& rhs)noexcept(is_nothrow_move_constructible_v<base>::value) : base{std::move(rhs)}, q_{std::move(rhs.q_)}{}
    self& operator=(self&& rhs) noexcept(is_nothrow_move_assignable<base>::value) { base::operator=(std::move(rhs)); q_ = std::move(rhs.q_); return *this; }
    self& operator=(self const& rhs) { ((base&)*this) = (base const&)rhs; q_ = rhs.q_; return *this; }
    bool empty() const noexcept { return q_.empty(); }
    size_type size() const noexcept { return q_.size(); } size_type max_size() const noexcept { return q_.max_size(); }
    void clear() noexcept { q_.clear(); }
    allocator_type get_allocator() const { return q_.get_allocator(); }
    const_reference top() const { return base::get_value(q_.front()); }
    void push(value_type const& v) { q_.push_back(base::make_node(v)); std::push_heap(q_.begin(), q_.end(), (base const&)*this); }
    void emplace<...Args>(Args&&...args) { q_.emplace_back(base::make_node(std::forward<Args>(args)...)); std::push_heap(q_.begin(), q_.end(), (base const&)*this); }
    void pop() { std::pop_heap(q_.begin(), q_.end(), (base const&)*this); q_.pop_back(); }
    void swap(self& rhs) noexcept(is_nothrow_move_constructible_v<base> && is_nothrow_move_assignable<base>) { base::swap(rhs); q_.swap(rhs.q_); }
    iterator begin() const noexcept { return {q_.begin()}; } iterator end() const noexcept { return {q_.end();} }
    void reserve(size_type element_count) { q_.reserve(element_count); }
    value_compare const& value_comp() const { return base::value_comp(); }
    bool operator< <H> (H const& rhs) const { return heap_compare(*this, rhs); } // and >, <=, >=
    bool operator== <H> (H const& rhs) const { return heap_equality(*this, rhs); } // and !=
};
```

-----
#### D-Ary Heap

```c++
struct detail::nop_index_updater { static void run<T>(T&, size_t) {} };
using detail::d_ary_heap_signature = parameter::parameters<required<tag::arity>,
  optional<tag::allocator>, optional<tag::compare>, optional<tag::stable>, optional<tag::stability_counter_type>, optional<tag::constant_time_size>>;
class detail::d_ary_heap<T,BoundArgs,IndexUpdater> : make_heap_base<T,BoundArgs,false>::type {
    using heap_base_maker = make_heap_base<T, BoundArgs, false>;
    using container_type = std::vector<internal_type, allocator_rebind<heap_base_maker::allocator_argument, internal_type>::type>;
    using container_iterator = container_type::const_iterator;
    using index_updater = IndexUpdater;
    static const unsigned D = parameter::binding<BoundArgs, tag::arity>::type::value;
    struct types: extract_allocator_types<heap_base_maker::allocator_argument> {
        using value_type = T;
        using value_compare = heap_base_maker::compare_argument;
        using allocator_type = container_type::allocator_type;
        struct ordered_iterator_dispatcher {
          static size_type max_index(const d_ary_heap* heap) { return heap->q_.size() - 1; }
          static bool is_leaf(const d_ary_heap* heap, size_type index) { return !heap->not_leaf(index); }
          static std::pair<size_type, size_type> get_child_nodes(const d_ary_heap* heap, size_type index)
          { return std::make_pair(d_ary_heap::first_child_index(index), heap->last_child_index(index)); }
          static internal_type const& get_internal_value(const d_ary_heap* heap, size_type index) { return heap->q_[index]; }
          static value_type const& get_value(internal_type const& arg) { return d_ary_heap::baes::get_value(arg); }
        };
        using ordered_iterator = ordered_adaptor_iterator<const value_type, internal_type, outer::base, allocator_type, d_ary_heap::internal_compare, ordered_iterator_dispatcher>;
        using iterator = stable_heap_iterator<const value_type, container_iterator, outer::base>;
        using const_iterator = iterator;
        using hanlde_type = void*;
    };
    using ordered_iterator_dispatcher = types::ordered_iterator_dispatcher;
    container_type q_;

    void reset_index(size_type index, size_type new_index) { index_updater::run(q_[index], new_index); }
    void siftdown(size_type index) {
      while (not_leaf(index)) {
        size_type max_child_index = top_child_index(index);
        if (!base::operator()(q_[max_child_index], q_[index])) {
          reset_index(index, max_child_index); reset_index(max_child_index);
          std::swap(q_[max_child_index], q_[index]); index=max_child_index;
        } else return;
    } }
    void siftup(size_type index) {
      while (index != 0) {
        size_type parent = parent_index(index);
        if (base::operator()(q_[parent], q_[index])) {
          reset_index(index, parent); reset_index(parent, index);
          std::swap(q_[parent], q_[index]); index=parent;
        } else return;
    } }
    bool not_leaf(size_type index) const { return first_child_index(index) < q_.size(); }
    size_type top_child_index(size_type index) const {
      auto first_index = first_child_index(index);
      auto first_child = q_.begin() + first_index, end = q_.end();
      auto last_child = (std::distance(first_child, end) > D) ? first_child + D : end;
      auto min_element = std::max_element(first_child, last_child, (base const&)*this);
      return min_element - q_.begin();
    }
    static size_type parent_index(size_type index) { return (index-1)/D; }
    static size_type first_child_index(size_type index) { return index * D + 1; }
    size_type last_child_index(size_type index) const { return std::min(first_child_index(index) + D - 1, size() - 1); }
    struct rebind<U,V,W,X> { using other = d_ary_heap<U,d_ary_heap_signature::bind<stable<heap_base_maker::is_stable>,
        stability_counter_type<heap_base_maker::stability_counter_type>, arity<D>, compare<V>, allocator<W>>::type, X>;
    };
    void update(size_type index) {
      if (index==0) { siftdown(index); return; }
      size_type parent = parent_index(index);
      if (base::operator()(q_[parent], q_[index])) siftup(index); else siftdown(index);
    }
    void erase(size_type index) {
      while (index!=0) {
        size_type parent = parent_index(index);
        reset_index(index, parent); reset_index(parent, index);
        std::swap(q_[parent], q_[index]); index=parent;
      }
      pop();
    }
    void increase(size_type index) { siftup(index); }
    void decrease(size_type index) { siftdown(index); }
public: using value_type = T;
    using size_type = types::size_type; // difference_type, value_compare, allocator_type, <const>_reference, <const>_pointer, <const>_iterator, ordered_iterator, handle_type
    static const bool is_stable = extract_stable<BoundArgs>::value;

    explicit ctor(value_compare const& cmp={}) : base{cmp} {}
    ctor(self const& rhs) : base{rhs}, q_{rhs.q_}{}
    ctor(self&& rhs) : base{std::move(rhs)}, q_{std::move(rhs.q_)}{}
    self& operator=(self&& rhs) { base::operator=(std::move(rhs)); q_ = std::move(rhs.q_); return *this; }
    self& operator=(self const& rhs) { ((base&)*this) = (base const&)rhs; q_ = rhs.q_; return *this; }
    bool empty() const { return q_.empty(); }
    size_type size() const { return q_.size(); } size_type max_size() const { return q_.max_size(); }
    void clear() { q_.clear(); }
    allocator_type get_allocator() const { return q_.get_allocator(); }
    value_type const& top() const { return base::get_value(q_.front()); }
    void push(value_type const& v) { q_.push_back(base::make_node(v)); reset_index(size()-1, size()-1); siftup(q_.size()-1); }
    void emplace<...Args>(Args&&...args) { q_.emplace_back(base::make_node(std::forward<Args>(args)...)); reset_index(size()-1, size()-1); siftup(q_.size()-1); }
    void pop() { std::swap(q_.front(), q_.back()); q_.pop_back(); if (q_.empty()) return; reset_index(0,0); siftdown(0); }
    void swap(self& rhs) { base::swap(rhs); q_.swap(rhs.q_); }
    iterator begin() const { return {q_.begin()}; } iterator end() const { return {q_.end();} }
    ordered_iterator ordered_begin() const { return {0, this, base::get_internal_cmp()}; }
    ordered_iterator ordered_end() const { return {size(), this, base::get_internal_cmp()}; }
    void reserve(size_type element_count) { q_.reserve(element_count); }
    value_compare const& value_comp() const { return base::value_comp(); }
};

struct detail::select_dary_heap<T,BoundArgs> {
  using type = std::conditional_t<extract_mutable<BoundArgs>::value,
    priority_queue_mutable_wrapper<d_ary_heap<T,BoundArgs, nop_index_updater>>, d_ary_heap<T,BoundArgs,nop_index_updater>>;
};

class d_ary_heap<T, A0=parameter::void_, ..., A5=parameter::void_> : public select_dary_heap<T,d_ary_heap_signature::bind<A0,...,A5>::type> {
  using bound_args = d_ary_heap_signature::bind<A0,...,A5>::type;
  static const bool is_mutable = extract_mutable<bound_args>::value;
  struct types { using size_type = base::size_type; }; // difference_type, value_compare, allocator_type, <const>_reference, <const>_pointer, <const>_iterator, ordered_iterator, handle_type
public: static const bool constant_time_size{true}, has_ordered_iterators{true}, is_mergable{false}, has_reserve{true}, is_stable{base::is_stable};
  using value_type = T;
  using size_type = types::size_type; // all types member types
  using base::ctor; using base::operator=;
  using base::empty; using base::size; using base::max_size; using base::clear;
  using base::get_allocator; using base::top; using base::push; using base::emplace;
  bool operator< <H>(H const& rhs) const { return heap_compare(*this, rhs); } // and >, <=, >=
  bool operator== <H>(H const& rhs) const { return heap_equality(*this, rhs); } // and !=
  using base::update; using base::increase; using base::decrease; using base::erase;
  using base::s_handle_from_iterator; using base::pop; using base::swap;
  using base::begin; using base::end; using base::ordered_begin; using base::ordered_end;
  using base::reserve; using base::value_comp;
};
```

------
#### Binomial Heap

```c++
using detail::binomial_heap_signature = parameter::parameters<optional<tag::allocator>, optional<tag::compare>, optional<tag::stable>, optional<tag::constant_time_size>, optional<tag::stability_counter_type>>;
struct detail::make_binomial_heap_base<T,Parspec> {
  static const bool constant_time_size = parameter::binding<Parspec, tag::constant_time_size, std::true_type>::type::value;
  using hb = make_heap_base<T,Parspec, constant_time_size>;
  using base_type = hb::type; using allocator_argument = hb::allocator_argument; using compare_argument = hb::compare_argument;
  using node_type = parent_pointing_heap_node<base_type::internal_type>;
  using allocator_type = allocator_rebind<allocator_argument, node_type>::type;
  struct type : base_type, allocator_type {
    ctor(compare_argument const& arg) : base_type{arg}{}
    ctor(allocator_type const& alloc) : allocator_{alloc}{}
    ctor(self const& rhs) : base_type{rhs}, allocator_type{rhs}{}
    ctor(self&& rhs) : base_type{std::move((base_type&)rhs)}, allocator_type{std::move((allocator_type&)rhs)} {}
    self& operator=(self&& rhs) { base_type::operator=(std::move((base_type&)rhs)); allocator_type::operator=(std::move((allocator_type&)(rhs))); return *this; }
    self& operator=(self const& rhs) { base_type::operator=(rhs); allocator_type::operator=(rhs); return *this; }
  };
};
class binomial_heap<T,A0=parameter::void_,...,A3=parameter::void_> : make_binomial_heap_base<T,binomial_heap_signature::bind<A0,...,A3>::type>::type {
    using bound_args = binomial_heap_signature::bind<A0,...,A3>::type;
    using base_maker = make_binomial_heap_base<T, bound_args>;
    using size_holder = base::size_holder_type;
    using allocator_argument = base_maker::allocator_argument;
    struct types: extract_allocator_types<base_maker::allocator_argument> {
        using value_type = T;
        using value_compare = base_maker::compare_argument;
        using allocator_type = base_maker::allocator_type;
        using node = base_maker::node_type;
        using <const>_node_pointer = allocator_<const>_pointer<allocator_type>::type;
        using handle_type = node_handle<node_pointer, outer::base, reference>;
        using node_type = base_maker::node_type;
        using node_list_type = intrusive::list<heap_node_base<false>, intrusive::constant_time_size<true>>;
        using node_list_<const>_iterator = node_list_type::<const>_iterator;
        using value_extractor = value_extractor<value_type, internal_type, outer::base>;
        using iterator = recursive_tree_iterator<node_type, node_list_const_iterator, const value_type, value_extractor, list_iterator_converter<node_type, node_list_type>>;
        using const_iterator = iterator;
        using ordered_iterator = tree_iterator<node_type, const value_type, allocator_type, value_extractor, list_iterator_converter<node_type, node_list_type>, true, true, value_compare>;
    };
    using node_type = types::node_type; // node_list_type, <const>_node_pointer, node_list_<const>_iterator
    node_pointer top_element{nullptr}; node_list_type trees;

    void merge_and_clear_nodes(self& rhs);
    void clone_forest(self const& rhs) {
      using node_cloner = node_type::node_cloner<allocator_type>;
      trees.clone_from(rhs.trees, node_cloner{*this, nullptr}, nop_disposer{});
      update_top_element();
    }
    struct force_inf { bool operator()<X>(X const&, X const&) const { return false; } };
    void siftup<Copmare>(node_pointer n, Compare const& cmp);
    void siftdown(node_pointer n);
    void insert_node(node_list_iterator it, node_pointer n);
    explicit ctor(value_compare const& cmp, node_list_type& child_list, size_type size) : base{cmp} {/*...*/}
    node_pointer merge_trees(node_pointer node1, node_pointer node2);
    void update_top_element() { top_element = find_max_child<node_list_type, node_type, internal_compare>(trees, base::get_internal_cmp()); }
    void sorted_by_degree() const{/*...*/}
    void sanity_check(){/*...*/}
public:
    static const bool constant_time_size = base::constant_time_size, has_ordered_iterators{true}, is_mergable{true}, is_stable = extract_stable<bound_args>::value, has_reserve{false};
    using value_type = T;
    using size_type = types::size_type; // difference_type, value_compare, allocator_type, <const>_reference, <const>_pointer, <const>_iterator, ordered_iterator, handle_type

    explicit ctor(value_compare const& cmp={}) : base{cmp} {}
    explicit ctor(allocator_type const& alloc): base{alloc} {}
    ctor(self const& rhs) : base{rhs} { if (rhs.empty()) return; clone_forest(rhs); size_holder::set_size(rhs.get_size()); }
    ctor(self&& rhs) : base{std::move(rhs)}, top_element{rhs.top_element}{ trees.splice(trees.begin(), rhs.trees); rhs.top_element = nullptr; }
    self& operator=(self const& rhs) {
      clear(); size_holder::set_size(rhs.get_size()); ((base&)(*this)) = rhs;
      if (rhs.empty()) top_element = nullptr; else clone_forest(rhs);
      return *this;
    }
    self& operator=(self&& rhs) {
      clear(); base::operator=(std::move(rhs)); trees.splice(trees.begin(), rhs.trees);
      top_element = rhs.top_element; rhs.top_element = nullptr;
      return *this;
    }
    ~dtor() { clear(); }
    bool empty() const { return top_element == nullptr; }
    size_type size() const {
      if (constant_time_size) return size_holder::get_size();
      if (empty()) return 0; else return count_list_nodes<node_type, node_list_type>(trees);
    }
    size_type max_size() const { return allocator_max_size((const allocator_type&)*this); }
    void clear() {
      using disposer = node_disposer<node_type, node_list_type::value_type, allocator_type>;
      trees.clear_and_dispose(disposer{*this});
      size_holder::set_size(0); top_element = nullptr;
    }
    allocator_type get_allocator() const { return *this; }
    void swap(self& rhs) { base::swap(rhs); std::swap(top_element, rhs.top_element); trees.swap(rhs.trees); }
    const_reference top() const { return base::get_value(top_element->value); }
    handle_type push(value_type const& v) {
      node_pointer n = this->allocate(1); new(n) node_type{base::make_node(v)}; insert_node(trees.begin(), n);
      if (!top_element||base::operator()(top_element->value, n->value)) top_element = n;
      size_holder::increment(); sanity_check(); return {n};
    }
    handle_type emplace<...Args>(Args&&...args) {
      node_pointer n = this->allocate(1); new(n) node_type{base::make_node(std::forward<Args>(args)...)}; insert_node(trees.begin(), n);
      if (!top_element||base::operator()(top_element->value, n->value)) top_element = n;
      size_holder::increment(); sanity_check(); return {n};
    }
    void pop() {
      trees.erase(node_list_type::s_iterator_to(*top_element)); size_holder::decrement();
      if (element->child_count()) {
        binomial_heap children{value_comp(), element->children, (1<<element->child_count())-1};
        if (trees.empty()) { size_t size = constant_time_size ? size_holder::get_size() : 0;
          swap(children); base::set_stability_count(base::get_stability_count());
          if (constant_time_size) size_holder::set_size(size);
        } else merge_and_clear_nodes(children);
      }
      if (trees.empty()) top_element = nullptr; else update_top_element();
      element->~node_type(); this->deallocate(element, 1); sanity_check();
    }
    void update(handle_type handle, const_reference v)
    { if (base::operator()(base::get_value(handle.node_->value), v)) increase(handle, v); else decrease(handle, v); }
    void update(handle_type handle) {
      if (handle.node_->parent) {
        if (base::operator()(base::get_value(handle.node_->parent->value), base::get_value(handle.node_->value)))
          increase(handle); else decrease(handle);
      } else decrease(handle);
    }
    void increase(handle_type handle, const_reference v) { handle.node_->value = base::make_node(v); increase(handle); }
    void increase(handle_type handle) { siftup(handle.node_, *this); update_top_element(); sanity_check(); }
    void decrease(handle_type handle, const_reference v) { handle.node_->value = base::make_node(v); decrease(handle); }
    void decrease(handle_type handle) { siftdown(handle.node_); update_top_element(); }
    void merge(binomial_heap& rhs) {
      if (rhs.empty()) return; if (empty()) { swap(rhs); return; }
      size_type new_size = size_holder::get_size() + rhs.get_size();
      merge_and_clear_nodes(rhs);
      size_holder::set_size(new_size); rhs.set_size(0); rhs.top_element = nullptr;
      base::set_stability_count(std::max(base::get_stability_count(), rhs.get_stability_count())); rhs.set_stability_count(0);
    }
    iterator begin() const { return {trees.begin()} } iterator end() const { return {trees.end();} }
    ordered_iterator ordered_begin() const { return {trees.begin(), trees.end(), top_element, base::value_comp()}; }
    ordered_iterator ordered_end() const { return {nullptr, base::value_comp()}; }
    void erase(handle_type handlej) { siftup(handle.node_, force_inf{}); top_element = n; pop(); }
    static handle_type s_handle_from_iterator(iterator const& it) { return {it.get_node()}; }
    value_compare const& value_comp() const { return base::value_comp(); }
    bool operator< <H> (H const& rhs) const { return heap_compare(*this, rhs); } // and >, <=, >=
    bool operator== <H> (H const& rhs) const { return heap_equality(*this, rhs); } // and !=
};
```

------
#### Fibonacci Heap

```c++
```

------
#### Pairing Heap

```c++
```

------
#### Skew Heap

```c++
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.ConceptCheck

* `<boost/concept_check.hpp>`
* `<boost/concept/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`
* `<boost/cstdint.hpp>`

#### Boost.Core

* `<boost/core/allocator_access.hpp>`

#### Boost.Intrusive

* `<boost/intrusive/list.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_adaptor.hpp>`

#### Boost.Parameter

* `<boost/parameter/**.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

------
### Standard Facilities
