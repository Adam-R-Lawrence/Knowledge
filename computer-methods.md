<!--
title: Computer Methods
tags: [algorithms, numerical-methods]
-->

## Computer Methods

Brief notes on computational techniques, algorithms, and numerical methods.

### GMRES (Generalized Minimal Residual)
- Purpose: Iterative Krylov-subspace solver for nonsymmetric, sparse linear systems `Ax = b`.
- Idea: Builds an orthonormal basis with Arnoldi, minimizes the residual over the growing subspace at each iteration.
- Restart: `GMRES(m)` restarts after `m` steps to cap memory/orthogonalization cost; too-small `m` can slow or stall convergence.
- Preconditioning: Essential for performance; apply left/right preconditioners to reduce the condition number and cluster eigenvalues.

### Preconditioners
- Identity preconditioner: `M = I` (no preconditioning); baseline for comparisons.
- Jacobi (diagonal) preconditioner: `M = diag(A)`; inexpensive and parallel-friendly; effective when row scaling dominates.
- Incomplete LU (ILU): Factorization `A ≈ L\,U` with restricted fill; stronger than Jacobi but more setup cost; common variants ILU(0), ILU(k), ILUT.
