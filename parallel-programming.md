<!--
title: Parallel Programming
tags: [parallel, concurrency]
-->

## Parallel Programming

Parallel programming structures computations to run simultaneously across cores, vector units, or nodes. Hardware behavior strongly shapes performance; the notes below group key concepts for quick recall.

### Background
- Moore's Law: Transistor counts roughly double periodically, enabling exponential compute growth (historical trend; now slowing).
- Amdahl's Law: Overall speedup = `1 / ((1 - p) + p / s)`; diminishing returns from parallelism when serial fractions remain significant—optimize sequential bottlenecks first.
- Encore Multimax: Early shared-memory multiprocessor (mid-1980s) with snoopy caches; a reminder that topology, cache coherence, and scheduling policies matter as much as raw core count.
- Execution semantics: Formal meaning of a parallel program—the orderings guaranteed (or allowed) by the memory model and runtime; reason about them to avoid relying on undefined interleavings.

### Instruction-Level Parallelism (CPU Execution)
- Pipelining: Overlaps instruction stages to increase throughput without raising clock speed.
- Branch prediction: Anticipates control flow to keep pipelines busy; mispredictions flush work and waste cycles.
- Data forwarding: Bypasses produced values directly to dependent stages, reducing read-after-write stalls.
- ILP (instruction-level parallelism): Count of independent operations the core can issue simultaneously; boost by reordering code, unrolling, and minimizing dependencies.
- Instructions in flight: Out-of-order cores keep dozens to hundreds of instructions queued across pipeline stages to hide latency; a single instruction in flight still occupies resources until it graduates, so dependency chains limit this window.
- Register renaming: Hardware maps architectural registers onto a larger physical pool to eliminate false dependencies (WAR/WAW) and sustain ILP.
- Pipeline bubble: An empty slot in the pipeline caused by hazards or cache misses; too many bubbles drop IPC and waste issue bandwidth.
- Asynchronous load instructions: Architectures like NVIDIA GPUs (`cp.async`, `ldg`) and Arm/PowerPC prefetch ops let software request a load and use the data later, overlapping memory latency with independent computation.
- Simultaneous multithreading (SMT): Allows one core to issue instructions from multiple hardware threads in the same cycle (a.k.a. simultaneous multi-threading); boosts utilization when one thread stalls, but threads contend for caches and execution units.
- Load-store interleaving: Mix independent loads and stores so the memory subsystem stays busy while arithmetic pipelines work, hiding latency and reducing structural hazards.

### OpenMP Essentials
- Programming model: Shared-memory API that uses compiler directives (`#pragma omp`) to spawn threads and manage work sharing.
- Master vs workers: The master thread executes serial regions and distributes parallel work; worker threads cooperate inside parallel regions.
- Master thread: Thread 0 in an OpenMP team; handles serial regions before and after parallel constructs and executes `single` regions when `omp_get_thread_num() == 0`.
- `#pragma omp master`: Restricts a structured block to the master thread while worker threads skip it without synchronization; add an explicit `#pragma omp barrier` afterward if they must rendezvous before continuing.
- `#pragma omp parallel`: Creates a team of threads to execute a block; use `num_threads` or environment variables to size the team.
- `#pragma omp parallel for`: Combines team creation with loop work sharing; default scheduling is implementation-defined but often static.
- Scheduling:
  - Chunk: Group of consecutive loop iterations assigned together; chunk size trades load balance for overhead (bigger chunks = fewer handoffs).
  - `schedule(static[, chunk])`: Predictable round-robin distribution; best for uniform loop bodies and cache reuse.
  - `schedule(dynamic[, chunk])`: Threads grab chunks on demand; smooths imbalance at the cost of extra synchronization.
  - `schedule(guided[, chunk])`: Starts with large chunks that shrink toward the provided chunk size; good for workloads whose cost drops over time.
  - `schedule(runtime)`: Defer choice to `OMP_SCHEDULE` at execution time—handy for tuning without recompiling.
  - `schedule(auto)`: Let the compiler/runtime pick based on heuristics; check generated code to ensure the choice matches expectations.
  - Guided schedule with chunk: Combine both to cap the minimum chunk size, e.g., `schedule(guided, 32)` to avoid hyper-fragmented tail work.
- Data scoping:
  - `private(var)`: Each thread gets an uninitialized private copy; updates do not affect the original variable.
  - `firstprivate(var)`: Like `private`, but copies the initial value from the master thread into each thread's private copy.
  - `shared(var)`: Declares variables visible (and potentially contend) across all threads; use sparingly because writes require synchronization.
  - `lastprivate(var)`: Copies the value from the lexicographically last iteration (per sequential order) back to the original variable after the parallel loop.
- Stack vs private vs shared variables: Automatic (stack) locals inside parallel regions default to private copies unless declared `shared`; globals and heap objects default to shared—override with clauses when that default is wrong.
- Reductions: `reduction(op:var)` accumulates thread-local partials using `op` (add, multiply, max, etc.) and combines them at region end; operations must be associative and commutative—division is disallowed.
  - Example: `#pragma omp parallel for reduction(min:best)` keeps the minimum value across iterations; initialize `best` to a sentinel larger than any candidate.
- Loop-carried dependencies: Only parallelize loops when iterations are independent; use techniques like loop skewing, prefix sums, or privatization to break dependencies before adding `parallel for`.
- Implicit synchronization: Parallel regions and `parallel for` include an implicit barrier at the end; add `nowait` when safe to skip it and avoid idle threads.
- `default(none)`: Forces explicit data-sharing clauses on every variable, preventing accidental sharing and catching bugs at compile time.
- `default(private)`: Gives each thread a private copy by default; useful when most variables should be thread-local, but requires explicit `shared` for coordinated data.

### OpenMP Tasking & Clauses
- `#pragma omp task`: Defers execution of a block to be run by any thread in the current team; tasks capture shared/private data environment just like parallel regions.
- `#pragma omp taskwait`: Synchronizes the generating task with all outstanding child tasks, ensuring dependents finish before continuing.
- `priority(n)`: Optional clause hinting to the runtime which tasks should be scheduled first; higher `n` usually executes earlier but is advisory.
- `collapse(n)`: Applies to nested loops; flattens the first `n` loops into a single iteration space so the runtime sees more iterations to distribute evenly.
- `private(list)`: Explicit data-sharing clause; declare scalars or temporaries `private` when each thread must have its own instance instead of sharing and racing on a single variable.
- Padding: Add unused array slots or structure members so thread-private data lands on different cache lines, reducing false sharing in OpenMP worksharing loops.
- Output multiplicity: Count of output elements an iteration/task may update; track it to choose between atomic updates, reductions, or privatized scratch buffers.

### OpenMP Directive Cheatsheet
- `#pragma omp parallel`: Spawns a team of threads to execute a structured block once per thread; combine with clauses like `default(shared)` or `num_threads(n)`.
- `#pragma omp parallel for`: Creates a team and distributes loop iterations automatically; default scheduling follows implementation but can be overridden with `schedule`.
- `#pragma omp sections` / `#pragma omp section`: Divide work into independent blocks executed by different threads; great for heterogeneous tasks but still imply a barrier at the end.
- `#pragma omp critical`: Serializes entry to the enclosed block so only one thread executes it at a time; keep regions small to limit contention.
- `#pragma omp atomic`: Provides low-overhead, atomic read-modify-write for simple updates (add, min, max); use instead of `critical` when updating a single scalar.
- `#pragma omp barrier`: All threads in the current team pause until everyone arrives; use to enforce phase ordering within a parallel region.
- `#pragma omp single`: Executes the enclosed block on just one thread (not necessarily the master) while others skip ahead; pair with `nowait` to avoid the implicit barrier if safe.
- `#pragma omp ordered`: Enforces sequential order within a `for` loop when iterations must commit results in index order; minimize use to preserve parallel efficiency.
- `#pragma omp ordered depend(...)`: OpenMP 5+ feature enabling fine-grained dependencies between iterations; tag producers with `depend(source)` and consumers with `depend(sink: var)` (optionally listing multiple indices) to express partial order without full barriers.

### Synchronization Concepts
- Mutual exclusion: Guarantee that only one thread executes a critical section at a time; implement with locks, `critical`, or atomic primitives.
- Mutual exclusion enforcement: Combine mutexes/locks with disciplined lock ordering so every shared update runs under the same guard, preventing inconsistent views.
- Critical section: Code region protected by mutual exclusion; keep it short and side-effect free to minimize contention.
- Global critical: An unnamed `#pragma omp critical` that serializes across the entire program; all such regions share a single lock.
- Named critical: `#pragma omp critical(name)` isolates contention to matching names so unrelated regions can proceed concurrently.
- Global lock: Single `omp_lock_t` shared across threads; emulates a process-wide mutex—use sparingly because it serializes all holders.
- Named lock: Separate `omp_lock_t` instances for independent resources; avoids unrelated contention.
- Barrier: Synchronization point where threads wait until all have reached the same program location; use sparingly to avoid idle time.
- Atomic operation: Hardware-supported read-modify-write (e.g., fetch-add) that completes indivisibly; ideal for counters or reductions.
- Lock routine: Explicit lock APIs (`omp_init_lock`, `omp_set_lock`, `omp_unset_lock`) for cases where critical regions must span multiple functions or require manual control.
- Locks: Use `omp_lock_t` (or `omp_nest_lock_t` for recursive locks) to protect shared data; always pair `omp_init_lock` with `omp_destroy_lock` to avoid leaks.
- Race condition: Occurs when multiple threads access the same data without proper synchronization and at least one access is a write; manifests as nondeterministic outputs.

### Memory Consistency & Ordering
- Happens-before relation: Defines when one memory operation is guaranteed to be visible before another; create edges via barriers, atomics, locks, or task dependencies.
- Flush directive: Forces a thread to synchronize its view of specified shared variables with memory, making updates visible to other threads observing the same flush set.
- `#pragma omp flush(varlist)`: OpenMP directive for the flush; name variables explicitly to limit scope, and pair producers/consumers with matching flushes.
- Point-to-point synchronization: Coordinate just the communicating threads (e.g., producer/consumer pairs) using atomics, locks, or condition variables instead of whole-team barriers.
- Busy-wait loop (spin-wait): Thread repeatedly polls a condition without sleeping; insert pause/backoff instructions and ensure the polled variable is touched via atomic/flush so updates are observable.
### OpenMP Runtime Library
- `omp_get_thread_num()`: Returns the calling thread’s ID (0-based) within the current team; valid inside parallel regions.
- `omp_get_num_threads()`: Reports the number of threads in the current team; returns 1 when executed outside a parallel region.
- `omp_get_max_threads()`: (related) Gives the size of the team that would be created by the next `parallel` region, honoring environment settings and `omp_set_num_threads`.

### Threading Models
- Threads: OS-scheduled execution contexts sharing a process address space; lighter than processes but still incur context-switch and synchronization overhead.
- Pthreads: POSIX threads API (`pthread_create`, `pthread_join`, locks, condition variables); foundation for many C/C++ parallel runtimes including OpenMP implementations.
- Implicit vs explicit threading: OpenMP hides thread creation behind directives, whereas pthreads requires manual lifecycle management—pick the level matching your control needs.

### MPI Basics
- MPI rank: Unique integer identifier assigned to each process within a communicator; drives data partitioning and determines which workload slice a process owns.
- Address space: The private memory owned by a rank; one-sided MPI operations expose selected regions via windows so peers can access them without attaching to the host process's call stack.
- Oversubscription: Running more software threads or MPI ranks than available hardware execution contexts; increases latency and cache contention, so use only when hiding I/O stalls or testing scalability on limited hardware.

### MPI Synchronization & Collectives
- `MPI_Barrier`: Blocking collective that forces all ranks in a communicator to wait until everyone arrives; helpful for phase transitions or timing sections but avoid overuse because it serializes progress.
- `MPI_Allreduce`: Combines values from all ranks with an associative operation (sum, max, logical and, etc.) and broadcasts the result to everyone; essential for global norms, dot products, and convergence checks.

### MPI Remote Memory Access (RMA)
- `MPI_Get`: One-sided read that fetches data from a target rank's exposed window without involving the target CPU in the critical path; complete the access with matching fence or lock/unlock calls to ensure ordering.
- `MPI_Put`: One-sided write pushing data into a target window; enables overlap by letting the origin rank continue once the transfer is initiated, subject to subsequent synchronization.
- `MPI_Accumulate`: Atomic read-modify-write on a remote window using a specified operation (e.g., sum, max); ideal for distributed counters or assembling halo contributions without explicit receive loops.

### MPI Thread Support Levels
- `MPI_THREAD_SINGLE`: Only one thread exists in the process; simplest level with minimal internal locking overhead.
- `MPI_THREAD_SERIALIZED`: Multiple application threads may exist, but at most one may make MPI calls at a time; enforce with external mutexes or thread funnels.
- `MPI_THREAD_MULTIPLE`: Fully concurrent MPI calls from multiple threads are permitted; offers maximal flexibility but depends on the implementation's locking granularity for performance.

### Memory Hierarchy & Locality
- Cache: Small, fast memory that keeps recently used data to reduce average access latency.
- Cache line: Minimum transfer unit between memory and cache (e.g., 64 bytes).
- Cache block: Alternative term for a cache line; caches move and tag memory in these fixed-size chunks.
- Multi-word cache lines: Lines contain adjacent words to exploit spatial locality.
- Spatial locality: Accesses tend to cluster near recent addresses; contiguous layouts benefit.
- Access stride: Step size between successive memory touches (e.g., 1, 2, 64); stride-1 enables prefetching and cache reuse, while large or irregular strides degrade locality.
- Indirection array: Auxiliary index array used to reorder or gather scattered data; helpful for locality but adds extra memory traffic and can hinder vectorization when access patterns become unpredictable.
- Indirection: Any extra level of pointer chasing or index lookups; increases latency and reduces spatial locality—flatten structures when profiling points to pointer-heavy bottlenecks.
- Overview: Registers → L1 → L2 → L3 (LLC) → DRAM → storage; capacity grows and latency/bandwidth worsen down the hierarchy.
- DRAM: Off-chip main memory with high capacity and ~50–100 ns latency; saturates on bandwidth-intensive workloads without sufficient cache reuse.
- Working set: The subset of data touched over a short interval; keep it within L1/L2 to sustain hit rates, and reshape algorithms when the working set eclipses those capacities.
- Cold cache: State where caches are empty (startup, context switch, flush); expect compulsory misses until the working set is populated.
- Row-major format: Consecutive elements of a row are adjacent in memory (C/C++ default); iterate rows innermost to stream through cache lines.
- Column-major format: Consecutive elements of a column are adjacent (Fortran, MATLAB); swap loop order to traverse columns sequentially on such layouts.
- C array of pointers: Stores an array of row pointers instead of a flat contiguous block; simplifies ragged matrices but hurts spatial locality and increases TLB/cache pressure.
- Array access patterns: Reorder loops and data layouts so threads traverse memory with stride-1 access; mixed patterns (gathers, scatters) benefit from tiling, structure-of-arrays, or software-managed staging buffers.
- Cross-reference: See “Memory Hierarchy & Locality”, “Cache Organization”, and “Cache Levels” sections for details.

### Cache Organization
- Direct-mapped cache: Each memory block maps to exactly one cache line; fast lookups but susceptible to conflict misses.
- Set-associative cache: Cache sets contain multiple ways so a block can reside in any way within its set, trading modest lookup cost for fewer conflicts.
- Fully associative cache: Any block can occupy any cache line; minimizes conflicts at the expense of complex lookup hardware.
- Direct vs set vs fully associative: Direct-mapped caches are effectively 1-way set associative; increasing associativity reduces conflict misses but raises hit latency, area, and power.

### Replacement Policies
- LRU (least recently used): Evicts the least recently used line to capitalize on temporal locality; often approximated in hardware.
- Random replacement: Selects a victim line uniformly at random; cheap and effective when access patterns lack predictability.

### Cache Miss Types
- Compulsory miss: First access (cold) miss; data has not been loaded yet.
- Conflict miss: Limited associativity maps multiple blocks to the same set, causing avoidable misses.
- Capacity miss: Working set exceeds cache size, causing misses even with optimal placement.
- Read miss: Load request that finds the target line absent; can stall execution until the line returns from the next level.
- Write miss: Store request to a line not present; handled via write-allocate (fetch then update) or write-no-allocate (write-around) policies.
- Eviction: Removal of a line to make space for another; dirty evictions trigger write-back, while clean lines can be dropped.

### Cache Thrashing
- Definition: Pathological sequence of accesses that repeatedly evicts and reloads cache lines, typically due to many active addresses mapping to the same set (conflict-driven). Results in very high miss rates and low effective cache reuse.
- Not the same as a cache miss: A cache miss is a single event; thrashing is a behavior that causes excessive misses (most often conflict misses). Thrashing can also occur at TLB levels.
- Triggers: Power-of-two strides, arrays whose leading dimensions align poorly with set mapping, or multiple threads hammering the same sets.
- Mitigations: Change strides or loop order; apply padding/array realignment; use blocking/tiling; increase associativity (hardware) or use software techniques like index hashing; avoid false sharing across threads.

### Cache Coherence Traffic
- Definition: Volume of cache-line transfers triggered by the coherence protocol to keep shared data consistent across cores, often measured as invalidations or snoop messages.
- Cost: Excess traffic inflates memory latency, burns bandwidth on interconnects, and can serialize updates—especially for fine-grained sharing or ping-ponging.
- Reduction: Favor thread-local data, apply privatization or reductions, align write-heavy data so each thread updates disjoint cache lines, and batch communication into coarser phases.

### Cache Coherence Protocols
- Cache coherence protocol: Ruleset that ensures all cores observe consistent values for shared memory despite private caches; defines states, transitions, and communication.
- Snoopy protocol: Broadcast-based coherence where caches observe (“snoop”) a shared bus for actions by peers—simple and effective for small SMPs like Encore Multimax.
- Directory-based protocol: Stores sharer lists in a directory per line; scales to many cores by sending targeted invalidations instead of broadcasts.
- MSI protocol: Minimal three-state (Modified, Shared, Invalid) coherence; see “MSI Cache Coherence Protocol” for transitions and performance notes.

### Cache Performance Metrics
- Hit ratio: Fraction of memory accesses served from a given cache level; rising hit ratios correlate with better locality and reduced lower-level traffic.
- Reuse ratio: Fraction of accesses that revisit already-fetched data; low reuse suggests streaming patterns that benefit from tiling or software-managed buffers.
- Miss penalty: Additional latency or cycles required to service a miss from the next level (hundreds of cycles when falling back to DRAM).
- One-eighth miss ratio: Rule-of-thumb target where only 1 in 8 accesses miss (12.5%); keeping miss ratios near or below this level prevents bandwidth from dominating execution time.
- Throughput: Useful work per unit time (e.g., FLOPs/s, tasks/s); depends jointly on compute resources, memory bandwidth, and the miss penalty profile.

### MESI Protocol
- States: Modified, Exclusive, Shared, Invalid—each cache line lives in one state per core, dictating read/write permissions and coherence actions.
- Transitions: Writes promote lines to Modified (invalidating others), reads move Invalid → Shared/Exclusive depending on sharers, and evictions of Modified lines trigger write-back to memory.
- Performance: MESI keeps single-owner writes fast but punishes ping-pong patterns; minimize shared-line bouncing via data partitioning and avoiding false sharing.

### False Sharing
- Problem: Independent thread-local variables mapped to the same cache line cause coherence invalidations with every write, tanking performance despite no true data dependency.
- Detection: Look for strided accesses to structs/arrays of small scalars in profiling tools (`perf stat`, Intel VTune false-sharing analysis).
- Fixes: Add cache-line padding between per-thread data, restructure arrays-of-structs into struct-of-arrays, or use OpenMP `aligned`/`declare simd` clauses with attentiveness to layout.

### Metrics & Operations
- FLOPs: Floating-point operations per second; a common compute-throughput metric.
- Load operations: Instructions that read memory into registers; may hit or miss in cache and dominate latency.
- Word size: Fundamental data unit; single precision uses 32-bit (4-byte) words, double precision uses 64-bit (8-byte) words—affects cache footprint, bandwidth, and SIMD lane occupancy.
- Throughput: Amount of useful work completed per unit time; improve by balancing compute, memory, and synchronization so no stage starves the others.
- Ceiling function: Smallest integer ≥ `x`; handy for partitioning work evenly across threads (`ceil(n / p)` gives at-most-one-element imbalance).

### Logic & Bitwise Ops
- Bitwise AND/OR/XOR: Operate on individual bits of integers; crucial for masks, SIMD lane control, and lock-free data structures.
- Logical AND/OR: Evaluate boolean expressions with short-circuit semantics; watch for unintended side effects when converting between logical and bitwise forms.

### Cache Levels (L1, L2, L3)
- L1: Smallest, fastest, per-core caches (separate I/D on many CPUs); tens of KB; few-cycle latency.
- L2: Larger per-core (hundreds of KB to a few MB); higher latency; buffers L1 misses.
- L3 (LLC): Shared among cores on a socket; multi-MB; higher latency; reduces DRAM traffic and supports coherence.
- Inclusive/exclusive policies: Some architectures duplicate lines across levels (inclusive) vs store uniquely (exclusive).
- Performance tip: Keep working sets within L1/L2 when possible; measure with cachegrind/perf.

### Virtual Memory & Translation
- Virtual memory: Provides each process with a large, contiguous address space mapped onto physical memory pages; enables isolation, overcommit, and transparent paging to disk.
- Translation Lookaside Buffer (TLB): Small cache of recent virtual→physical translations; TLB misses trigger page-table walks that stall execution, so keep working sets within TLB capacity and mind page sizes.

### Vectorization (SIMD)
- SIMD (single instruction, multiple data): Executes the same operation across a vector of elements in one instruction, amortizing decode/issue overhead.
- Vectorization: Compiler or manual rewrite that restructures loops to leverage SIMD hardware; requires predictable strides and minimal data dependencies.
- Compiler vectorization: Auto-vectorizers scan loops for uniform strides, no-alias guarantees, and reductions; help them by using `restrict`, aligning data, and hoisting complex control flow.
- Data-parallel operations apply one instruction to multiple elements (e.g., SSE/AVX/AVX-512, NEON, SVE).
- Alignment and contiguous memory layouts improve throughput; minimize gather/scatter and misaligned accesses.
- Loop structure: Favor simple, stride-1 loops with no data dependencies; enable compiler auto-vectorization.
- Vector intrinsics: Architecture-specific functions (e.g., `_mm256_add_ps`) that expose SIMD instructions directly; trade readability for precise control.
- VFPU (vector floating-point unit): Hardware block that executes SIMD floating-point math; back-end throughput hinges on VFPU width and pipeline depth.
- FPU (floating-point unit): Scalar floating-point execution unit; feeds SIMD lanes when vectors are unavailable or width is 1.
- SSE / AVX: x86 SIMD ISAs—SSE is 128-bit, AVX extends to 256-bit, and AVX-512 provides 512-bit vectors plus mask registers.
- Memory-bound limits: SIMD helps only if bandwidth/latency isn’t the bottleneck; consider blocking and prefetching.
- FMA (fused floating-point multiply-add): Performs `a * b + c` in one instruction with a single rounding; doubles floating-point throughput on capable hardware and improves numerical accuracy (also called floating-point multiply-add).

### Tiling (Blocking)
- Definition: Reorder loops to operate on small subproblems (tiles)—also called loop blocking/loop tiling—that fit into a target cache level to maximize data reuse before eviction.
- Benefits: Reduces capacity/conflict misses, improves TLB locality, and exposes reuse for SIMD and multi-threading.
- Multi-level: Nest tiles to target L1, then L2/L3; choose tile sizes so the working set per tile stays well below the cache size with headroom.
- See also: “Arithmetic Intensity, Tiling, Unrolling, Prefetching” for additional guidance and heuristics.

### Loop Transformations
- Loop fusion: Merges adjacent loops over the same iteration space to reduce pass count, improve cache reuse, and shrink synchronization overhead (beware register pressure if fused bodies get large).
- Loop interchange: Swaps loop nesting order to traverse stride-1 dimensions innermost; critical for matching row-major vs column-major layout and shrinking TLB/cache thrash.
- Cache-aware loop design: Combine tiling, fusion, interchange, and prefetch distance tuning so each loop nest feeds caches sequentially while keeping per-thread working sets disjoint.
- Loop refactoring: General restructuring (split, combine, reorder) of loop nests to expose parallelism, enhance locality, or simplify dependences prior to applying other transformations.
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

### Aliasing & `restrict`
- Aliasing: Occurs when two expressions reference the same memory location; forces compilers to assume writes may affect reads, limiting reordering and vectorization.
- Pointer aliasing: Multiple pointers into overlapping regions; analyze hot loops to ensure each thread operates on disjoint slices when possible.
- `restrict` keyword: C qualifier promising that a pointer is the sole reference to its pointed-to data; frees the compiler to reorder loads/stores and emit wider SIMD when the promise holds (never violate it—undefined behavior).
- `restrict` qualifier: Applies the same promise to pointer-qualified struct members or typedefs; combine with const/volatile as needed but ensure the non-alias contract is actually met.
- Array overlap: When different indices or slices of an array refer to intersecting memory; restructure data or copy to temporaries so hot kernels see disjoint regions and the compiler can vectorize aggressively.

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

### CPU Issue Metrics
- IPC (instructions per cycle): Measures how many instructions retire each cycle; approaching the machine width (e.g., 4-wide) indicates good ILP and low stalls.
- CPI (cycles per instruction): Reciprocal of IPC; lower CPI means higher throughput but can spike when memory or branch stalls dominate.


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

- Prefetcher: Hardware or software mechanism that predicts future accesses and pulls lines into cache early, hiding miss latency when patterns are regular.
- Hardware: Modern CPUs/GPU SMs detect streams/strides and issue prefetches; effectiveness drops for irregular access patterns.
- Software: Insert explicit prefetch hints (e.g., `_mm_prefetch`) with a distance ≈ memory_latency / cycles_per_iteration; use non-temporal hints for one-time streams.
- Pitfalls: Over-prefetching can pollute caches or waste bandwidth; tune distances and target levels (L1/L2) with profiling.
- Complementary: Combine with tiling and layout changes to make access patterns more predictable for both hardware and software prefetchers.

### Minimize Communication, Maximize Locality
- Principle: Every data transfer across threads, sockets, or nodes has overhead; restructure algorithms to reuse data where it already resides before sending it elsewhere.
- Techniques: Partition domains so each thread works on spatially contiguous blocks, replicate read-only parameters instead of sharing write-heavy state, and overlap communication with computation when movement is unavoidable.
- Diagnostics: Track bytes transferred per FLOP and MPI/OpenMP bandwidth metrics; rising ratios often signal that communication, not compute, is the bottleneck.

### Loop Parallelization Heuristics
- Prefer parallelizing the outermost loops so each thread processes large, contiguous chunks—this improves locality and reduces synchronization relative to inner-loop parallelism.
- Ensure vectorization on the innermost loop, then expose thread-level parallelism one level up; balance grain size so each chunk amortizes scheduling cost yet leaves enough work for all threads.
- Use OpenMP `collapse(n)` to expose more iterations when outer loops are too small, and pair it with `schedule(dynamic, chunk)` carefully to avoid fragmenting cache-resident tiles.

### Linear Algebra Patterns
- Back substitution: Solves upper-triangular systems in reverse order (from last equation to first); inherently sequential but can use block partitioning or task pipelining to extract limited parallelism.
- Prefix sum (scan): Computes running totals across a vector; use work-efficient upsweep/downsweep algorithms or segmented scans to expose parallelism while preserving order-dependent semantics.

### Pipeline Algorithm
- Pattern: Break a computation into ordered stages, each handled by a different thread or hardware unit; data items stream through the stages like an assembly line.
- Benefit: Overlaps work on different items, hiding per-stage latency and raising throughput once the pipeline fills.
- Caveats: Throughput is limited by the slowest stage; balance workloads or replicate bottleneck stages, and add buffering/queues to absorb burstiness without blocking upstream stages.

### Diagonal (Wavefront) Algorithm
- Description: Traverse dependence graphs along diagonals (wavefronts) so computations with satisfied dependencies execute in parallel, common in dynamic programming and stencil updates.
- Implementation: Process anti-diagonals of a matrix or time-space grid; points on the same wavefront are independent, while successive wavefronts provide natural synchronization boundaries.
- Benefit: Avoids fine-grained locks by aligning work with dependency cones and often improves cache locality compared to naive row/column ordering.

### Saturating Pipelines
- Goal: Keep hardware execution units busy by providing enough independent work to cover pipeline latency; avoid bubbles from stalls or imbalanced instruction issue.
- Tactics: Unroll loops, interleave independent operations, and schedule memory accesses early so data is ready when needed. Pair with software pipelining or out-of-order-friendly instruction mixes to feed all stages continuously.

### Correctness & Formal Methods
- Theorem proving: Uses automated or interactive proof assistants (Coq, Isabelle, Lean) to mathematically verify that concurrent algorithms satisfy specifications; valuable for lock-free data structures and protocol proofs.
- Correct code vs optimized code: Always establish functional correctness and race freedom before micro-optimizing; unsafe speedups risk heisenbugs that erase performance gains when debugging or rolling back.
