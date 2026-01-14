# Boost.Range

* lib: `boost/libs/iostreams`
* repo: `boostorg/iostreams`
* commit: `8561bdb`, 2024-12-16

------
### Concept

```c++
// Devices
concept Device<D, Ch,Cat> = requires(D dev, Ch* s1, const Ch* s2, std::streamsize n, stream_offset off,
    ios_base::seekdir way, ios_base::openmode which) {
    {char_type_of<D>::type}->same_as<Ch>; {category_of<D>::type}->same_as<Cat>;
    {io::read(dev,s1,n)}->same_as<std::streamsize>; // when Cat->input && !Cat->direct_tag
    {dev.input_sequence()}->same_as<std::pair<Ch*,Ch*>>; // when Cat->input && Cat->direct_tag
    {io::write(dev,s2,n)}->same_as<std::streamsize>; // when Cat->output && !Cat->direct_tag
    {dev.output_sequence()}->same_as<std::pair<Ch*,Ch*>>; // when Cat->output && Cat->direct_tag
    {io::seek(dev,off,way)}->same_as<std::streampos>; // when Cat->seekable && !Cat->direct_tag
    {dev.seek(dev,off,way,which)}->same_as<std::streampos>; // when Cat->dual_seekable || Cat->bidirectional_seekable && !Cat->direct_tag
};

concept Source<D,Ch,Cat> = Device<D,Ch,Cat> && convertible_to<Cat,input>; // read, input_sequence
concept Sink<D,Ch,Cat> = Device<D,Ch,Cat> && convertible_to<Cat,output>; // write, output_sequence
concept Peekable<D,Ch,Cat> = Source<D,Ch,Cat> && requires(D p, Ch c) { {io::putback(p,c)->same_as<bool>; }};
concept Blocking<D,Ch,Cat> = Device<D,Ch,Cat>;
concept Direct<D,Ch,Cat> = Device<D,Ch,Cat>;

concept BidirectionalDevice<D,Ch,Cat> = Source<D,Ch,Cat> && Sink<D,Ch,Cat>; // read/write, input/output_seqeunce
concept SeekableDevice<D,Ch,Cat> = Source<D,Ch,Cat> && Sink<D,Ch,Cat>; // read/write/seek, input/output_seqeunce

// Filters
concept Filter<F,Ch,Cat,Mode>

// Other
```

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`
* `<boost/cstdint.hpp>`
* `<boost/config/auto_link.hpp>`
* `<boost/config/abi_prefix.hpp>`, `<boost/config/abi_suffix.hpp>`

#### Boost.Core

* `<boost/checked_delete.hpp>`
* `<boost/core/type.hpp>`
* `<boost/core/typeinfo.hpp>`
* `<boost/noncopyable.hpp>`
* `<boost/ref.hpp>`
* `<boost/utility/enable_if.hpp>`, `<boost/core/enable_if.hpp>`

#### Boost.Detail

* `<boost/detail/is_incrementable.hpp>`

#### Boost.Function

* `<boost/function.hpp>`

#### Boost.Integer

* `<boost/integer_traits.hpp>`

#### Boost.Iterator

* `<boost/next_prior.hpp>`

#### Boost.MPL

* `<boost/mpl/*.hpp>`

#### Boost.Numeric/Conversion

* `<boost/numeric/conversion/cast.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

#### Boost.Random

* `<boost/random/linear_congruential.hpp.hpp>`
* `<boost/random/uniform_smallint.hpp>`

#### Boost.Range

* `<boost/range/iterator_range.hpp>`
* `<boost/range/value_type.hpp>`

#### Boost.Regex

* `<boost/regex.hpp>`

#### Boost.SmartPtr

* `<boost/shared_ptr.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

#### Boost.Utility

* `<boost/call_traits.hpp>`
* `<boost/utility/base_from_member.hpp>`
* `<boost/utility/result_of.hpp>`

------
### Standard Facilities
