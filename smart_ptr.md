# Boost.SmartPtr

* lib: `boost/libs/smart_ptr`
* repo: `boostorg/smart_ptr`
* commit: `5a0b5ea`, 2025-09-24

------
### Scoped Pointer/Array

```c++
class scoped_ptr<T> { // no copy, no self equality
    T* px;
public: using element_type = T;
    explicit ctor(T* p=0) noexcept :px{p}{}
    ~dtor() noexcept { checked_delete(px); }
    void reset(T* p=0) noexcept { self(p).swap(*this); }
    T& operator*() const noexcept { return *px; }
    T* operator->() const noexcept { return px; }
    T* get() const noexcept { return px; }
    explicit operator bool() const noexcept { return px!=0; }
    void swap(self& b) noexcept { T* tmp=b.px; b.px=px; px=tmp; }
};
bool operator==(scoped_ptr<T> const& p, nullptr_t) { return p.get() == 0; } // and !=, and inversed
void swap<T>(scoped_ptr<T>& a, scoped_ptr<T>& b) noexcept { a.swap(b); }
T* get_pointer<T>(scoped_ptr<T> const& p) noexcept { return p.get(); }

class scoped_array<T> { // no copy, no self equality
    T* px;
public: using element_type = T;
    explicit ctor(T* p=0) noexcept :px{p}{}
    ~dtor() noexcept { checked_array_delete(px); }
    void reset(T* p=0) noexcept { self(p).swap(*this); }
    T& operator[](ptrdiff_t i) const noexcept { return px[i]; }
    T* get() const noexcept { return px; }
    explicit operator bool() const noexcept { return px!=0; }
    void swap(self& b) noexcept { T* tmp=b.px; b.px=px; px=tmp; }
};
bool operator==(scoped_array<T> const& p, nullptr_t) { return p.get() == 0; } // and !=, and inversed
void swap<T>(scoped_array<T>& a, scoped_array<T>& b) noexcept { a.swap(b); }
```

------
### Pointer Cast & Traits

```c++
// all 4 casts
T* static_pointer_cast<T,U>(U* ptr) noexcept { return static_cast<T*>(ptr); }
using std::static_pointer_cast; // for std::shared_ptr
std::unique_ptr<T> static_pointer_cast<T,U>(std::unique_ptr<U>&& r) noexcept
{ return { static_cast<std::unique_ptr<T>::element_type*>(r.release()) }; }

struct pointer_to_other<T,U>;
struct pointer_to_other<Sp<T>,U> { using type = Sp<U>; };
struct pointer_to_other<Sp<T,T2>,U> { using type = Sp<U,T2>; };
struct pointer_to_other<Sp<T,T2,T3>,U> { using type = Sp<U,T2,T3>; };
struct pointer_to_other<T*,U> { using type = U*; };
```

------
### Configuratino

* `SP_ENABLE_DEBUG_HOOKS`:
    * `sp_{scalar|array}_constructor_hook`, `sp_{scalar|array}_destructor_hook`

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>` - used for debugging

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`
* `<boost/config/header_deprecated.hpp>`
* `<boost/config/pragma_message.hpp>`
* `<boost/cstdint.hpp>`

#### Boost.Core

* `<boost/core/addressof.hpp>`
* `<boost/core/allocator_access.hpp>`
* `<boost/core/alloc_construct.hpp>`
* `<boost/core/checked_delete.hpp>`
* `<boost/core/default_allocator.hpp>`
* `<boost/core/empty_value.hpp>`
* `<boost/core/first_scalar.hpp>`
* `<boost/core/noinit_adaptor.hpp>`
* `<boost/core/pointer_traits.hpp>`
* `<boost/core/typeinfo.hpp>`
* `<boost/core/yield_primitives.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

------
### Standard Facilities

Library: `shared_ptr`, `weak_ptr`, `unique_ptr`, `static_pointer_cast` family (C++11)
