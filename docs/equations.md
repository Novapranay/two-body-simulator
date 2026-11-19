# Equations and where they are implemented

All physics lives in the `<script id="physics-module">` block of
[`two_body_simulator.html`](../two_body_simulator.html). Each function there has a comment tag `[EQ-n]`, and
the Theory card in the app uses the same tags. Line numbers refer to the current version of the file.

Notation: r = r₂ − r₁ (relative position), v = v₂ − v₁ (relative velocity), M = m₁ + m₂,
μ = m₁m₂/M, GM = G(m₁ + m₂).

## Dynamics and integration

| Tag | Equation | Function | Line |
|---|---|---|---|
| EQ-1 | F = G m₁ m₂ / r², along the line of centres, F₁₂ = −F₂₁ | `accelerations` | 302 |
| EQ-2 | a₁ = +G m₂ r / r³, a₂ = −G m₁ r / r³ (Newton's 2nd law) | `accelerations` | 302 |
| EQ-25 | drag pair (perturbation): F₂ = −k m₂ v, F₁ = −F₂ | `accelerations` | 309 |
| EQ-3 | Velocity-Verlet: v½ = v + a(x)·dt/2; x′ = x + v½·dt; v′ = v½ + a(x′)·dt/2 | `verletStep` | 319 |
| EQ-4 | RK4: s′ = s + dt/6·(k₁ + 2k₂ + 2k₃ + k₄) | `derivative`, `rk4Step` | 331, 336 |

## Conserved quantities

| Tag | Equation | Function | Line |
|---|---|---|---|
| EQ-5 | KE = ½ m₁ v₁² + ½ m₂ v₂² | `energies` | 357 |
| EQ-6 | PE = −G m₁ m₂ / r | `energies` | 357 |
| EQ-7 | E = KE + PE | `energies` | 357 |
| EQ-8 | R = (m₁r₁ + m₂r₂)/M, V = (m₁v₁ + m₂v₂)/M | `centreOfMass` | 365 |
| EQ-9 | P = m₁v₁ + m₂v₂ | `linearMomentum` | 372 |
| EQ-10 | L = Σ mᵢ (rᵢ − R) × (vᵢ − V) | `angularMomentumCOM` | 378 |
| EQ-11 | L = μ r × v (same value, reduced-mass form) | `angularMomentumReduced` | 386 |
| EQ-23 | Kepler's 2nd law: dA/dt = \|r × v\|/2 = L/(2μ) | `arealRate`; per-step sector area in `advance` | 392, 695 |

## Reduced mass, orbit shape and orbit type

| Tag | Equation | Function | Line |
|---|---|---|---|
| EQ-12 | μ = m₁ m₂ / (m₁ + m₂) | `reducedMass` | 349 |
| EQ-13 | specific orbital energy ε = v²/2 − GM/r | `orbitalElements` | 401 |
| EQ-14 | h = r × v; eccentricity vector e = [(v² − GM/r) r − (r·v) v] / GM | `orbitalElements` | 401 |
| EQ-15 | p = h²/GM, a = −GM/(2ε), r_p = p/(1+e), r_a = p/(1−e) | `orbitalElements` | 401 |
| EQ-27 | classifier: ε < 0 → circle (e < 10⁻³) or ellipse; ε ≈ 0 or e ≈ 1 → parabola; ε > 0 → hyperbola | `orbitalElements` | 401 |
| EQ-16 | Kepler's 3rd law: T = 2π √(a³ / GM) | `keplerPeriod` | 395 |
| EQ-17 | vis-viva: v² = GM (2/r − 1/a) | `visViva` | 398 |
| EQ-18 | v_circ = √(GM/r), v_esc = √(2GM/r), v_periapsis = √(GM(1+e)/r_p) | `speedForOrbit`, `circularSpeed` | 424, 457 |
| EQ-19 | conic: r(ν) = p/(1 + e cos ν), v_r = √(GM/p)·e sin ν, v_t = √(GM/p)·(1 + e cos ν) | `conicState` | 435 |
| EQ-20 | COM-frame start: r₁ = −(m₂/M) r, r₂ = (m₁/M) r (velocities the same way) | `initialState` | 446 |

## Geosynchronous orbits

| Tag | Equation | Function | Line |
|---|---|---|---|
| EQ-21 | r_geo = (G M T_sid² / 4π²)^(1/3), T_sid = 86,164.1 s | `geoRadius` | 456 |
| EQ-22 | Earth rotation angle θ_E = θ₀ + (2π/T_sid)·t | `earthAngle` | 460 |
| EQ-24 | east–west drift rate λ̇ = 2π/T − ω_E | `geoDriftRate` | 463 |
| EQ-26 | ground track: tilt the orbit plane by i about the node line, then subtract θ_E → (latitude, longitude) | `groundTrack` | 467 |

## Where the equations are used in the application

| Function | Line | Role |
|---|---|---|
| `resetSim` | 606 | builds the initial state (EQ-18, EQ-20); keeps r₀ = r_geo when Earth's mass changes (EQ-21) |
| `setTiming` | 650 | fixed step dt = 2π√(r_p³/GM) / N (EQ-16 at periapsis) |
| `advance` | 680 | fixed sub-steps with the chosen integrator; Kepler sector areas; collision check |
| `burn` | 716 | external impulse Δv = ±10 % of \|v\| on body 2 |
| `updateDashboard` | 886 | live values, drift badges, geosynchronous readouts |
| `render` | 997 | canvas: trails, osculating orbit (EQ-19), sectors, rotating Earth, vectors |
| `drawGraph`, `drawInset` | 1148, 1172 | conservation graphs; Earth-fixed ground track (EQ-26) |
| `runSelfTests` | 1237 | tests T1–T6 (see the README) |

## Short derivations

**Reduced mass.** Subtract the two equations of motion:
r̈ = r̈₂ − r̈₁ = −G m₁ r/r³ − G m₂ r/r³ = −G(m₁+m₂) r/r³. Multiplying by μ gives μ r̈ = −G m₁ m₂ r/r³:
a single particle of mass μ moving around a fixed mass M = m₁ + m₂.

**Geosynchronous radius.** For a circular orbit, gravity supplies the centripetal force:
G M m / r² = m ω² r with ω = 2π/T_sid, so r³ = G M T_sid² / 4π².

**Why L is conserved.** The gravitational pair acts along r, so the total torque about the COM is
r × F = 0 and dL/dt = 0. This is also why each Verlet step conserves L exactly: the position update is
along v½ and the velocity kicks are along r.

**Why E is conserved.** Gravity is conservative (F = −∇U with U = −G m₁ m₂/r). Velocity-Verlet is
symplectic, so it exactly conserves a slightly perturbed "shadow" energy. The measured energy therefore
oscillates within a bounded band instead of drifting.
