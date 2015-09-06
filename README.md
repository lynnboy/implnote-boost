Boost C++ Libraries

* `assert` -- **Assert** | _Customizable assert macros._
  * Assertion custom handler support.
  * STD: `<cassert>`.

* `static_assert` -- **Static Assert** | _Static assertions (compile time assertions)._
  * STD: `static_assert` (C++11).

* `conversion` -- **Conversion** | _Polymorphic casts._
  * `implicit_cast`, `polymorphic_[pointer_][down]cast`, etc.

* `disjoint_sets`
  * Aux algorithm.

* `any` -- **Any** | _Safe, generic container for single values of different value types._
  * STD: `<any>` (Library Fundamentals TS v1).

* `compatibility` -- **Compatibility** | _Help for non-conforming standard libraries._
  * STD: `<cxxxx>`.

* `throw_exception` -- **ThrowException** | _A common infrastructure for throwing exceptions from Boost libraries._
  * Add throwing position as exception error-info.
  * STD: `<exception>` `current_exception` support (C++11).

* `array` -- **Array** | _STL compliant container wrapper for arrays of constant size._
  * STD: `<array>` (tr1, C++11).

* `io` -- **IO State Savers** | _The I/O sub-library of Boost helps segregate the large number of Boost headers._
  * IOS state savers.
  * STD: `quoted` manipulator (C++14).

* `timer` -- **Timer** | _Event timer, progress timer, and progress display classes._
  * Timer classes.

* `rational` -- **Rational** | _A rational number class._
  * STD: proposal N3611, pending.

* `logic` -- **Tribool** | _3-state boolean type library._
  * `tribool` related.

* `tokenizer` -- **Tokenizer** | _Break of a string or other character sequence into a series of tokens._
  * Tokenizer function, iterator, view range.

* `align` -- **Align** | _Memory alignment functions, allocators, and adaptors._
  * `is_aligned`, `aligned_alloc`, `aligned_allocator`, `aligned_delete`
  * STD: `align`, `alignment_of` (C++11)

* `type_index` -- **Type Index** | _Runtime/Compile time copyable type info._
  * STD: `<typeindex>` (C++11)

* `foreach` -- **Foreach** | _BOOST_FOREACH iterates over sequences for us, freeing us from having to deal directly with iterators or write predicates._
  * STD: Range based `for` (C++11)

* `uuid` -- **Uuid** | _A universally unique identifier._
  * Uuid data type & generator/factories
  * SHA1 hasher
  * URNG seed generator (C++11 `random_device`)

* `tuple` -- **Tuple** | _Ease definition of functions returning multiple values, and more._
  * STD: `<tuple>` (C++11)

* `integer` -- **Integer** | _Take advantage of <stdint.h> types from the 1999 C standard._
  * Type selector, mask maker
  * Compile-time and Runtime GCD/LCM and Log2.
  * STD: `<cstdint>` (C++11)

* `assign` -- **Assign** | _Filling containers with constant or generated data has never been easier._
  * STD: `<initializer_list>` (C++11)

* `scope_exit` -- **Scope Exit** | _Execute arbitrary code at scope exit._
  * STD: N4189 - Generic Scope Guard and RAII Wrapper for the Standard Library

* `optional` -- **Optional** | _A value-semantic, type-safe wrapper for representing 'optional' (or 'nullable') objects of a given type._
  * STD: Library fundamentals v1

* `coroutine2` -- **Coroutine2** | _(C++14) Coroutine library._
  * Asymmetric coroutine
  * STD: N4499 - Draft wording for Coroutines (Revision 2)

* `system` -- **System** | _Operating system support, including the diagnostics support that will be part of the C++0x standard library._
  * STD: `<system_error>` (C++11)

* `core`
* `convert`
* `dynamic_bitset`
* `exception`
* `concept_check`
* `endian`
* `detail`
* `function`
* `crc`
* `chrono/stopwatches`
* `lockfree`
* `signals`
* `flyweight`
* `tti`
* `multi_array`
* `lexical_cast`
* `ratio`
* `property_map`
* `functional`
* `winapi`
* `tr1`
* `dll`
* `format`
* `pool`
* `numeric/conversion`
* `sort`
* `numeric/interval`
* `statechart`
* `parameter`
* `iterator`
* `utility`
* `predef`
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

Concept Check

    Tools for generic programming.

    Author(s)
        Jeremy Siek
    First Release
        1.19.0
    Standard
         
    Categories
        Correctness and testing, Generic Programming

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

Convert

    An extendible and configurable type-conversion framework.

    Author(s)
        Vladimir Batov
    First Release
        1.59.0
    Standard
         
    Categories
        String and text processing

Core

    A collection of simple core utilities with minimal dependencies.

    Author(s)
        Peter Dimov, Glen Fernandes and Andrey Semashev
    First Release
        1.56.0
    Standard
         
    Categories
        Miscellaneous

Coroutine

    Coroutine library.

    Author(s)
        Oliver Kowalke
    First Release
        1.53.0
    Standard
         
    Categories
        Concurrent Programming

CRC

    The Boost CRC Library provides two implementations of CRC (cyclic redundancy code) computation objects and two implementations of CRC computation functions. The implementations are template-based.

    Author(s)
        Daryle Walker
    First Release
        1.22.0
    Standard
         
    Categories
        Domain Specific

Date Time

    A set of date-time libraries based on generic programming concepts.

    Author(s)
        Jeff Garland
    First Release
        1.29.0
    Standard
         
    Categories
        Domain Specific, System

Dynamic Bitset

    The dynamic_bitset class represents a set of bits. It provides accesses to the value of individual bits via an operator[] and provides all of the bitwise operators that one can apply to builtin integers, such as operator& and operator<<. The number of bits in the set is specified at runtime via a parameter to the constructor of the dynamic_bitset.

    Author(s)
        Jeremy Siek and Chuck Allison
    First Release
        1.29.0
    Standard
         
    Categories
        Containers

Enable If

    Selective inclusion of function template overloads.

    Author(s)
        Jaakko Järvi, Jeremiah Willcock and Andrew Lumsdaine
    First Release
        1.31.0
    Standard
         
    Categories
        Generic Programming

Endian

    Types and conversion functions for correct byte ordering and more regardless of processor endianness.

    Author(s)
        Beman Dawes
    First Release
        1.58.0
    Standard
         
    Categories
        Input/Output, Math and numerics

Exception

    The Boost Exception library supports transporting of arbitrary data in exception objects, and transporting of exceptions between threads.

    Author(s)
        Emil Dotchevski
    First Release
        1.36.0
    Standard
         
    Categories
        Language Features Emulation

Filesystem

    The Boost Filesystem Library provides portable facilities to query and manipulate paths, files, and directories.

    Author(s)
        Beman Dawes
    First Release
        1.30.0
    Standard
         
    Categories
        System

Flyweight

    Design pattern to manage large quantities of highly redundant objects.

    Author(s)
        Joaquín M López Muñoz
    First Release
        1.38.0
    Standard
         
    Categories
        Patterns and Idioms

Format

    The format library provides a class for formatting arguments according to a format-string, as does printf, but with two major differences: format sends the arguments to an internal stream, and so is entirely type-safe and naturally supports all user-defined types; the ellipsis (...) can not be used correctly in the strongly typed context of format, and thus the function call with arbitrary arguments is replaced by successive calls to an argument feeding operator%.

    Author(s)
        Samuel Krempp
    First Release
        1.29.0
    Standard
         
    Categories
        Input/Output, String and text processing

Function

    Function object wrappers for deferred calls or callbacks.

    Author(s)
        Doug Gregor
    First Release
        1.23.0
    Standard
        TR1
    Categories
        Function objects and higher-order programming, Programming Interfaces

Function Types

    Boost.FunctionTypes provides functionality to classify, decompose and synthesize function, function pointer, function reference and pointer to member types.

    Author(s)
        Tobias Schwinger
    First Release
        1.35.0
    Standard
         
    Categories
        Generic Programming, Template Metaprogramming

Functional

    The Boost.Function library contains a family of class templates that are function object wrappers.

    Author(s)
        Mark Rodgers
    First Release
        1.16.0
    Standard
         
    Categories
        Function objects and higher-order programming

Functional/Factory

    Function object templates for dynamic and static object creation

    Author(s)
        Tobias Schwinger
    First Release
        1.43.0
    Standard
         
    Categories
        Function objects and higher-order programming

Functional/Forward

    Adapters to allow generic function objects to accept arbitrary arguments

    Author(s)
        Tobias Schwinger
    First Release
        1.43.0
    Standard
         
    Categories
        Function objects and higher-order programming

Functional/Hash

    A TR1 hash function object that can be extended to hash user defined types.

    Author(s)
        Daniel James
    First Release
        1.33.0
    Standard
        TR1
    Categories
        Function objects and higher-order programming

Functional/Overloaded Function

    Overload different functions into a single function object.

    Author(s)
        Lorenzo Caminiti
    First Release
        1.50.0
    Standard
         
    Categories
        Function objects and higher-order programming

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

Interval

    Extends the usual arithmetic functions to mathematical intervals.

    Author(s)
        Guillaume Melquiond, Hervé Brönnimann and Sylvain Pion
    First Release
        1.30.0
    Standard
         
    Categories
        Math and numerics

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

Iterator

    The Boost Iterator Library contains two parts. The first is a system of concepts which extend the C++ standard iterator requirements. The second is a framework of components for building iterators based on these extended concepts and includes several useful iterator adaptors.

    Author(s)
        Dave Abrahams, Jeremy Siek and Thomas Witt
    First Release
        1.21.0
    Standard
         
    Categories
        Iterators

Lambda

    Define small unnamed function objects at the actual call site, and more.

    Author(s)
        Jaakko Järvi and Gary Powell
    First Release
        1.28.0
    Standard
         
    Categories
        Function objects and higher-order programming

Lexical Cast

    General literal text conversions, such as an int represented a string, or vice-versa.

    Author(s)
        Kevlin Henney
    First Release
        1.20.0
    Standard
         
    Categories
        String and text processing

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

Lockfree

    Lockfree data structures.

    Author(s)
        Tim Blechmann
    First Release
        1.53.0
    Standard
         
    Categories
        Concurrent Programming

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

Multi-Array

    Boost.MultiArray provides a generic N-dimensional array concept definition and common implementations of that interface.

    Author(s)
        Ron Garcia
    First Release
        1.29.0
    Standard
         
    Categories
        Containers, Math and numerics

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

Numeric Conversion

    Optimized Policy-based Numeric Conversions.

    Author(s)
        Fernando Cacciola
    First Release
        1.32.0
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

Parameter

    Boost.Parameter Library - Write functions that accept arguments by name.

    Author(s)
        David Abrahams and Daniel Wallin
    First Release
        1.33.0
    Standard
         
    Categories
        Language Features Emulation, Programming Interfaces

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

Pool

    Memory pool management.

    Author(s)
        Steve Cleary
    First Release
        1.21.0
    Standard
         
    Categories
        Memory

Predef

    This library defines a set of compiler, architecture, operating system, library, and other version numbers from the information it can gather of C, C++, Objective C, and Objective C++ predefined macros or those defined in generally available headers.

    Author(s)
        Rene Rivera
    First Release
        1.55.0
    Standard
         
    Categories
        Miscellaneous

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

Property Map

    Concepts defining interfaces which map key objects to value objects.

    Author(s)
        Jeremy Siek
    First Release
        1.19.0
    Standard
         
    Categories
        Containers, Generic Programming

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

Ratio

    Compile time rational arithmetic. C++11.

    Author(s)
        Howard Hinnant, Beman Dawes and Vicente J. Botet Escriba
    First Release
        1.47.0
    Standard
        Proposed
    Categories
        Math and numerics

Ref

    A utility library for passing references to generic functions.

    Author(s)
        Jaako Järvi, Peter Dimov, Doug Gregor and Dave Abrahams
    First Release
        1.25.0
    Standard
        TR1
    Categories
        Function objects and higher-order programming

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

Signals (deprecated)

    Managed signals & slots callback implementation.

    Author(s)
        Doug Gregor
    First Release
        1.29.0
    Standard
         
    Categories
        Function objects and higher-order programming, Patterns and Idioms

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

Sort

    High-performance templated sort functions.

    Author(s)
        Steven Ross
    First Release
        1.58.0
    Standard
         
    Categories
        Algorithms

Spirit

    LL parser framework represents parsers directly as EBNF grammars in inlined C++.

    Author(s)
        Joel de Guzman, Hartmut Kaiser and Dan Nuffer
    First Release
        1.30.0
    Standard
         
    Categories
        Parsing, String and text processing

Statechart

    Boost.Statechart - Arbitrarily complex finite state machines can be implemented in easily readable and maintainable C++ code.

    Author(s)
        Andreas Huber Dönni
    First Release
        1.34.0
    Standard
         
    Categories
        State Machines

String Algo

    String algorithms library.

    Author(s)
        Pavol Droba
    First Release
        1.32.0
    Standard
         
    Categories
        Algorithms, String and text processing

Swap

    Enhanced generic swap function.

    Author(s)
        Joseph Gauterin
    First Release
        1.38.0
    Standard
         
    Categories
        Miscellaneous

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

TR1 (deprecated)

    The TR1 library provides an implementation of the C++ Technical Report on Standard Library Extensions. This library does not itself implement the TR1 components, rather it's a thin wrapper that will include your standard library's TR1 implementation (if it has one), otherwise it will include the Boost Library equivalents, and import them into namespace std::tr1.

    Author(s)
        John Maddock
    First Release
        1.34.0
    Standard
        TR1
    Categories
        Miscellaneous

TTI

    Type Traits Introspection library.

    Author(s)
        Edward Diener
    First Release
        1.54.0
    Standard
         
    Categories
        Generic Programming, Template Metaprogramming

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
