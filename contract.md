# Boost.Contract

* lib: `boost/libs/contract`
* repo: `boostorg/contract`
* commit: `4a4047f`, 2025-06-09

contract, contract_macro
call_if, check, constructor, destructor, function, old, public_function
core/: access, check_macro, constructor_precondition, exception
detail/: assert, auto_ptr, check, checking, config, debug, declspec, inlined, name, none, operator_safe_bool, static_local_var
detail/condition/: cond_base, cond_inv, cond_post, cond_subcontracting
detail/inlined/: core/exception, detail/checking, old
detail/operation/: constructor, destructor, function, public_function, static_public_function
detail/type_traits/: member_function_types, mirror, optional

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
