# Erratum

The PDFs in this directory were written before a geometry defect was found. They are preserved as
written rather than retroactively edited — the reasoning path, including the wrong turns, is part of
the record. This file states exactly what is superseded.

**Status: the defect has been corrected, re-run, and validated. See the repository README for final
results.**

## The defect

The six impeller blades were built as a Block primitive with Base Center `[0.0495, 0, 0.100]` and
length 0.026 m, spanning **r = 0.0365 → 0.0625 m** instead of the standard **r = 0.025 → 0.050 m**,
and sitting *above* the disc plane rather than straddling it.

Swept diameter was therefore **D = 0.125 m**, not the intended **0.100 m**.

Because only blade position changed while blade length, height, and disc diameter retained their
D = 0.100 m values, the as-built impeller was not a geometrically similar larger Rushton either.
The Po ≈ 5.0 correlation did not describe it, and neither the value computed with D = 0.100 m
(Po = 10.49) nor with the true D = 0.125 m was a valid benchmark comparison.

Source of the error: `RotorLab_Rebuild_Runbook.pdf`, Section 2.3, step 6, which instructs placing
each blade's inner edge overlapping the disc outer radius (0.0375 m) by ~1 mm. That instruction is
wrong. Correct placement puts the blade *tip* at D/2 = 0.050 m, with the blade extending inward to
r = 0.025 m and crossing the disc edge at 0.0375 m.

## The correction and its confirmation

Block length 0.026 → 0.025 m, Base Center → `[0.0375, 0, 0.090]`. Verified post-remesh by Maximum
`Position[X]` on a Z-thresholded slice of ImpellerWall: **0.0500 m exactly**.

| | Value |
|---|---|
| Predicted corrected torque (from (r₂⁴−r₁⁴) scaling) | ~0.199 N·m |
| Measured corrected torque, steady MRF | 0.192562 N·m |
| Measured corrected torque, sliding mesh (2-period mean) | 0.193983 N·m |
| **Corrected Po — MRF / sliding mesh** | **4.85 / 4.89** |
| Benchmark | ≈5.0 |

## Document-by-document

### `RotorLab_Rebuild_Runbook.pdf` — contains the defect

- **Section 2.3, step 6** — blade placement instruction is incorrect. Blades must be positioned with
  tips at r = 0.050 m (inner edge r = 0.025 m), straddling the disc edge at r = 0.0375 m, and
  centred on the disc plane in Z.
- All other dimensions, build ordering, Boolean-consumption rules, and failure modes remain correct.

### `RotorLab_Build_Decision_Log.pdf` — contains the defect

- Records the geometry as built, including the incorrect blade placement, without flagging it.
- The RotZone diagnosis, interface-type findings, meshing workarounds, and file-loss analysis are
  unaffected and remain correct.

### `RotorLab_MRF_Results_Report.pdf` — superseded

- **Section 4** — Po = 10.49 is computed against defective geometry. Not a valid benchmark
  comparison. Superseded by the corrected MRF result, Po = 4.85.
- **Section 6** — attributes the discrepancy to blade-baffle interaction unresolvable by steady MRF.
  Refuted: the transient sliding-mesh run landed within 1.7% of MRF rather than closing the gap.
- **Sections 3 and 5** (convergence, Wall Y+) remain methodologically valid.

### `RotorLab_Final_Comparison_Report.pdf` — superseded

- **Executive Summary and Section 1** — the Po comparison against the benchmark is invalid.
  Superseded by the corrected results.
- **Section 7** — identifies Realizable K-Epsilon closure bias as the leading remaining cause. This
  is wrong. Documented k-epsilon bias for Rushton geometries is roughly 10–30%; it cannot account
  for 110%. The magnitude should have been checked against the literature before the hypothesis was
  advanced. The actual cause was geometric.
- **Section 3** (periodicity) remains directionally valid but the corrected run supersedes it: the
  measured fundamental period is **0.0500 s** (blade-baffle passing, 4 baffles), not the 0.1 s
  half-revolution reported there.
- **Sections 2, 5, 6** (convergence, data-quality notes, Wall Y+) remain valid.

### `RotorLab_SlidingMesh_Setup_Log.pdf` — valid, with one addition

Every setting and rationale documented here is independent of the geometry defect and stands as
written. One addition from the corrected run: **10 inner iterations proved insufficient** on the
corrected mesh, stalling end-of-timestep Continuity at 4.75×10⁻². Raising to 25 improved it by 330×.
See the README section on steady-adequate meshes failing under transient.

## What the defect never invalidated

- **MRF vs. sliding-mesh agreement.** Both methods faithfully simulated whatever geometry they were
  given and converged consistently — 1.7% apart as-built, 0.74% apart corrected.
- All meshing, interface configuration, stopping-criterion, and monitor-triggering findings.
- The existence of periodic blade-baffle structure in the transient result, though the corrected run
  revises the period from 0.1 s to 0.05 s.
