<!--
title: Parallel Programming
tags: [parallel, concurrency]
-->

## Parallel Programming

Parallel programming structures computations to run simultaneously across cores, vector units, or nodes. Hardware behavior strongly shapes performance; the notes below group key concepts for quick recall.

### Background
- Moore's Law: Transistor counts roughly double periodically, enabling exponential compute growth (historical trend; now slowing).

### Instruction-Level Parallelism (CPU Execution)
- Pipelining: Overlaps instruction stages to increase throughput without raising clock speed.
- Branch prediction: Anticipates control flow to keep pipelines busy; mispredictions flush work and waste cycles.
- Data forwarding: Bypasses produced values directly to dependent stages, reducing read-after-write stalls.

### OpenMP Essentials
- Programming model: Shared-memory API that uses compiler directives (`#pragma omp`) to spawn threads and manage work sharing.
- Master vs workers: The master thread executes serial regions and distributes parallel work; worker threads cooperate inside parallel regions.
- `#pragma omp parallel`: Creates a team of threads to execute a block; use `num_threads` or environment variables to size the team.
- `#pragma omp parallel for`: Combines team creation with loop work sharing; default scheduling is implementation-defined but often static.
- Scheduling: `schedule(static)` assigns contiguous iteration chunks per thread, minimizing scheduling overhead; `schedule(dynamic)` hands out chunks at runtime to balance irregular workloads (extra overhead from coordination).
  - Examples:
    ```c
    #pragma omp parallel for schedule(static)
    for (int i = 0; i < n; ++i) { /* work */ }
    
    #pragma omp parallel for schedule(dynamic)
    for (int i = 0; i < n; ++i) { /* irregular work */ }
    ```
- Data scoping:
  - `private(var)`: Each thread gets an uninitialized private copy; updates do not affect the original variable.
  - `firstprivate(var)`: Like `private`, but copies the initial value from the master thread into each thread's private copy.
- Reductions: `reduction(op:var)` accumulates thread-local partials using `op` (add, multiply, max, etc.) and combines them at region end; operations must be associative and commutative—division is disallowed.
- Loop-carried dependencies: Only parallelize loops when iterations are independent; use techniques like loop skewing, prefix sums, or privatization to break dependencies before adding `parallel for`.

### Memory Hierarchy & Locality
- Cache: Small, fast memory that keeps recently used data to reduce average access latency.
- Cache line: Minimum transfer unit between memory and cache (e.g., 64 bytes).
- Multi-word cache lines: Lines contain adjacent words to exploit spatial locality.
- Spatial locality: Accesses tend to cluster near recent addresses; contiguous layouts benefit.
- Overview: Registers → L1 → L2 → L3 (LLC) → DRAM → storage; capacity grows and latency/bandwidth worsen down the hierarchy.
- Working set: Keep hot data within L1/L2 when possible; structure algorithms to maximize temporal/spatial locality.
- Cross-reference: See “Memory Hierarchy & Locality”, “Cache Organization”, and “Cache Levels” sections for details.

### Cache Organization
- Fully associative: Any block can occupy any cache line; minimizes conflicts with higher lookup cost.
- LRU (least recently used): Replacement policy that evicts the least recently used line, leveraging temporal locality.

### Cache Miss Types
- Compulsory miss: First access (cold) miss; data has not been loaded yet.
- Conflict miss: Limited associativity maps multiple blocks to the same set, causing avoidable misses.
- Capacity miss: Working set exceeds cache size, causing misses even with optimal placement.

### Cache Thrashing
- Definition: Pathological sequence of accesses that repeatedly evicts and reloads cache lines, typically due to many active addresses mapping to the same set (conflict-driven). Results in very high miss rates and low effective cache reuse.
- Not the same as a cache miss: A cache miss is a single event; thrashing is a behavior that causes excessive misses (most often conflict misses). Thrashing can also occur at TLB levels.
- Triggers: Power-of-two strides, arrays whose leading dimensions align poorly with set mapping, or multiple threads hammering the same sets.
- Mitigations: Change strides or loop order; apply padding/array realignment; use blocking/tiling; increase associativity (hardware) or use software techniques like index hashing; avoid false sharing across threads.

### Metrics & Operations
- FLOPs: Floating-point operations per second; a common compute-throughput metric.
- Load operations: Instructions that read memory into registers; may hit or miss in cache and dominate latency.

### Cache Levels (L1, L2, L3)
- L1: Smallest, fastest, per-core caches (separate I/D on many CPUs); tens of KB; few-cycle latency.
- L2: Larger per-core (hundreds of KB to a few MB); higher latency; buffers L1 misses.
- L3 (LLC): Shared among cores on a socket; multi-MB; higher latency; reduces DRAM traffic and supports coherence.
- Inclusive/exclusive policies: Some architectures duplicate lines across levels (inclusive) vs store uniquely (exclusive).
- Performance tip: Keep working sets within L1/L2 when possible; measure with cachegrind/perf.

### Vectorization (SIMD)
- Data-parallel operations apply one instruction to multiple elements (e.g., SSE/AVX/AVX-512, NEON, SVE).
- Alignment and contiguous memory layouts improve throughput; minimize gather/scatter and misaligned accesses.
- Loop structure: Favor simple, stride-1 loops with no data dependencies; enable compiler auto-vectorization.
- Intrinsics: Explicit control when the compiler struggles; balance readability vs performance.
- Memory-bound limits: SIMD helps only if bandwidth/latency isn’t the bottleneck; consider blocking and prefetching.

### Tiling (Blocking)
- Definition: Reorder loops to operate on small subproblems (tiles) that fit into a target cache level to maximize data reuse before eviction.
- Benefits: Reduces capacity/conflict misses, improves TLB locality, and exposes reuse for SIMD and multi-threading.
- Multi-level: Nest tiles to target L1, then L2/L3; choose tile sizes so the working set per tile stays well below the cache size with headroom.
- See also: “Arithmetic Intensity, Tiling, Unrolling, Prefetching” for additional guidance and heuristics.
### DVFS (Dynamic Voltage and Frequency Scaling)
- Trade-off between performance, power, and thermals by adjusting per-core or global frequency/voltage.
- Turbo/boost increases frequency opportunistically; thermal or power limits can throttle under sustained load.
- Latency-sensitive vs throughput workloads may benefit from different governor choices (e.g., performance vs ondemand).
- Parallel code may hit power limits earlier; watch aggregate socket power and frequency clipping.

### MSI Cache Coherence Protocol
- States: Modified (owned/dirty), Shared (clean, read-only), Invalid (not present/invalid).
- Loads move lines to S; stores upgrade to M, invalidating other S copies via coherence traffic.
- Snooping or directory-based mechanisms propagate invalidations/ownership changes among cores.
- Related: MESI/MOESI add Exclusive/Owned states to reduce write upgrades or memory writebacks.
- False sharing: Independent variables on the same cache line cause needless invalidations; mitigate with padding/tiling.

### Registers, Stack, Heap
- Registers: Fastest storage, limited quantity; compiler allocates for hot variables; spills go to stack.
- Stack: Per-thread, contiguous, LIFO allocation for call frames; fast, size-limited; great locality.
- Heap: Dynamic, long-lived allocations; flexible but slower, may fragment; allocator choice matters under contention.
- Guidance: Prefer stack/arena allocations for hot paths; reduce pointer chasing to aid cache and vectorization.

### Context Switch Time
- Switching threads/processes incurs saving/restoring registers, TLB effects, and cache working-set disruption.
- Frequent switches reduce effective cache residency and increase tail latency on shared cores.
- Mitigations: Use thread pools, pin threads (affinity), reduce lock contention, batch work to amortize overhead.
- Measure with perf tracepoints, scheduler stats, or microbenchmarks; consider isolating cores for critical threads.

### Cache Pollution
- Occurs when streaming/one-time data evicts hot working sets, harming hit rates.
- Mitigations: Software prefetch hints, non-temporal loads/stores, cache-bypassing for streams, and tiling/blocking.
- Algorithmic: Reuse-friendly orderings (loop interchange, blocking), compact data layouts, and separating hot/cold data.
- Observe via cache miss counters and LLC occupancy; validate with A/B profiling.

### I/O Waiting Time
- Time spent blocked on disk/network/IPC; often dominates latency in data pipelines.
- Overlap compute and I/O via async APIs, DMA, queues, and double-buffering; increase queue depth where safe.
- Batch small operations to amortize syscalls; use zero-copy paths and pin memory for high-throughput NICs.
- For mixed workloads, partition I/O and CPU threads to reduce interference and context switches.

### Process, Transistors, and Clocks
- Definition: Semiconductor process “node” (e.g., 7 nm) indicating the scale of transistor features and interconnects.
- Implications: Smaller features increase density and potential switching speed, but wire RC delay and leakage become limiting factors.
- System impact: More transistors enable more cores, wider SIMD, and larger caches; memory latency/bandwidth improve much more slowly than compute, widening the memory wall.

- Role: Fundamental switches implementing logic, registers, and SRAM (caches). Count largely determines core count, cache capacity, and accelerator scope.
- Power: Dynamic power ≈ α · C · V² · f; leakage rises at smaller nodes, tightening power/thermal budgets.
- Parallel impact: Abundant transistors are often “spent” on parallel resources (cores/SMs, vector width, cache) rather than single-thread frequency alone.

- Definition: `Tclk` is time per cycle, set by the critical path; deeper pipelines reduce logic per stage to shorten `Tclk`.
- Performance relation: Execution time ≈ cycles × `Tclk`; per-instruction time ≈ `CPI × Tclk`.
- Trade-offs: Shorter `Tclk` (higher frequency) can increase branch mispredict penalties (more pipeline stages) and power; may not help memory-bound code.

- Definition: `fclk = 1 / Tclk`. Higher `fclk` increases potential throughput if the workload isn’t bottlenecked by memory or I/O.
- DVFS: Frequency and voltage scale together; boosts are power/thermally limited and may drop under heavy parallel load.
- Guidance: Profile first—if stalled on memory, target locality, tiling, prefetching, and algorithmic reuse before chasing frequency.


### Arithmetic Intensity, Tiling, Unrolling, Prefetching
- Definition: Useful operations per byte moved from a given level of memory (often DRAM). `AI = FLOPs / Bytes`.
- Roofline: Attainable performance is min(compute peak, memory bandwidth × AI); low-AI kernels are memory-bound.
- Increase AI: Reuse data via blocking/tiling, fuse kernels, reduce precision/bytes moved when acceptable, and compress or reorder data.
- Measure: Use performance counters to estimate bytes transferred and FLOPs; validate shifts in AI after optimizations.

- Idea: Restructure computation to operate on subproblems that fit in a target cache level, increasing reuse before eviction.
- Multi-level: Choose tiles for L1, then nest larger tiles for L2/L3; consider TLB/page effects and associativity.
- Heuristic: Ensure (working arrays × tile_size × element_size) comfortably fits within a fraction of the cache (leave headroom for stack/metadata).
- Use cases: Matrix multiply/convolutions/stencils; combine with thread- and NUMA-aware partitioning to minimize cross-socket traffic.

- Purpose: Reduce branch overhead and expose instruction-level parallelism (ILP) to aid scheduling and vectorization.
- Trade-offs: Larger code size can pressure I-cache; higher register pressure may cause spills; tune unroll factors (e.g., 2–8) empirically.
- Variants: Unroll-and-jam for nested loops; pair with software pipelining and prefetching for streaming kernels.

- Hardware: Modern CPUs/GPU SMs detect streams/strides and issue prefetches; effectiveness drops for irregular access patterns.
- Software: Insert explicit prefetch hints (e.g., `_mm_prefetch`) with a distance ≈ memory_latency / cycles_per_iteration; use non-temporal hints for one-time streams.
- Pitfalls: Over-prefetching can pollute caches or waste bandwidth; tune distances and target levels (L1/L2) with profiling.
- Complementary: Combine with tiling and layout changes to make access patterns more predictable for both hardware and software prefetchers.
