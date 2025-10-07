<!--
title: Mathematics
tags: [math]
-->

## Mathematics

Brief notes on mathematical concepts, definitions, and problem-solving.

### Weak Derivative
- A way to talk about a “derivative” for rough functions by testing how they behave inside integrals with smooth probe functions; sharp corners and jumps are allowed.
- Idea: Define derivatives via integration against smooth test functions when classical derivatives may not exist.
- 1D definition: A function `w ∈ L^1_loc` is the weak derivative of `u` if for all `φ ∈ C_c^∞(a,b)`,
  `∫_a^b u(x) φ'(x) dx = - ∫_a^b w(x) φ(x) dx`.
- Multi-D: For multi-index `α`, `D^α u` is weak derivative if `∫_Ω u D^α φ = (-1)^{|α|} ∫_Ω (D^α u) φ` for all `φ ∈ C_c^∞(Ω)`.
- Context: Underpins Sobolev spaces `W^{k,p}` and weak formulations of PDEs.

### Integration by Parts
- Trade a derivative from one factor to the other inside an integral, plus a boundary term—think “product rule for integrals.”
- 1D: `∫_a^b u'(x) v(x) dx = [u v]_a^b - ∫_a^b u(x) v'(x) dx`.
- Multi-D (one form): For sufficiently smooth `u` and vector field `w`,
  `∫_Ω ∇u · w dx = - ∫_Ω u (∇·w) dx + ∫_{∂Ω} u (w·n) ds`.
- Weak forms: Boundary terms encode natural/essential boundary conditions depending on which functions vanish on `∂Ω`.

### Test Functions
- Smooth little “bump” functions that are zero near the boundary; safe probes to test other functions without causing trouble at the edges.
- Definition: Infinitely differentiable functions with compact support inside the domain, `C_c^∞(Ω)`.
- Role: Probe distributions/weak derivatives via duality; often chosen to vanish on the boundary to eliminate boundary terms.
- Spaces: The space of test functions is dense in many function spaces and defines distributions as continuous linear functionals on it.

### Jump Discontinuity
- A step change—values just before and after a point (or across a surface) differ; the jump is their difference.
- 1D: A point `x0` where left and right limits exist but differ: `u(x0^-) ≠ u(x0^+)`.
- Across an interface `Γ` in multi-D: The jump is `[u] = u^+ - u^-`, with `u^±` denoting traces from each side of `Γ` along the normal.
- Notes: Jumps appear in piecewise-defined solutions, shocks, or discontinuous Galerkin methods; derivatives may contain measures (e.g., Dirac terms) at jumps.

### Lax–Milgram Theorem
- If your energy-like bilinear form is well-behaved (bounded) and strongly positive (coercive), then your linear problem has exactly one solution and small input changes cause only small solution changes.
- Setting: Hilbert space `V`, continuous bilinear form `a(·,·): V×V→ℝ` (or `ℂ`) and continuous linear functional `f: V→ℝ`.
- Continuity (boundedness): `|a(u,v)| ≤ M ||u||_V ||v||_V` for some `M > 0` and all `u,v ∈ V`.
- Coercivity (ellipticity): `a(v,v) ≥ α ||v||_V^2` for some `α > 0` and all `v ∈ V`.
- Conclusion: There exists a unique `u ∈ V` such that `a(u,v) = f(v)` for all `v ∈ V`, with stability bound `||u||_V ≤ ||f||_{V'} / α`.
- Use: Guarantees well-posedness of weak formulations of many elliptic boundary value problems.

### Advection PDE
- Transports a quantity without smoothing; information moves along characteristics at a speed.
- 1D linear form: `u_t + a u_x = 0` with constant advection speed `a`; solution is a shift: `u(x,t) = u0(x - a t)`.
- Features: Preserves shape in the continuous model; numerical schemes must control dispersion/dissipation (CFL condition).

### Diffusion PDE
- Spreads/smooths a quantity over time; high frequencies decay fastest.
- 1D heat equation: `u_t = κ u_xx` with diffusivity `κ > 0`; fundamental solution is Gaussian with variance `2 κ t`.
- Features: Conserves total mass (with appropriate boundaries); solutions become smoother instantly for `t > 0`.

### Generalized-Alpha Method
- Family of second-order, unconditionally stable time integrators that add controllable high-frequency damping for ODEs/structural dynamics.
- Parameters via spectral radius at infinity `ρ∞ ∈ [0,1]` (0 = max damping, 1 = no extra damping):
  `α_m = (2 ρ∞ - 1)/(ρ∞ + 1)`, `α_f = ρ∞/(ρ∞ + 1)`, `γ = 1/2 + α_f - α_m`, `β = (1/4) (1 + α_f - α_m)^2`.
- Use: Choose `ρ∞ ≈ 0.5–0.8` to damp spurious high-frequency response while keeping low-frequency accuracy.

### Lp Norms (ℓ2, ℓ∞)
- For a vector `x ∈ ℝ^n` and `p ≥ 1`: `||x||_p = (∑ |x_i|^p)^{1/p}`; measures size with different sensitivity to components.
- ℓ2 (Euclidean): `||x||_2 = (∑ x_i^2)^{1/2}`; rotation-invariant; common in least-squares.
- ℓ∞ (max norm): `||x||_∞ = max_i |x_i|`; limit of `||·||_p` as `p → ∞`.
- All `||·||_p` satisfy positivity, homogeneity, and triangle inequality; different norms are equivalent up to constants in finite dimensions.

### Newton's Method
- Root finding for `f(x) = 0` using local linearization; iterate `x_{k+1} = x_k - f(x_k)/f'(x_k)` (scalar) or `x_{k+1} = x_k - J_f(x_k)^{-1} f(x_k)` (multivariate).
- Convergence: Quadratic near a simple root with a good initial guess; may diverge if started too far or with poor conditioning.
- Practicalities: Use damping/line search and solve the Newton system with direct or iterative linear solvers; reuse/precondition Jacobians when possible.

### Picard (Fixed-Point) Method
- Solve `x = g(x)` by iteration `x_{k+1} = g(x_k)`; derivative-free and simple.
- Convergence: Linear when `g` is a contraction (`||g'(x)|| < 1` in a suitable norm); slower than Newton but robust for some nonlinearities.
- Use: Common for nonlinear PDE discretizations (e.g., implicit diffusion/advection-reaction) and as a preconditioner or initializer for Newton.
