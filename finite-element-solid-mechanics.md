<!--
title: Finite Element Solid Mechanics
tags: [finite-elements, solid-mechanics, boundary-conditions, convergence]
-->

## Finite Element Solid Mechanics

### Robin Boundary Conditions in Solid Mechanics

For linear elasticity,

`-div(sigma) = b` in `Omega`.

The standard boundary conditions are:

- Dirichlet: `u = u_bar` on `Gamma_D`.
- Neumann: `sigma n = t_bar` on `Gamma_N`.
- Robin: traction is related to displacement, for example `sigma n + K_s u = t_bar` on `Gamma_R`.

A Robin condition therefore behaves like a distributed spring attached to the boundary. `K_s` is a boundary spring stiffness per unit boundary measure. If the body moves away from the preferred position, the boundary produces a restoring traction.

After integration by parts, the Robin condition contributes a boundary stiffness term to the weak form:

`int_Omega eps(v):sigma(u) dOmega + int_Gamma_R v . K_s u dGamma = int_Omega v . b dOmega + int_Gamma_N v . t_bar dGamma + int_Gamma_R v . t_bar dGamma`.

Thus a Robin condition is neither a prescribed displacement nor a prescribed traction: the traction depends on the unknown displacement. `K_s = 0` reduces to Neumann, while very large `K_s` increasingly constrains the displacement toward a prescribed value.

### Rigid-Body Modes, Zero Eigenvalues, and Pure Neumann Problems

The elastic energy is

`a(u,u) = int_Omega eps(u):C:eps(u) dOmega`.

A rigid-body motion has the form `u(x) = c + omega x x` in 3D and produces no strain: `eps(u) = 0`. It therefore stores no elastic energy. In the discrete problem this means

`K r = 0`,

so every rigid-body mode `r` is a null vector of the stiffness matrix and corresponds to a zero eigenvalue.

- In 2D there are 3 rigid modes: 2 translations and 1 rotation.
- In 3D there are 6 rigid modes: 3 translations and 3 rotations.

A purely Neumann elasticity problem does not restrain these modes, so `K` is positive semidefinite rather than positive definite and the displacement is not unique. If `u` is a solution, `u + r` is also a solution for any rigid-body mode `r`.

A static pure-Neumann problem also requires load compatibility: the applied body forces and tractions must have zero resultant force and zero resultant moment. Otherwise no static equilibrium solution exists.

Sufficient Dirichlet constraints, or Robin springs that restrain all rigid motions, remove the rigid-body nullspace and make the elastic problem unique, assuming there are no other mechanisms.

### Kronecker Delta Property and Dirichlet Boundary Conditions

For a nodal Lagrange finite-element basis,

`N_i(x_j) = delta_ij`,

where the Kronecker delta is `delta_ij = 1` when `i = j` and `0` otherwise.

This means the finite-element coefficient at node `j` is exactly the value of the interpolated field at that node. Therefore a strong Dirichlet condition is easy to impose: set the boundary nodal degree of freedom directly to the prescribed displacement.

The Kronecker-delta property is therefore mainly connected to strong Dirichlet enforcement, not weak enforcement.

With weak Dirichlet enforcement, such as Nitsche's method, the trial space is not forced to satisfy `u = g` pointwise at boundary nodes. Instead, boundary consistency, symmetry, and penalty terms are added to the variational form. This does not require an interpolatory basis with the Kronecker-delta property. This is especially useful for bases such as B-splines/NURBS or unfitted methods where coefficients are not simply point values.

Conceptually:

- Strong Dirichlet: boundary value is built directly into the discrete trial space.
- Weak Dirichlet: boundary value is enforced through additional terms in the weak form.
- Kronecker-delta interpolation makes strong enforcement particularly simple.

### Interface Alignment and FEM Convergence Rate

Suppose a material interface crosses the domain and a coefficient such as Young's modulus or conductivity jumps across it. The exact solution is often continuous, but its gradient has a jump because the physical flux or traction must remain continuous.

If the interface lies on element boundaries, each element contains a smooth branch of the solution. Standard finite elements can then recover their normal piecewise-smooth convergence rates. For polynomial degree `p`, the usual optimal rates are approximately

- energy/H1 error: `O(h^p)`,
- L2 error: `O(h^(p+1))`,

provided the solution has the required regularity on each material subdomain.

If the interface cuts through an element and the standard finite-element space is not enriched, the element is being asked to represent a derivative jump using one smooth polynomial. The approximation cannot place the kink at the true interface. The global regularity seen by the standard approximation is reduced, and convergence can therefore degrade.

For the canonical case of a solution that is continuous but has a finite jump in its first derivative, a standard unfitted mesh can show approximately

- H1 error: `O(h^(1/2))`,
- L2 error: `O(h^(3/2))`,

rather than the optimal rates, although the precise result depends on dimension, norms, coefficient treatment, and regularity. The essential point is that higher polynomial order alone cannot recover a derivative discontinuity that the approximation space is not allowed to represent.

Fitted meshes, XFEM/generalized FEM enrichment, immersed/interface methods, or suitable CutFEM formulations can restore optimal piecewise convergence by representing the interface jump correctly.

### Spring Stiffness vs Axial Rigidity

Spring stiffness is

`k = F / delta`,

with units `N/m`. It describes the force required to produce a given displacement of the whole spring or structural member.

For a uniform axial bar,

`delta = F L / (E A)`,

so

`F / delta = E A / L`.

The important distinction is:

- Axial rigidity: `EA`, units `N`. It combines material stiffness `E` and cross-sectional area `A` and is independent of member length.
- Axial stiffness of a bar: `EA/L`, units `N/m`. It is the equivalent spring stiffness of the whole bar.
- Spring stiffness: `k`, units `N/m`. For a linear axial bar, `k = EA/L`.

The 1D bar finite-element stiffness matrix is therefore

`K_e = (EA/L) [[1, -1], [-1, 1]]`,

which has exactly the same two-node form as a linear spring element with stiffness `k = EA/L`.

Intuition: `EA` says how hard the cross-section is to stretch per unit strain, while `EA/L` says how hard the entire member is to stretch by a specified displacement. Two bars with the same `EA` but different lengths have the same axial rigidity but different structural stiffness; the shorter bar is stiffer.

### Double Nodes at the Same Geometric Location

Two finite-element nodes may have identical spatial coordinates but different node IDs and therefore different degrees of freedom. Geometrically they occupy the same point, but topologically they are distinct unknowns.

This is useful when a field is allowed to be discontinuous across an interface, crack, cohesive zone, or contact surface. One element may connect to node `i^-` on one side and another element to node `i^+` on the other side, even though `x_i^- = x_i^+`. Then the approximation can have `u_i^- != u_i^+`, so a displacement jump can occur at that location.

If the two nodes are tied by a constraint such as `u_i^- = u_i^+`, they behave like a single continuous node. If they are intentionally left independent, they represent two coincident material points or two traces of the field. If duplicate coincident nodes are introduced accidentally and are not properly connected or constrained, they can create disconnected pieces, mechanisms, or extra zero-energy modes and make the stiffness matrix singular.

A coincident-node pair is therefore not automatically a bad mesh: it is a topological device whose meaning is determined by connectivity and constraints, not by coordinates alone.

### Zero in an Integral Sense in FEM

A weak formulation does not normally require the strong residual `R(u)` to vanish at every point. Instead it requires the residual to vanish when tested against admissible test functions:

`int_Omega v R(u) dOmega = 0` for all admissible `v`.

In the continuous weak problem, if `R` is an ordinary integrable function and this identity holds for every sufficiently rich test function, then `R = 0` almost everywhere. The residual may differ from zero on a set of measure zero without changing the integral statement.

In discrete Galerkin FEM the statement is weaker:

`int_Omega v_h R(u_h) dOmega = 0` for every `v_h` in the finite-dimensional test space `V_h`.

This does not imply that `R(u_h) = 0` pointwise or even almost everywhere. It means only that the residual is orthogonal to the discrete test space. The residual can be nonzero between nodes and inside elements while all Galerkin weighted residual equations are exactly satisfied.

This distinction is central to FEM: the discrete solution satisfies the PDE in a projected or weak sense, rather than by forcing the strong-form residual to be zero at every spatial point.
