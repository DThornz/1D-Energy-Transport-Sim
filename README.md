# 1D Energy Transport Simulator

An interactive browser-based solver for the one-dimensional heat (thermal diffusion) equation. Supports Cartesian, cylindrical, and spherical coordinate systems with Dirichlet and Neumann boundary conditions. Part of the [A. Mirza interactive simulator suite](https://dthornz.github.io/website-cv-tools/).

**[Live Demo →](https://dthornz.github.io/1D-Energy-Transport-Sim/)**

---

## Features

- **Three coordinate systems** — Cartesian (plane wall/slab), cylindrical radial (rod/pipe), cylindrical axial (same as Cartesian), and spherical radial (sphere/droplet)
- **Full boundary condition support** — independent Dirichlet (fixed temperature) or Neumann (fixed heat flux) conditions at each boundary; inner symmetry boundary automatically enforced for radial geometries
- **Real-time simulation** — explicit forward-Euler / central-difference finite difference scheme with per-frame canvas rendering
- **Von Neumann stability enforcement** — maximum stable Δt computed automatically for each geometry; time step capped with a visual warning when the user-requested value would cause instability
- **Analytical steady-state overlay** — exact steady-state profile solved via the Thomas algorithm (O(N) tridiagonal solver) and rendered as a dashed overlay
- **Configurable material properties** — specify thermal diffusivity α directly, or derive it from conductivity k, density ρ, and specific heat cₚ
- **Accessibility panel** — dark/light mode, three font sizes, and three font families, all persisted via `localStorage`
- **Zero dependencies** — single HTML + CSS, runs entirely in the browser

---

## Physics

### Governing Equation

The simulator solves the unsteady heat conduction equation under the assumption of constant, isotropic thermal properties and no internal heat generation:

```
∂T/∂t = α ∇²T      where α = k / (ρ cₚ)
```

In one spatial dimension the Laplacian takes the following coordinate-specific forms:

| Geometry | Equation |
|---|---|
| Cartesian (x) | ∂T/∂t = α ∂²T/∂x² |
| Cylindrical radial (r) | ∂T/∂t = (α/r) ∂/∂r(r ∂T/∂r) |
| Cylindrical axial (z) | ∂T/∂t = α ∂²T/∂z² |
| Spherical radial (r) | ∂T/∂t = (α/r²) ∂/∂r(r² ∂T/∂r) |

### Boundary Conditions

- **Dirichlet** — prescribed temperature: T|_Γ = T_w
- **Neumann** — prescribed heat flux: −k (∂T/∂n)|_Γ = q (W m⁻²; q > 0 out of domain at right, into domain at left)
- **Symmetry** — Neumann q = 0 is automatically applied at r = 0 for all radial geometries (cylindrical and spherical) to satisfy the physical symmetry condition ∂T/∂r|_{r=0} = 0

### Numerical Method

**Time integration** — explicit forward Euler:

```
T_i^{n+1} = T_i^n + r (T_{i+1}^n - 2T_i^n + T_{i-1}^n)   [Cartesian]
r = α Δt / Δx²
```

Cylindrical and spherical interior nodes include an additional first-derivative term discretized with central differences. The singular 1/r terms at r = 0 are handled via L'Hôpital's rule, yielding coefficients of 4r (cylindrical) and 6r (spherical) for the origin update.

**Neumann BCs** — implemented via second-order ghost-point extrapolation, giving O(Δx²) accuracy at boundaries.

**Stability limits** (Von Neumann analysis):

| Geometry | Condition | Limit |
|---|---|---|
| Cartesian / Cyl. axial | r ≤ 0.5 | Δt ≤ Δx² / (2α) |
| Cylindrical radial | r ≤ 0.25 | Δt ≤ Δx² / (4α) |
| Spherical | r ≤ 1/6 | Δt ≤ Δx² / (6α) |

The tighter limits for radial geometries arise from the larger stencil coefficients at r = 0.

**Steady-state solver** — setting ∂T/∂t = 0 yields a tridiagonal linear system solved in O(N) operations using the Thomas algorithm. In Cartesian geometry the exact solution is linear; in cylindrical it is logarithmic; in spherical it is T = A − B/r.

---

## Usage

Open `index.html` in any modern web browser. No build step or server required.

| Control | Action |
|---|---|
| **Coordinate System** | Switch between Cartesian, cylindrical (radial or axial), and spherical |
| **Length / Grid points** | Set domain size and spatial resolution |
| **α, k, ρ, cₚ** | Material thermal properties (α overrides the computed value only if changed independently) |
| **BC Type + Value** | Dirichlet (K) or Neumann (W m⁻²) at each boundary |
| **T₀** | Uniform initial temperature |
| **Δt** | Requested time step (automatically capped at the stability limit) |
| **Run / Pause** | Start or pause the simulation |
| **Reset** | Reset to initial conditions |
| **Steady State** | Compute and overlay the analytical steady-state solution |

### Keyboard shortcuts

| Key | Action |
|---|---|
| `Space` | Run / Pause |
| `R` | Reset |
| `S` | Toggle steady-state overlay |

---

## File Structure

```
index.html   — page structure, math sections, simulator HTML, simulation JavaScript
style.css    — design system (shared with A. Mirza portfolio; DM Sans/Serif/Mono, teal palette)
README.md    — this file
LICENSE.md   — research use only license
```

---

## References

1. Fourier, J.B.J. (1822). *Théorie analytique de la chaleur*. Firmin Didot, Paris.
2. Incropera, F.P. et al. (2007). *Fundamentals of Heat and Mass Transfer* (6th ed.). Wiley.
3. Crank, J. (1975). *The Mathematics of Diffusion* (2nd ed.). Oxford University Press.
4. LeVeque, R.J. (2007). *Finite Difference Methods for Ordinary and Partial Differential Equations*. SIAM.
5. Ozisik, M.N. (1993). *Heat Conduction* (2nd ed.). Wiley.

---

*Part of the [A. Mirza computational science portfolio](https://dthornz.github.io/website-cv-tools/). For research and educational use only — see [LICENSE.md](LICENSE.md).*
