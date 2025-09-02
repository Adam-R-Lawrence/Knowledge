<!--
title: Parallel Programming
tags: [parallel, concurrency]
-->

## Parallel Programming

- Moore's Law: Transistor counts roughly double periodically, driving exponential compute growth.
- Pipelining: Overlaps instruction stages to raise throughput without faster clocks.
- Branch prediction: Predicts branch direction to keep pipelines busy; mispredictions flush.
- Data forwarding: Bypasses results to later stages, preventing read-after-write stalls.
