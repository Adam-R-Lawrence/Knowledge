<!--
title: C++
tags: [c++, build, compilation]
-->

## C++

### `-march=native`
- Meaning: Compiler flag (GCC/Clang) to target the host CPU’s micro-architecture and enable its instruction set (e.g., AVX2/AVX-512).
- Pros: Unlocks CPU-specific vector/SIMD and tuning for best performance.
- Cons: Binaries may not run on older/different CPUs; prefer portable defaults and enable per-target builds when distributing.
- Related: Pair with `-O3`/`-Ofast` judiciously; use runtime dispatch or fat binaries for portability.

### `.inl` files
- Purpose: Convention for “inline implementation” files included into headers, often for templates or small functions kept in headers for ODR/visibility.
- Usage: Included from a header (e.g., `foo.hpp` includes `foo.inl`) to keep declarations and definitions logically separated but still available to all translation units.
- Caution: Maintain include guards and keep `.inl` only included by a single public header to avoid multiple-definition issues.

### `std::filesystem::absolute`
- Purpose: Produces an absolute path from a relative or mixed input, resolving `.`/`..` segments using the process’s current working directory.
- Behavior: Returns a `std::filesystem::path` without touching the filesystem; it does not check existence. Call `std::filesystem::canonical` when you need symlink resolution and validation.
- Portability: Available since C++17 in `<filesystem>`; use `std::filesystem::absolute(p, base)` to anchor resolution against a custom base instead of `current_path()`.

### `std::filesystem::directory_iterator`
- Usage: Range-style iteration over the immediate children of a directory; emits `directory_entry` objects with cached status/info.
- Lifetime: Default-constructs to an end iterator; increment throws on errors unless you pass in a `std::error_code` reference overload to collect errors non-throwingly.
- Tips: Prefer `recursive_directory_iterator` for depth-first traversal; set `directory_options::skip_permission_denied` to keep walking when encountering restricted subtrees.

### `std::quoted`
- Header: `<iomanip>` manipulator for stream insertion/extraction of quoted strings with escapes.
- Behavior: Writing wraps text in double quotes and escapes embedded quotes and backslashes; reading strips the quotes and unescapes.
- Parameters: `std::quoted(str, delim, escape)` lets you customize delimiters and escape characters for formats such as CSV or custom protocols.

### `std::move`
- Purpose: Casts an lvalue to an rvalue reference, signaling that its resources may be moved from; enables move constructors and move assignment operators.
- Semantics: Performs no move itself—it's a cast; the destination object decides how to steal resources. Only call when you will not use the source in its original state afterward (but it must remain valid).
- Pitfalls: Never `std::move` when you still need the value; avoid moving from the same object twice unless the type documents that it is safe.

### `std::invalid_argument`
- Location: `<stdexcept>` exception thrown to signal that a function received an argument outside its valid domain.
- Usage: Prefer for precondition violations detected at runtime (e.g., negative sizes, malformed strings); include a descriptive message to aid debugging.
- Handling: Catch by reference (`catch (const std::invalid_argument& e)`) to inspect `e.what()`; use alongside `std::domain_error`/`std::out_of_range` to express different contract breaches.

### POSIX
- Acronym: Portable Operating System Interface; a family of IEEE/ISO standards defining common Unix APIs (processes, files, signals, threads).
- Scope: Governs system calls (`open`, `fork`, `waitpid`), shells, utilities, and threading primitives (`pthread_*`); ensures source-level portability across Unix-like systems.
- C++ interplay: Many C libraries and platform APIs assume POSIX semantics; when writing cross-platform C++, isolate POSIX-specific code and provide WIN32 alternatives or use higher-level wrappers (Boost.Asio, `std::filesystem`, `std::thread`) where possible.
