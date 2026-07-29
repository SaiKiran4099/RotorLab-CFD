# RotorLab — Rushton Turbine Stirred-Tank CFD Benchmark

**Steady MRF vs. transient sliding mesh in Simcenter STAR-CCM+, benchmarked against the published Power Number correlation.**

A self-directed CFD validation study on a standard baffled Rushton turbine stirred tank. The project builds the same case twice — once with a steady Moving Reference Frame (MRF) approximation, once with a fully transient sliding mesh — and compares both against the published Po ≈ 5.0 benchmark to test *why* they disagree.

---

## Headline Result

| Metric | Steady MRF | Sliding Mesh | Published Benchmark |
|---|---|---|---|
| Converged torque | 0.41666 N·m | 0.42362 N·m (period-averaged) | — |
| **Power Number (Po)** | **10.49** | **10.67** | **≈5.0** |
| Ratio vs. benchmark | ≈2.10× | ≈2.13× | 1.00× |
| Agreement with each other | — | within 1.7% of MRF | — |

**The interesting finding is the negative one.** The working hypothesis going into the transient phase was that steady MRF's inability to resolve blade-passage unsteadiness drove the ~2.1× over-prediction. Sliding mesh was the direct test of that hypothesis — and it failed to close the gap, landing within 1.7% of the MRF result rather than moving toward the benchmark.

That reframes the diagnosis: a factor **common to both runs** is responsible. The leading candidate is the Realizable K-Epsilon turbulence closure, documented in the literature to over-predict power draw for Rushton impellers regardless of whether the solve is steady or transient. Mesh resolution was independently checked and ruled out as a primary driver (Wall Y+ predominantly well below target across the wetted impeller surface).

## What Sliding Mesh *Did* Deliver

A genuine periodic torque signal with a **~0.1 s fundamental period — half a revolution, not a full one.** This matches the geometry exactly: with 6 blades and 4 baffles, `gcd(6,4) = 2`, so the combined rotor–stator spatial pattern repeats every 180°. Steady MRF is structurally incapable of producing this, since it freezes the rotor at a single angular position for the entire solve.

![Velocity magnitude](images/velocity_magnitude_plane.png)

*Velocity magnitude (Lab Reference Frame) on a plane section. The classic Rushton signature: a radial discharge jet firing outward from the impeller and splitting into upper and lower circulation loops through the bulk tank — confirming both fluid regions are fully coupled across the sliding interface.*

---

## Case Setup

| Parameter | Value |
|---|---|
| Tank diameter (T) | 0.300 m |
| Impeller diameter (D) | 0.100 m |
| Impeller | 6-blade Rushton disc turbine |
| Baffles | 4 × wall-mounted, 0.030 m wide, full height |
| Rotation rate (N) | 5 rev/s (300 rpm) = 31.416 rad/s |
| Fluid | Water — 998 kg/m³, 0.001 Pa·s |
| Mesh | 788,728 cells, trimmed + prism layers, two regions |
| Turbulence | Realizable K-Epsilon Two-Layer, Two-Layer All Y+ |
| Interface | Internal / In-place / Imprinted, `Reset on Relative Motion` enabled for the transient case |
| Timestep (transient) | 5.5556×10⁻⁴ s (1° per timestep, 360 steps/rev) |
| Run length (transient) | ~0.81 s ≈ 8 revolutions, 10 inner iterations/timestep |

**Power Number:** `Po = Torque × 2πN / (ρN³D⁵)` — reduces to `Po ≈ Torque × 25.18` for these parameters.

![Geometry and mesh](images/geometry_mesh_tank.png)

---

## Convergence

![Residuals](images/residuals_full_run.png)

The first ~1500 iterations are the steady MRF phase converging conventionally. After the switch to sliding mesh, residuals stop trending toward zero and settle into a repeating banded pattern — the correct signature of a solution locked onto a genuine periodic state, not a stalled solve. In a transient periodic problem there is no single final answer for the across-timestep residual trend to shrink toward; what matters is the *within-timestep* drop across inner iterations, which held at roughly 5–10× per timestep throughout.

![Wall Y+](images/wall_yplus_impeller.png)

*Wall Y+ on the impeller: range 0.113–49.1, visually dominated by low values across most of the wetted surface.*

---

## Repository Contents

```
docs/
  RotorLab_Final_Comparison_Report.pdf   # Capstone: MRF vs sliding mesh vs benchmark
  RotorLab_MRF_Results_Report.pdf        # Steady-phase results and discussion
  RotorLab_SlidingMesh_Setup_Log.pdf     # Every transient setting changed, and why
  RotorLab_Build_Decision_Log.pdf        # Geometry, meshing, and debugging narrative
  RotorLab_Rebuild_Runbook.pdf           # Validated from-scratch rebuild spec
data/
  moment_periodic_window.csv             # Torque time series over the averaging window
images/                                  # Figures used above
```

The `docs/` set is written as an engineering record rather than a polished writeup — including the failure modes, the wrong theories that got disproved, and the reasoning that corrected them.

---

## Selected Engineering Problems Solved

- **3D-CAD primitive convention trap.** A Cylinder primitive's "Center" field specifies the center of the *base 2D profile* before extrusion, not the volumetric center of the resulting solid. Entering the intended mid-height Z produced a rotating zone offset by half its height — a defect that passed 100% mesh face-validity checks and was only caught by direct body-extents measurement. Two plausible-but-wrong root causes were disproved by evidence before the real one was found.
- **Trimmed mesher multi-part limitation.** The Trimmed Cell Mesher cannot process multiple parts in one Automated Mesh operation; resolved by splitting into two operations, which also simplified impeller-local refinement.
- **Interface type correctness.** Direct Region Interfaces silently produce a Heat Exchanger–type interface that blocks flow. Correct configuration requires an Internal/In-place/Imprinted interface, plus `Reset on Relative Motion` for the sliding case so the flux mapping recomputes as the rotor sweeps.
- **Stopping-criterion semantics.** An Asymptotic torque criterion tuned for steady convergence repeatedly triggered prematurely on slow monotonic drift, then became conceptually invalid altogether once the signal turned periodic — convergence had to be judged by cycle-to-cycle waveform repeatability instead.
- **Transient monitor triggering.** Monitors left on iteration-triggering sample inner-loop noise rather than converged per-timestep values; switching to timestep-triggering is what made the periodic signal legible at all.

## Known Limitations

- Sliding-mesh torque was averaged over a 130-timestep window (~0.736–0.810 s), slightly short of one full 0.1 s fundamental period — a solid working estimate, not a laboratory-grade final digit.
- No flow-field animation was produced. Repeated stop/resume cycles on the remote session desynced the Solution History file's internal snapshot numbering, causing recurring HDF5 write failures; the recording was disabled. Core solve integrity was independently verified unaffected (Time-Step and Physical Time stayed exactly consistent with Δt across every resume boundary).
- Turbulence-model sensitivity was not tested. Given the evidence, this is the highest-value next step — an SST k-omega or scale-resolving run would directly test the remaining hypothesis.

---

## Tools

Simcenter STAR-CCM+ · 3D-CAD · Trimmed/prism meshing · MRF and rigid-body-motion rotating machinery · RANS turbulence modeling · Python (post-processing, report generation)
