# Erratum

The PDFs in this directory were written before a geometry defect was found. They are preserved as
written rather than retroactively edited — the reasoning path, including the wrong turns, is part of
the record. This file states exactly what is superseded.

## The defect

The six impeller blades were built with inner edges at the disc's outer radius rather than
straddling it. Blades span **r = 0.0365 → 0.0615 m** instead of the standard **r = 0.025 → 0.050 m**.

Swept diameter is therefore **D = 0.123 m**, not the intended **0.100 m** — 23% oversized.

Since blade torque scales as (r₂⁴ − r₁⁴) at fixed blade height, this predicts a torque ratio of
**2.139** relative to correct geometry. The measured over-prediction against the benchmark-implied
torque (0.1986 N·m at Po = 5.0) was **2.134**. A 0.3% match — the discrepancy is fully accounted for.

Source of the error: `RotorLab_Rebuild_Runbook.pdf`, Section 2.3, step 6, which instructs placing
each blade's inner edge overlapping the disc outer radius (0.0375 m) by ~1 mm. That instruction is
wrong. Correct placement puts the blade *tip* at D/2 = 0.050 m, with the blade extending inward to
r = 0.025 m and crossing the disc edge at 0.0375 m.

## Document-by-document

### `RotorLab_Rebuild_Runbook.pdf` — contains the defect

- **Section 2.3, step 6** — blade placement instruction is incorrect. Blades must be positioned with
  tips at r = 0.050 m (inner edge r = 0.025 m), straddling the disc edge at r = 0.0375 m.
- All other dimensions, build ordering, Boolean-consumption rules, and failure modes remain correct.

### `RotorLab_Build_Decision_Log.pdf` — contains the defect

- Records the geometry as built, including the incorrect blade placement, without flagging it.
- The RotZone diagnosis, interface-type findings, meshing workarounds, and file-loss analysis are
  unaffected and remain correct.

### `RotorLab_MRF_Results_Report.pdf` — superseded

- **Section 4** — Po = 10.49 is computed with D = 0.100 m against as-built geometry of D = 0.123 m.
  Not a valid benchmark comparison.
- **Section 6** — attributes the discrepancy to blade-baffle interaction unresolvable by steady MRF.
  Refuted: the transient sliding-mesh run landed within 1.7% of MRF rather than closing the gap.
- **Sections 3 and 5** (convergence, Wall Y+) remain valid.

### `RotorLab_Final_Comparison_Report.pdf` — superseded

- **Executive Summary and Section 1** — the Po comparison against the benchmark is invalid for the
  reason above.
- **Section 7** — identifies Realizable K-Epsilon closure bias as the leading remaining cause. This
  is wrong. Documented k-epsilon bias for Rushton geometries is roughly 10–30%; it cannot account
  for 110%. The magnitude should have been checked against the literature before the hypothesis was
  advanced.
- **Section 3** (periodicity, ~0.1 s half-revolution period, gcd(6,4)=2) remains valid — it depends
  on blade and baffle counts, not radial placement.
- **Sections 2, 5, 6** (convergence, data-quality notes, Wall Y+) remain valid.

### `RotorLab_SlidingMesh_Setup_Log.pdf` — valid

Every setting and rationale documented here is independent of the geometry defect and stands as
written.

## What the defect does not invalidate

- **MRF vs. sliding-mesh agreement (1.7%).** Both methods faithfully simulated the same geometry and
  converged consistently. The cross-validation holds.
- **The ~0.1 s half-revolution periodic torque structure** and its geometric explanation.
- All meshing, interface configuration, stopping-criterion, and monitor-triggering findings.

## Correction pending

Move the six blades radially inward by 0.0115 m so tips land at r = 0.050 m, rebuild the Boolean
chain and mesh, and re-run. Predicted corrected torque ≈ **0.199 N·m**, corresponding to Po ≈ 5.0.
Confirming that figure would validate both the benchmark and this diagnosis in a single pass.
