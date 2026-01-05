# Boost.Utility/CompressedPair

* lib: `boost/libs/utility`
* repo: `boostorg/utility`
* commit: `dda83c5`, 2025-08-26

------
### Compressed Pair

* Header `<boost/compressed_pair.hpp>`

```c++
using detail::compressed_pair_empty<T> = std::bool_constant<!is_final_v<T> && is_empty_v<T>>;
struct detail::compressed_pair_switch<T1,T2, isSame, empty1, empty2> {
    static const int value = (!isSame) ?
        (!empty1 && !empty2) ? 0 : (empty1 && !empty2) ? 1 : (!empty1 && empty2) ? 2 : 3 :
        empty1 ? 4 : 5;
};
void detail::cp_swap<T>(T& t1, T& t2) { using std::swap; swap(t1, t2); }

class detail::compressed_pair_imp<T1,T2,0> {
    T1 first_; T2 second_;
public: using first_type = T1; using second_type = T2;
    using {first|second}_param_type = call_traits<{first|second}_type>::param_type;
    using {first|second}_<const>_reference = call_traits<{first|second}_type>::<const>_reference;
    ctor(); ctor(first_param_type x, second_param_type y); ctor(first_param_type x);  ctor(second_param_type y); 
    first_<const>_reference first() <const>; second_<const>_reference second() <const>;
    void swap(self& y) { cp_swap(first_, y.first()); cp_swap(second_, y.second()); }
};
class detail::compressed_pair_imp<T1,T2,1> : protected remove_cv_t<T1> {
    T2 second_;
public: // same types and members, first is empty base subobj, swap only second_
};
class detail::compressed_pair_imp<T1,T2,2> : protected remove_cv_t<T2> {
    T1 first_;
public: // same types and members, second is empty base subobj, swap only first__
};
class detail::compressed_pair_imp<T1,T2,3> : protected remove_cv_t<T1>, protected remove_cv_t<T2> {
public: // same types and members, both are empty base subobj, swap noop
};
class detail::compressed_pair_imp<T1,T2,4> : protected remove_cv_t<T1> { // same empty type
    T2 second_; // first is empty base subobj, store second separately, to give different address
public: // same types and members, no ctor for second, swap noop
};
class detail::compressed_pair_imp<T1,T2,2> : protected remove_cv_t<T2> { // same nonempty type
    T1 first_; T2 second_;
public: // same types and members, no ctor for second, swap both
};

struct compressed_pair<T1,T2> : private compressed_pair_imp<T1,T2,
    compressed_pair_switch<T1,T2,is_same_v<remove_cv_t<T1>,remove_cv_t<T2>>,compressed_pair_empty<T1>::value,compressed_pair_empty<T2>::value>::value> {
    // all types and members from base
};
struct compressed_pair<T,T> : private compressed_pair_imp<T,T,
    compressed_pair_switch<T1,T2,true,compressed_pair_empty<T>::value,compressed_pair_empty<T>::value>::value> {
    // all types and members from base
};
void swap<T1,T2>(compressed_pair<T1,T2>& x, compressed_pair<T1,T2>& y);
```

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/**.hpp>`

#### Boost.Utility/CallTraits

* `<boost/call_traits.hpp>`

------
### Standard Facilities
