---
title: Parallel Programming
tags: [parallel, concurrency]
---

1. Moore's Law - Transistor counts roughly double periodically, driving exponential compute growth.
2. Pipelining - Overlaps instruction stages to raise throughput without faster clocks.
3. Branch Prediction - Predicts branch direction to keep pipelines busy; mispredictions flush.
4. Data Forwarding - Bypasses results to later stages, preventing read-after-write stalls.
