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

