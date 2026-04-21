<!--
title: Fluid Mechanics and Environmental Hydrodynamics
tags: [fluid-dynamics, hydrodynamics, environment]
-->

## Fluid Mechanics and Environmental Hydrodynamics

- Stage
- Backwater Zone

### Continuum and Kinematics

- Material Point: An idealized infinitesimal fluid parcel used as the basic entity in continuum mechanics, carrying properties such as velocity, pressure, density, and temperature.
- Material derivative: The rate of change experienced by a moving fluid parcel, `D()/Dt = ∂()/∂t + u · ∇()`, combining local and advective effects.
- Cross-stream motion: Velocity component normal to the primary flow direction (e.g., transverse or secondary flow), important in mixing, shear-layer growth, and curved-channel dynamics.

### Rarefaction and Thermodynamic Closure

- Knudsen Number: Dimensionless ratio `Kn = λ / L` (molecular mean free path over characteristic length) used to classify continuum validity and rarefaction effects.
- Mean Free path: Average molecular travel distance between collisions; sets the microscopic scale that underlies transport coefficients and rarefaction behavior.
- Equation of state: Constitutive relation connecting thermodynamic variables (commonly `p(ρ,T)`) to close compressible-flow models.

### Pressure and Incompressibility

- Pressure acts as a Lagrange multiplier that enforces incompressibility: In incompressible Navier–Stokes, pressure is introduced to satisfy `∇ · u = 0` rather than evolved from an independent thermodynamic equation.
- Pressure is solved via a global elliptic problem: Projection and pressure-correction methods obtain pressure from a domain-wide Poisson-type equation constrained by continuity and boundary conditions.
- Pressure redistributes momentum instantaneously to maintain zero divergence: In the incompressible limit, pressure communicates constraints across the domain so corrected velocity remains solenoidal.
- Pressure is non-local: Pressure at a point depends on the velocity and boundary data throughout the domain through the elliptic solve.
- Flow can be driven through a pressure gradient: Spatial pressure differences (`-∇p`) provide a body-force-like acceleration mechanism that sustains internal and external flows.

### Rotational and Invariance Concepts

- Vorticity (curl of `u`): Vector field `ω = ∇ × u` measuring local rotation and circulation intensity of the velocity field.
- Galilean invariance: Governing equations retain form under constant-velocity frame shifts, ensuring physics is independent of inertial observer translation.
