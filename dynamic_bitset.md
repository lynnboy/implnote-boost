# Boost.DynamicBitset

* lib: `boost/libs/dynamic_bitset`
* repo: `boostorg/dynamic_bitset`
* commit: `340822f9`, 2015-06-03

------
### Dynamic bitset

#### Header

* `<boost/dynamic_bitset.hpp>`

#### API difference with `std::bitset`

* Argumented by `Block` type and `Allocator`, by default are `ulong` and `std::allocator<Block>`
* Wraps a `std::vector<Block>` for internal storage.
* Member types `block_type`, `allocator_type`, `size_type`
* Member constants `bits_per_block` and `npos`
* Member `reference` provides `|=`, `&=`, `^=`, and `-=` (in addition to `std::bitset::reference`)
* Constructors have additional allocator parameter
* Provides Block-range oriented constructor and `assign`, `append`, and `to_block_range`, `from_block_range` functions.
* Additionally provides move-constructor and move-assignment, and `swap`
* Provides `vector`-like members `resize`, `clear`, `push_back`, `pop_back`, `empty`, `size`
* Provides capacity members `num_blocks`, `max_size`, `capacity`, `reserve`, `shrink_to_fit`
* Provides set operations `-=`, `-`, `is_subset_of`, `is_proper_subset_of`, `intersects`
* Provides lookup: `find_first`, `find_next`
* Provides ordering `<` etc.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`, `<boost/detail/workaround.hpp>`.
* `<boost/limits.hpp>

#### Boost.Core

* `<boost/utility/addressof.hpp>`
* `<boost/core/no_exceptions_support.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Move

* `<boost/move/move.hpp>` - if compiler supports rvalue-ref

#### Boost.Integer

* `<boost/pending/integer_log2.hpp>`

------
### Standard Facilities
