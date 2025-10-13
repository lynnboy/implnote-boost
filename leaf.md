# Boost.LEAF - Lightweight Error Augmentation Framework

* lib: `boost/libs/leaf`
* repo: `boostorg/leaf`
* commit: `61e4b0d`, 2025-06-02

------
### Common Bits

* Header `<boost/leaf/detail/demangle.hpp>`

```c++
constexpr int check_prefix <s1,s2> (char const (&str)[s1], char const (&prefix)[s2]) noexcept; // str starts with prefix
constexpr int check_suffix <s1,s2> (char const (&str)[s1], char const (&suffix)[s2]) noexcept; // str starts with prefix
struct n::r { char const* name; int len; }; // init-ctor, and inserter <<
r n::p <T> () { // Get demangled string for T within n::p()'s signature
    int const p01 = check_prefix(__PRETTY_FUNCTION__, "r boost::leaf::n::p() [T = "); // and p02~p24 for all possible cc/mangle styles
    int const s01 = check_suffix(__PRETTY_FUNCTION__, "]"); // and s02 for ">(void)"
    int const p = ..., s = ...; // and validate possible cases, then
    return n::r{ __PRETTY_FUNCTION__ + p, s - p };
}
using parsed = n::r;
parsed parse <T> () { return n::p<T>(); }
```

* Header `<boost/leaf/detail/optional.hpp>`

```c++
class detail::optional<T> { // key marked value holder, all constexpr
    int key_{0}; union { T value_; }; // key==0 for no value; T is placement-new ed on `value`
public: using value_type = T;
    ctor() noexcept; ctor(int key, T {const&|&&} v);
    // and copy/move-ctor, copy/move-assign, dtor
    bool empty() const noexcept; int key() const noexcept;
    void reset() noexcept { if (key_) { value_.~T(); key_=0; } } // call dtor to destory
    T& load(int key) { reset(); new(&value_) T; key_ = key; return value_; }
    T& load(int key, T const& v); T& load(int key, T && v) noexcept; // rval moved-in
    T<const>* has_value_any_key()<const> noexcept { return key_ ? &value_ : nullptr; }
    T<const>* has_value(int key)<const> noexcept { return key_ == key ? &value_ : nullptr; }
    T{const&|&|const&&} value(int key) {const&|&|const&&} noexcept { assert(has_value(key)); return value_; }
    T value(int key) && noexcept { assert(has_value(key)); T tmp(std::move(value_)); reset(); return tmp; } // move-out
    T& value_or_default(int key) noexcept { if (T* v = has_value(key)) return *v; else return load(key); }
};
```

* Header `<boost/leaf/detail/mp11.hpp>` - Selected parts from Boost.MP11.

* Header `<boost/leaf/detail/function_traits.hpp>`

```c++
struct detail::remove_noexcept<T> { using type = T; };
struct detail::remove_noexcept<R(*)(A...) noexcept> { using type = R(*)(A...); };
struct detail::remove_noexcept<R(C::*)(A...)<const> noexcept> { using type = R(C::*)(A...)<const>; };

struct detail::function_traits_impl<F,=void> { constexpr static int arity = -1; }; // primary, for non-functions
struct detail::function_traits_impl<F,void_t<decltype(&F::operator())>> { // class supports op()
    using tr = function_traits_impl<remove_noexcept<decltype(&F::operator())>::type>; // tr for op()
    using return_type = tr::return_type; constexpr static int arity = tr::arity - 1; // no this param
    using mp_args = mp_rest<tr::mp_args>; struct arg<i> : tr::arg<i+1> {}; // off-by-1 for this param
};
struct detail::function_traits_impl<R(A...)> { // function types
    using return_type = R; constexpr static int arity = sizeof...(A);
    using mp_args = mp_list<A...>; struct arg<i> { using type = A...[i]; };
};
struct detail::function_traits_impl<F{&|&&}> : function_traits_impl<F> {};
struct detail::function_traits_impl<R(*<&|const>)(A...)> : function_traits_impl<R(A...)> {}; // fptr-type to fntype
struct detail::function_traits_impl<R(C::*)(A...)<const>> : function_traits_impl<R(C<const>&,A...)> {}; // expos this on func-mptr
struct detail::function_traits_impl<R(C::*)> : function_traits_impl<R(C&)> {}; // data mptr
using function_traits<F> = function_traits_impl<remove_noexcept<F>::type>;
using fn_result_type<F> = function_traits<F>::return_type;
using fn_arg_type<F,i> = function_traits<F>::arg<i>::type;
using fn_mp_args<F> = function_traits<F>::mp_args;
```

* Header `<boost/leaf/config/tls_xxx.hpp>`

```c++
using detail::atomic_unsigned_int = std::atomic<unsigned int>; // just unsigned int when no threading

struct tls::ptr<T> { static <thread_local> T* p; }; // ptr for each T, no `thread_local` when no threading
T* tls::read_ptr <T> () noexcept { return ptr<T>::p; }
void tls::write_ptr <T> (T* p) noexcept { ptr<T>::p = p; }

struct tls::tagged_uint <Tag> { static <thread_local> unsigned x; }; // value for each Tag type
unsigned tls::read_uint <Tag> () noexcept { return tagged_uint<Tag>::x; }
void tls::write_uint <Tag> (unsigned x) noexcept { tagged_uint<Tag>::x = x; }
```

* Header `<boost/leaf/detail/capture_list.hpp>`

```c++
class detail::capture_list { // no copy. all non-virtual functions constexpr
protected:
    class node {
        virtual void unload(int err_id) = 0;
        virtual void print(std::ostream&, error_id const& e, char const*& prefix) const = 0;
    protected: virtual ~dtor() noexcept{};
        node* next_; // single-linked
        constexpr explicit node(node **& last) noexcept;
    } *first_; // link head
    void for_each <F> (F f) const { for (node *p = first_; p; p=p->next_) f(*p); }
public:
    explicit ctor(note* first) noexcept; // init
    ctor(self&& other) noexcept; // move
    ~dtor() noexcept; // delete each node
    void unload(int err_id) {
        capture_list moved(first_); first_ = nullptr; // move to temp
        tls::write_uint<tls_tag_id_factory_current_id>((unsigned)err_id); // this thread's current error id
        moved.foreach([err_id](node& n) { n.unload(err_id); }); // foreach call unload before dtor
    }
    void print<Ch,Tr>(std::basic_stream<Ch,Tr>& os, error_id const& e, char const*& prefix) const; // for_each node call print
};
```

* Header `<boost/leaf/detail/print.hpp>`

```c++
struct show_in_diagnostics<E> : std::true_type {}; // primary, default is show

using detail::is_printable<T> = requires(std::ostream& os, T const& t){os<<t;} && show_in_diagnostics<T>;
using detail::has_printable_member_value<T> = requires(std::ostream& os, T const& t){os<<t.value;} && show_in_diagnostics<T>;

void detail::print_name <T> (std::ostream& os, char const*& prefix, char const* delimiter)
{ char const* p = prefix; prefix = nullptr;     os << (p?p:delimiter) << parse<T>(); }
void detail::print_impl <T,PInfo> (std::ostream& os, char const*& prefix, char const* delimiter, char const* mid, PInfo const{&|*}x)
{ print_name<T>(os, prefix, delimiter);  if (mid) os << mid << (x ? x : "<nullptr>"); return true; }

struct detail::diagnostic<Wrapper, showInDiag=show_in_diagnostics<Wrapper>::value,
    printW=is_printable<Wrapper>::value, printV=has_printable_member_value<Wrapper>::value,
    isEx=std::is_base_of_v<std::exception,Wrapper>, isEnum=std::is_enum_v<Wrapper>>
{
    static bool print(std::ostream& os, char const*& prefix, char const* delimiter, Wrapper const& x) {
        if constexpr (!showInDiag) return false;
        else if constexpr (isEx) { // Wrapper is exception type
            if (print_impl<Wrapper>(os, prefix, delimiter, ": \"", x.what())) return os<<'"', true;
            else return false }
        else if constexpr (printW) return print_impl<Wrapper>(os, prefix, delimiter, ": ", x);
        else if constexpr (printV) return print_impl<Wrapper>(os, prefix, delimiter, ": ", x.value);
        else if constexpr (isEnum) return print_impl<Wrapper>(os, prefix, delimiter, ": ", std::underlying_type_t<Wrapper>(x));
        else                       return print_impl<Wrapper>(os, prefix, delimiter, nullptr, 0);
    }
};
```

------
### Error Reporting

* Header `<boost/leaf/common.hpp>`

```c++
struct e_api_function { char const * value; };
struct e_file_name { std::string value; };
struct e_errno { int value; explicit ctor(int v=errno); }; // and hidden friend op<<
struct e_type_info_name { char const * value; };
struct e_at_line { int value; };
struct windows::e_LastError { unsigned value; explicit ctor(unsigned val); ctor() : value(GetLastError()) {} }; // and op<<
```

* Header `<boost/leaf/error.hpp>`

```c++
#define BOOST_LEAF_ASSIGN(v,r) \
    auto&& <tmp> = r; if (!<tmp>) return <tmp>.error(); \
    v = std::forward<decltype(<tmp>)>(<tmp>).value();
#define BOOST_LEAF_AUTO(v,r)    BOOST_LEAF_ASSIGN(auto v, r)

#define BOOST_LEAF_CHECK(r) \
    { auto&& <tmp> = (r); if (!<tmp>) return <tmp>.error(); }

#define BOOST_LEAF_NEW_ERROR detail::inject_loc{__FILE__,__LINE__,__FUNCTION__} + new_error

struct e_source_location { char const* file; int line; char const* function; }; // and friend op<<
struct show_in_diagnostics<e_source_location> : std::false_type {}; // dont show

class detail::exception_base { // interface for exceptions
protected: ctor() noexcept{} ~dtor() noexcept{}
public: virtual error_id get_error_id() const noexcept = 0;
    virtual void print_type_name(std::ostream&) const = 0; // diag only
};

class detail::slot<E> : public optional<E> { // imp inherit. delete copy-ctor/copy-assign
    slot<E>* prev_{nullptr}; // linked list
public: ctor(); ctor(self&&)noexcept; ~dtor()noexcept; // def-ctor, move-ctor and dtor
    void activate() noexcept { prev_ = tls::read_ptr<self>(); tls::write_ptr<self>(this); } // set this as new list head
    void deactivate() const noexcept {
        if constexpr(std::is_same_v<E,dynamic_allocator>)
            if (auto da = this->has_value_any_key()) da->deactivate();
        tls::write_ptr<self>(prev_); // remove this from list head
    }
    void unload(int err_id) noexcept(...) {
        if constexpr(std::is_same_v<E,dynamic_allocator>)
        { if (auto da = this->has_value_any_key()) da->unload(err_id); }
        else { // not for dynamic_allocator
            if (this->key() != err_id) return;
            if (auto p = tls::read_ptr<self>()) { if (!p->has_value(err_id)) *p = std::move(*this); }
            else dynamic_load<false>(error_id, std::move(*this).value(err_id)); // only when CFG_CAPTURE
        }
    }
    void print<ErrID> (std::ostream& os, ErrId to_print, char const*& prefix) const {
        if (int k = this->key()) {
            if (to_print && to_print.value() != k) return;
            if (diagnostic<E>::print(os,prefix,BOOST_LEAF_CFG_DIAGNOSTICS_DELIMITER,this->value(k)) && !to_print) os << '(' << k/4 << ')';
        }
    }
};

class detail::dynamic_allocator : capture_list { // only when capture is enabled. delete copy-ctor/copy-assign
    class capturing_node : public node { // base interface
    protected: constexpr explicit ctor(node**&last) noexcept : base(last) {}
    public: virtual void deactivate() const noexcept = 0;
    };
    class capturing_slot_node<E>: public capturing_node, public slot<E> { // delete copy
        void deactivate() const noexcept final override { slot<E>::deactivate(); }
        void unload(int err_id) final override { slot<E>::unload(err_id); }
        void print(std::ostream& os, error_id const& to_print, char const*& prefix) const final override { slot<E>::print(os, to_print, prefix); }
    public: constexpr ctor(node**&last, int error_id, T&& e) : capturing_node(last) { slot<E>::load(err_id, (T&&)(e)); }
    };
    class capturing_exception_node: public capturing_node { // delete copy
        std::exception_ptr const ex_;
        void deactivate() const noexcept final override { }
        void unload(int) final override { std::rethrow_exception(ex_); }
        void print(std::ostream& os, error_id const& to_print, char const*& prefix) const final override { }
    public: constexpr ctor(node**&last, std::exception_ptr&& ex) noexcept : capturing_node(last), ex_(std::move(ex)) { }
    };

    node ** last_; // list end. `first_` from capture_list

public: ctor() : last_(&first){} ctor(self&&) noexcept; // def and move
    std::decay_t<E>& dynamic_load <E> (int err_id, E&& e) {
        using T = std::decay_t<E>;
        auto csn = new capturing_slot_node<T>(last_, err_id, (E&&)e); // allocate one node into list
        csn->activate(); return csn->value(err_id);
    }
    void deactivate() const noexcept { for_each([](node const& n) {((capturing_node const&)n).deactivate();}); } // deactivate all nodes
    LeafRes extract_capture_list <LeafRes> (int err_id) {
        if (auto ex = std::current_exception()) new capturing_exception_node(last_, std::move(ex)); // current ex
        auto f = first_; first_ = nullptr; last_ = &first; // exchange out the list
        return {err_id, capture_list(f)};
    }
};
struct show_in_diagnostics<dynamic_allocator> : std::false_type{};

void dynamic_load_<E> (int err_id, E&& e) {
    if (auto sl = tls::read_ptr<slot<dynamic_allocator>>()) {
        if (auto da = sl->has_value_any_key()) da->dynamic_load(err_id, (E&&)e);
        else sl->load(err_id).dynamic_load(err_id, (E&&)e);
    }
}
void dynamic_accumulate_<E,F> (int err_id, F&& f) {
    if (auto sl = tls::read_ptr<slot<dynamic_allocator>>()) {
        if (auto da = sl->has_value(err_id)) ((F&&)f)( da->dynamic_load(err_id, E{}) );
        else ((F&&)f)( sl->load(err_id).dynamic_load(err_id, E{}) );
    }
}
void dynamic_load <onError,E> (int err_id, E&& e) noexcept(onError) {
    if (onError) try { dynamic_load_(err_id, (E&&)e); } catch(...){}
    else dynamic_load_(err_id, (E&&)e);
}
void dynamic_accumulate <onError,E,F> (int err_id, F&& f) noexcept(onError) {
    if (onError) try { dynamic_accumulate_(err_id, (F&&)f); } catch(...){}
    else dynamic_accumulate_(err_id, (F&&)f);
}

int detail::load_slot <onError,E> (int err_id, E&& e) noexcept(onError) {
    using T = std::decay_t<E>;
    if (auto p = tls::read_ptr<slot<T>>())
    { if (!onError || !p->has_value(err_id)) p->load(err_id, (E&&)e); }
    else dynamic_load<onError>(err_id, (E&&)e); // only when CFG_CAPTURE
    return 0;
}
int detail::load_slot_deferred <onError,F> (int err_id, F&& e) noexcept(onError) {
    using E = function_traits<F>::return_type; using T = std::decay_t<E>;
    if (auto p = tls::read_ptr<slot<T>>())
    { if (!onError || !p->has_value(err_id)) p->load(err_id, (F&&)f); }
    else dynamic_load<onError>(err_id, (F&&)f); // only when CFG_CAPTURE
    return 0;
}
int detail::load_slot_accumulate <onError,F> (int err_id, F&& e) noexcept(onError) {
    using T = std::decay_t<fn_arg_type<F,0>>;
    if (auto sl = tls::read_ptr<slot<T>>())
    { if (auto v = sl->has_value(err_id)) ((F&&)f)(*v); else ((F&&)f)(sl->load(err_id, T{})); }
    else dynamic_load_accumulate<onError,T>(err_id, (F&&)f); // only when CFG_CAPTURE
    return 0;
}

struct detail::load_item<T, arity=function_traits<T>::arity> {};
struct detail::load_item<E,-1> // not function
{ static int load_(int err_id, E&&e) noexcept { return load_slot<false>(err_id, (E&&)e); } };
struct detail::load_item<F,0> // f()
{ static int load_(int err_id, F&&f) noexcept { return load_slot_deferred<false>(err_id, (F&&)f); } };
struct detail::load_item<F,1> // f()
{ static int load_(int err_id, F&&f) noexcept { return load_slot_accumulate<false>(err_id, (F&&)f); } };

struct detail::id_factory< =void> {
    static inline atomic_unsigned_int counter = {1};
    static unsigned generate_next_id() noexcept { auto id = (counter+=4); assert(id&3 == 1); return id; }
};
int detail::current_id() noexcept
{ unsigned id = tls::read_uint<tls_tag_id_factory_current_id>(); return int(id); }
int detail::new_id() noexcept
{ unsigned id = id_factory<>::generate_next_id(); tls::write_uint<tls_tag_id_factory_current_id>(id); return int(id); }
struct detail::inject_loc { char const* file; int line; char const* fn;
    friend T operator+ <T> (inject_loc loc, T&& x) noexcept
    { x.load_source_location_(loc.file, loc.line, loc.fn); return std::move(x); }
};

class detail::leaf_error_category final : public std::error_category { // only when CFG_STD_SYSTEM_ERROR
    // override equivalent() { return false; }, override name() and message() { return "LEAF error"; }
};
struct detail::get_leaf_error_category< =void> { static inline leaf_error_category cat; }
int detail::import_error_code(std::error_code const& ec) noexcept {
    if (int err_id = ec.value()) { auto& cat = get_leaf_error_category<>::cat;
        if (&ec.category() == &cat) return (err_id&~3)|1;
        else { err_id = new_id(); load_slot<false>(err_id, ec); return (err_id&~3)|1; }
    } else return 0;
}
bool detail::is_error_id(std::error_code const& ec) noexcept { return &ec.category() == &detail::get_leaf_error_category<>::cat; }

class error_id {

};
```

------
### Configurations

* Platforms: `BOOST_LEAF_TLS_FREERTOS`, `BOOST_LEAF_EMBEDDED`, `BOOST_LEAF_CFG_WIN32`, 
* Options: `BOOST_LEAF_CFG_DIAGNOSTICS`, `BOOST_LEAF_CFG_STD_SYSTEM_ERROR`, `BOOST_LEAF_CFG_STD_STRING`, `BOOST_LEAF_CFG_CAPTURE`
* Lang/Lib: `BOOST_LEAF_ASSERT`(`assert`), `BOOST_LEAF_CFG_GNUC_STMTEXPR`, `BOOST_LEAF_PRETTY_FUNCTION`(`__PRETTY_FUNCTION__`), `BOOST_LEAF_NO_EXCEPTIONS`
* TLS impl: `BOOST_LEAF_USE_TLS_ARRAY` to use `tls_array`, otherwise use global or `thread_local` based on threading off/on.
* Diagnostic format: `BOOST_LEAF_CFG_DIAGNOSTICS_FIRST_DELIMITER`, `BOOST_LEAF_CFG_DIAGNOSTICS_DELIMITER`

------
#### Dependency

------
### Standard Facilities
