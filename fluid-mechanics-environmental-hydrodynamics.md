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

### Streamlines, Streamtubes, and Streamtube Pinching

- Streamline: A curve everywhere tangent to the instantaneous velocity field. At a fixed instant, fluid velocity has no component normal to a streamline.
- Streamtube: A tube whose side surface is formed entirely from streamlines. Because velocity is tangent to its sides, there is no flow through the lateral surface; fluid enters and leaves only through the tube's end cross-sections.
- For steady incompressible flow, mass conservation through a streamtube gives `rho A V = constant`, and with constant `rho`, `A V = constant` for cross-section-averaged speed `V`.
- Streamtube pinching means neighboring streamlines move closer together so the streamtube cross-sectional area decreases. Continuity then requires the average speed to increase: smaller `A` implies larger `V` for the same volume flow rate.
- Conversely, streamtube expansion generally corresponds to decreasing average speed.
- Pinching should not automatically be interpreted as a pressure decrease. For inviscid steady flow along a streamline with negligible elevation change, Bernoulli gives higher speed with lower static pressure, but viscosity, unsteadiness, body forces, and losses can modify this relationship.
- Intuition: streamlines are like the walls of an imaginary flexible hose. If the hose narrows while the same amount of incompressible fluid must pass through each second, the fluid has to move faster.

### Coanda Effect

- Coanda effect: The tendency of a fluid jet emerging near a surface to bend toward and remain attached to that surface rather than continuing along its original straight trajectory.
- A free jet entrains surrounding fluid into itself. When a wall is close to one side of the jet, fluid cannot be replenished from that side as easily, producing a lower-pressure region between the jet and the wall. Higher ambient pressure on the opposite side pushes the jet toward the surface.
- Once the jet bends, a cross-stream pressure gradient supplies the centripetal acceleration required for the curved fluid trajectories. The wall-side pressure is generally lower than the pressure on the outside of the curved jet.
- Viscosity is important because it creates shear layers, entrainment, and near-wall momentum transfer, but the effect should not be described simply as the fluid being 'stuck' to the wall by viscosity.
- Attachment persists only while the near-wall jet has enough momentum to overcome an adverse pressure gradient. If near-wall momentum becomes too small, the flow separates from the surface.
- Applications include blown flaps, fluidic devices, ventilation jets, circulation-control airfoils, and the familiar tendency of a thin water stream to follow a curved spoon or similar surface.

### Vorticity and the Biot-Savart Law

- Vorticity is `omega = curl(u)`. The Biot-Savart law reconstructs the velocity induced at one location by a distribution of vorticity elsewhere, under the usual incompressible unbounded-domain setting.
- For a three-dimensional vorticity field, the induced velocity is `u(x) = (1/(4 pi)) int [omega(x') x (x - x')] / |x - x'|^3 dV'`, up to any additional irrotational or boundary-induced velocity required by the problem.
- Each small piece of vorticity induces a velocity perpendicular both to the local vorticity direction and to the vector connecting the source point to the observation point. The cross product determines this direction, while the inverse-distance dependence makes nearby vorticity contribute more strongly.
- For a thin vortex filament of circulation `Gamma`, the law becomes `u(x) = (Gamma/(4 pi)) int [dl' x (x - x')] / |x - x'|^3`.
- For an infinitely long straight vortex filament, this reduces to the familiar azimuthal speed `u_theta = Gamma/(2 pi r)`: the velocity circles around the vortex and decays like `1/r`.
- Intuition: vorticity is a source of rotational velocity. Biot-Savart tells how every piece of that vorticity contributes to the velocity at the point you care about, and the total velocity is obtained by summing those contributions.
- The analogy is direct with magnetostatics: electric current induces magnetic field through the electromagnetic Biot-Savart law, while vortex strength/vorticity induces velocity in vortex dynamics.

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
