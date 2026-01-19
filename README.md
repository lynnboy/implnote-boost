Boost C++ Libraries

* [`accumulators`](accumulators.md) (C++03) -- **Accumulators** | _Framework for incremental calculation, and collection of statistical accumulators._
  * Statistics accumulators: `count`, `max`, `min`, `mean`, `sum`, ...

* [`algorithm`](algorithm.md) (C++03) -- **Algorithm** | _A collection of useful generic algorithms._
  * STD: searchers (C++17), new algorithms of C++11, C++14, C++17, C++20
  * Supplimentary algo: `find_not`, `power`, `gather`, `hex`, `unhex`, `is_palindrome`, ....

* [`algorithm/minmax`](algorithm-minmax.md) (C++03) -- **Min-Max** | _Standard library extensions for simultaneous min/max and min/max element computations._
  * STD: `minmax`, `minmax_element` (C++11)

* [`algorithm/string`](algorithm-string.md) (C++03) -- **String Algo** | _String algorithms library._
  * case conv, compare/contain, classify, trim, finder, formatter, find/split iterator, find, erase, replace, split, join, regex tools
  * STD: `starts_with` etc (C++20,C++23), `regex_search`, `regex_replace` (C++23)

* [`align`](align.md) (C++03) -- **Align** | _Memory alignment functions, allocators, traits._
  * `is_aligned`, `aligned_alloc`, `aligned_allocator`, `aligned_delete`
  * STD: `alignof`, `alignas` (C++11), aligned `new`/`delete` (C++17), `alignment_of`, `align`, `aligned_alloc`, `is_sufficiently_aligned`, `assume_aligned`

* [`any`](any.md) (C++11) -- **Any** | _Safe, generic container for single values of different value types._
  * STD: `<any>` (Library Fundamentals TS v1, C++17).

* [`array`](array.md) (C++03) -- **Array** | _STL compliant container wrapper for arrays of constant size._
  * STD: `<array>` (tr1, C++11).

* [`asio`](asio.md) (C++11) -- **Asio** | _Portable networking and other low-level I/O, including sockets, timers, hostname resolution, socket iostreams, serial ports, file descriptors and Windows HANDLEs._
  * --

* [`assert`](assert.md) (C++03) -- **Assert** | _Customizable assert macros._
  * Assertion custom handler support.
  * STD: `<cassert>`.
  * `source_location` (LFTS v2, C++20).

* [`assign`](assign.md) (C++11) -- **Assign** | _Filling containers with constant or generated data has never been easier._
  * STD: `<initializer_list>` (C++11)

* [`atomic`](atomic.md) (C++11) -- **Atomic** | _C++11-style atomic types._
  * --

* [`beast`](beast.md) (C++11) -- **Beast** | _Portable HTTP, WebSocket, and network operations using only C++11 and Boost.Asio._
  * --

* [`bimap`](bimap.md) (C++11) -- **Bimap** | _Bidirectional maps library for C++. With Boost.Bimap you can create associative containers in which both types can be used as key._

* [`bind`](bimap.md) (C++11) -- **Bind** | _boost::bind is a generalization of the standard functions std::bind1st and std::bind2nd. It supports arbitrary function objects, functions, function pointers, and member function pointers, and is able to bind any argument to a specific value or route input arguments into arbitrary positions._
  * STD: lambda, `<bind>`

* [`bind/mem_fn`](bimap-mem_fn.md) (C++11) -- **Bind/MemFn** | _Generalized binders for function/object/pointers and member functions._
  * STD: `mem_fn` (C++11)

* [`bloom`](bloom.md) (C++11) -- **Bloom** | _Bloom filters._

* [`callable_traits`](callable_traits.md) (C++11) -- **CallableTraits** | _A spiritual successor to Boost.FunctionTypes, Boost.CallableTraits is a header-only C++11 library for the compile-time inspection and manipulation of all 'callable' types. Additional support for C++17 features._

* [`charconv`](charconv.md) (C++11) -- **CharConv** | _An implementation of <charconv> in C++11._
  * --

* [`chrono`](chrono.md) (C++11) -- **Chrono** | _Useful time utilities. C++11._
  * STD: `<chrono>` (C++11)

* [`chrono/stopwatches`](chrono-stopwatches.md) (C++11) -- **Chrono/StopWatches** | _stopwatch classes_
  * removed

* [`circular_buffer`](circular_buffer.md) (C++03) -- **Circular Buffer** | _A STL compliant container also known as ring or cyclic buffer._

* [`cobalt`](cobalt.md) (C++20) -- **Cobalt** | _Coroutines. Basic Algorithms & Types_

* [`compat`](compat.md) (C++11) -- **Compat** | _C++11 implementations of standard components added in later C++ standards._

* [`compatibility`](compatibility.md) (C++03) -- **Compatibility** | _Help for non-conforming standard libraries._
  * STD: `<cxxxx>`.

* [`compute`](compute.md) (C++03) -- **Compute** | _Parallel/GPU-computing library_
  * --

* [`concept_check`](concept_check.md) (C++03) -- **Concept Check** | _Tools for generic programming._
  * STD: Concept Lite TS, C++20

* [`config`](config.md) (C++03) -- **Config** | _Helps Boost library developers adapt to compiler idiosyncrasies; not intended for library users._

* [`container`](container.md) (C++03) -- **Container** | _Standard library containers and extensions._
  * --

* [`container_hash`](container_hash.md) (C++11) -- **ContainerHash** | _An STL-compatible hash function object that can be extended to hash user defined types._
  * STD: `hash` (C++11)

* [`context`](context.md) (C++11) -- **Context** | _(C++11) Context switching library._
  * Fiber, call/cc continuation, and stack allocation

* [`contract`](contract.md) (C++11) -- **Contract** | _Contract programming for C++. All contract programming features are supported: Subcontracting, class invariants, postconditions (with old and return values), preconditions, customizable actions on assertion failure (e.g., terminate or throw), optional compilation and checking of assertions, etc._
  * STD: contracts (C++26)

* [`conversion`](convert.md) (C++11) -- **Conversion** | _Polymorphic casts._
  * `implicit_cast`, `polymorphic_[pointer_][down]cast`, etc.

* [`convert`](convert.md) (C++11) -- **Convert** | _An extendible and configurable type-conversion framework._
  * Pluggable conversion API, `lexical_cast`, `printf`, `stream`, `strtol`, `spirit` based converters.
  * STD: `<charconv>` (C++17)

* [`core`](core.md) (C++11) -- **Core** | _A collection of simple core utilities with minimal dependencies._
  * `demangle`, `typeinfo` and `demangled_name`
  * `ignore_unused`, Lightweight Test, Dummy try/catch, `noncopyable`, `null_deleter`, `visit_each`
  * STD: `addressof`, `default_delete`, `explicit operator bool`, `is_same`, `enum class`

* [`core/enable_if`](core-enable_if.md) (C++03) -- **Enable If** | _Selective inclusion of function template overloads._
  * `disable_if`, `lazy_enable_if`, etc.
  * STD: `enable_if` (C++11)

* [`core/ref`](core-ref.md) (C++03) -- **Ref** | _A utility library for passing references to generic functions._
  * Trait `is_reference_wrapper` and `unwrap_reference`, `unwrap_ref()`
  * STD: `ref`, `cref`, `reference_wrapper` (C++11)

* [`core/swap`](core-swap.md) (C++03) -- **Swap** | _Enhanced generic swap function._
  * STD: `swap` for array (C++11)

* [`coroutine`](coroutine.md) (C++03) -- **Coroutine** | _Coroutine library._
  * Asymmetric/symmetric coroutine
  * STD: N4402, N4403, P0057, coroutine (C++20)

* [`coroutine2`](coroutine2.md) (C++11) -- **Coroutine2** | _(C++14) Coroutine library._
  * Asymmetric coroutine
  * STD: P0057, P0073, coroutine (C++20)

* [`crc`](crc.md) (C++11) -- **CRC** | _The Boost CRC Library provides two implementations of CRC (cyclic redundancy code) computation objects and two implementations of CRC computation functions. The implementations are template-based._
  * CRC checksum calculator

* [`date_time`](date_time.md) (C++03) -- *DateTime** | _A set of date-time libraries based on generic programming concepts._
  * --

* [`describe`](describe.md) (C++03) -- **Describe** | _A C++14 reflection library._
  * static reflection (C++26)

* [`detail`](detail.md) (C++03) -- **Detail** | _This library contains a set of header only utilities used internally by Boost C++ Libraries to facilitate their implementation._
  * STD: `binary_search`, `codecvt_utf8` (C++11), `is_sorted` (C++11)

* [`disjoint_sets`](disjoint_sets.md) (C++03)
  * removed, moved into **Boost.Graph**

* [`dll`](dll.md) (C++11) -- **DLL** | _Library for comfortable work with DLL and DSO._
  * Declare symbol alias, import symbol
  * Load shared library, query symbols
  * STD: P0275

* [`dynamic_bitset`](dynamic_bitset.md) (C++11) -- **Dynamic Bitset** | *The `dynamic_bitset` template represents a set of bits.*
  * It provides access to the value of individual bits via `operator[]` and provides all of the bitwise operators that one can apply to builtin integers, such as `operator&` and `operator<<`. The number of bits in the set can change at runtime.
  * Dynamic version of `bitset`

* [`endian`](endian.md) (C++11) -- **Endian** | _Types and conversion functions for correct byte ordering and more regardless of processor endianness._
  * Endian conversion, Endian-aware buffer, Endian-aware arithmetic type
  * STD: P0463

* [`exception`](exception.md) (C++03) -- **Exception** | _The Boost Exception library supports transporting of arbitrary data in exception objects, and transporting of exceptions between threads._
  * STD: `exception_ptr` (C++11)
  * STD: proposal N3757 and N3758, pending.

* [`fiber`](fiber.md) (C++11) -- **Fiber** | _(C++11) Userland threads library._

* [`filesystem`](filesystem.md) (C++11) -- **FileSystem** | _The Boost Filesystem Library provides portable facilities to query and manipulate paths, files, and directories._
  * STD: `<filesystem>` (C++11)

* [`flyweight`](flyweight.md) (C++03) -- **Flyweight** | _Design pattern to manage large quantities of highly redundant objects._
  * Automatic intern values in repository, flyweight wrapper for handles behave as immutable value.

* [`foreach`](foreach.md) (C++03) -- **Foreach** | *`BOOST_FOREACH` iterates over sequences for us, freeing us from having to deal directly with iterators or write predicates.*
  * STD: Range based `for` (C++11)

* [`format`](format.md) (C++11) -- **Format** | _The format library provides a class for formatting arguments according to a format-string._
  * Formating to string.
  * STD: `<format>` (C++20)

* [`function`](function.md) (C++11) -- **Function** | _Function object wrappers for deferred calls or callbacks._
  * STD: `function` (C++11), `move_only_function` (C++23), `copyable_function`, `function_ref` (C++26)

* [`functional`](functional.md) (C++03) -- **Functional** | _The Boost.Function library contains a family of class templates that are function object wrappers._
  * STD: `<functional>`

* [`functional/factory`](functional-factory.md) (C++03) -- **Functional/Factory** | _Function object templates for dynamic and static object creation)_
  * wrapper for object creation

* [`functional/forward`](functional-forward.md) (C++03) -- **Functional/Forward** | _Adapters to allow generic function objects to accept arbitrary arguments_
  * STD: rvalue-reference, `forward` (C++11)

* [`functional/overloaded_function`](functional-overloaded_function.md) (C++03) -- **Functional/Overloaded Function** | _Overload different functions into a single function object._
  * STD: Proposal P0045, P0051

* [`function_types`](function_types.md) (C++11) -- **Function Types** | _Boost.FunctionTypes provides functionality to classify, decompose and synthesize function, function pointer, function reference and pointer to member types._

* [`fusion`](fusion.md) (C++03) -- **Fusion** | _Library for working with tuples, including various containers, algorithms, etc._
  * --

* [`geometry`](geometry.md) (C++14) -- **Geometry** | _The Boost.Geometry library provides geometric algorithms, primitives and spatial index._
  * --

* [`gil`](gil.md) (C++14) -- **GIL** | _(C++14) Generic Image Library_
  * --

* [`graph`](graph.md) (C++14) -- **Graph** | _The BGL graph interface and graph components are generic, in the same sense as the Standard Template Library (STL)._
  * --

* [`graph_parallel`](graph_parallel.md) (C++03) -- **GraphParallel** | _The PBGL graph interface and graph components are generic, in the same sense as the Standard Template Library (STL)._
  * --

* [`hana`](hana.md) (C++14) -- **Hana** | _A modern C++ metaprogramming library. It provides high level algorithms to manipulate heterogeneous sequences, allows writing type-level computations with a natural syntax, provides tools to introspect user-defined types and much more._
  * --

* [`hash2`](hash2.md) (C++11) -- **Hash2** | _An extensible hashing framework._
  * XXH32/XXH64/XXH3-128, SipHash/HalfSipHash, HMAC, MD5, SHA1/SHA2/SHA3, RIPEMD, BLAKE2

* [`heap`](heap.md) (C++14) -- **Heap** | _Priority queue data structures._
  * D-ary, Binomial, Fibonacci, Pairing, Skew

* [`histogram`](histogram.md) (C++14) -- **Histogram** | _Fast multi-dimensional histogram with convenient interface for C++14_

* [`hof`](hof.md) (C++11) -- **HOF** | _Higher-order functions for C++_
  * Adaptors: combine, compose, decorate, first_of, fix, flip, flow, fold, implicit, indirect, infix, lazy, match, mutable, partial, pipable, proj, protect, result, reveal, reverse_fold, rotate, static, unpack
  * Decorators: capture, if, limit, repeat, repeat_while
  * Functions: always, arg, construct, decay, identity
  * Placeholders

* [`icl`](icl.md) (C++03) -- **ICL** | _Interval Container Library, interval sets and maps and aggregation of associated values_

* [`integer`](integer.md) (C++03) -- **Integer** | _Take advantage of <stdint.h> types from the 1999 C standard._
  * Type selector, mask maker
  * Compile-time and Runtime GCD/LCM and Log2.
  * STD: `<cstdint>` (C++11), gcd/lcm (LFTS v2, C++17)

* [`interprocess`](interprocess.md) (C++03) -- **Interprocess** | _Shared memory, memory mapped files, process-shared mutexes, condition variables, containers and allocators._
  * --

* [`intrusive`](intrusive.md) (C++03) -- **Intrusive** | _Intrusive containers and algorithms._
  * --

* [`io`](io.md) (C++03) -- **IO** | _Utilities for the standard I/O library._
  * IOS state savers.
  * STD: `quoted` manipulator (C++14).

* [`iostreams`](iostreams.md) (C++03) -- **Iostreams** | _Boost.IOStreams provides a framework for defining streams, stream buffers and i/o filters._

* [`iterator`](iterator.md) (C++11) -- **Iterator** | _The Boost Iterator Library contains two parts. The first is a system of concepts which extend the C++ standard iterator requirements. The second is a framework of components for building iterators based on these extended concepts and includes several useful iterator adaptors._
  * STD: `reverse_iterator`, `<ranges>`, `<generator>`

* [`json`](json.md) (C++11) -- **JSON** | _JSON parsing, serialization, and DOM in C++11_
  * --

* [`lambda`](lambda.md) (C++03) -- **Lambda** | _Define small unnamed function objects at the actual call site, and more._
  * STD: lambda (C++11)

* [`lambda2`](lambda2.md) (C++14) -- **Lambda2** | _A C++14 lambda library._
  * STD: lambda (C++11)

* [`leaf`](leaf.md) (C++14) -- **LEAF** | _A lightweight error handling library for C++11._

* [`lexical_cast`](lexical_cast.md) (C++11) -- **Lexical Cast** | _General literal text conversions, such as an int represented a string, or vice-versa._
  * Casting using stringstream by concept, keeps precision.
  * STD: Proposal P0117

* [`locale`](locale.md) (C++11) -- **Locale** | _Provide localization and Unicode handling tools for C++._
  * --

* [`local_function`](local_function.md) (C++03) -- **Local Function** | _Program functions locally, within other functions, directly within the scope where they are needed._
  * STD: lambda as local function

* [`lockfree`](lockfree.md) (C++14) -- **Lockfree** | _Lockfree data structures._
  * Lock-free `queue`, `stack`, wait-free `spsc_queue`, `spsc_value`

* [`log`](log.md) (C++11) -- **Log** | _Logging library._
  * --

* [`logic`](logic.md) (C++03) -- **Tribool** | _3-state boolean type library._
  * `tribool` related.

* [`math`](math.md) (C++14) -- **Math** | _Boost.Math includes several contributions in the domain of mathematics._
  * FP utils, specific-width fp types, constants, statistical-distrubitions, special functions, root finding, polynomials, interpolation, integration and differentiation
  * --

* [`metaparse`](metaparse.md) (C++03) -- **Metaparse** | _A library for generating compile time parsers parsing embedded DSL code as part of the C++ compilation process_
  * parsers: alphanum, digit, letter, lit, one_char, range, space, digit_val, int_, empty, fail, keyword, return
  * parser combiners: accept_when, except, transform_error, foldl, foldr, iterate, repeated, if, one_of, optional, first_of, last_of, middle_of, nth_of, sequence, always, transform

* [`move`](move.md) (C++03) -- **Move** | _Portable move semantics for C++03 and C++11 compilers._
  * STD: rvalue-ref, move-semantic, `unique_ptr`, `forward`, `move`

* [`mp11`](mp11.md) (C++11) -- **MP11** | _A C++11 metaprogramming library._

* [`mpi`](mpi.md) (C++03) -- **MPI** | _Message Passing Interface library, for use in distributed-memory parallel application programming._

* [`mpl`](mpl.md) (C++03) -- **MPL** | _The Boost.MPL library is a general-purpose, high-level C++ template metaprogramming framework of compile-time algorithms, sequences and metafunctions._

* [`mqtt`](mqtt.md) (C++17) -- **MQTT** | _MQTT5 client library built on top of Boost.Asio._

* [`multi_array`](multi_array.md) (C++03) -- **Multi-Array** | _Boost.MultiArray provides a generic N-dimensional array concept definition and common implementations of that interface._
  * Multi dimensional array & array view
  * STD: N4355, N4512 (Fundamentals TS v2), P0009, P0122, `<mdspan>`

* [`multi_index`](multi_index.md) (C++03) -- **Multi-Index** | _The Boost Multi-index Containers Library provides a class template named multi_index_container which enables the construction of containers maintaining one or more indices with different sorting and access semantics._

* [`msm`](msm.md) (C++03) -- **Meta State Machine** | _A very high-performance library for expressive UML2 finite state machines._

* [`multiprecision`](multiprecision.md) (C++14) -- **Multiprecision** | _Extended precision arithmetic types for floating point, integer, and rational arithmetic._
  * --

* [`mysql`](mysql.md) (C++11) -- **MySQL** | _MySQL client library built on top of Boost.Asio._
  * --

* [`nowide`](nowide.md) (C++11) -- **Nowide** | _Standard library functions with UTF-8 API on Windows._

* [`numeric/conversion`](numeric-conversion.md) (C++03) -- **Numeric Conversion** | _Optimized Policy-based Numeric Conversions._
  * Numeric conversion/cast with detailed control.

* [`numeric/interval`](numeric-interval.md) (C++03) -- **Interval** | _Extends the usual arithmetic functions to mathematical intervals._
  * Interval arithmetics

* [`numeric/odeint`](numeric-odeint.md) (C++11) -- **Odeint** | _Solving ordinary differential equations._
  * --

* [`numeric/ublas`](numeric-ublas.md) (C++11) -- **uBLAS** | _uBLAS provides tensor, matrix, and vector classes as well as basic linear algebra routines. Several dense, packed and sparse storage schemes are supported._
  * --

* [`openmethod`](openmethod.md) (C++17) -- **OpenMethod** | _Open-methods for C++17 and above._
  * --

* [`optional`](optional.md) (C++03) -- **Optional** | _A value-semantic, type-safe wrapper for representing 'optional' (or 'nullable') objects of a given type. An optional object may or may not contain a value of the underlying type._
  * STD: Library fundamentals v1, C++17

* [`outcome`](outcome.md) (C++14) -- **Outcome** | _A deterministic failure handling library partially simulating lightweight exceptions._
  * --

* [`parameter`](parameter.md) (C++03) -- **Parameter** | _Boost.Parameter Library - Write functions that accept arguments by name._

* [`parameter_python`](parameter_python.md) (C++03) -- **Parameter Python Bindings** | _Boost.Parameter Library Python bindings._

* [`parser`](parser.md) (C++17) -- **Parser** | _A parser combinator library._
  * --

* [`pfr`](pfr.md) (C++14) -- **PFR** | _Basic reflection for user defined types._
  * --

* [`phoenix`](phoenix.md) (C++03) -- **Phoenix** | _Define small unnamed function objects at the actual call site, and more._
  * --

* [`poly_collection`](poly_collection.md) (C++11) -- **PolyCollection** | _Fast containers of polymorphic objects._

* [`polygon`](polygon.md) (C++03) -- **Polygon** | _Voronoi diagram construction and booleans/clipping, resizing/offsetting and more for planar polygons with integral coordinates._
  * --

* [`pool`](pool.md) (C++03) -- **Pool** | _Memory pool management._
  * Fixed element size free-list (segregated) pool and allocators.

* [`predef`](predef.md) (C++98) -- **Predef** | _This library defines a set of compiler, architecture, operating system, library, and other version numbers from the information it can gather of C, C++, Objective C, and Objective C++ predefined macros or those defined in generally available headers._

* [`preprocessor`](preprocessor.md) (C++03) -- **Preprocessor** | _Preprocessor metaprogramming tools including repetition and recursion._

* [`process`](process.md) (C++11) -- **Process** | _Library to create processes in a portable way._
  * --

* [`program_options`](program_options.md) (C++11) -- **Program Options** | _The program_options library allows program developers to obtain program options, that is (name, value) pairs from the user, via conventional methods such as command line and config file._

* [`property_map`](property_map.md) (C++11) -- **Property Map** | _Concepts defining interfaces which map key objects to value objects._
  * Abstraction for `put`/`get`/`[]`

* [`property_map/parallel`](property_map-parallel.md) (C++03) -- **Property Map (Parallel)** | _Parallel extensions to Property Map for use with Parallel Graph._
  * Distributed property map, upon MPI

* [`property_tree`](property_tree.md) (C++11) -- **Property Tree** | _A tree data structure especially suited to storing configuration data._
  * Parsers: Info, INI, JSON, XML

* [`proto`](proto.md) (C++03) -- **Proto** | _Expression template library and compiler construction toolkit for domain-specific embedded languages._

* [`ptr_container`](ptr_container.md) (C++11) -- **Pointer Container** | _Containers for storing heap-allocated polymorphic objects to ease OO-programming._
  * `ptr_xxx` containers

* [`python`](python.md) (C++03) -- **Python** | _The Boost Python Library is a framework for interfacing Python and C++._
  * --

* [`qvm`](qvm.md) (C++03) -- **QVM** | _Generic C++ library for working with Quaternions Vectors and Matrices._
  * --

* [`random`](random.md) (C++11) -- **Random** | _A complete system for random number generation._
  * --

* [`range`](range.md) (C++03) -- **Range** | _A new infrastructure for generic algorithms that builds on top of the new iterator concepts._
  * STD: `<range>` (C++11)

* [`ratio`](ratio.md) (C++11) -- **Ratio** | _Compile time rational arithmetic. C++11._
  * STD: `ratio` (C++11)

* [`rational`](rational.md) (C++11) -- **Rational** | _A rational number class._
  * STD: proposal N3611, pending.

* [`redis`](redis.md) (C++17) -- **Redis** | _Redis async client library built on top of Boost.Asio._

* [`regex`](regex.md) (C++11) -- **Regex** | _Regular expression library._
  * STD: `<regex>` (C++11)
  * --

* [`safe_numerics`](safe_numerics.md) -- **Sfae Numerics** (C++14) | _Guaranteed Correct Integer Arithmetic_

* [`scope`](scope.md) (C++11) -- **Scope** | _A collection of scope guards and a unique_resource wrapper._
  * STD: LFV3: `scope_exit`, `scope_success`, `scope_fail`, `unique_resource`

* [`scope_exit`](scope_exit.md) (C++03) -- **Scope Exit** | _Execute arbitrary code at scope exit._
  * STD: P0052 - Generic Scope Guard and RAII Wrapper for the Standard Library

* [`serialization`](serialization.md) (C++03) -- **Serialization** | _Serialization for persistence and marshalling._
  * --

* [`signals`](signals.md) (C++03) -- **Signals** (deprecated) | _Managed signals & slots callback implementation._
  * Signal/slot, disconnect notify trackable targets, pullable combiner API

* [`signals2`](signals2.md) (C++03) -- **Signals2** | _Managed signals & slots callback implementation (thread-safe version 2)._
  * Signal/slot, disconnect notify trackable targets, pullable combiner API

* [`smart_ptr`](smart_ptr.md) (C++11) -- **Smart Ptr** | _Smart pointer class templates._
  * `intrusive_ptr`, `scoped_ptr`, `local_shared_ptr`, `atomic_shared_ptr`
  * STD: `shared_ptr`, `weak_ptr`, etc.

* [`sort`](sort.md) (C++03) -- **Sort** | _High-performance templated sort functions._
  * Spread sort algorithms for integer, floating, and strings.

* [`spirit`](spirit.md) (C++03) -- **Spirit** | _LL parser framework represents parsers directly as EBNF grammars in inlined C++._
  * --

* [`stacktrace`](stacktrace.md) (C++11) -- **Stacktrace** | _Gather, store, copy and print backtraces._
  * STD: `<stacktrace>` (C++23)

* [`statechart`](statechart.md) (C++03) -- **Statechart** | _Arbitrarily complex finite state machines can be implemented in easily readable and maintainable C++ code._
  * Flexible structure, not so efficient as MSM.

* [`static_assert`](static_assert.md) (C++03) -- **Static Assert** | _Static assertions (compile time assertions)._
  * STD: `static_assert` (C++11).

* [`static_string`](static_string.md) (C++11) -- **Static String** | _A fixed capacity dynamically sized string._

* [`stl_interfaces`](stl_interfaces.md) (C++14) -- **STL Interfaces** | _C++14 and later CRTP templates for defining iterators, views, and containers._

* [`sync`](sync.md) (C++03) -- **Sync** | _Implementation of synchronization primitives._
  * removed

* [`system`](system.md) (C++11) -- **System** | _Extensible error reporting._
  * STD: `<system_error>` (C++11)

* [`test`](test.md) (C++11) -- **Test** | _Support for simple program testing, full unit testing, and for program execution monitoring._
  * --

* [`thread`](thread.md) (C++11) -- **Thread** | _Portable C++ multi-threading. C++11, C++14, C++17._
  * --

* [`throw_exception`](throw_exception.md) (C++03) -- **ThrowException** | _A common infrastructure for throwing exceptions from Boost libraries._
  * Add throwing position as exception error-info.
  * STD: `<exception>` `current_exception` support (C++11).

* [`timer`](timer.md) (C++03) -- **Timer** | _Event timer, progress timer, and progress display classes._
  * Timer classes: `timer`, `progress_timer`, `progress_display`

* [`tokenizer`](tokenizer.md) (C++03) -- **Tokenizer** | _Break of a string or other character sequence into a series of tokens._
  * Tokenizer function, iterator, view range.

* [`tr1`](tr1.md) (C++98) -- **TR1** (deprecated) | _The TR1 library provides an implementation of the C++ Technical Report on Standard Library Extensions._
  * STD: TR1
  * removed

* [`tti`](tti.md) (C++03) -- **TTI** | _Type Traits Introspection library._
  * Query members of a type.
  * STD: reflection proposals: N4428, N4447, N4451

* [`tuple`](tuple.md) (C++03) -- **Tuple** | _Ease definition of functions returning multiple values, and more._
  * STD: `<tuple>` (C++11)

* [`type_erasure`](type_erasure.md) (C++03) -- **Type Erasure** | _Runtime polymorphism based on concepts._

* [`type_index`](type_index.md) (C++11) -- **Type Index** | _Runtime/Compile time copyable type info._
  * STD: `<typeindex>` (C++11)

* [`typeof`](typeof.md) (C++11) -- **TypeOf** | _Typeof operator emulation._

* [`type_traits`](type_traits.md) (C++03) -- **Type Traits** | _Templates for fundamental properties of types._
  * STD: `<type_traits>` (C++11)
  * --

* [`units`](units.md) (C++03) -- **Units** | _Zero-overhead dimensional analysis and unit/quantity manipulation and conversion._
  * --

* [`unordered`](unordered.md) (C++11) -- **Unordered** | _Unordered associative containers._
  * --

* [`url`](url.md) (C++11) -- **URL** | _URL parsing in C++11_
  * --

* [`utility`](utility.md) (C++03) -- **Utility** | _Various utilities, such as base-from-member idiom and binary literals in C++03._

* [`utility/call_traits`](utility-call_traits.md) (C++03) -- **Utility/CallTraits** | _Defines types for passing parameters._

* [`utility/compressed_pair`](utility-compressed_pair.md) (C++03) -- **Utility/CompressedPair** | _A pair class with empty member optimization._

* [`utility/identity_type`](utility-identity_type.md) (C++03) -- **Utility/Identity Type** | _Wrap types within round parenthesis so they can always be passed as macro parameters._

* [`utility/in_place_factories`](utility-in_place_factories.md) (C++03) -- **Utility/In Place Factory, Typed In Place Factory** | _Generic in-place construction of contained objects with a variadic argument-list._

* [`utility/operators`](utility-operators.md) (C++03) -- **Utility/Operators** | _Templates to simplify operator definition in arithmetic classes and iterators._

* [`utility/result_of`](utility-result_of.md) (C++03) -- **Utility/Result Of** | _Determines the type of a function call expression._
  * STD: `decltype`, `result_of`, `invoke_result`

* [`utility/string_view`](utility-string_view.md) (C++03) -- **Utility/String View** | _String view templates._
  * STD: `<string_view>` (C++17)

* [`utility/value_init`](utility-value_init.md) (C++03) -- **Utility/Value Initialized** | _Wrapper for uniform-syntax value initialization, based on the original idea of David Abrahams._

* [`uuid`](uuid.md) (C++11) -- **UUID** | _A universally unique identifier._
  * Uuid data type & generator/factories
  * SHA1 hasher
  * URNG seed generator (C++11 `random_device`)

* [`variant`](variant.md) (C++11) -- **Variant** | _Safe, generic, stack-based discriminated union container._
  * STD: `<variant>`, `<bind>`, `mem_fn` (C++11)

* [`variant2`](variant2.md) (C++11) -- **Variant2** | _A never-valueless, strong guarantee implementation of `std::variant`._
  * STD: `<variant>`, `<bind>`, `mem_fn` (C++11)

* [`vmd`](vmd.md) (C++03) -- **VMD** | _Variadic Macro Data library._
  * --

* [`wave`](wave.md) (C++11) -- **Wave** | _The Boost.Wave library is a Standards conformant, and highly configurable implementation of the mandated C99/C++ preprocessor functionality packed behind an easy to use iterator interface._
  * --

* [`winapi`](winapi.md) (C++03) -- **WinAPI** | _Windows API abstraction layer._
  * Windows API declaration.

* [`xpressive`](xpressive.md) (C++03) -- **Xpressive** | _Regular expressions that can be written as strings or as expression templates, and which can refer to each other and themselves recursively with the power of context-free grammars._
  * --

* [`yap`](yap.md) (C++14) -- **YAP** | _An expression template library for C++14 and later._
