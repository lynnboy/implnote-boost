# Boost.Contract

* lib: `boost/libs/contract`
* repo: `boostorg/contract`
* commit: `4a4047f`, 2025-06-09

------
#### Commons

```c++
struct exception { virtual ~dtor(); };
class bad_virtual_result_cast: public std::bad_cast, public exception {
    std::string what_;
public: explicit ctor(char const* from_type_name, char const* to_type_name); ~dtor();
};
class assertion_failure : public std::exception, public exception {
    char const* file_; unsigned long line_; char const* code_; std::string what_;
public: explicit ctor(char const* file="", unsigned long line=0, char const* code="");
    explicit ctor(char const* code); ~dtor();
    char const* what() const; char const* file() const; unsigned long line() const; char const* code() const;
};
enum from { from_constructor, from_destructor, from_function };
using from_failure_handler = function<void(from)>;
using failure_handler = function<void()>;
namespace exception_ {
failure_handler const& set_check_failure_<un>locked(failure_handler const& f) noexcept;
failure_handler get_check_failure_<un>locked() noexcept;
void check_failure_<un>locked();
from_failure_handler const& set_{pre|post|except|old|entry_inv|exit_inv}_failure_<un>locked(from_failure_handler const& f) noexcept;
from_failure_handler get_{pre|post|except|old|entry_inv|exit_inv}_failure_<un>locked() noexcept;
void {pre|post|except|old|entry_inv|exit_inv}_failure_<un>locked(from where);
}
failure_handler const& set_check_failure(failure_handler const& f);
failure_handler get_check_failure();
void check_failure();
from_failure_handler const& set_{pre|post}condition_failure(from_failure_handler const& f);
from_failure_handler get_{pre|post}condition_failure();
void {pre|post}condition_failure(from where);
from_failure_handler const& set_{except|old}_failure(from_failure_handler const& f);
from_failure_handler get_{except|old}_failure();
void {except|old}_failure(from where);
from_failure_handler const& set_{entry|exit}_invariant_failure(from_failure_handler const& f);
from_failure_handler get_{entry|exit}_invariant_failure();
void {entry|exit}_invariant_failure(from where);
from_failure_handler const& set_invariant_failure(from_failure_handler const& f);
```

#### Asserts

* `CHECK(cond)`, `CHECK_AUDIT(cond)`, `CHECK_AXIOM(cond)`
* `ASSERT(cond)`, `ASSERT_AUDIT(cond)`, `ASSERT_AXIOM(cond)`

#### Core Classes

```c++
struct specify_nothing{};
struct specify_except {
    specify_nothing except<F>(F const& f);
};
struct specify_postcondition_except<VirtualResult> {
    specify_except postcondition<F>(F const& f);
    specify_nothing except<F>(F const& f);
};
struct specify_old_postcondition_except<VirtualResult> {
    specify_postcondition_except<VirtualResult> old(F const& f);
    specify_except postcondition<F>(F const& f);
    specify_nothing except<F>(F const& f);
};
struct specify_precondition_old_postcondition_except<VR> {
    specify_old_postcondition_except<VirtualResult> precondition(F const& f);
    specify_postcondition_except<VirtualResult> old(F const& f);
    specify_except postcondition<F>(F const& f);
    specify_nothing except<F>(F const& f);
};
class virtual_ {
    enum action_enum { no_action, check_entry_inv, check_pre, check_exit_inv,
        push_old_init_copy, call_old_ftor, push_old_ftor_copy, pop_old_ftor_copy, check_post, check_except };
    action_enum action_; bool failed_;
    std::queue<shared_ptr<void>> old_init_copies_, old_ftor_copies_;
    any result_ptr_; char const* result_type_name_; bool result_optional_;
};
struct constructor_precondition<Class> {
    ctor(); explicit ctor<F>(F const& f);
};

```

Usage:
```c++
int foo(int& x) {
    int result;
    boost::contract::old_ptr<int> old_x = BOOST_CONTRACT_OLDOF(x);
    boost::contract::check c = boost::contract::function()
        .precondition([&] {
            BOOST_CONTRACT_ASSERT(x < std::numeric_limits<int>::max());
        })
        .postcondition([&] {
            BOOST_CONTRACT_ASSERT(x == *old_x + 1);
            BOOST_CONTRACT_ASSERT(result == *old_x);
        })
        .except([&] {
            BOOST_CONTRACT_ASSERT(x == *old_x);
        })
    ;

    return result = x++; // Function body.
}
```

#### `call_if`

```c++
struct call_if_statement<pred,Then,ThenResult=none>{};
struct call_if_statement<true,Then,none> : call_if_statement<true,Then,result_of_t<Then()>>{};
struct call_if_statement<true,Then,ThenResult> {
  explicit ctor(Then f) : r_{make_shared<ThenResult>{f()}}{}
  operator ThenResult() const { return *r_; }
  ThenResult else_<Else>(Else const& f) const { return *r_; }
  self else_if_c<elseIfPred,ElseIfThen>(ElseIfThen const& f) const { return *this; }
  self else_if<ElseIfPred,ElseIfThen>(ElseIfThen const& f) const { return *this; }
private: shared_ptr<ThenResult> r_;
};
struct call_if_statement<true,Then,void> {
  explicit ctor(Then f) {f();}
  void else_<Else>(Else const& f) const {}
  self else_if_c<elseIfPred,ElseIfThen>(ElseIfThen const& f) const { return *this; }
  self else_if<ElseIfPred,ElseIfThen>(ElseIfThen const& f) const { return *this; }
};
struct call_if_statement<false,Then,none> {
  explicit ctor(Then const& f) {}
  result_of_t<Else()> else_<Else>(Else f) const { return f(); }
  self<elseIfPred,ElseIfThen> else_if_c<elseIfPred,ElseIfThen>(ElseIfThen f) const { return {f}; }
  self<ElseIfPred::value,ElseIfThen> else_if<ElseIfPred,ElseIfThen>(ElseIfThen f) const { return {f}; }
};
call_if_statement<pred,Then> call_if_c<pred,Then>(Then f) { return {f}; }
call_if_statement<Pred::value,Then> call_if<Pred,Then>(Then f) { return {f}; }
bool condition_if_c<pred,Then>(Then f, bool else_=true) { return pred ? f() : else_; }
bool condition_if<Pred,Then>(Then f, bool else_=true) { return condition_if_c<Pred::value>(f, else_); }
```

#### Checking Entrances

```c++
class check {
public: ctor<F>(F const& f) { CHECK({f();}) }
  ctor(self const& other) : cond_{((check&)other).cond_.release()}{}
  ctor<VirtualResult> (specify_precondition_old_postcondition_except<VirtualResult> const& contract);
  ctor<VirtualResult> (specify_old_postcondition_except<VirtualResult> const& contract);
  ctor<VirtualResult> (specify_postcondition_except<VirtualResult> const& contract);
  ctor<VirtualResult> (specify_except const& contract);
  ctor<VirtualResult> (specify_nothing const& contract);
  ~dtor();
private: auto_ptr<cond_base> cond_;
};

specify_old_postcondition_except<> constructor<Class>(Class* obj);
specify_old_postcondition_except<> destructor<Class>(Class* obj);
specify_old_postcondition_except<> function();
specify_precondition_old_postcondition_except<> public_function<Class>();
specify_precondition_old_postcondition_except<> public_function<Class>(Class* obj);
specify_precondition_old_postcondition_except<> public_function<Class>(virtual_* v, Class* obj);
specify_precondition_old_postcondition_except<VirtualResult> public_function<VirtualResult,Class>(virtual_* v, VirtualResult& r, Class* obj);
specify_precondition_old_postcondition_except<> public_function<Override,F,Class,...Args>(virtual_* v, F f, Class* obj, Args&...args);
specify_precondition_old_postcondition_except<VirtualResult> public_function<Override,VirtualResult,F,Class,...Args>(virtual_* v, VirtualResult& r, F f, Class* obj, Args&...args);

struct is_old_value_copyable<T> : is_copy_constructible<T> {};
struct is_old_value_copyable<old_value> : true_type {};
struct old_value_copy<T> {
  explicit ctor(T const& old) : old_{old}{}
  T const& old() const { return old_; }
private: T const old_;
};
class old_ptr<T> {
public: using element_type = T;
  ctor(){}
  T const& operator*() const { return typed_copy_->old(); }
  T const* operator->() const { if (typed_copy_) return &typed_copy_->old(); return 0; }
  explicit operator bool() const;
private: shared_ptr<old_value_copy<T>> typed_copy_;
};
class old_ptr_if_copyable<T> {
public: using element_type = T;
  ctor(){}
  ctor(old_ptr<T> const& other) : typed_copy_{other.typed_copy_}{}
  T const& operator*() const { return typed_copy_->old(); }
  T const* operator->() const { if (typed_copy_) return &typed_copy_->old(); return 0; }
  explicit operator bool() const;
private: shared_ptr<old_value_copy<T>> typed_copy_;
};
class old_value {
public: ctor<T>(T const& old) : untyped_copy_{new old_value_copy<T>(old)}{}
  ctor<T>(T const& old){}
private: shared_ptr<void> untyped_copy_;
};
class old_pointer {
public: operator old_ptr_if_copyable<T>() { return get<old_ptr_if_copyable<T>>(); }
  operator old_ptr<T>() { return get<old_ptr<T>>(); }
private: explicit ctor(virtual_* v, old_value const& old): v_{v}, untyped_copy_{old.untyped_copy_}{}
  Ptr get<Ptr>();
  virtual_* v_; shared_ptr<void> untyped_copy_;
};
old_value null_old();
old_pointer make_old(old_value const& old);
old_pointer make_old(virtual_* v, old_value const& old);
bool copy_old();
bool copy_old(virtual_* v);
```

* `OLDOF(par)`

#### Invariants

```c++
class T {
public: // must be public
  void invariant() const {...}
  static void static_invariant() {...}
};
```

#### Subcontracting

* `BASE_TYPES(...)`, `OVERRIDE(name)`, `OVERRIDES(name, list)`

Usage:
```c++
#define BASES public B1, private B2
class T : BASES {
public:     typedef BOOST_CONTRACT_BASE_TYPES(BASES) base_types; // Bases typedef.
#undef BASES
    void foo() override{}
    BOOST_CONTRACT_OVERRIDE(foo)
};
```

-----
### Configuration

* `DYN_LINK`, `STATIC_LINK`, `HEADER_ONLY`, `DISABLE_THREADS`
* `MAX_ARGS`: 10, `BASES_TYPEDEF`: `base_types`, `INVARIANT_FUNC`: `invariant`, `STATIC_INVARIANT_FUNC`: `static_invariant`
* `PERMISSIVE`, `ON_MISSING_CHECK_DECL`, `PRECONDITIONS_DISABLE_NO_ASSERTION`, `ALL_DISABLE_NO_ASSERTION`, `AUDITS`
* `NO_CHECKS`, `NO_PRECONDITIONS`, `NO_POSTCONDITIONS`, `NO_EXCEPTS`, `NO_ENTRY_INVARIANTS`, `NO_EXIT_INVARIANTS`, `NO_INVARIANTS`

-----
### Dependencies

#### Boost.Any

* `<boost/any.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/config/auto_link.hpp>`

#### Boost.Core

* `<boost/noncopyable.hpp>`
* `<boost/utility/enable_if.hpp>`

#### Boost.Exception

* `<boost/exception/diagnostic_information.hpp>`

#### Boost.Function

* `<boost/function.hpp>`

#### Boost.FunctionTypes

* `<boost/function_types/function_arity.hpp>`
* `<boost/function_types/parameter_types.hpp>`
* `<boost/function_types/property_tags.hpp>`
* `<boost/function_types/result_type.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.Optional

* `<boost/optional.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.SmartPtr

* `<boost/make_shared.hpp>`
* `<boost/shared_ptr.hpp>`
* `<boost/smart_ptr/allocate_unique.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.Thread

* `<boost/thread/lock_guard.hpp>`
* `<boost/thread/mutex.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`
* `<boost/utility/declval.hpp>`

#### Boost.TypeOf

* `<boost/typeof/typeof.hpp>`

#### Boost.Utility

* `<boost/utility/result_of.hpp>`

------
### Standard Facilities

Language: Contracts (C++26)
