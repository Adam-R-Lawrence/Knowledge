<!--
title: Structural Engineering
tags: [structures, statics]
-->

## Structural Engineering

### Statics & Resultants

- Equilibrium: Condition where net force and net moment on a body are zero.
- Equations of equilibrium: In 2D, ΣFx = 0, ΣFy = 0, ΣM = 0.
- Force systems:
  - Colinear forces: Lines of action on the same straight line; combine algebraically along that line.
  - Non-colinear forces: Lines of action differ; require vector sums and can induce a resultant moment.
  - Couple: Two equal, opposite, parallel forces separated by distance; zero net force, pure moment M = F·d independent of point. Intuition: push up on the left end and down on the right; forces cancel in translation but both spin the bar the same way, so rotations add.
- Resultants: An equivalent single force and/or moment that replaces a system of forces.
- Moment: Tendency of a force to cause rotation about a point or axis.
- Lever arm: Perpendicular distance from a force's line of action to the pivot; moment M = F × d.

### Loads & Supports

- Loads:
  - Dead loads: Permanent, static loads from self-weight and fixed construction.
  - Live loads: Transient, movable loads from occupancy and use.
- Load path: Sequence of load transfer through elements and connections to the supports/foundations; requires continuity without unintended gaps.
- Types of support:
  - Fixed support: Restrains translation and rotation; provides force and moment reactions.
  - Pin support: Restrains translation, allows rotation; provides two force reactions, no moment.
  - Roller support: Restrains one direction only; allows rotation and in-plane movement; provides a single normal reaction.

- Abutment: End support of a bridge that carries bearing reactions and retains approach fill; transfers loads to the foundation and resists earth pressures.

### Structural Elements

- Structural elements:
  - Load-bearing element: Member that supports structural loads and participates in the load path to supports.
  - Beam: Member primarily resisting bending from transverse loads; typically horizontal.
  - Column: Compression member primarily resisting axial load; buckling governs slender members.
  - Cantilever: Member fixed at one end and free at the other.

### Material & Section Behavior

- Material and section behavior:
  - Stress: Internal force per area; normal component due to axial/bending actions.
  - Axial force: Internal force acting along the member's axis; tension positive and compression negative by sign convention.
  - Tension: Axial action causing elongation; induces tensile normal stress.
  - Compression: Axial action causing shortening; may trigger buckling in slender members.
  - Shear stress: Tangential stress from transverse shear or torsion; drives sliding along planes.
  - Torsion: Twisting due to applied torque; produces shear stress over the cross-section.
  - Young's modulus: Material stiffness parameter E; slope of elastic stress–strain.
  - Stiffness: Structural resistance to deformation (k = load/deflection); depends on E and geometry.
  - Strength: Capacity before failure (yield/ultimate limits); governs allowable stress/load.

### Structural Response

- Bending: Curvature due to internal moments; tension on one side, compression on the other.
- Buckling: Sudden lateral instability of compression members above a critical load.

### Determinacy & Stability

- Determinacy:
  - Statically determinate: Unknowns equal equilibrium equations; solvable by equilibrium only.
  - Statically indeterminate: Unknowns exceed equations; need compatibility. Degree n = excess unknowns.
- Stability: Stable means no rigid-body motion; unstable forms a mechanism. Determinacy ≠ stability.

### Trusses

- Truss: Triangulated structure of pin-connected members carrying primarily axial forces; ideal loads applied at joints with negligible member bending.
- Simple truss: Built from basic triangular units added joint-by-joint; typically statically determinate under ideal truss assumptions.
- Compound truss: Two or more simple trusses connected to act together; overall determinacy depends on the connection scheme.
- Complex truss: Configuration not classed as simple or compound; often requires advanced analysis and may be statically indeterminate.

- Span: Clear distance between primary supports; for trusses, often taken center-to-center of bearings.
- Bay: Repeated spacing between adjacent frames or trusses, or the distance between adjacent panel points along a truss.

- Top chord: Upper longitudinal members of a truss; typically in compression under gravity loads.
- Bottom chord: Lower longitudinal members of a truss; typically in tension under gravity loads.
- End post: Outermost inclined member at the end of a through truss connecting the top and bottom chords; often in compression.
- Gusset plate: Steel plate connecting multiple truss members at a joint; transfers forces via bolts/welds in shear, bearing, and block-shear.

- Roof truss: Truss used to support a roof; purlins bear on the top chord; efficient for moderate-to-long roof spans.
- Purlin: Secondary roof member spanning between trusses or rafters; supports roof sheeting and delivers loads to the trusses.

- Bridge truss: Truss used as a bridge superstructure (deck or through types; e.g., Pratt, Howe, Warren) with a floor system (stringers/floor beams) and portal/lateral bracing.
- Stringer: Longitudinal beam carrying deck loads to floor beams or trusses (common in bridge floors; also used in stairs).
- Portal bracing: Lateral bracing between end posts/columns at the portal (end) of a through truss/frame to resist wind and control sway.
- Knee brace: Diagonal brace between a column and a beam/rafter to stiffen the frame corner and reduce sway.

- Sway: Lateral displacement of a frame or truss due to horizontal loads or asymmetry; limited with bracing or moment frames.
- Internal stability: Adequacy of member connectivity and triangulation to prevent mechanisms within the structure.
- External stability: Adequacy of supports and overall restraint to prevent rigid-body motion of the whole structure.
