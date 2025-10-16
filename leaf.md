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

-----
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
    int value_{0};     ctor(int value) noexcept :value_{value} {}; // data and init
public: ctor() noexcept{}
    explicit ctor(std::error_code const& ec) noexcept : value_(import_error_code(std::error_code(ec))) {} // only when CFG_STD_SYSTEM_ERROR
    ctor <Enum> (Enum e) noexcept requires std::is_error_code_enum_v<Enum>  : value_(import_error_code(std::error_code(ec))) {} // only when CFG_STD_SYSTEM_ERROR
    operator T() const noexcept requires is_constructible_v<T,std::error_code> { return std::error_code(value_, get_leaf_error_category<>::cat); } // only when CFG_STD_SYSTEM_ERROR

    error_id load() const noexcept { return *this; }
    error_id load<...Item> (Item&&...item) const noexcept
    { if (int err_id = value()) load_item<Item>::load_(err_id, (Item&&)item)...; return *this; }
    int value() const noexcept { return value_; }
    explicit operator bool() const noexcept { return value_ != 0; }
    friend bool operator==(self a, self b) noexcept { return a.value_ == b.value_; } // and also !=, <
    friend std::ostream& operator<< (std::ostream& os, error_id x) { return os << (x.value_/4); }
    void load_source_location_(char const* file, int line, char const* fn) const noexcept
    { load(e_source_location{file, line, fn}); }
};

error_id detail::make_error_id(int err_id) noexcept { return error_id((err_id&~3)|1); }
error_id new_error() noexcept { return make_error_id(new_id()); }
error_id new_error<...Item> (Item&&...item) noexcept { return make_error_id(new_id()).load((Item&&)item...); }
error_id current_error() noexcept { return make_error_id(current_id()); }

struct is_result_type<R> : std::false_type {}; // primary
struct is_result_type<R const> : is_result_type<R> {};
```

* Current error-id is stored in TLS with tag `tls_tag_id_factory_current_id`.

-----
* Header `<boost/leaf/exception.hpp>`

```c++
#define BOOST_LEAF_THROW_EXCEPTION detail::throw_with_loc{__FILE__,__LINE__,__FUNCTION__}+detail::make_exception
[[noreturn]] void detail::throw_exception_impl <T> (T&& e) { throw std::move(e); }
struct detail::throw_with_loc { char const* const file; int const line; char const* const fn;
    friend void operator+ <Ex> (throw_with_loc loc, Ex&& ex)
    { ex.load_source_location_(loc.file, loc.line, loc.fn); throw_exception_impl(std::move(ex)); }
};

void detail::enforce_std_exception(std::exception const&) noexcept {}
class detail::exception<Ex> : public Ex, public exception_base, public error_id {
    mutable bool clear_current_error_; // owner flag for current exception
    bool is_current_exception() const noexcept { return tls::read_uint<tls_tag_id_factory_current_id>() == value(); }
    error_id get_error_id() const noexcept final override { clear_current_error_ = false; return *this; }
    void print_type_name(std::ostream& os) const final override { demangle_and_print(os, typeid(Ex).name()); } // only when CFG_DIAGNOSTICS
public: ctor(self const&); ctor(self &&); // copy and move
    ctor(error_id id, <Ex {const&|&&} ex>) noexcept : <Ex(<std::move>(ex))>, error_id(id), clear_current_error_(true) { enforce_std_exception(*this); }
    ~dtor() noexcept { if (clear_current_error_ && is_current_exception()) tls::write_uint<tls_tag_id_factory_current_id>(0); }
};

struct detail::at_list_one_derives_from_std_exception<...T>
{ value = false || ... || std::is_base_of_v<std::exception,std::remove_reference_t<T>>; };
decltype(auto) detail::make_exception <...E> (error_id err, E&& ...e) noexcept {
    if constexpr (sizeof...(E) == 0) return exception<std::exception>{err};
    else { using {Ex,...Ey} = {E...[0],E...[1:]}; static_assert(!at_list_one_derives_from_std_exception<Ey...>::value);
        if constexpr (!std::is_base_of_v<std::exception, Ex>) return exception<std::exception>( err.load((E&&)e...) );
        else return exception<std::remove_reference_t<Ex>>( err.load(forward(e...[1:]), (Ex&&)e...[0]));
    }
}
decltype(auto) detail::make_exception <...E> (E&& ...e) noexcept {
    if constexpr (sizeof...(E) == 0) return exception<std::exception>{new_error()};
    else { using {Ex,...Ey} = {E...[0],E...[1:]}; static_assert(!at_list_one_derives_from_std_exception<Ey...>::value);
        if constexpr (!std::is_base_of_v<std::exception, Ex>) return exception<std::exception>( new_error().load((E&&)e...) );
        else return exception<std::remove_reference_t<Ex>>( new_error().load(forward(e...[1:]), (Ex&&)e...[0]));
    }
}
[[return]] void throw_exception<...E> (E&&...e) { throw_exception_impl(make_exception((E&&)e...)); }

error_id detail::catch_exception_helper <...E> (std::exception const& ex, mp_list<E...>) {
    if constexpr (sizeof...(E) == 0) return new_error(std::current_exception());
    else if (auto p = dynamic_cast<Ex const*>(&ex)) return catch_exception_helper(ex, mp_rest<mp_list<E...>>{}).load(*p);
    else return catch_exception_helper(ex, mp_rest<mp_list<E...>>{});
}
struct detail::deduce_exception_to_result_return_type_impl<T> { using type = result<T>; };
struct detail::deduce_exception_to_result_return_type_impl<result<T>> { using type = result<T>; };
using detail::deduce_exception_to_result_return_type<T> = deduce_exception_to_result_return_type_impl<T>::type;
auto exception_to_result<...Ex,F> (F&& f) noexcept -> deduce_exception_to_result_return_type<fn_return_type<F>> {
    try { return ((F&&)f)(); }
    catch (std::exception const& ex) { return catch_exception_helper(ex, mp_list<Ex...>{}); }
    catch (...) { return new_error(std::current_exception()); }
}
```

-----
* Header `<boost/leaf/on_error.hpp>`

```c++
class error_monitor {
    int const uncaught_exceptions_, err_id_;
public:
    ctor() noexcept : uncaught_exceptions_{std::uncaught_exceptions()}, err_id_(current_id()) {}
    int check_id() const noexcept {
        int err_id = current_id();
        if (err_id != err_id_) return err_id;
        else if (std::uncaught_exceptions() > uncaught_exceptions_) return new_id();
        else return 0;
    }
    int get_id() const noexcept {
        int err_id = current_id();
        if (err_id != err_id_) return err_id; else return new_id();
    }
    error_id check() const noexcept { return make_error_id(check_id()); }
    error_id assigned_error_id() const noexcept { make_error_id(get_id()); }
};

struct detail::tuple_for_each_preload<i,Tup> //recursion on Tup
{ static void trigger(Tup& tup, int err_id) noexcept {
    if constexpr (i==0) return;
    else { tuple_for_each_preload<i-i,Tup>::trigger(tup,err_id); std::get<i-1>(tup).trigger(err_id); }
} };
struct detail::preloaded_item<E> {
    std::decay_t<E> e_; ctor(E&& e) : e_((E&&)e) {} // data && init
    void trigger(int err_id) noexcept { load_slot<true>(err_id, std::move(e_)); }
};
struct detail::deferred_item<F,Ret=function_traits<F>::return_type> {
    F f_;       ctor(F&& f) noexcept : f_(std::forward<F>(f)) {} // data && init
    void trigger(int err_id) noexcept
    { if constexpr (std::is_same_v<Ret,void>) f_(); else load_slot_deferred<true>(err_id, f_); }
};
struct detail::accumulating_item<F,A0=fn_arg_type<F,0>,arity=function_traits<F>::arity>;
struct detail::accumulating_item<F,A0&,1> { // 1-arg
    F f_;       ctor(F&& f) noexcept : f_(std::forward<F>(f)) {} // data && init
    void trigger(int err_id) noexcept { load_slot_accumulate<true>(err_id, std::move(f_)); }
};
class preloaded<...Item> { // trigger all on dtor
    std::tuple<Item...> p_; bool moved_{false}; error_monitor id_;
public: explicit ctor(Item&&...i): p_(std::forward<Item>(i)...) {}
    ctor(self&&) noexcept; //support move, delete copy
    ~dtor() noexcept { if (!moved_) if (auto id = id_.check_id()) tuple_for_each_preload<sizeof...(Item),decltype(p_)>::trigger(p_,id); }
};
struct detail::deduce_item_type<F,arity=function_traits<F>::arity>;
struct detail::deduce_item_type<F,-1> { using type = preloaded_item<F>; }.
struct detail::deduce_item_type<F,0> { using type = deferred_item<F>; };
struct detail::deduce_item_type<F,1> { using type = accumulating_item<F>; };

auto on_error <...Item> (Item&&...i) -> preloaded<deduce_item_type<Item>::type...>
{ return {std::forward<Item>(i)...}; }
```

-----
* Header `<boost/leaf/result.hpp>`

```c++
class bad_result : public std::exception { /*what*/ };

struct detail::result_value_pointer<T,printable=is_printable<T>::value>
{ static void print(std::ostream& s, T const& x) { if constexpr(printable) s << x; else s << "{not printable}"; } };
void detail::print_result_value <T> (std::ostream& s, T const& x) { result_value_pointer<T>::print(s, x); }

struct detail::stored<T> { using type = T; // and also specialization for `T&`
    using value_no_ref = T; using value_no_ref_const = T const;
    using value_cref = T const&; using value_ref = T&;
    using value_rv_cref = T const&&; using value_rv_ref = T&&;
    static value_no_ref_<const>* cptr(type <const>& v) noexcept { return &v; }
};
class detail::result_discriminant { int state_; // low 3-bits is kind
public: enum kind_t { err_id_zero, err_id, err_id_capture_list, val };
    explicit ctor(error_id id) noexcept : state_(id.value()) {}
    explicit ctor(int err_id, capture_list const&) noexcept : state_((err_id&~3)|2) {} // only when CFG_CAPTURE
    struct kind_val {}; explicit ctor(kind_val) noexcept : state_(val) {}
    kind_t kind() const noexcept { return kind_t(state_&3); }
    error_id get_error_id() const noexcept { return make_error_id((state_&~3)|1); }
};

class result <T> {
    using stored_type = stored<T>::type; // using other types from stored<T>
    union { stored_type stored_; mutable capture_list cap_; };
    result_discriminant what_;
    struct error_result { // delete copy, allow move
        result& r_; ctor(self& r) noexcept : r_(r) {} // ref member and init
        operator result<U>() noexcept {
            switch (auto k = r_.what_.kind()) {
                case val: return result<U>(error_id());
                case err_id_capture_list: return result<U>(r_.what_.get_error_id().value(), std::move(r_.cap_));
                default: return result<U>(r_.what_.get_error_id());
            }
        }
        operator error_id() const noexcept { return r_.what_.kind() == val ? error_id() : r_.what_.get_error_id(); }
    };
    void destroy() const noexcept {
        switch (auto k = what_.kind()) {
            case err_id_capture_list: cap_.~capture_list(); break;
            case val: stored_.~stored_type();
        }
    }
    result_discriminant move_from <U> (result<U>&& x) noexcept {
        switch (auto k =x.what_.kind()) {
            case err_id_capture_list: new(&cap_) capture_list(std::move(x.cap_)); break;
            case val: new(&stored_) stored_type(std::move(x.stored_));
        }
        return x.what_;
    }
    error_id get_error_id() const noexcept { return what_.get_error_id(); }
    stored_type <const>* get() <const> noexcept { return has_value() ? &stored_ : nullptr; }
protected:
    ctor(int err_id, capture_list&& cap) noexcept : cap_(std::move(cap)), what_(err_id, cap) {} // only when CFG_CAPTURE
    void enforce_value_state() const {
        switch (what_.kind()) {
            case err_id_capture_list: cap_.unload(what_.get_error_id().value());
            case err_id_zero: case err_id: throw_exception(get_error_id(), bad_result{});
        }
    }
    void move_assign <U> (result<U>&& x) noexcept { destroy(); what_ = move_from(std::move(x)); }
    void print_error_result(std::ostream& os) const {
        error_id const err_id = what_.get_error_id();
        os << "Error serial #" << err_id;
        if (what_.kind() == err_id_capture_list) { cap_.print(os, err_id, "\nCaptured:"); os << "\n"; }
    }
public: using value_type = T;
    ctor(self&& x) noexcept : what_(move_from(std::move(x))) {}
    ctor <U> (result<U>&& x) noexcept requires is_convertible_v<U,T> : what_(move_from(std::move(x))) {}
    ctor() : stored_(stored_type{}), what_(result_discriminant::kind_val{}) {}
    ctor(value_no_ref {const&|&&} v) :stored_(<std::forward<value_no_ref>>(v)), what_(result_discriminant::kind_val{}) {}
    ctor <...A> (A&&...a) noexcept requires is_constructible_v<T,A...> :stored_(std::forward<A>(a)...), what_(result_discriminant::kind_val{}) {}
    ctor(error_id err) noexcept : what_(err) {}
    ctor <A> (A&& a) noexcept requires is_constructible_v<T,A> && is_convertible_v<A,T>
        :stored_(std::forward<A>(a)), what_(result_discriminant::kind_val{}) {}
    ctor <Enum> (Enum e) noexcept requires is_error_code_enum_v<Enum> : what_(error_id(e)) {}
    ~dtor() noexcept { destroy(); }
    self & operator=(self&& x) noexcept { move_assign(std::move(x)); return *this;}
    self & operator= <U> (result<U>&& x) noexcept requires is_convertible_v<U,T> { move_assign(std::move(x)); return *this;}

    bool has_value() const noexcept { return what_.kind() == val; }
    bool has_error() const noexcept { return !has_value(); }
    explicit operator bool() const noexcept { return has_value(); }
    value_<c>ref value() <const>& { enforce_value_state(); return stored_; }
    value_rv_<c>ref value() <const>&& { enforce_value_state(); return std::move(stored_); }
    value_no_ref<_const>* operator->() <const>noexcept { return has_value()? stored<T>::<c>ptr(stored_):nullptr; }
    value_<c>ref operator*() <const>& noexcept { return *get(); }
    value_rv_<c>ref operator*() <const>&& noexcept { return std::move(*get()); }
    error_result error() noexcept { return error_result{*this}; }

    error_id load<...Item> (Item&&...item) noexcept { return error_id(error()).load(std::forward<Item>(item)...); }
    void unload() { if (what_.kind() == err_id_capture_list) cap_.unload(what_.get_error_id().value()); }

    friend std::ostream& operator<<(std::ostream& os, self const& r) {
        if (r.what_.kind() == val) print_result_value(os, r.value());
        else r.print_error_result(os);
        return os;
    }
};

struct detail::void_{};
class result<void> : result<detail::void_> {
    ctor(int err_id, capture_list&& cap) noexcept : base(err_id, std::move(cap)) {} // only when CFG_CAPTURE
public: // using base op==, op bool, get_error_id, error, load, unload
    ctor(self&& x) noexcept : base(std::move(x)) {}
    ctor() noexcept {} 
    ctor(error_id err) noexcept : base(err) {}
    ctor(std::error_code const& ec) noexcept : base(ec) {}
    ctor <Enum> (Enum e) noexcept requires is_error_code_enum_v<Enum> : base(e) {}
    ~dtor() noexcept {}
    self & operator=(self&& x) noexcept { move_assign(std::move(x)); return *this; }

    void value() const { enforce_value_state(); }
    void <const>* operator->() <const> noexcept { return base::operator->(); }
    void operator*() const noexcept {}

    friend std::ostream& operator<<(std::ostream& os, self const& r) {
        if (r) os << "No error";
        else r.print_error_result(os);
        return os;
    }
};

struct is_result_type<R>;
struct is_result_type<result<T>>: std::true_type {};
```

------
### Error Handling

* Headder `<boost/leaf/context.hpp>`

```c++
struct is_predicate<T>: false_type {};
struct detail::is_exception<T> : is_base_of<std::exception, std::decay_t<T>> {};
struct detail::handler_argument_traits<E>;
struct detail::handler_argument_traits_defaults<E,isPred=is_predicate<E>::value>;
struct detail::handler_argument_traits_defaults<E,false> {
    using error_type = std::decay_t<E>; using context_types = mp_list<error_type>;
    constexpr static bool always_available = false;
    constexpr static error_type <const>* check(Tup <const>&, error_info const&) noexcept;
    constexpr static E get<Tup> (Tup& tup, error_info const& ei) noexcept { return *check(tup, ei); }
};
struct detail::handler_argument_traits_defaults<Pred,false> : handler_argument_traits_defaults<Pred::error_type> {
    constexpr static bool check(Tup const& tup, error_info const& ei) noexcept
    { auto e = base::check(tup, ei); return e && Pred::evaluate(*e); }
    constexpr static Pred get<Tup> (Tup const& tup, error_info const& ei) noexcept { return Pred{*base::check(tup, ei)}; }
};

```

-----
* Headder `<boost/leaf/diagnostics.hpp>`

```c++
```

-----
* Headder `<boost/leaf/handle_errors.hpp>`

```c++
```

-----
* Headder `<boost/leaf/pred.hpp>`

```c++
```

-----
* Headder `<boost/leaf/to_variant.hpp>`

```c++
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
