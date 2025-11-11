<!--
title: Continuum Mechanics
tags: [mechanics, materials]
-->

## Continuum Mechanics

### Stress & Strain Fundamentals

- Stress: Internal force per unit area; represented by a second-order tensor capturing how traction depends on plane orientation.
- Normal stress: Component of stress perpendicular to a plane; tension positive, compression negative.
- Strain: Dimensionless measure of deformation defined as change in length divided by original length; tracks fiber stretch or shortening.
- Normal strain: Axial strain aligned with the member or fiber direction; governs elongation or contraction under axial loading.
- Young's modulus: Material stiffness constant `E` defining the slope of the linear elastic stress-strain curve in uniaxial tension or compression.

### Boundary Geometry

- Unit outward normal vector `n`: A unit-length vector perpendicular to the boundary surface pointing outward from the domain; used to define tractions `t = σ · n`, impose Neumann/flux conditions, and orient surface integrals in divergence theorem applications.

### Elastic Behavior

- Linear elasticity: Regime where stress is proportional to strain and deformations are fully recoverable after unloading.
- Hookean material: Ideal linear elastic solid obeying Hooke's law (`sigma = E * epsilon`) within its elastic limit.
- Non-Hookean material: Exhibits nonlinear stress-strain response such as plasticity, hyperelasticity, or viscoelasticity; tangent or secant moduli depend on load level and history.

### Energy Concepts

- Conservation of energy: For an isolated mechanical system, total energy remains constant; in quasi-static structural problems, external work balances the increase in internal strain energy.
- Energy principle: Minimum total potential energy states that among kinematically admissible displacement fields, the actual equilibrium configuration minimizes `Pi = U - W`, where `U` is strain energy and `W` is work of external loads.
- Elastic energy: Strain energy stored during reversible deformation; for linear elastic bodies `U = (1/2) * integral(sigma : epsilon dV)`, reducing to `sigma^2 / (2E)` for axial members.

### Loading Regimes & Dissipation

- Quasistatic loading: Evolves slowly enough that inertial forces are negligible compared to static equilibrium forces.
- Damping: Mechanisms that dissipate vibrational energy (material hysteresis, friction, fluid drag); characterized by damping ratio, loss factor, or complex stiffness.
- Inertial losses: Energy consumed in repeatedly accelerating masses or entrained media during cyclic motion; appears as apparent damping and reduces dynamic amplification.

### Viscosity Measures

- Dynamic viscosity `μ`: Proportionality constant between shear stress and velocity gradient (`τ = μ ∂u/∂y`) describing a fluid’s internal resistance to deformation; SI units Pa·s and strongly temperature dependent.
- Kinematic viscosity `ν = μ / ρ`: Dynamic viscosity normalized by density, yielding a diffusion-like property with units m²/s that governs how momentum diffuses and sets nondimensional groups such as Reynolds and Kolmogorov scales.
