---
name: redistricting-design
description: 'Analyze a redistricting question before implementation. Use to identify legal/substantive criteria, translate them into estimands, decide whether sampling, optimization, descriptive scoring, or communication support is appropriate, and determine what data are required.'
---

# Redistricting Analysis Design

Use this skill to turn a redistricting question into an analysis specification. The goal is not to format a memo; it is to determine what should be measured, what criteria matter, which criteria can be encoded in a model, and which method answers the question.

## Identify the Question Type

Classify the user's question before selecting a method:

| User question | Analysis target | Method family |
|---------------|-----------------|---------------|
| "Is this plan unusual?" | Enacted/proposed plan rank in a neutral distribution | Sampling |
| "What do typical valid plans look like?" | Distribution of metrics across valid plans | Sampling |
| "How many seats could a party win?" | Extreme achievable partisan outcome | Optimization, usually compared to sampling |
| "What is the fewest county splits possible?" | Extreme achievable administrative criterion | Optimization |
| "Does this plan satisfy criteria?" | Scores for one or more submitted plans | Descriptive scoring |
| "Which criteria apply?" | Legal/substantive design question | Criteria analysis |

Sampling estimates typicality. Optimization finds extreme achievable examples. Descriptive scoring measures existing plans. These are different estimands; do not substitute one for another.

## Identify Applicable Criteria

Separate criteria into four classes:

1. **Hard constraints**: must be enforced or the plan is invalid for the analysis.
2. **Soft constraints**: should influence the ensemble but may vary across valid plans.
3. **Diagnostics/outputs**: should be measured, not necessarily constrained.
4. **Context only**: relevant for interpretation but not directly modeled.

Common criteria:

| Criterion | Usually modeled as | Notes |
|-----------|-------------------|-------|
| Equal population | Hard constraint via `pop_tol` | Tolerance depends on chamber, jurisdiction, and analysis purpose |
| Contiguity | Hard constraint via adjacency and algorithm | Verify islands and water connections before simulating |
| Compactness | Soft constraint or diagnostic | Choose metric: Polsby-Popper, Reock, edge compactness, BBR, etc. |
| County/municipal splits | Soft constraint or diagnostic | Decide whether to penalize any split, multi-splits, or total pieces |
| Communities of interest | Usually context or custom constraint | Requires data/proxy; often hard to encode cleanly |
| Core retention / least change | Soft constraint or diagnostic | Requires reference plan |
| Incumbent protection/pairing | Soft constraint or diagnostic | Requires incumbent locations |
| Partisan fairness | Estimand or diagnostic | Usually not a neutral constraint unless explicitly studying constrained maps |
| Minority representation / VRA | Legal/context, constraint, and diagnostic | Requires group, denominator, and electoral performance theory |

Do not encode every legal or substantive criterion as a simulation constraint. Some criteria are better reported as diagnostics so the ensemble can reveal tradeoffs.

## Research Legal Criteria

When jurisdiction is known:

1. Check state redistricting criteria from a current source such as NCSL: <https://www.ncsl.org/elections-and-campaigns/redistricting-criteria>
2. Check state constitution, statutes, commission rules, or court orders when the project requires legal precision.
3. Record federal requirements separately from state/local criteria: population equality and constitutional/Voting Rights Act protections.
4. For each criterion, identify whether it is mandatory, permissive, traditional, or merely relevant background.
5. Mark unresolved criteria as unresolved. Do not invent thresholds or convert vague legal standards into precise constraints without justification.

## VRA and Minority Representation Design

Treat VRA-related design as an evidentiary question, not only a threshold choice.

Identify:

- Minority group or coalition being studied
- Denominator: VAP, CVAP, citizen voting-age by race/ethnicity, or another measure
- Relevant elections for performance analysis
- Whether racially polarized voting evidence exists or is assumed from prior work
- Whether the analysis asks about majority-minority districts, opportunity-to-elect districts, cracking, packing, or descriptive representational capacity

Use group hinge constraints only when the analysis has a clear reason to encourage or discourage specific group shares. Always plan diagnostics that report group shares and electoral performance separately.

## Choose the Method

### Use Sampling When

- The question is about typicality, outlier status, or the distribution of possible plans.
- The enacted/proposed plan will be compared to a neutral baseline.
- The output needs percentiles, ranks, histograms, or uncertainty about what is common.

Default implementation: SMC with `redist_smc()`. Use MCMC/merge-split as a cross-check or when SMC is difficult for the geography/constraints.

### Use Optimization When

- The question asks what is achievable under constraints.
- The target is an extreme: maximum seats, minimum splits, maximum compactness, maximum minority opportunity districts, etc.
- The output is an example plan or bound-like result, not a representative ensemble.

Optimization should usually be interpreted against a sampled neutral ensemble. Run multiple restarts and describe results as local optima unless exact enumeration is possible.

### Use Descriptive Scoring When

- The user only needs metrics for existing plans.
- There is no need to characterize the full space of possible plans.
- The task is QA, comparison among submitted plans, or preparing tables.

### Use Exact Enumeration When

- The map is very small.
- The question requires exact probabilities or complete characterization.
- The number of units is low enough for enumeration to be computationally feasible.

## Data Requirements

Match data to the selected method and estimand:

| Need | Required data |
|------|---------------|
| Any simulation/optimization | Geography, population, district count, adjacency |
| Enacted-plan comparison | Enacted/proposed assignment aligned to map units |
| Partisan metrics | Election returns or composite vote columns |
| Minority representation | Demographic population and voting-age/citizen voting-age data as appropriate |
| County/municipal splits | Administrative unit identifiers |
| Core retention / least change | Prior district assignment |
| Incumbent analysis | Incumbent locations or home units |
| Official plan ingestion | Block assignment files or district geometries |

Before implementation, verify that the data unit, plan assignment unit, and election/demographic unit can be joined without losing or duplicating population.

## Criteria Operationalization

For each criterion, the key modeling attributes are:

- Source: legal citation, substantive theory, data availability, or analyst choice
- Role: hard constraint, soft constraint, diagnostic, or context
- Operationalization: variable, threshold, metric, or scorer
- Risk: what could go wrong if the criterion is too tight, too vague, or unsupported by data

Example:

| Criterion | Source | Role | Operationalization | Risk |
|-----------|--------|------|--------------------|------|
| Equal population | Federal/state law or analysis design | Hard | `pop_tol = ...` | Too loose or too tight for chamber |
| County preservation | State law or substantive design | Soft + diagnostic | `add_constr_splits()` and `splits_admin()` | Over-constrains ensemble |
| Partisan bias | Research question | Estimand | `part_egap()`, `part_mean_median()` | Single metric may be misleading |
| BVAP opportunity | VRA/substantive | Constraint + diagnostic | group hinge, BVAP/CVAP plots, election performance | Threshold without RPV/performance evidence |

## Method Selection Cues

- If comparing enacted/proposed plan to "normal" plans: use sampling.
- If searching for a best/worst case: use optimization.
- If optimization result will be interpreted substantively: also run or cite a neutral ensemble.
- If only scoring provided plans: do descriptive metrics, not simulation.
- If legal criteria are uncertain: keep them as diagnostics or sensitivity variants rather than hard-coding them.
- If constraints are central to the result: plan sensitivity runs that vary their strength.
- If VRA is central: plan demographic and electoral-performance diagnostics, not only threshold counts.
