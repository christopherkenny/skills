---
name: redistricting-review
description: 'Review redistricting simulation or optimization output for data validity, diagnostics, constraint behavior, sensitivity, and substantive interpretability. Use after analysis artifacts exist.'
---

# Redistricting Analysis Review

## Review Dimensions

- Data and plan validity
- Simulation or optimization diagnostics
- Constraint behavior and sensitivity
- Fatal issues that require rerunning or rebuilding data
- Substantive warnings that affect interpretation

## Default Diagnostic Targets

| Diagnostic | Pass | Action if failing |
|-----------|------|-------------------|
| Between-run agreement | Stable enough for the estimand | Increase runs/samples, adjust tempering, or relax over-tight constraints |
| Effective sample size / weights | Not dominated by a few plans | Increase samples or reduce weight concentration from constraints |
| Plan diversity | Broad enough for the question; no collapse to identical plans | Check adjacency, constraints, and initialization |
| Population deviation | Matches stated `pop_tol` and chamber expectations | Large unexpected deviations suggest wrong data, plan, or tolerance |
| Reference plan alignment | Reference plan is complete and comparable to sampled plans | Rejoin or re-ingest plan assignments |

Treat these as default targets, not universal rules. Document any deviation and whether it changes the substantive conclusion.

**Plan weights:** A long right tail means a few plans dominate the ensemble and effective sample size is low. The concern is not the exact range of weights, but whether weighted estimates are being driven by a small number of plans.

**Compactness:** Large enacted-vs-simulated discrepancies can indicate a real substantive difference, a missing criterion, or a data/adjacency problem. Interpret compactness alongside split, population, and diversity diagnostics.

**Administrative splits:** Compare reference-plan splits to the simulated distribution only after confirming that administrative unit IDs are clean and the split metric matches the design question.

**Minority representation diagnostics (if VRA or representation questions apply):** Check for cracking, packing, and opportunity-to-elect evidence using the relevant group, denominator, and election set. Do not treat a fixed VAP threshold as legally sufficient without supporting analysis.

## Data Validity Checks

- Correct jurisdiction, chamber, cycle, and number of districts
- Correct Census vintage and population field
- Enacted/proposed plan assignment is complete and aligned to map units
- No unexpected missing or zero population values
- Adjacency graph has no unexplained islands or disconnected components
- Election columns match the intended elections and have plausible totals
- Administrative units used for split constraints are present and clean

## Common Issues

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Very skewed plan weights | Constraint too tight or too few samples | Loosen constraint strength or increase sample size |
| Very low plan diversity | Adjacency error, over-constrained problem, or fixed initialization | Check `adj`, isolated units, constraints, and initial plans |
| Population deviation unexpectedly high | Wrong population field, wrong census vintage, or plan joined incorrectly | Re-check population column, year, and plan assignment join |
| Between-run diagnostics disagree | Insufficient mixing or unstable weights | Increase runs/samples, tune algorithm settings, or relax tight constraints |
| Reference plan is an extreme outlier on many unrelated metrics | Real outlier, missing legal criterion, or data mismatch | Check design criteria, reference-plan alignment, and metric definitions |

## Sensitivity Analysis

For each substantively important active constraint, test robustness by varying its strength:

| Design choice | Sensitivity test |
|---------------|------------------|
| Administrative split penalty | Weaken and strengthen enough to visibly change split rates |
| Compactness penalty | Compare against a weaker/no compactness setting and a stronger setting |
| Minority representation constraint | Vary threshold/strength according to the substantive theory, then compare group share and electoral-performance diagnostics |
| Population tempering or relaxed balance | Compare against stricter balance if computationally feasible |
| Alternative election sets | Recompute partisan or performance metrics with plausible election subsets/composites |

Report whether the primary estimand is robust or sensitive across the tested range. Define the tolerance in context rather than relying on a universal percentage shift.

## Issue Severity

Treat the following as fatal when they affect the stated estimand: wrong jurisdiction, chamber, cycle, district count, or Census vintage; invalid or incomplete reference plans; population balance or contiguity failures inconsistent with the design; material adjacency errors; or diagnostics showing unusable convergence or plan-diversity collapse.

Treat the following as substantive warnings when they affect interpretation but do not invalidate the analysis: marginal between-run diagnostics, low effective sample size, limited plan diversity, constraint outcomes that drift from the design intent, primary results that move under plausible sensitivity checks, or missing minority-representation diagnostics when VRA or representation constraints were used.

Treat the following as reproducibility warnings: unclear Census year, election set, plan labels, or population field; unrecoverable constraint settings or algorithm parameters; missing random seed or stochastic metadata when needed; or sensitivity runs that cannot be distinguished from the primary run.
