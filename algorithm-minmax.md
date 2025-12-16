# Boost.Algorithm/Minmax

* lib: `boost/libs/algorithm`
* repo: `boostorg/algorithm`
* commit: `0edbfe8`, 2025-08-25

------
#### API

* Header `<boost/algorithm/minmax.hpp>`

```c++
tuple<T const&, T const&> minmax<T>(T const& a, T const& b);
tuple<T const&, T const&> minmax<T,BinPred>(T const& a, T const& b, BinPred comp);
```

* Header `<boost/algorithm/minmax_element.hpp>`

```c++
std::pair<FwdIt,FwdIt> minmax_element<FwdIt>(FwdIt f, FwdIt l);
std::pair<FwdIt,FwdIt> minmax_element<FwdIt,BinPred>(FwdIt f, FwdIt l, BinPred comp);
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Core

* `<boost/ref.hpp>`

#### Boost.Tuple

* `<boost/tuple/tuple.hpp>`

------
### Standard Facilities

Library: `minmax`, `minmax_element` (C++11)
