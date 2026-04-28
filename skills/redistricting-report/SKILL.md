---
name: redistricting-report
description: 'Identify the substantive, evidentiary, and reproducibility information needed for a redistricting report. Use to check that final writeups have enough support for their claims without prescribing the report format or prose structure.'
---

# Redistricting Report Content

Use this skill to determine what information a redistricting report must contain. It does not prescribe section order, prose style, or interaction flow. The purpose is to ensure that claims about plans, ensembles, criteria, and diagnostics are supported by the analysis artifacts.

## Claim Evidence Categories

### Question and Estimand

Include enough information to identify:

- Jurisdiction, chamber, cycle, and plan or plans evaluated
- Substantive question being answered
- Estimand or comparison target
- Whether the analysis is sampling, optimization, descriptive scoring, exact enumeration, or a combination
- Direction and interpretation of key metrics

### Data Provenance

Identify:

- Geographic units and source
- Population data source, year, and population variables
- Demographic data source and denominator used for group-share measures
- Election data source, election years, offices, and any composite construction
- Reference/enacted/proposed plan source and join method
- Any unit harmonization, spatial interpolation, aggregation, or exclusion

### Criteria and Modeling Choices

Distinguish:

- Legal criteria from substantive or analyst-imposed criteria
- Criteria encoded as hard constraints
- Criteria encoded as soft constraints
- Criteria measured only as diagnostics
- Criteria mentioned only as context

For each modeled criterion, include its operationalization: variable, metric, threshold, score function, or constraint.

### Method Information

Include the details needed to understand the analysis:

- Algorithm or method family
- Population tolerance and contiguity handling
- Constraint specification and strength choices
- Number of plans, independent runs, restarts, thinning, or enumeration scope
- Whether plans are weighted and how weights are used
- Reference plans added for comparison
- Sensitivity variants that were run

### Diagnostics and Validity

Report the diagnostics relevant to the method:

- Between-run agreement or convergence evidence
- Effective sample size, weight concentration, or plan diversity
- Population deviation checks
- Adjacency and contiguity checks
- Reference-plan alignment checks
- Sensitivity of the primary result to key modeling choices
- Any failed or marginal diagnostics and their implications

### Results

For each major result, specify:

- Metric or quantity
- Value for the reference plan, if any
- Ensemble distribution, percentile, rank, interval, or comparison baseline
- Direction of interpretation
- Whether the result is robust to sensitivity checks
- Which figure, table, or computed object supports the claim

### VRA and Minority Representation

If the analysis discusses VRA or minority representation, identify:

- Group or groups analyzed
- Denominator: VAP, CVAP, or another population measure
- Relevant election set for performance analysis
- Whether racially polarized voting evidence is used, assumed, or not assessed
- Whether the claim is legal compliance, opportunity-to-elect, majority-minority capacity, cracking/packing, or descriptive representation
- Demographic and electoral-performance diagnostics supporting the claim

Do not let a group-share threshold alone carry a legal conclusion.

### Limitations

State limitations that affect interpretation:

- Data vintage or quality limits
- Missing elections, demographics, or plan information
- Unresolved legal criteria
- Model constraints that are proxies for legal/substantive concepts
- Diagnostics that are weak, marginal, or not run
- Optimization local-optimum limits
- Sensitivity results that change the main conclusion

### Reproducibility

Include enough information to reproduce or audit the analysis:

- Code or workflow location
- Data source references and download/access dates when relevant
- Random seeds or stochastic run metadata
- Package versions or computational environment
- Saved analysis objects or regeneration instructions
- Figure/table generation source

## Claim Support Rules

- A plan-outlier claim requires a reference plan value and an ensemble distribution, percentile, rank, or comparable baseline.
- A "typical plans" claim requires a sampling design and diagnostics showing the ensemble is usable for the estimand.
- An "achievable" claim requires an optimization or enumeration method, objective definition, and local/global-optimum caveat.
- A legal-criteria claim requires a legal/source citation and a separate explanation of how the analysis operationalizes that criterion.
- A partisan claim requires clear metric direction and election data provenance.
- A minority-representation claim requires group denominator, election-performance context when relevant, and careful legal caveats.
- A robustness claim requires an explicit sensitivity comparison.
- A reproducibility claim requires enough code, data, parameters, and environment information to audit the result.
