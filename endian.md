# Boost.Endian

* lib: `boost/libs/endian`
* repo: `boostorg/endian`
* commit: `900fe18`, 2024-04-21

------
### Endian Conversion

Header `<boost/endian/conversion.hpp>`

```c++
enum class order { big, little, native }; // native == big or little

[u]int{8,16,32,64}_t endian_reverse([u]int{8,16,32,64}_t x) noexcept; // use intrinsic if available

template <typename T> concept bool EndianReversible = CopyConstructible<T> &&
  requires (T x) { { endian_reverse(x) } -> T; };
template <typename T> concept bool EndianReversibleInPlace = CopyConstructible<T> &&
  requires (T x) { endian_reverse_inplace(x); };

EndianReversible {big,little}_to_native(EndianReversible x) noexcept; // endian_reverse(x) if need
EndianReversible native_to_{big,little}(EndianReversible x) noexcept; // ditto
EndianReversible conditional_reverse<order, order>(EndianReversible x) noexcept; // ditto
EndianReversible conditional_reverse(EndianReversible x, order, order) noexcept; // ditto

void endian_reverse_inplace(EndianReversible& x) noexcept; // x = endian_reverse(x)
void {big,little}_to_native_inplace(EndianReversibleInplace& x) noexcept; // endian_reverse_inplace(x) or x
void native_to_{big,little}_inplace(EndianReversibleInplace& x) noexcept; // ditto
void conditional_reverse_inplace<order, order>(EndianReversibleInplace& x) noexcept; // ditto
void conditional_reverse_inplace(EndianReversibleInplace& x, order, order) noexcept; // ditto

intN_t load_{big,little}_{s,u}{16,24,32,40,48,56,64}(unsigned char const*) noexcept;
void store_{big,little}_{s,u}{16,24,32,40,48,56,64}(unsigned char const*, intN_t) noexcept;
```

Overload `endian_reverse` for custom types.

------
### Endian-aware Buffer Types

Header `<boost/endian/buffers.hpp>`

```c++
enum class align { no, yes };

template <order Order, typename T, std::size_t n_bits, align A=align::no>
class endian_buffer {
  unsigned char value_[n_bits/8];
public:
  typedef T value_type;
  endian_buffer() noexcept = default;
  explicit endian_buffer(T v) noexcept;
  endian_buffer& operator=(T v) noexcept;
  value_type value() const noexcept;
  unsigned char [const]* data() [const] noexcept;
};
std::basic_ostream<C,T>& operator<<(std::basic_ostream<C,T>&, endian_buffer<O,T,N,A> const &);
std::basic_istream<C,T>& operator>>(std::basic_istream<C,T>&, endian_buffer<O,T,N,A> &);

using {big,little}_[u]int{8,16,32,64}_buf_at = // exact size aligned buffer
  endian_buffer<order::{big,little}, [u]int{8,16,32,64}, {8,16,32,64}, align::yes>;

using {big,little}_float{32,64}_buf_at =
  endian_buffer<order::{big,little}, {float,double}, {32,64}, align::yes>;

using {big,little,native}_[u]int{8,16,24,32,40,48,56,64}_buf_t = // unaligned buffer
  endian_buffer<order::{big,little,native}, [u]int_least{8,16,32,64}, {8,16,24,32,40,48,56,64}>;

using {big,little,native}_float{32,64}_buf_t =
  endian_buffer<order::{big,little,native}, {float,double}, {32,64}, align::no>;
```

* Use `native_to_{big,little}` conversions internally.
* Specializations for aligned buffers store typed value.
* Specializations for unaligned buffers store value in char array.
* Supports stream I/O `<<` & `>>` for `endian_buffer`.

------
### Endian Specified Arithmetic Types (Endian Types)

Header `<boost/endian/arithmetic.hpp>`

```c++
template <order Order, typename T, std::size_t n_bits, align A=align::no>
class endian_arithmetic {
  typedef endian_buffer<Order, T, n_bits, Align> buffer_type;
  buffer_type buf_;
public:
  typedef T value_type;

  endian_arithmetic() noexcept = default;
  endian_arithmetic(T v) noexcept :buf_(v) {} // implicit
  endian_arithmetic& operator=(T v) noexcept;
  value_type value() const noexcept { return buf_.value(); }
  unsigned char [const]* data() [const] noexcept { return buf_.data(); }
  operator value_type() const noexcept { return value(); } // implicit
  operator buffer_type [const]& () noexcept { return buf_; }

  T operator+() const noexcept { return value(); }
  endian_arithmetic & operator{+,-,*,/,%,&,|,^,<<,>>}=(T y) noexcept;
  endian_arithmetic & operator{++,--}([int]) noexcept;
  friend std::basic_ostream<C,T>& operator<<(std::basic_ostream<C,T>&, endian_arithmetic const &);
  friend std::basic_istream<C,T>& operator>>(std::basic_istream<C,T>&, endian_arithmetic &);
};

using {big,little}_[u]int{8,16,32,64}_at = // exact size aligned
  endian_arithmetic<order::{big,little}, [u]int{8,16,32,64}, {8,16,32,64}, align::yes>;

using {big,little}_float{32,64}_at =
  endian_arithmetic<order::{big,little}, {float,double}, {32,64}, align::yes>;

using {big,little,native}_[u]int{8,16,24,32,40,48,56,64}_t = // unaligned
  endian_arithmetic<order::{big,little,native}, [u]int_least{8,16,32,64}, {8,16,24,32,40,48,56,64}>;

using {big,little,native}_float{32,64}_t =
  endian_arithmetic<order::{big,little,native}, {float,double}, {32,64}, align::no>;
```

Full operator set is covered according to implicit conversion to `value_type`.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/header_deprecated.hpp>`

------
### Standard Facilities
