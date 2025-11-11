<!--
title: Turbulence
tags: [fluid-dynamics, turbulence]
-->

## Turbulence

Notes on turbulent flow scales and modeling concepts.

### Kolmogorov Scale
- Smallest turbulence length scale where viscous dissipation dominates; derived from balancing inertial energy cascade with viscosity.
- Size: `η = (ν^3 / ε)^{1/4}`, with kinematic viscosity `ν` and dissipation rate `ε`; sets the grid spacing for direct numerical simulation.
- Implication: Resolving eddies smaller than `η` is unnecessary for DNS but LES/subgrid models must model their net effect on larger scales.

### Drag Crisis
- Bluff bodies such as circular cylinders or spheres experience a drag crisis when the boundary layer transitions to turbulence, delaying separation and slashing pressure drag near `Re ≈ 3×10^5`.
- Predicting the onset requires tracking surface roughness, free-stream turbulence, and Reynolds number; CFD grids must resolve the near-wall instabilities or include transition models to capture the sudden drag drop.

### Vorticity Equation
- The vorticity transport equation
  ```
  ∂ω/∂t + (u · ∇)ω = (ω · ∇)u + ν ∇²ω + (∇ρ × ∇p)/ρ²
  ```
  balances local/convective changes of vorticity with vortex stretching/tilting, viscous diffusion, and baroclinic torque.
- Solvers often advance this equation to focus on rotational dynamics, using the baroclinic term to capture vorticity generation in variable-density flows.

### Vortex Stretching
- The `(ω · ∇)u` term amplifies vorticity magnitude when velocity gradients align with existing ω, transferring energy from large to small scales—a hallmark of three-dimensional turbulence.
- Accurately capturing stretching requires sufficient resolution of velocity gradients; under-resolved simulations damp this pathway and mispredict cascade rates.

### Enstrophy Production
- Enstrophy `E = ½ ∫ |ω|² dV` measures the intensity of rotational motion; its production is governed by vortex stretching (source) and viscous diffusion (sink).
- Monitoring enstrophy budgets highlights where mesh refinement or subgrid modeling must focus to sustain realistic dissipation rates.

### Baroclinic Torque
- When pressure and density gradients are misaligned (`∇ρ × ∇p ≠ 0`), the baroclinic term injects new vorticity even in initially irrotational flows, coupling combustion, stratification, or multiphase physics to turbulence.
- Baroclinic torque dominates vorticity generation in shock–density-interface interactions and buoyancy-driven flows; numerical schemes need aligned gradients and low dissipation to capture it faithfully.

### Chaotic Statistical Predictability
- Chaotic systems (turbulence, Lorenz models) lose pointwise predictability quickly, yet their statistical quantities—means, spectra, Reynolds stresses—vary slowly, enabling useful forecasts and closure modeling even when trajectories diverge exponentially.
