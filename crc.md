# Boost.CRC

* lib: `boost/libs/crc`
* repo: `boostorg/crc`
* commit: `d547322`, 2025-08-23

------
### CRC Checksum Calculator

#### Header

* `<boost/crc.hpp>`

------
#### Bit-wise (Slow) CRC Calculator

```c++
template<std::size_t Bits>
class crc_basic {
public:
  typedef uint_t<Bits>::fast value_type;
  static const std::size_t bit_count = Bits;

  explicit crc_basic(value_type truncated_polynomial
    value_type initial_remainder = 0, value_type final_xor_value = 0,
    bool reflect_input = false, bool reflect_remainder = false);

  // accessors for the five arguments: 'get_xxx() const'
  value_type get_interim_remainder() const;   void reset(value_type new_rem);
  void reset();
  
  void process_bit(bool);
  void process_bits(unsigned char bits, std::size_t bit_length);
  void process_byte(unsigned char);
  void process_block(void const*, void const*);
  void process_bytes(void const*, std::size_t);
  
  value_type checksum() const;
};
```

------
#### Optimized Word-wise CRC Calculator

```c++
template<std::size_t Bits, uint_t<Bits>::fast TruncPoly=0, // static version of attributes
  uint_t<Bits>::fast InitRem=0, uint_t<Bits>::fast FinalXor=0,
  bool ReflectIn=false, bool ReflectRem=false>
class crc_optimal {
public:
  typedef uint_t<Bits>::fast value_type;
  static const std::size_t bit_count = Bits;
  // constants for the five arguments

  explicit crc_optimal(value_type init_rem = initial_remainder);

  // accessors for the five arguments: 'get_xxx() const'

  value_type get_interim_remainder() const;
  void reset(value_type new_rem = initial_remainder);

  void process_byte(unsigned char);             void operator() (unsigned char);
  void process_block(void const*, void const*);
  void process_bytes(void const*, std::size_t);
  
  value_type checksum() const;                  value_type operator() () const;
};

using crc_16_type =                                     // ARC, CRC-16, CRC-IBM etc.
  crc_optimal<16, 0x8005, 0, 0, true, true>;
using crc_ccitt_type = crc_ccitt_false_t =              // CRC-16/CCITT-FALSE
  crc_optimal<16, 0x1021, 0xFFFF, 0, false, false>;
using crc_ccitt_true_t =                                // CRC-16/CCITT, KERMIT, etc.
  crc_optimal<16, 0x1021, 0, 0, true, true>;
using crc_xmodem_t =                                    // XMODEM, CRC-16/ACORN, etc.
  crc_optimal<16, 0x1021, 0, 0, false, false>;
using crc_32_type =                                     // CRC-32, PKZIP, etc.
  crc_optimal<32, 0x04C11DB7, 0xFFFFFFFF, 0xFFFFFFFF, true, true>;
```

* Use strategy types to handle input/checksum reflection
* Use strategy types to make crc lookup table for different cases:
  * For `Bits > CHAR_BIT` normal case and optimize for `Bits <= CHAR_BIT`
  * For reflected or not reflected
* Lookup tables are static local `boost::array` variables, initialized at first use.

------
#### CRC Functions

```c++
template < std::size_t Bits, uint_t<Bits>::fast TruncPoly,
           uint_t<Bits>::fast InitRem, uint_t<Bits>::fast FinalXor,
           bool ReflectIn, bool ReflectRem >
uint_t<Bits>::fast crc(void const *, std::size_t);    // use crc_optimal to calculate

template < std::size_t Bits, uint_t<Bits>::fast TruncPoly >
uint_t<Bits>::fast augmented_crc(void const *, std::size_t, uint_t<Bits>::fast initial_remainder);
```

------
### Dependency

------
### Standard Facilities
