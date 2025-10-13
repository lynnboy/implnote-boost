# Boost.LEAF - Lightweight Error Augmentation Framework

* lib: `boost/libs/leaf`
* repo: `boostorg/leaf`
* commit: `61e4b0d`, 2025-06-02

------
### Common Bits

* Header `<boost/leaf/common.hpp>`

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

struct e_api_function { char const * value; };
struct e_file_name { std::string value; };
struct e_errno { int value; explicit ctor(int v=errno); }; // and hidden friend op<<
struct e_type_info_name { char const * value; };
struct e_at_line { int value; };
struct windows::e_LastError { unsigned value; explicit ctor(unsigned val); ctor() : value(GetLastError()) {} }; // and op<<
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
    T<const>* has_value_and_key()<const> noexcept { return key_ ? &value_ : nullptr; }
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
