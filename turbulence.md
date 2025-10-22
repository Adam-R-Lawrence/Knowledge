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
