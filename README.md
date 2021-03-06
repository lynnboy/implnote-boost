Boost C++ Libraries

* [`assert`](assert.md) -- **Assert** | _Customizable assert macros._
  * Assertion custom handler support.
  * STD: `<cassert>`.
  * `source_location` (LFTS v2).

* [`static_assert`](static_assert.md) -- **Static Assert** | _Static assertions (compile time assertions)._
  * STD: `static_assert` (C++11).

* [`conversion`](conversion.md) -- **Conversion** | _Polymorphic casts._
  * `implicit_cast`, `polymorphic_[pointer_][down]cast`, etc.

* [`disjoint_sets`](disjoint_sets.md)
  * Aux algorithm.

* [`any`](any.md) -- **Any** | _Safe, generic container for single values of different value types._
  * STD: `<any>` (Library Fundamentals TS v1, C++17).

* [`compatibility`](compatibility.md) -- **Compatibility** | _Help for non-conforming standard libraries._
  * STD: `<cxxxx>`.

* [`array`](array.md) -- **Array** | _STL compliant container wrapper for arrays of constant size._
  * STD: `<array>` (tr1, C++11).

* [`throw_exception`](throw_exception.md) -- **ThrowException** | _A common infrastructure for throwing exceptions from Boost libraries._
  * Add throwing position as exception error-info.
  * STD: `<exception>` `current_exception` support (C++11).

* [`io`](io.md) -- **IO State Savers** | _The I/O sub-library of Boost helps segregate the large number of Boost headers._
  * IOS state savers.
  * STD: `quoted` manipulator (C++14).

* [`timer`](timer.md) -- **Timer** | _Event timer, progress timer, and progress display classes._
  * Timer classes.

* [`rational`](rational.md) -- **Rational** | _A rational number class._
  * STD: proposal N3611, pending.

* [`logic`](logic.md) -- **Tribool** | _3-state boolean type library._
  * `tribool` related.

* [`tokenizer`](tokenizer.md) -- **Tokenizer** | _Break of a string or other character sequence into a series of tokens._
  * Tokenizer function, iterator, view range.

* [`align`](align.md) -- **Align** | _Memory alignment functions, allocators, and adaptors._
  * `is_aligned`, `aligned_alloc`, `aligned_allocator`, `aligned_delete`
  * STD: `align`, `alignment_of` (C++11)

* [`foreach`](foreach.md) -- **Foreach** | *`BOOST_FOREACH` iterates over sequences for us, freeing us from having to deal directly with iterators or write predicates.*
  * STD: Range based `for` (C++11)

* [`tuple`](tuple.md) -- **Tuple** | _Ease definition of functions returning multiple values, and more._
  * STD: `<tuple>` (C++11)

* [`uuid`](uuid.md) -- **Uuid** | _A universally unique identifier._
  * Uuid data type & generator/factories
  * SHA1 hasher
  * URNG seed generator (C++11 `random_device`)

* [`integer`](integer.md) -- **Integer** | _Take advantage of <stdint.h> types from the 1999 C standard._
  * Type selector, mask maker
  * Compile-time and Runtime GCD/LCM and Log2.
  * STD: `<cstdint>` (C++11), gcd/lcm (LFTS v2, C++17)

* [`assign`](assign.md) -- **Assign** | _Filling containers with constant or generated data has never been easier._
  * STD: `<initializer_list>` (C++11)

* [`scope_exit`](scope_exit.md) -- **Scope Exit** | _Execute arbitrary code at scope exit._
  * STD: P0052 - Generic Scope Guard and RAII Wrapper for the Standard Library

* [`system`](system.md) -- **System** | _Operating system support, including the diagnostics support that will be part of the C++0x standard library._
  * STD: `<system_error>` (C++11)

* [`core`](core.md) -- **Core** | _A collection of simple core utilities with minimal dependencies._
  * `demangle`, `typeinfo` and `demangled_name`
  * `ignore_unused`, Lightweight Test, Dummy try/catch, `noncopyable`, `null_deleter`, `visit_each`
  * STD: `addressof`, `default_delete`, `explicit operator bool`, `is_same`, `enum class`

* [`core/enable_if`](core-enable_if.md) -- **Enable If** | _Selective inclusion of function template overloads._
  * `disable_if`, `lazy_enable_if`, etc.
  * STD: `enable_if` (C++11)

* [`core/ref`](core-ref.md) -- **Ref** | _A utility library for passing references to generic functions._
  * Trait `is_reference_wrapper` and `unwrap_reference`, `unwrap_ref()`
  * STD: `ref`, `cref`, `reference_wrapper` (C++11)

* [`core/swap`](core-swap.md) -- **Swap** | _Enhanced generic swap function._
  * STD: `swap` for array (C++11)

* [`convert`](convert.md) -- **Convert** | _An extendible and configurable type-conversion framework._
  * Pluggable conversion API, `lexical_cast`, `printf`, `stream`, `strtol`, `spirit` based converters.

* [`dynamic_bitset`](dynamic_bitset.md) -- **Dynamic Bitset** | *The `dynamic_bitset` class represents a set of bits.*
  * Dynamic version of `bitset`

* [`type_index`](type_index.md) -- **Type Index** | _Runtime/Compile time copyable type info._
  * STD: `<typeindex>` (C++11)

* [`exception`](exception.md) -- **Exception** | _The Boost Exception library supports transporting of arbitrary data in exception objects, and transporting of exceptions between threads._
  * STD: `exception_ptr` (C++11)
  * STD: proposal N3757 and N3758, pending.

* [`concept_check`](concept_check.md) -- **Concept Check** | _Tools for generic programming._
  * STD: Concept Lite TS

* [`endian`](endian.md) -- **Endian** | _Types and conversion functions for correct byte ordering and more regardless of processor endianness._
  * Endian conversion, Endian-aware buffer, Endian-aware arithmetic type
  * STD: P0463

* [`detail`](detail.md)
  * STD: `binary_search`, `codecvt_utf8` (C++11), `is_sorted` (C++11)

* [`function`](function.md) -- **Function** | _Function object wrappers for deferred calls or callbacks._
  * STD: `function` (C++11)

* [`crc`](crc.md) -- **CRC** | _The Boost CRC Library provides two implementations of CRC (cyclic redundancy code) computation objects and two implementations of CRC computation functions. The implementations are template-based._
  * CRC checksum calculator

* [`chrono/stopwatches`](chrono-stopwatches.md)
  * Stopwatch, output formatting, supersedes *Boost.Timer*

* [`optional`](optional.md) -- **Optional** | _A value-semantic, type-safe wrapper for representing 'optional' (or 'nullable') objects of a given type._
  * STD: Library fundamentals v1, C++17

* [`signals`](signals.md) -- **Signals** (deprecated) | _Managed signals & slots callback implementation._
  * Signal/slot, disconnect notify trackable targets, pullable combiner API

* [`flyweight`](flyweight.md) -- **Flyweight** | _Design pattern to manage large quantities of highly redundant objects._
  * Automatic intern values in repository, flyweight wrapper for handles behave as immutable value.

* [`lockfree`](lockfree.md) -- **Lockfree** | _Lockfree data structures._
  * Lock-free `queue`, `stack`, wait-free `spsc_queue`

* [`coroutine2`](coroutine2.md) -- **Coroutine2** | _(C++14) Coroutine library._
  * Asymmetric coroutine
  * STD: P0057, P0073

* [`tti`](tti.md) -- **TTI** | _Type Traits Introspection library._
  * Query members of a type.
  * STD: reflection proposals: N4428, N4447, N4451

* [`multi_array`](multi_array.md) -- **Multi-Array** | _Boost.MultiArray provides a generic N-dimensional array concept definition and common implementations of that interface._
  * Multi dimensional array & array view
  * STD: N4355, N4512 (Fundamentals TS v2), P0009, P0122

* [`lexical_cast`](lexical_cast.md) -- **Lexical Cast** | _General literal text conversions, such as an int represented a string, or vice-versa._
  * Casting using stringstream by concept, keeps precision.
  * STD: Proposal P0117

* [`ratio`](ratio.md) -- **Ratio** | _Compile time rational arithmetic. C++11._
  * STD: `ratio` (C++11)

* [`property_map`](property_map.md) -- **Property Map** | _Concepts defining interfaces which map key objects to value objects._
  * Abstraction for `put`/`get`/`[]`

* [`property_map/parallel`](property_map-parallel.md)
  * Distributed property map, upon MPI

* [`tr1`](tr1.md) -- **TR1** (deprecated) | _The TR1 library provides an implementation of the C++ Technical Report on Standard Library Extensions._
  * STD: TR1

* [`format`](format.md) -- **Format** | _The format library provides a class for formatting arguments according to a format-string._
  * Formating to string.

* [`functional`](functional.md) -- **Functional** | _The Boost.Function library contains a family of class templates that are function object wrappers._
  * STD: `<functional>`

* [`functional/factory`](functional-factory.md) -- **Functional/Factory** | _Function object templates for dynamic and static object creation)_
  * wrapper for object creation

* [`functional/forward`](functional-forward.md) -- **Functional/Forward** | _Adapters to allow generic function objects to accept arbitrary arguments_
  * STD: rvalue-reference, `forward` (C++11)

* [`functional/hash`](functional-hash.md) -- **Functional/Hash** | _A TR1 hash function object that can be extended to hash user defined types._
  * STD: `hash` (C++11), Proposal P0029

* [`functional/overloaded_function`](functional-overloaded_function.md) -- **Functional/Overloaded Function** | _Overload different functions into a single function object._
  * STD: Proposal P0045, P0051

* [`numeric/conversion`](numeric-conversion.md) -- **Numeric Conversion** | _Optimized Policy-based Numeric Conversions._
  * Numeric conversion/cast with detailed control.

* [`pool`](pool.md) -- **Pool** | _Memory pool management._
  * Fixed element size free-list (segregated) pool and allocators.

* [`numeric/interval`](numeric-interval.md) -- **Interval** | _Extends the usual arithmetic functions to mathematical intervals._
  * Interval arithmetics

* [`sort`](sort.md) -- **Sort** | _High-performance templated sort functions._
  * Spread sort algorithms for integer, floating, and strings.

* [`statechart`](statechart.md) -- **Statechart** | _Arbitrarily complex finite state machines can be implemented in easily readable and maintainable C++ code._
  * Flexible structure, not so efficient as MSM.

* [`dll`](dll.md) -- **DLL** | _Library for comfortable work with DLL and DSO._
  * Declare symbol alias, import symbol
  * Load shared library, query symbols
  * STD: P0275

* [`winapi`](winapi.md)
  * Windows API declaration.

* [`parameter`](parameter.md) -- **Parameter** | _Write functions that accept arguments by name._
  * Named function parameter
  * (lazy) default value
  * named template parameter

* [`iterator`](iterator.md) -- **Iterator** | _The Boost Iterator Library contains two parts. The first is a system of concepts which extend the C++ standard iterator requirements. The second is a framework of components for building iterators based on these extended concepts and includes several useful iterator adaptors._
  * STD: `reverse_iterator`

* [`predef`](predef.md) -- **Predef** | _This library defines a set of compiler, architecture, operating system, library, and other version numbers from the information it can gather of C, C++, Objective C, and Objective C++ predefined macros or those defined in generally available headers._

* `utility`
* `signals2`
* `bind`
* `program_options`
* `local_function`
* `variant`
* `heap`
* `move`
* `filesystem`
* `coroutine`
* `ptr_container`
* `context`
* `unordered`
* `circular_buffer`
* `smart_ptr`
* `property_tree`
* `config`
* `chrono`
* `type_erasure`
* `sync`
* `atomic`
* `lambda`
* `bimap`
* `mpi`
* `algorithm`
* `accumulators`
* `random`
* `multi_index`
* `vmd`
* `range`
* `icl`
* `iostreams`
* `function_types`
* `units`
* `date_time`
* `type_traits`
* `locale`
* `msm`
* `python`
* `serialization`
* `xpressive`
* `graph_parallel`
* `regex`
* `numeric/odeint`
* `test`
* `compute`
* `thread`
* `multiprecision`
* `wave`
* `polygon`
* `intrusive`
* `gil`
* `interprocess`
* `proto`
* `container`
* `log`
* `graph`
* `numeric/ublas`
* `preprocessor`
* `asio`
* `math`
* `mpl`
* `typeof`
* `geometry`
* `spirit`
* `fusion`
* `phoenix`




Accumulators

    Framework for incremental calculation, and collection of statistical accumulators.

    Author(s)
        Eric Niebler
    First Release
        1.36.0
    Standard
         
    Categories
        Math and numerics

Algorithm

    A collection of useful generic algorithms.

    Author(s)
        Marshall Clow
    First Release
        1.50.0
    Standard
         
    Categories
        Algorithms

Asio

    Portable networking and other low-level I/O, including sockets, timers, hostname resolution, socket iostreams, serial ports, file descriptors and Windows HANDLEs.

    Author(s)
        Chris Kohlhoff
    First Release
        1.35.0
    Standard
         
    Categories
        Concurrent Programming, Input/Output

Atomic

    C++11-style atomic<>.

    Author(s)
        Helge Bahmann, Tim Blechmann and Andrey Semashev
    First Release
        1.53.0
    Standard
         
    Categories
        Concurrent Programming

Bimap

    Bidirectional maps library for C++. With Boost.Bimap you can create associative containers in which both types can be used as key.

    Author(s)
        Matias Capeletto
    First Release
        1.35.0
    Standard
         
    Categories
        Containers, Data structures

Bind

    boost::bind is a generalization of the standard functions std::bind1st and std::bind2nd. It supports arbitrary function objects, functions, function pointers, and member function pointers, and is able to bind any argument to a specific value or route input arguments into arbitrary positions.

    Author(s)
        Peter Dimov
    First Release
        1.25.0
    Standard
        TR1
    Categories
        Function objects and higher-order programming

Call Traits

    Defines types for passing parameters.

    Author(s)
        John Maddock, Howard Hinnant, et al
    First Release
        1.13.0
    Standard
         
    Categories
        Generic Programming

Chrono

    Useful time utilities. C++11.

    Author(s)
        Howard Hinnant, Beman Dawes and Vicente J. Botet Escriba
    First Release
        1.47.0
    Standard
        Proposed
    Categories
        Domain Specific, System

Circular Buffer

    A STL compliant container also known as ring or cyclic buffer.

    Author(s)
        Jan Gaspar
    First Release
        1.35.0
    Standard
         
    Categories
        Containers

Compressed Pair

    Empty member optimization.

    Author(s)
        John Maddock, Howard Hinnant, et al
    First Release
        1.13.0
    Standard
         
    Categories
        Data structures, Patterns and Idioms

Config

    Helps Boost library developers adapt to compiler idiosyncrasies; not intended for library users.

    Author(s)
         
    First Release
        1.9.0
    Standard
         
    Categories
        Broken compiler workarounds

Container

    Standard library containers and extensions.

    Author(s)
        Ion Gaztañaga
    First Release
        1.48.0
    Standard
         
    Categories
        Containers, Data structures

Context

    Context switching library.

    Author(s)
        Oliver Kowalke
    First Release
        1.51.0
    Standard
         
    Categories
        Concurrent Programming, System

Coroutine

    Coroutine library.

    Author(s)
        Oliver Kowalke
    First Release
        1.53.0
    Standard
         
    Categories
        Concurrent Programming

Date Time

    A set of date-time libraries based on generic programming concepts.

    Author(s)
        Jeff Garland
    First Release
        1.29.0
    Standard
         
    Categories
        Domain Specific, System

Filesystem

    The Boost Filesystem Library provides portable facilities to query and manipulate paths, files, and directories.

    Author(s)
        Beman Dawes
    First Release
        1.30.0
    Standard
         
    Categories
        System

Function Types

    Boost.FunctionTypes provides functionality to classify, decompose and synthesize function, function pointer, function reference and pointer to member types.

    Author(s)
        Tobias Schwinger
    First Release
        1.35.0
    Standard
         
    Categories
        Generic Programming, Template Metaprogramming

Fusion

    Library for working with tuples, including various containers, algorithms, etc.

    Author(s)
        Joel de Guzman, Dan Marsden and Tobias Schwinger
    First Release
        1.35.0
    Standard
         
    Categories
        Data structures, Template Metaprogramming

Geometry

    The Boost.Geometry library provides geometric algorithms, primitives and spatial index.

    Author(s)
        Barend Gehrels, Bruno Lalande, Mateusz Loskot, Adam Wulkiewicz and Menelaos Karavelas
    First Release
        1.47.0
    Standard
         
    Categories
        Algorithms, Data structures, Math and numerics

GIL

    Generic Image Library

    Author(s)
        Lubomir Bourdev and Hailin Jin
    First Release
        1.35.0
    Standard
         
    Categories
        Algorithms, Containers, Generic Programming, Image processing, Iterators

Graph

    The BGL graph interface and graph components are generic, in the same sense as the the Standard Template Library (STL).

    Author(s)
        Jeremy Siek and a University of Notre Dame team; now maintained by Andrew Sutton and Jeremiah Willcock.
    First Release
        1.18.0
    Standard
         
    Categories
        Algorithms, Containers, Iterators

Heap

    Priority queue data structures.

    Author(s)
        Tim Blechmann
    First Release
        1.49.0
    Standard
         
    Categories
        Data structures

ICL

    Interval Container Library, interval sets and maps and aggregation of associated values

    Author(s)
        Joachim Faulhaber
    First Release
        1.46.0
    Standard
         
    Categories
        Containers, Data structures

Identity Type

    Wrap types within round parenthesis so they can always be passed as macro parameters.

    Author(s)
        Lorenzo Caminiti
    First Release
        1.50.0
    Standard
         
    Categories
        Preprocessor Metaprogramming

In Place Factory, Typed In Place Factory

    Generic in-place construction of contained objects with a variadic argument-list.

    Author(s)
        Fernando Cacciola
    First Release
        1.32.0
    Standard
         
    Categories
        Generic Programming

Interprocess

    Shared memory, memory mapped files, process-shared mutexes, condition variables, containers and allocators.

    Author(s)
        Ion Gaztañaga
    First Release
        1.35.0
    Standard
         
    Categories
        Concurrent Programming

Intrusive

    Intrusive containers and algorithms.

    Author(s)
        Ion Gaztañaga
    First Release
        1.35.0
    Standard
         
    Categories
        Containers

Iostreams

    Boost.IOStreams provides a framework for defining streams, stream buffers and i/o filters.

    Author(s)
        Jonathan Turkanis
    First Release
        1.33.0
    Standard
         
    Categories
        Input/Output, String and text processing

Lambda

    Define small unnamed function objects at the actual call site, and more.

    Author(s)
        Jaakko Järvi and Gary Powell
    First Release
        1.28.0
    Standard
         
    Categories
        Function objects and higher-order programming

Local Function

    Program functions locally, within other functions, directly within the scope where they are needed.

    Author(s)
        Lorenzo Caminiti
    First Release
        1.50.0
    Standard
         
    Categories
        Function objects and higher-order programming

Locale

    Provide localization and Unicode handling tools for C++.

    Author(s)
        Artyom Beilis
    First Release
        1.48.0
    Standard
         
    Categories
        String and text processing

Log

    Logging library.

    Author(s)
        Andrey Semashev
    First Release
        1.54.0
    Standard
         
    Categories
        Miscellaneous

Math

    Boost.Math includes several contributions in the domain of mathematics: The Greatest Common Divisor and Least Common Multiple library provides run-time and compile-time evaluation of the greatest common divisor (GCD) or least common multiple (LCM) of two integers. The Special Functions library currently provides eight templated special functions, in namespace boost. The Complex Number Inverse Trigonometric Functions are the inverses of trigonometric functions currently present in the C++ standard. Quaternions are a relative of complex numbers often used to parameterise rotations in three dimentional space. Octonions, like quaternions, are a relative of complex numbers.

    Author(s)
        various
    First Release
        1.23.0
    Standard
         
    Categories
        Math and numerics

Math Common Factor

    Greatest common divisor and least common multiple.

    Author(s)
        Daryle Walker
    First Release
        1.26.0
    Standard
         
    Categories
        Math and numerics

Math Octonion

    Octonions.

    Author(s)
        Hubert Holin
    First Release
        1.23.0
    Standard
         
    Categories
        Math and numerics

Math Quaternion

    Quaternions.

    Author(s)
        Hubert Holin
    First Release
        1.23.0
    Standard
         
    Categories
        Math and numerics

Math/Special Functions

    A wide selection of mathematical special functions.

    Author(s)
        John Maddock, Paul Bristow, Hubert Holin and Xiaogang Zhang
    First Release
        1.35.0
    Standard
         
    Categories
        Math and numerics

Math/Statistical Distributions

    A wide selection of univariate statistical distributions and functions that operate on them.

    Author(s)
        John Maddock and Paul Bristow
    First Release
        1.35.0
    Standard
         
    Categories
        Math and numerics

Member Function

    Generalized binders for function/object/pointers and member functions.

    Author(s)
        Peter Dimov
    First Release
        1.25.0
    Standard
        TR1
    Categories
        Function objects and higher-order programming

Meta State Machine

    A very high-performance library for expressive UML2 finite state machines.

    Author(s)
        Christophe Henry
    First Release
        1.44.0
    Standard
         
    Categories
        State Machines

Min-Max

    Standard library extensions for simultaneous min/max and min/max element computations.

    Author(s)
        Hervé Brönnimann
    First Release
        1.32.0
    Standard
         
    Categories
        Algorithms

Move

    Portable move semantics for C++03 and C++11 compilers.

    Author(s)
        Ion Gaztañaga
    First Release
        1.48.0
    Standard
         
    Categories
        Language Features Emulation

MPI

    Message Passing Interface library, for use in distributed-memory parallel application programming.

    Author(s)
        Douglas Gregor and Matthias Troyer
    First Release
        1.35.0
    Standard
         
    Categories
        Concurrent Programming

MPL

    The Boost.MPL library is a general-purpose, high-level C++ template metaprogramming framework of compile-time algorithms, sequences and metafunctions. It provides a conceptual foundation and an extensive set of powerful and coherent tools that make doing explict metaprogramming in C++ as easy and enjoyable as possible within the current language.

    Author(s)
        Aleksey Gurtovoy
    First Release
        1.30.0
    Standard
         
    Categories
        Template Metaprogramming

Multi-Index

    The Boost Multi-index Containers Library provides a class template named multi_index_container which enables the construction of containers maintaining one or more indices with different sorting and access semantics.

    Author(s)
        Joaquín M López Muñoz
    First Release
        1.32.0
    Standard
         
    Categories
        Containers, Data structures

Multiprecision

    Extended precision arithmetic types for floating point, integer andrational arithmetic.

    Author(s)
        John Maddock and Christopher Kormanyos
    First Release
        1.53.0
    Standard
         
    Categories
        Math and numerics

Odeint

    Solving ordinary differential equations.

    Author(s)
        Karsten Ahnert and Mario Mulansky
    First Release
        1.53.0
    Standard
         
    Categories
        Math and numerics

Operators

    Templates ease arithmetic classes and iterators.

    Author(s)
        Dave Abrahams and Jeremy Siek
    First Release
        1.9.0
    Standard
         
    Categories
        Generic Programming, Iterators, Math and numerics

Phoenix

    Define small unnamed function objects at the actual call site, and more.

    Author(s)
        Joel de Guzman, Dan Marsden, Thomas Heller and John Fletcher
    First Release
        1.47.0
    Standard
         
    Categories
        Function objects and higher-order programming

Pointer Container

    Containers for storing heap-allocated polymorphic objects to ease OO-programming.

    Author(s)
        Thorsten Ottosen
    First Release
        1.33.0
    Standard
         
    Categories
        Containers, Data structures

Polygon

    Voronoi diagram construction and booleans/clipping, resizing/offsetting and more for planar polygons with integral coordinates.

    Author(s)
        Lucanus Simonson and Andrii Sydorchuk
    First Release
        1.44.0
    Standard
         
    Categories
        Algorithms, Data structures, Math and numerics

Preprocessor

    Preprocessor metaprogramming tools including repetition and recursion.

    Author(s)
        Vesa Karvonen and Paul Mensonides
    First Release
        1.26.0
    Standard
         
    Categories
        Preprocessor Metaprogramming

Program Options

    The program_options library allows program developers to obtain program options, that is (name, value) pairs from the user, via conventional methods such as command line and config file.

    Author(s)
        Vladimir Prus
    First Release
        1.32.0
    Standard
         
    Categories
        Input/Output

Property Tree

    A tree data structure especially suited to storing configuration data.

    Author(s)
        Marcin Kalicinski and Sebastian Redl
    First Release
        1.41.0
    Standard
         
    Categories
        Containers, Data structures

Proto

    Expression template library and compiler construction toolkit for domain-specific embedded languages.

    Author(s)
        Eric Niebler
    First Release
        1.37.0
    Standard
         
    Categories
        Template Metaprogramming

Python

    The Boost Python Library is a framework for interfacing Python and C++. It allows you to quickly and seamlessly expose C++ classes functions and objects to Python, and vice-versa, using no special tools -- just your C++ compiler.

    Author(s)
        Dave Abrahams
    First Release
        1.19.0
    Standard
         
    Categories
        Inter-language support

Random

    A complete system for random number generation.

    Author(s)
        Jens Maurer
    First Release
        1.15.0
    Standard
        TR1
    Categories
        Math and numerics

Range

    A new infrastructure for generic algorithms that builds on top of the new iterator concepts.

    Author(s)
        Niel Groves and Thorsten Ottosen
    First Release
        1.32.0
    Standard
         
    Categories
        Algorithms

Regex

    Regular expression library.

    Author(s)
        John Maddock
    First Release
        1.18.0
    Standard
        TR1
    Categories
        String and text processing

Result Of

    Determines the type of a function call expression.

    Author(s)
         
    First Release
        1.32.0
    Standard
         
    Categories
        Function objects and higher-order programming

Serialization

    Serialization for persistence and marshalling.

    Author(s)
        Robert Ramey
    First Release
        1.32.0
    Standard
         
    Categories
        Input/Output

Signals2

    Managed signals & slots callback implementation (thread-safe version 2).

    Author(s)
        Frank Mori Hess
    First Release
        1.39.0
    Standard
         
    Categories
        Function objects and higher-order programming, Patterns and Idioms

Smart Ptr

    Smart pointer class templates.

    Author(s)
        Greg Colvin, Beman Dawes, Peter Dimov, Darin Adler and Glen Fernandes
    First Release
        1.23.0
    Standard
        TR1
    Categories
        Memory

Spirit

    LL parser framework represents parsers directly as EBNF grammars in inlined C++.

    Author(s)
        Joel de Guzman, Hartmut Kaiser and Dan Nuffer
    First Release
        1.30.0
    Standard
         
    Categories
        Parsing, String and text processing

String Algo

    String algorithms library.

    Author(s)
        Pavol Droba
    First Release
        1.32.0
    Standard
         
    Categories
        Algorithms, String and text processing

Test

    Support for simple program testing, full unit testing, and for program execution monitoring.

    Author(s)
        Gennadiy Rozental
    First Release
        1.21.0
    Standard
         
    Categories
        Correctness and testing

Thread

    Portable C++ multi-threading. C++11, C++14.

    Author(s)
        Anthony Williams and Vicente J. Botet Escriba
    First Release
        1.25.0
    Standard
        Proposed
    Categories
        Concurrent Programming, System

Type Erasure

    Runtime polymorphism based on concepts.

    Author(s)
        Steven Watanabe
    First Release
        1.54.0
    Standard
         
    Categories
        Data structures

Type Traits

    Templates for fundamental properties of types.

    Author(s)
        John Maddock, Steve Cleary, et al
    First Release
        1.13.0
    Standard
        TR1
    Categories
        Generic Programming, Template Metaprogramming

Typeof

    Typeof operator emulation.

    Author(s)
        Arkadiy Vertleyb and Peder Holt
    First Release
        1.34.0
    Standard
         
    Categories
        Language Features Emulation

uBLAS

    uBLAS provides matrix and vector classes as well as basic linear algebra routines. Several dense, packed and sparse storage schemes are supported.

    Author(s)
        Joerg Walter and Mathias Koch
    First Release
        1.29.0
    Standard
         
    Categories
        Math and numerics

Units

    Zero-overhead dimensional analysis and unit/quantity manipulation and conversion.

    Author(s)
        Matthias Schabel and Steven Watanabe
    First Release
        1.36.0
    Standard
         
    Categories
        Domain Specific

Unordered

    Unordered associative containers.

    Author(s)
        Daniel James
    First Release
        1.36.0
    Standard
        TR1
    Categories
        Containers

Utility

    Class noncopyable plus checked_delete(), checked_array_delete(), next(), prior() function templates, plus base-from-member idiom.

    Author(s)
        Dave Abrahams and others
    First Release
        1.13.0
    Standard
         
    Categories
        Algorithms, Function objects and higher-order programming, Memory, Patterns and Idioms

Value Initialized

    Wrapper for uniform-syntax value initialization, based on the original idea of David Abrahams.

    Author(s)
        Fernando Cacciola
    First Release
        1.9.0
    Standard
         
    Categories
        Miscellaneous

Variant

    Safe, generic, stack-based discriminated union container.

    Author(s)
        Eric Friedman and Itay Maman
    First Release
        1.31.0
    Standard
         
    Categories
        Containers, Data structures

Wave

    The Boost.Wave library is a Standards conformant, and highly configurable implementation of the mandated C99/C++ preprocessor functionality packed behind an easy to use iterator interface.

    Author(s)
        Hartmut Kaiser
    First Release
        1.33.0
    Standard
         
    Categories
        String and text processing

Xpressive

    Regular expressions that can be written as strings or as expression templates, and which can refer to each other and themselves recursively with the power of context-free grammars.

    Author(s)
        Eric Niebler
    First Release
        1.34.0
    Standard
         
    Categories
        String and text processing
