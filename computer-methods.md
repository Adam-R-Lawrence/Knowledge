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

### Gauss–Seidel Iteration
- Scheme: Splits `A = L + D + U` (lower, diagonal, upper); updates solution components sequentially using freshest values: `x_i^{k+1} = (b_i - Σ_{j<i} a_{ij} x_j^{k+1} - Σ_{j>i} a_{ij} x_j^k) / a_{ii}`.
- Convergence: Guaranteed for strictly diagonally dominant or symmetric positive definite matrices; otherwise depends on spectral radius of the iteration matrix.
- Usage: Simple to implement, good smoother for multigrid; sequential data dependence limits parallelism but red-black ordering or block variants unlock concurrency.

### Finite Element Meshes
- Non-conforming mesh: Adjacent elements do not share a node-to-node correspondence along their common interface (e.g., mismatched element sizes or polynomial orders); continuity is enforced with constraint equations, multipoint constraints, or mortar methods.
- Hanging node: Node that lies on the edge or face of a neighboring element without being a vertex of that element; typical in adaptive mesh refinement and requires constraint handling to maintain field continuity.

### Consistency (Numerical PDE Theory)
- Definition: A discrete operator is consistent if its truncation error vanishes as the mesh is refined; discrete equations converge to the continuous PDE.
- Role: Together with stability, consistency implies convergence via the Lax equivalence theorem for linear problems.
- Practice: Derive leading truncation terms to guide mesh refinement or stabilization; ensure boundary conditions are discretized with matching order to avoid dominating the global error.

### Stabilized FEM (SUPG, mPSPG, RBVMS)
- SUPG (Streamline Upwind/Petrov–Galerkin): Adds directional diffusion along the flow to control oscillations in advection-dominated problems while preserving sharp fronts.
- mPSPG (modified Pressure-Stabilizing/Petrov–Galerkin): Provides pressure stabilization for equal-order velocity/pressure spaces in incompressible flow by penalizing the residual of continuity; the “modified” form adjusts weighting for improved near-wall accuracy.
- RBVMS (Residual-Based Variational Multiscale): Splits the solution into coarse and fine scales; models fine-scale effects via residual-driven terms, unifying SUPG/PSPG ideas and improving consistency for transient problems.
- Implementation: Stabilization parameters depend on local element size, flow speed, viscosity, and time step; calibrate with dimensionless groups like Peclet or Courant numbers.

### LES (Large-Eddy Simulation) Models
- Concept: Resolve large turbulent eddies directly while modeling subgrid-scale (SGS) motions that lie below the mesh filter width.
- SGS models: Smagorinsky, dynamic Smagorinsky, WALE, and Vreman provide eddy viscosity closures to represent energy transfer from resolved to unresolved scales.
- Requirements: Filter width is typically tied to the cube root of cell volume; need fine near-wall resolution or wall models to capture boundary layers without DNS-level cost.

### Flow Diagnostics Around Bluff Bodies
- Bluff body: Shape with separated flow and dominant pressure drag (e.g., cylinder, square prism); wakes shed vortices and create unsteady forces.
- Boundary layer separation: Point where adverse pressure gradient causes the boundary layer to detach from the surface, forming recirculation zones and enlarging drag.
- Von Kármán vortex street: Alternating vortex shedding pattern in the wake of a bluff body at moderate Reynolds numbers, leading to lift oscillations and tonal noise.
- Wake: Region of flow downstream of a body where velocity deficit and turbulence persist; length and structure depend on Reynolds number and body geometry.
- Strouhal number: Non-dimensional shedding frequency `St = f D / U`; relates vortex shedding frequency `f` to body size `D` and free-stream velocity `U`, guiding sensor placement and vibration mitigation.

### q-Criterion
- Definition: Scalar field `q = 0.5 (||Ω||^2 - ||S||^2)` comparing rotation rate tensor `Ω` and strain-rate tensor `S`; positive `q` identifies regions where vorticity dominates strain.
- Usage: Visualize coherent vortical structures in CFD by plotting iso-surfaces of `q > 0`; filter weak noise by choosing thresholds relative to mean shear.
- Notes: Complement with λ2 or swirling strength criteria to avoid misinterpreting shear layers as vortices.
