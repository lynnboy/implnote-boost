# Boost.LocalFunction

* lib: `boost/libs/local_function`
* repo: `boostorg/local_function`
* commit: `ff0934f`, 2025-06-26

------
### Usage

```c++
struct n {/*...*/};
BOOST_TYPEOF_REGISTER_TYPE(n) // required by capturing

int main(){
  n x{};
  void BOOST_LOCAL_FUNCTION(const bind& x) { // type name is __LINE__ encoded
    x.value; // use x
  } BOOST_LOCAL_FUNCTION_NAME(f) // object name
  f(); // invoke op() of f
}
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`

#### Boost.MPL

* `<boost/mpl/**.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.ScopeExit

* `<boost/scope_exit.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

#### Boost.TypeOf

* `<boost/typeof/typeof.hpp>`

#### Boost.Utility

* `<boost/call_traits.hpp>`
* `<boost/utility/identity_type.hpp>`

------
### Standard Facilities
Language: Lambda expression (C++11)
