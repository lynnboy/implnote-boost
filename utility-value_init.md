# Boost.Utility/ValueInit

* lib: `boost/libs/utility`
* repo: `boostorg/utility`
* commit: `dda83c5`, 2025-08-26

------
### `initialized` and `value_initialized`

* Header `<boost/utility/value_init.hpp>`

```c++
struct detail::zero_init{ ctor(); ctor(void*p, size_t n) { memset(p,0,n); } };
class initialized<T> : zero_init { // force memset 0 by ctor()
    T data_;
public: ctor() : base(&data_, sizeof(T)) {}
    ctor(T const& arg): data_(arg) {}
    T <const>& data() <const> { return data_; }
    void swap(self& arg) { invoke_swap(data(), arg.data()); }
    operator T <const>&() <const>;
};
T <const>& get<T>( initialized<T><const>& x) { return x.data(); }
void swap<T> (initialized<T>& x, initialized<T>& y) { x.swap(y); }

class value_initialized<T> { // another layer wrapping T
    initialized<T> m_data;
public: // default ctors
    T<const>& data() <const>;
    void swap(self&);
    operator T<const>&() <const>;
};
T <const>& get<T>( value_initialized<T><const>& x) { return x.data(); }
void swap<T> (value_initialized<T>& x, value_initialized<T>& y) { x.swap(y); }

struct initialized_value_t { operator T() const { return initialized<T>().data(); } };
initialized_value_t const initialized_value = {} // get initialized rvalue by conv-op
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Core

* `<boost/core/invoke_swap.hpp>`

------
### Standard Facilities
