---
title: C++ Coding Guidelines
date: 2025-09-15
categories:
  - Theory, Data Structures, Algorithms, Programming Languages, Design Patterns
tags:
  - Reference
---

## Guiding Philosophy

- Prioritize modularity, clarity, and maintainability over premature optimization or legacy patterns.
    - Code should be readable, easily understood, and safe by default.
- Favor Python-style expressiveness with modern C++ strengths:
    - Use strong type systems, STL containers, concepts, range-based and higher-order functions.
    - Avoid C-style loops, macros, and pointer tricks that hide intent.
- Make the compiler's job as easy as possible:
    - Clear ownership, resource lifetime, and data flow allow compilers to optimize code aggressively.
    - By avoiding hidden dependencies and using STL idioms, enable the compiler to perform superior vectorization, inlining, and memory/layout optimizations.
- Design for trivial portability to GPU, FPGA, and other accelerators:
    - Functional, range-based, and stateless/RAII idioms map directly to bulk-parallel and dataflow hardware.
    - State machines using `std::variant` or explicit types, instead of enums and scattered flags, make auto-parallelization and offloading straightforward.
    - Consistent type size/platform rules yield code predictable for high-level synthesis and cross-ISA portability.
    - By steering clear of manual low-level hacks, code lifts cleanly to modern frameworks like CUDA, SYCL, or high-level FPGA toolchains.
- C++ is treated as a platform-level language like Python. The guidelines do not merely govern surface syntax and C++ standard features, but unambiguously specify expectations for the *entire execution environment*:
    - ABI (Application Binary Interface)
    - OS and system libraries (e.g. POSIX semantics, thread/process/signaling behavior)
    - Consistency of type sizes, object layout, exception handling, RTTI, and dynamic loading
    - Deterministic handling and semantics for all STL and language/library features

## Lexical & Filing Conventions

### Indentation

- Always use 4 spaces per indentation level (Python style).
- Tabs, 2-space or 8-space indents are forbidden.

### Functions & Variables

- All function and variable names (including members): `snake_case`, all lowercase.
    - `int max_value;`, `double amount_sum;`
- No CamelCase, PascalCase, or mixing.

### Types/Classes/Structs

- Class, struct, and enum names: `CamelCase` (first letter of each word capitalized), **no underscores.**
    - `class FileReader { ... };`, `struct UserInfo { ... };`

### File Extensions

- `.c`: C-style source code.
- `.m`, `.mm`: Objective-C style source code.
    - We explicitly allow and encourage the usage of POSIX-compliant Objective-C OUTSIDE the Cocoa framework, whenever Python-style "look up by name" is required.
- `.cpp`: C++-style source code.
- `.h`: Only interface declarations, never function/template bodies.
- `.hpp`: Can include full implementations, templates, inline functions.

No other file extensions.

### Include Guards

- Every header (`.h`, `.hpp`) must use an include guard named as the file name in all caps, dots replaced with underscores, with no added prefix or suffix, e.g., 
    - `foo_bar.h` - `#ifndef FOO_BAR_H`
    - `my_class.hpp` - `#ifndef MY_CLASS_HPP`

### File Naming

- All source/header files: `snake_case`, all lowercase, use underscores, e.g.:
    - `network_manager.h`, `data_loader.cpp`
- No CamelCase, hyphens, or uppercase.

## Type System and Data Model

### Type Size Assumptions

| Type         | Bytes |
|--------------|-------|
| `char`       | 1     |
| `short`      | 2     |
| `int`        | 4     |
| `wchar_t`    | 4     |
| `float`      | 4     |
| `long`       | 8     |
| `double`     | 8     |

- Any violation makes the platform "unsupported".

### Struct vs Class Semantics

- Struct: For plain data (POD).
    - Structs NEVER own resources, nor clean up.
    - Always default-initializable to valid state.
    - No custom Big Five (destructor, copy constructor, etc.)
    - Never allocate struct on the heap (no `new StructType`). They must live on stack or inside containers/smart pointers.
    - Do not store pointers/references to struct instance for long-term use.
- Class: For resource management.
    - Must define explicit constructor(s) and Big Five.
    - No raw pointers escape outside internal implementation; prefer smart pointers/containers.

### Function Parameters

- If struct/class is 16 bytes: always pass by reference (`const T&`/`T&`/`T&&` and `std::move`).
- Value passing only for simple types and small STL types.

### Strings & Unicode

- `std::string`: Always UTF-8 encoded.
- `std::wstring` (POSIX): Each `wchar_t` holds a decoded unicode codepoint; use for character-based logic.
    - Serialize or send as UTF-8 (`std::wstring` -> `std::string`) for output/network.

### Subtyping

- Use C++20 concepts and duck typing with templates, NOT class inheritance, whenever possible.
    - Prioritize using the standard library's concepts, and use them according to Python conventions.
        - See Appendix for details.

### Functors

- Always use functors (including lambdas) for generalization/high-order logic.
- C-style function pointers are only allowed for low-level ABI/C-interfacing.

### Container & Data Structure Policy

- Always prefer STL: `vector`, `map`, `set`, `optional`, `variant`, etc.
- Only use third-party containers (boost, absl, folly) with full documentation and technical review.
- Never hand-write data structures for "preemptive optimization".

## Operational Semantics

### Control Flow

- Strongly discourage the use of counting-based for-loops. Hard to read, hard to optimize.
- Only allow `goto` for control flow in plain-POD non-owner code, never with objects needing destruction.
- Use modern functional idioms (map/reduce/filter/apply_if) whenever possible.
- Use explicit patterns like State Machine whenever possible.
    - Important program states must use concrete types and variants, never enums+switches+separate local variables, e.g.,
        - `struct IdleState { /* ... */ };`
        - `struct RunningState { /* ... */ };`
        - `using State = std::variant<IdleState, RunningState>;`
- Use C++20 coroutines for complex control flow.
    - Follow Python generator conventions (`co_yield` = `yield`, "delegating" = `yield from`).
    - Document coroutines in Pythonic style for zero cognitive gap.

### Exceptions & Error Handling

#### Exception Policy

- Use exceptions as standard; do not rewrite everything to error codes or forbid exceptions.
- Only exclude exceptions in hard-constrained "special" cases (must be fully justified & documented).

#### Fail Fast, Fail Loudly

- On detection of error/inconsistency, immediately abort/throw/assert with full context.
    - Never clip, silence, or tolerate errors unless in controlled, reviewed, and documented edge cases.

## Platform & Build System

### Platform, ABI

- Only POSIX-conformant platforms with full POSIX C API for file/IO, threads, sockets, signals, etc. are supported.
- Only Itanium C++ ABI is supported.

### Build Toolchain

- All build/link flows use GCC-style CLI tools, preferably LLVM/Clang.
    - Assume availability of all GCC extensions.
- Always specify all needed options/libraries, e.g.:  
    - `-pthread` (threads)
    - `-lc++` (clang libc++)
    - `-ldl` (dlopen)

### Intermediate Representation

- Always use LLVM IR (`.ll`, `.bc`) as the intermediate layer for cross-platform, analysis, optimization.
- Never use native assembly.

## Appendix: C++ Concepts vs Python Typing Constructs

### Equality, Ordering

| C++ Concept                   | Example                       | Meaning                        | Python Typing Equivalent        |
|-------------------------------|-------------------------------|--------------------------------|---------------------------------|
| `std::equality_comparable<T>` | `equality_comparable<MyType>` | Can test equality (`==`, `!=`) | `typing.SupportsRichComparison` |
| `std::totally_ordered<T>`     | `totally_ordered<MyType>`     | All `<, <=, >, >=` comparisons | `typing.SupportsRichComparison` |

### Numeric and Mathematical Types

| C++ Concept                 | Example                       | Meaning                       | Python Typing Equivalent |
|-----------------------------|-------------------------------|-------------------------------|--------------------------|
| `std::integral<T>`          | `integral<int>`               | Is an integral (integer) type | `numbers.Integer`        |
| `std::signed_integral<T>`   | `signed_integral<int>`        | Is a signed integer type      |                          |
| `std::unsigned_integral<T>` | `unsigned_integral<uint32_t>` | Is an unsigned integer type   |                          |
| `std::floating_point<T>`    | `floating_point<double>`      | Is a floating point type      | `numbers.Real`           |

### Containers

| C++ Concept                       | Example                           | Meaning                                 | Python Typing Equivalent    |
|-----------------------------------|-----------------------------------|-----------------------------------------|-----------------------------|
| `std::ranges::range<T>`           | `range<std::vector<int>>`         | Iterable range concept                  | `typing.Iterable[T]`        |
| `std::ranges::input_range<T>`     | `input_range<std::istream>`       | Readable, single pass                   | `typing.Iterator[T]`        |
| `std::ranges::sized_range<T>`     | `sized_range<std::array<int, 3>>` | Has a known size (`.size()`)            | `typing.Sized`              |
| `std::ranges::output_range<T, V>` |                                   | Writable range (output iterators)       | `typing.MutableSequence[T]` |
| `std::ranges::view<T>`            |                                   | Lightweight, non-owning range           |                             |
| `std::input_iterator<T>`          | `input_iterator<Iter>`            | Supports `++`, deref, read              | `typing.Iterator[T]`        |
| `std::forward_iterator<T>`        | `forward_iterator<Iter>`          | Multi-pass input iterator               | `typing.Iterator[T]`        |
| `std::bidirectional_iterator<T>`  | `bidirectional_iterator<Iter>`    | Forward/backward iteration              | `typing.Sequence[T]`        |
| `std::random_access_iterator<T>`  | `random_access_iterator<Iter>`    | Supports `it[n]` indexing, etc.         | `typing.Sequence[T]`        |
| `std::contiguous_iterator<T>`     | `contiguous_iterator<Iter>`       | Underlying data is contiguous in memory | `typing.Sequence[T]`        |


### Callables

| C++ Concept                  | Example              | Meaning                                  | Python Typing Equivalent     |
|------------------------------|----------------------|------------------------------------------|------------------------------|
| `std::invocable<F, Args...>` | `invocable<Fn, int>` | Callable object (function, lambda, etc.) | `typing.Callable[..., T]`    |
| `std::predicate<F, Args...>` | `predicate<Fn, int>` | Callable returns `bool`                  | `typing.Callable[..., bool]` |

### Type Identity and Conversion

| C++ Concept                           | Example (C++)                                  | Meaning                                    | Python `typing` Equivalent         |
|---------------------------------------|------------------------------------------------|--------------------------------------------|------------------------------------|
| `std::same_as<T, U>`                  | `same_as<int, int>`                            | Types are exactly the same                 |                                    |
| `std::convertible_to<From, To>`       | `convertible_to<int, float>`                   | Can be converted (`static_cast<To>(from)`) |                                    |
| `std::derived_from<D, B>`             | `derived_from<class A, class Base>`            | D inherits from B                          |                                    |
| `std::constructible_from<T, Args...>` | `constructible_from<std::string, const char*>` | Constructible from given args              | `typing.Callable` for constructors |
| `std::default_initializable<T>`       |                                                | Default (no-arg) constructable             |                                    |

### Copying, Moving, Assignment & Swap

| C++ Concept                  | Example                         | Meaning                           | Python Typing Equivalent |
|------------------------------|---------------------------------|-----------------------------------|--------------------------|
| `std::move_constructible<T>` | `move_constructible<MyType>`    | Supports move semantics           |                          |
| `std::copy_constructible<T>` | `copy_constructible<MyType>`    | Can be copy-constructed           |                          |
| `std::assignable_from<T, U>` | `assignable_from<MyType&, int>` | lhs can be assigned rhs           |                          |
| `std::swappable<T>`          | `swappable<MyType>`             | Can exchange values (swap)        |                          |
| `std::swappable_with<T, U>`  | `swappable_with<A, B>`          | Can swap values with another type |                          |

