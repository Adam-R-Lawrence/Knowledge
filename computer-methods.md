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

### Conjugate Gradient Method
- Scope: Iterative solver for symmetric positive definite (SPD) systems that minimizes the `A`-norm of the error over growing Krylov subspaces.
- Mechanism: Builds mutually `A`-orthogonal search directions using residuals and scalar recurrence coefficients, requiring only matrix-vector products and vector dot products per iteration.
- Practical tips: Preconditioning preserves SPD when the preconditioner is SPD, and finite precision can break orthogonality—restart or reorthogonalize if convergence stalls.

### Gauss–Seidel Iteration
- Scheme: Splits `A = L + D + U` (lower, diagonal, upper); updates solution components sequentially using freshest values: `x_i^{k+1} = (b_i - Σ_{j<i} a_{ij} x_j^{k+1} - Σ_{j>i} a_{ij} x_j^k) / a_{ii}`.
- Convergence: Guaranteed for strictly diagonally dominant or symmetric positive definite matrices; otherwise depends on spectral radius of the iteration matrix.
- Usage: Simple to implement, good smoother for multigrid; sequential data dependence limits parallelism but red-black ordering or block variants unlock concurrency.

### Structured Matrix Classes
- Stieltjes matrix: Symmetric positive definite matrix with nonpositive off-diagonals; equivalent to an M-matrix that is also SPD, guaranteeing inverse positivity and energy-based convergence of iterative solvers modeling diffusion-like physics.
- Monotone matrix: Matrix whose inverse is entrywise nonnegative (`A^{-1} ≥ 0`), ensuring that forcing terms of one sign produce solutions of the same sign; Stieltjes/M-matrices are monotone, making them indispensable for preserving maximum principles in discrete diffusion systems.

### Anderson Acceleration
- Multisecant extrapolation that mixes the current residual with a window of prior updates to accelerate fixed-point iterations (`x_{k+1} = G(x_k)`).
- Builds a least-squares combination of past residuals to approximate the inverse Jacobian, often turning slow Picard iterations into superlinearly convergent schemes without forming Jacobians explicitly.
- Practical tips: Limit the history size (`m = 3–10`) to control memory/conditioning, restart when the least-squares problem becomes ill-conditioned, and apply damping when residual norms grow.

### Sparse Matrix Storage Formats
- ELLPACK (ELL): Stores each row with a fixed number of columns equal to the maximum nonzeros per row, padding shorter rows with zeros and column indices; ideal for GPUs/SIMD because memory accesses become strided and predictable, but wastes space when sparsity patterns vary widely.

### Inexact Newton Method
- Idea: Relax the accuracy of the linearized Jacobian solve at each Newton step, solving `J δ = -F` only to a tolerance that tightens as the nonlinear residual shrinks.
- Benefit: Saves work early in the iteration when the search direction is rough, while still recovering quadratic convergence near the solution as tolerances tighten.
- Implementation: Choose forcing terms (e.g., Eisenstat–Walker criteria) to adapt the inner-solve tolerance automatically rather than hard-coding iteration counts.

### Line Search Newton Method
- Goal: Globalize Newton's method by backing off the full step length when the residual or merit function fails to decrease.
- Mechanics: Compute the Newton direction, then use backtracking or Wolfe-condition line search to find `α` such that `x_{k+1} = x_k + α δ` satisfies descent criteria.
- Use cases: Stabilizes nonlinear solves with poor initial guesses or highly nonlinear regions; pair with trust-region strategies when line search fails repeatedly.

### Consistent Tangent Operators
- In nonlinear finite element formulations, the consistent tangent is the Gateaux derivative of the discretized residual (or mixed residual) with respect to nodal DOFs—i.e., the exact Jacobian that the Newton linear solve assumes.
- Approximating the tangent (secant or numerical differentiation) reduces setup cost but degrades convergence to linear; use consistent tangents when plasticity, contact, or multiphysics coupling makes iterations sensitive to linearization errors.
- Implementation: Differentiate constitutive updates analytically (e.g., return-mapping algorithms) and include geometric stiffness terms so the tangent matches the residual linearization used in the global solve.

### Implicit vs Explicit Linear Solvers
- Explicit solver: Updates unknowns via direct formulas (e.g., forward Euler, Jacobi) that use only previously computed values—cheap per step but bound by stability limits on time step or relaxation factors.
- Implicit solver: Solves a coupled system (often via factorization or iterative methods) that includes the unknown at the new state—more robust for stiff problems but demands linear solves each step.
- Trade-off: Explicit schemes favor low-cost, easily parallelizable iterations, whereas implicit schemes permit larger steps and unconditional stability at the expense of solving larger algebraic systems.

### Finite Element Meshes
- Non-conforming mesh: Adjacent elements do not share a node-to-node correspondence along their common interface (e.g., mismatched element sizes or polynomial orders); continuity is enforced with constraint equations, multipoint constraints, or mortar methods.
- Hanging node: Node that lies on the edge or face of a neighboring element without being a vertex of that element; typical in adaptive mesh refinement and requires constraint handling to maintain field continuity.
- Element triangle radius: Quality metric comparing an element’s actual inradius/circumradius to the ideal equilateral triangle; large circumradius-to-inradius ratios flag skinny triangles that amplify interpolation error and condition numbers.

### Mass Lumping
- Replace the consistent mass matrix with a diagonal (lumped) approximation so explicit dynamics, diffusion solves, or preconditioners avoid solving coupled systems each step.
- Achieved by row-summing the consistent matrix or integrating with special quadrature rules (Gauss–Lobatto) that collapse off-diagonal entries; preserves total mass but sacrifices higher-order accuracy.
- Use lumping when stability/efficiency of explicit time-stepping dominates; revert to consistent mass matrices for wave propagation or when modal accuracy is critical.

### Signed Distance Functions
- Signed distance function (SDF) stores the shortest distance to an interface with a sign convention (negative inside, positive outside), giving a smooth implicit representation of geometry.
- SDFs enable level-set methods, ghost-fluid boundaries, and cut-cell FEM by letting integrals over irregular domains be evaluated with standard meshes plus localized reinitialization to keep `|∇φ| = 1`.

### Collocated Grids
- Collocated grid (occasionally typoed “collocated gtrid”) stores all primary variables (pressure, velocity components, scalars) at the same control-volume center instead of staggering them; simplifies data structures and mesh adaptation.
- Requires special interpolation (e.g., Rhie–Chow momentum interpolation) or pressure stabilization to suppress checkerboard modes because gradients computed from collocated samples can decouple adjacent cells.

### Mesh Adaptivity Strategies
- h-method refinement: Halving characteristic element size (`h → h/2`) multiplies element count by ≈4 in 2D surfaces or ≈8 in 3D volumes, boosting resolution at the cost of denser matrices.
- Node balancing: Redistribute elements or adjust partition weights so each processor receives comparable node counts, preventing load imbalance after local refinement or derefinement.
- Edge swapping: Replace diagonal connections in quadrilaterals or tetrahedra to improve element quality metrics (aspect ratio, minimum angle) without changing mesh density.
- Coarsening approaches:
  - Derefinement as inverse refinement: Collapse previously refined elements back to their parents using stored refinement trees to maintain conformity.
  - Arbitrary derefinement: Remove elements based on error indicators even without a refinement history, reconstructing surrounding connectivity on the fly.
- R-method: Relocate mesh nodes toward regions of large solution gradients while keeping element connectivity fixed, clustering resolution where physics demand it.
- Vorticity refinement: Drive adaptive refinement using `|∇ × u|` magnitude so the mesh tracks shear layers, vortex sheets, and coherent structures—essential for LES/DNS where vortical content carries primary dynamics.

### Consistency (Numerical PDE Theory)
- Definition: A discrete operator is consistent if its truncation error vanishes as the mesh is refined; discrete equations converge to the continuous PDE.
- Role: Together with stability, consistency implies convergence via the Lax equivalence theorem for linear problems.
- Practice: Derive leading truncation terms to guide mesh refinement or stabilization; ensure boundary conditions are discretized with matching order to avoid dominating the global error.

### Variational and Weak Formulations
- Well-posed PDE: Requires existence of a solution, uniqueness, and continuous dependence on data; check via Lax–Milgram or Babuska–Nečas theorems before discretization.
- Variational principle: Rewrite the governing PDE as a stationary point of an energy or action functional, enabling integral formulations and highlighting conservation laws.
- Weighted residual method: Multiply the strong-form residual by admissible test functions and integrate over the domain so that projection of the residual onto the test space vanishes.
- Galerkin form: Choose test functions from the same space as the trial functions, yielding symmetric systems for self-adjoint problems and ensuring optimality in the energy norm.
- Continuous weak form: Integrate by parts to lower derivative order, impose boundary conditions weakly, and pose the problem on infinite-dimensional trial/test spaces.
- Discretized weak form: Restrict trial and test spaces to finite-dimensional bases (e.g., finite elements) to obtain algebraic systems suitable for numerical solution.
- Tikhonov regularization: Add a weighted penalty (e.g., `α ||L u||^2`) to the variational functional to stabilize ill-posed or noisy problems and control solution smoothness.

### Petrov–Galerkin Test Spaces
- Keep the trial space unchanged but choose modified test functions to encode desired directional sensitivity (e.g., streamline upwinding in advection-dominated problems).
- Petrov–Galerkin formulations break the symmetry of standard Galerkin to embed stabilization directly in the weighting functions while preserving consistency of the underlying PDE.

### Divergence Theorem in Weak Forms
- Applying the divergence theorem converts volume integrals of `∇ · (σ)` into surface integrals of `σ · n`, allowing strong-form equilibrium equations to transform into weak statements that balance internal virtual work with boundary tractions.
- The unit outward normal anchors the surface term orientation, ensuring Neumann boundary conditions (tractions, fluxes) enter the weak form naturally while Dirichlet conditions remain enforced through the trial space or penalty terms.

### Integration by Parts to Lower Derivatives
- “Integration for converting strong form to wak form” refers to repeated integration by parts: move derivatives off the trial function and onto test functions so only first derivatives appear, making `C^0` finite elements admissible.
- Each integration-by-parts step introduces boundary terms; keep or drop them based on whether corresponding boundary conditions are enforced weakly (Neumann/Robin) or strongly (Dirichlet).

### Stabilized FEM (SUPG, mPSPG, RBVMS)
- SUPG (Streamline Upwind/Petrov–Galerkin): Adds directional diffusion along the flow to control oscillations in advection-dominated problems while preserving sharp fronts.
- mPSPG (modified Pressure-Stabilizing/Petrov–Galerkin): Provides pressure stabilization for equal-order velocity/pressure spaces in incompressible flow by penalizing the residual of continuity; the “modified” form adjusts weighting for improved near-wall accuracy.
- RBVMS (Residual-Based Variational Multiscale): Splits the solution into coarse and fine scales; models fine-scale effects via residual-driven terms, unifying SUPG/PSPG ideas and improving consistency for transient problems.
- Implementation: Stabilization parameters depend on local element size, flow speed, viscosity, and time step; calibrate with dimensionless groups like Peclet or Courant numbers.

### Discontinuity Capturing
- Discontinuity capturing (sometimes typoed “discontinuy capturing”) augments Galerkin or SUPG formulations with nonlinear sensors that add isotropic/artificial diffusion only near steep gradients or shocks, preventing Gibbs oscillations without smearing smooth regions.
- Typical sensors monitor residuals, density gradients, or characteristic speeds to localize the added dissipation; coupling with RBVMS keeps the base scheme high-order elsewhere.

### TVD Schemes
- Total Variation Diminishing (TVD) discretizations enforce `TV(u^{n+1}) ≤ TV(u^n)` so spurious oscillations or new extrema cannot arise after an update.
- Flux limiters (e.g., minmod, Superbee) blend high- and low-order fluxes based on local smoothness indicators to maintain TVD while retaining second-order accuracy in smooth regions.

### Entropy Capturing & Stability
- Entropy capturing adds dissipation tailored to enforce a discrete entropy inequality, ensuring weak solutions of hyperbolic systems (Euler, shallow water) satisfy the second law and remain physically admissible.
- Entropy stability extends the idea by constructing fluxes/test functions whose Jacobians are symmetrizable so the discrete scheme admits an entropy function decreasing in time, providing nonlinear stability beyond energy norms.

### TVD vs DMP
- TVD bounds total variation globally, making it well-suited for 1D conservation laws but less strict than the Discrete Maximum Principle (DMP), which enforces local upper/lower bounds at every degree of freedom.
- DMP guarantees no new overshoots beyond boundary/initial extremes, whereas TVD allows plateaus or bounded oscillations so long as overall variation does not increase; high-resolution schemes often combine both for robustness.

### Bubble-Supported Fine Scales
- Introduce bubble functions (shape functions that vanish on element boundaries) to represent unresolved fine scales inside each element; their support makes it easy to statically condense the fine-scale DOFs and derive stabilization terms analytically.
- Bubble-supported models capture subgrid dissipation without altering inter-element continuity, providing a bridge between classical bubble enrichment and modern residual-based multiscale approaches.

### Adjoint Scale Modeling
- Residual-based VMS often defines fine scales via an adjoint operator, yielding `u' ≈ τ R(u)` where `τ` is an approximate inverse of the linearized operator; “adjoint scale” highlights that the weighting comes from solving (approximately) the adjoint problem.
- Computing `τ` from element-wise Green’s functions or spectral estimates ensures that stabilization respects both primal and adjoint accuracy, which is essential for goal-oriented error control and data assimilation.

### Discrete Maximum Principle (DMP)
- A discretization satisfies the DMP if, in diffusion-dominated problems, the discrete solution’s extrema occur on the boundary just like the continuous problem—preventing spurious high-order overshoots.
- High-order elements or under-diffused schemes often violate the DMP; limiters, positivity-preserving fluxes, or mesh/step restrictions enforce it so temperature, concentration, or volume fraction fields stay within physical bounds.

### Algebraic Flux Correction (AFC)
- AFC modifies the stiffness/mass matrices after assembly to remove negative off-diagonal entries that would break monotonicity, blending a low-order monotone operator with a high-order one via limiters.
- Implemented at the algebraic level (independent of element type), making it attractive for unstructured meshes; limiters compare antidiffusive fluxes to local bounds so the final solution honors the DMP while retaining as much high-order accuracy as possible.

### Discontinuous Galerkin Stabilization Toolkit
- Discontinuous Galerkin (DG) methods (occasionally spelled `discontinous galerkin`) rely on element-local polynomial spaces coupled through numerical fluxes, enabling hp-adaptivity, natural upwinding, and embarrassingly parallel assembly.
- Slope limiting: Apply TVB/minmod or hierarchical limiters so DG solutions remain bounded near shocks without sacrificing high-order accuracy in smooth regions; limiter choice drives monotonicity and convergence.
- Entropy stabilization: Add entropy-conserving or entropy-viscous fluxes to enforce a discrete entropy inequality, preventing nonphysical states for compressible Euler/Navier–Stokes solves.
- Dissipation/dispersion control (`disspiation`, dispersion): Evaluate the scheme’s modified wavenumber or eigenvalues to balance numerical dissipation with dispersion, especially when propagating acoustic waves or vortical structures.
- Filtering and high-frequency filtering: Modal/spectral filters damp only the highest polynomial modes each step, suppressing aliasing noise while preserving resolved content; essential when p-refining without remeshing.
- Explicit time stepping: SSP Runge–Kutta, Adams–Bashforth, or LSRK schemes leverage DG locality but face strict CFL limits scaling with `(2p+1)^2/h`, so subcycling or local time stepping can recover efficiency.
- Analytical viscosity: Introduce analytical/artificial viscosity targeted by shock sensors to regularize steep gradients while leaving smooth regions untouched, complementing slope limiting.
- Regularization: Add small penalty or Laplacian terms to the weak form so ill-posed inverse problems, phase interfaces, or noisy data remain solvable without spurious oscillations.
- Frequency domain & Fourier transform diagnostics (`fourier transofmr`): Map DG operators into the frequency domain to quantify dispersion/dissipation, tune filters, and design low-alias quadrature strategies for wave problems.
- Phase field model: DG discretizations of Cahn–Hilliard/Allen–Cahn phase-field models benefit from energy-stable time splitting, entropy stabilization, and localized filtering to keep interfaces sharp.
- Taylor–Couette flow (`taylor coutte flow`): Rotating-cylinder benchmark used to validate DG slope limiters, entropy stabilization, and filtering strategies across laminar, transitional, and turbulent regimes.

### Spectral Collocation & Dealiasing
- Gauss–Lobatto–Legendre (GLL) points include endpoints and support high-order Lagrange bases with diagonal mass matrices under GLL quadrature, making them the default nodes for spectral-element methods.
- Chebyshev polynomials/nodes cluster points near boundaries (`x_k = cos(kπ/N)`), yielding fast cosine transforms and well-conditioned interpolants for problems with boundary layers.
- Dealiasing suppresses nonlinear aliasing errors (e.g., `u^2` projecting onto too-low modes) by oversampling—common choices include the 3/2-rule zero-padding in Fourier space or evaluating nonlinear terms on denser GLL/Chebyshev grids before filtering.

### Mixed Methods for Poisson Problems
- Mixed formulations introduce both scalar potential `u` and flux `q = -k ∇u`, letting Raviart–Thomas or Brezzi–Douglas–Marini elements enforce `∇ · q = f` exactly at the discrete level.
- These methods satisfy local conservation and capture discontinuous coefficients, making them attractive for heterogeneous diffusion or Darcy models where flux continuity matters more than potential smoothness.
- Stability hinges on the discrete inf-sup condition between flux and potential spaces; pairing incompatible spaces causes spurious modes, so follow canonical `(RT_k, P_k)` or `(BDM_{k}, P_{k-1})` families when in doubt.

### Perturbing the Solution Space
- “Pertubate the solution space” (i.e., perturb the current iterate or search basis) by injecting small random, modal, or physics-informed directions when Newton/Krylov methods stagnate on a saddle point or symmetric null space.
- Controlled perturbations help escape mesh-aligned artifacts, nudge bifurcating solutions onto a desired branch, or seed multi-start globalization strategies while still honoring trust-region/line-search safeguards.

### Single-Precision Error Accumulation
- Single precision (≈7 decimal digits) can cause rounding errors to accumulate when iterative updates differ by orders of magnitude, especially in Krylov solvers or long explicit time integrations.
- Mitigations include mixed-precision refinement (factorization in FP32, residual correction in FP64), periodic re-orthogonalization, and scaling variables so working ranges stay well inside the normal exponent window.

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
