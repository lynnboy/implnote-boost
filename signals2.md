# Boost.Signals2

* lib: `boost/libs/signals2`
* repo: `boostorg/signals2`
* commit: `5dcb2af`, 2025-03-21

------
### API

```c++
void detail::adl_predestruct(...) {}
class postconstructor_invoker<T> {
    shared_ptr<T> _sp; mutable bool _postconstructed{false};
    ctor(const shared_ptr<T>& sp) : _sp(sp) {}
public:
    operator const shared_ptr<T>& () const { return postconstruct(); }
    const shared_ptr<T>& postconstruct<...Args>(Args&&...args) const
    { if (!_postconstructed) { adl_postconstruct(_sp, sp.get(), std::forward<Args>(args)...); _postconstructed=true;} return _sp; }
};
struct detail::sp_aligned_storage<n,a> { union type{ char data_[n]; boost::type_with_alignment<a>::type align_; }; };
class detail::deconstruct_deleter<T> {
    bool initialized_{false}; sp_aligned_storage<sizeof(T), alignment_of_v<T>>::type storage_;
    void destroy() { if (initialized_) { using adl_predestruct; adl_predestruct(p); p->~T(); initialized_=false; }} // remove const
public: ctor() {} ctor(self&const) {}
    ~dtor {destroy();} void operator()(T*) { destroy();}
    void* address() { return storage_.data_; }
    void set_initialized() { initialized_ = true; }
};
class deconstruct_access {
    static postconstructor_invoker<T> deconstruct<T,...Args>(Args&&...args) {
        shared_ptr<T> pt{nullptr, deconstruct_deleter<T>{}};
        auto* pd = get_deleter(pt); void* pv = pd->address();
        new(pv) T(std::forward<Args>(args)...); pd->set_initialized();
        shared_ptr<T> retval{pt, (T*)pv}; sp_enable_shared_from_this(&retval, retval.get(), retval.get());
        return retval;
    }
};
postconstructor_invoker<T> deconstruct<T,...Args>(Args&&...args) { return deconstruct_access::deconstruct<T>(std::forward<Args>(args)...); }

class postconstructible { // disabled ADL
protected: ctor(){} virtual ~dtor(){}
    virtual void postconstruct() = 0;
    friend void adl_postconstruct<T>(const shared_ptr<T>&, self* p) { p->postconstruct(); }
};
class predestructible { // disable ADL
protected: ctor(){} virtual ~dtor(){}
    virtual void predestruct() = 0;
    friend void adl_predestruct(self* p) { p->predestruct(); }
    friend void adl_postconstruct<T>(const shared_ptr<T>&, ...) {}
};

void detail::do_postconstruct(const postconstructible* ptr) { ptr->postconstruct(); } // remove const
void detail::do_postconstruct(...) {}
void detail::do_predestruct(const predestruct* ptr) { try {ptr->predestruct();} catch(...){assert(false);} }
void detail::do_predestruct(...) {}
struct predestructing_deleter { void operator()(const T* ptr) const { do_predestruct(ptr); checked_delete(ptr); } };
shared_ptr<T> deconstruct_ptr<T>(T* ptr) {
    if (!ptr) return shared_ptr<T>{ptr};
    shared_ptr<T> shared{ptr, predestructing_deleter<T>{}}; do_postconstruct(ptr); return shared;
}
shared_ptr<T,D> deconstruct_ptr<T>(T* ptr, D deleter) {
    shared_ptr<T> shared(ptr, deleter); if (!ptr) return shared;
    do_postconstruct(ptr); return shared;
}

struct dummy_mutex { void lock(){} bool try_lock() { retrun true; } void unlock() {} };

struct expired_slot: bad_weak_ptr { /* what*/ };
struct no_slots_error: std::exception { /*what*/ };

struct last_value<T> { using result_type = T;
    T operator() <InIt>(InIt first, InIt last) const {
        if (first == last) throw_exception(no_slots_error{});
        optional<T> value;
        for (;first != last;++first){ try{ value = move_if_not_lvalue_reference<T>(*first); }catch(const expired_slot&){} }
        if (value) return value.get(); else throw_exception(no_slots_error{});
    }
};
struct last_value<void> { using result_type = void;
    void operator() <InIt>(InIt first, InIt last) const
    { for (;first != last;++first){ try{ *first; }catch(const expired_slot&){} } }
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

#### Boost.Core

* `<boost/core/addressof.hpp>`
* `<boost/core/allocator_access.hpp>`
* `<boost/core/checked_delete.hpp>`
* `<boost/core/enable_if.hpp>`
* `<boost/core/invoke_swap.hpp>`
* `<boost/core/no_exceptions_support.hpp>`
* `<boost/core/noncopyable.hpp>`
* `<boost/core/ref.hpp>`
* `<boost/core/visit_each.hpp>`

#### Boost.Function

* `<boost/function.hpp>`

#### Boost.Iterator

* `<boost/iterator/reverse_iterator.hpp>`
* `<boost/iterator/iterator_traits.hpp>`

#### Boost.Move

* `<boost/move/utility_core.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.Optional

* `<boost/optional.hpp>`

#### Boost.Parameter

* `<boost/parameter/**.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.SmartPtr

* `<boost/scoped_ptr.hpp>`
* `<boost/shared_ptr.hpp>`, `<boost/shared_ptr/bad_weak_ptr.hpp>`
* `<boost/weak_ptr.hpp>`
* `<boost/smart_ptr/make_shared.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Tuple

* `<boost/tuple/tuple.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

#### Boost.Variant

* `<boost/variant/apply_visitor.hpp>`
* `<boost/variant/variant.hpp>`

------
### Standard Facilities

* Preprocessor: `__func__`, `__FILE__`, `__LINE__`.
* Standard Library: `<cassert>`, `source_location` (C++20)
