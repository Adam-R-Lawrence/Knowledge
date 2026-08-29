<!--
title: Mathematical Analysis
tags: [measure-theory, lebesgue-integration, pde, linear-operators]
-->

## Mathematical Analysis

### Sets of Measure Zero

A set has Lebesgue measure zero if it occupies no length, area, or volume in the measure-theoretic sense. Formally, a set `E` has measure zero if for every `epsilon > 0`, it can be covered by countably many intervals, rectangles, or boxes whose total measure is less than `epsilon`.

Examples in Euclidean space include:

- a single point in 1D, 2D, or 3D,
- any finite or countable collection of points,
- a curve in a 2D domain,
- a surface in a 3D volume.

A measure-zero set need not be empty. It may even contain infinitely many points. The statement is only that it contributes zero measure to a volume integral.

### Lebesgue Integral and Almost-Everywhere Equality

The Lebesgue integral is insensitive to changes on sets of measure zero. If two measurable functions satisfy `f = g` almost everywhere, written `f = g a.e.`, then their Lebesgue integrals are the same whenever the integrals exist:

`int_Omega f dx = int_Omega g dx`.

For example, changing the value of a function at one point does not change its integral. More generally, changing it on any measure-zero set does not change its Lebesgue integral.

This is why function spaces such as `L2(Omega)` identify functions that differ only on sets of measure zero. Mathematically, an `L2` object is an equivalence class of functions rather than a unique pointwise representative.

An important consequence is that the statement `f = 0 almost everywhere` is usually the natural notion of equality for weak PDE theory and finite-element analysis, not `f(x) = 0` at literally every point.

### Homogeneous Solutions of Linear Problems

For a linear operator `L`, consider

`L u = f`.

The associated homogeneous problem is

`L u_h = 0`.

Any solution of this zero-forcing equation is called a homogeneous solution. If `u_p` is one particular solution of `L u = f`, then every solution of the original linear equation has the form

`u = u_p + u_h`,

where `u_h` belongs to the null space of `L`.

This follows from linearity:

`L(u_p + u_h) = L u_p + L u_h = f + 0 = f`.

Thus homogeneous solutions describe the freedom left after one particular forced response has been found. Boundary conditions usually determine which homogeneous component is allowed.

For an ODE such as

`u'' - u = 3`,

a particular solution is `u_p = -3`, while the homogeneous equation

`u_h'' - u_h = 0`

has solutions `u_h = C1 exp(x) + C2 exp(-x)`. Therefore

`u = -3 + C1 exp(x) + C2 exp(-x)`.

In PDE and FEM language, homogeneous solutions are closely connected to the kernel or nullspace of the differential operator. For example, rigid-body motions are homogeneous zero-energy solutions of unconstrained linear elasticity. A pure-Neumann elasticity problem retains these homogeneous modes, which is why its displacement is not unique until the rigid-body freedom is removed.
