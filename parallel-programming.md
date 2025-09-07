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

### Memory Hierarchy & Locality
- Cache: Small, fast memory that keeps recently used data to reduce average access latency.
- Cache line: Minimum transfer unit between memory and cache (e.g., 64 bytes).
- Multi-word cache lines: Lines contain adjacent words to exploit spatial locality.
- Spatial locality: Accesses tend to cluster near recent addresses; contiguous layouts benefit.

### Cache Organization
- Fully associative: Any block can occupy any cache line; minimizes conflicts with higher lookup cost.
- LRU (least recently used): Replacement policy that evicts the least recently used line, leveraging temporal locality.

### Cache Miss Types
- Compulsory miss: First access (cold) miss; data has not been loaded yet.
- Conflict miss: Limited associativity maps multiple blocks to the same set, causing avoidable misses.
- Capacity miss: Working set exceeds cache size, causing misses even with optimal placement.

### Metrics & Operations
- FLOPs: Floating-point operations per second; a common compute-throughput metric.
- Load operations: Instructions that read memory into registers; may hit or miss in cache and dominate latency.
