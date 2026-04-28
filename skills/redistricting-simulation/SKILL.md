---
name: redistricting-simulation
description: 'Generate or score a neutral redistricting ensemble in R using the redistverse. Covers pre-run QA, map construction, adjacency validation, constraint specification, SMC/MCMC simulation, convergence checking, plan metrics, and reproducibility.'
---

# Redistricting Simulation

## Preflight QA

Before running a simulation, check:

- Geometry is valid, projected appropriately for spatial operations, and has one row per unit
- Population columns are present, non-missing, and sum to expected totals
- District count matches the chamber and enacted/proposed plan
- Enacted/proposed plan IDs are present, complete, and use consistent district labels
- Adjacency has no unexpected isolated units or disconnected components
- Election columns exist and match the intended election set
- Administrative units used for split constraints are present and clean
- Stochastic settings and saved-object locations are planned

## Key Data Structures

**`redist_map`** — `sf` tibble, one row per precinct. Stores adjacency graph, population column, number of districts, and population tolerance. Standard columns: `pop`, `pop_black`, `pop_hisp`, `pop_asian`, `pop_white`, `pop_vap`, `pop_bvap`, `ndv`, `nrv`, `geometry`.

**`redist_plans`** — tibble, one row per district per plan (`nsims × ndists` rows, plus reference plans). Access the plan matrix with `get_plans_matrix()`. Add metrics via `mutate()`.

## Setup

```r
library(redistverse)  # loads redist, redistmetrics, ggredist, geomander, sf, adj

set.seed(<seed>)

map <- redist_map(
  shp,
  pop_col = "pop",
  existing_plan = existing_plan,
  pop_tol = <tolerance>,
  ndists = <number_of_districts>
)
```

If a validated precomputed ensemble already answers the question, score and compare that ensemble instead of regenerating plans.

## Adjacency

```r
adj <- geomander::get_adjacent(shp)

# Remove edges across water or mountain barriers
adj <- geomander::subtract_edge(adj, which(shp$GEOID == "A"), which(shp$GEOID == "B"))

# Add edges for physical connections (ferries, bridges)
adj[[i]] <- sort(c(adj[[i]], j))
adj[[j]] <- sort(c(adj[[j]], i))
```

## Constraints

```r
cons <- redist_constr(map) |>
  # Administrative subdivision splits
  add_constr_splits(strength = 2.0, admin = county)
```

Add VRA-related constraints only when the legal or substantive theory has been specified. Record the group, denominator, threshold, and reason for the constraint.

```r
cons <- cons |>
  # Example: encourage districts above a specified group-share threshold
  add_constr_grp_hinge(
    strength = <strength>,
    group_pop = map$<group_population>,
    total_pop = map$<denominator_population>,
    tgts_group = <threshold>
  ) |>
  # Example: sensitivity check for packing above a specified ceiling
  add_constr_grp_hinge(
    strength = <strength>,
    group_pop = map$<group_population>,
    total_pop = map$<denominator_population>,
    tgts_group = <threshold>,
    form = "inv_hinge"  # use inv_hinge when goal is "keep below" a ceiling
  )
```

Equal population is set via `pop_tol` in `redist_map()`, not as a separate constraint.

## Population Tolerance

| Map type | `pop_tol` | Legal basis |
|----------|-----------|-------------|
| Congressional | chosen for the analysis and jurisdiction | Equal population requirement |
| State legislative | chosen for the analysis and jurisdiction | Equal population requirement |

Treat `pop_tol` values as modeling choices to document, not universal legal conclusions.

## Simulation

```r
plans <- redist_smc(
  map,
  nsims = <number_of_plans>,
  runs = <independent_runs>, # must be >= 2
  ncores = <cores>,
  constraints = cons,
  pop_temper = <temper>
)

# Assess convergence on the full pre-thinned sample
summary(plans)

# Then thin sampled plans to the planned final size.
# Preserve reference plans separately if the object includes them.
sampled <- plans |> filter(!draw %in% subset_ref(plans)$draw)
refs <- subset_ref(plans)
keep_draws <- sample(unique(sampled$draw), min(<final_sample_size>, n_distinct(sampled$draw)))
plans_final <- bind_rows(sampled |> filter(draw %in% keep_draws), refs)
```

**Default convergence targets:** R-hat ≤ 1.05 · substantial ESS% in final splits · log weight SD < 3.0 · no collapse in diversity. Treat these as review targets; document deviations and reruns.

**If convergence is poor:** increase independent runs or sample size, tune population tempering, weaken over-tight constraints, or cross-check with an alternative algorithm.

## Metrics

```r
plans <- plans |>
  mutate(
    eg     = part_egap(map, plans, ndv, nrv),        # efficiency gap
    mm     = part_mean_median(map, plans, ndv, nrv),  # mean-median
    pp     = comp_polsby(plans, map),                  # Polsby-Popper compactness
    splits = splits_admin(plans, map, county)          # county splits
  )
```

## Reproducibility

- `map`: final `redist_map` after data cleaning and adjacency edits
- `cons`: final `redist_constr` object or equivalent code
- Raw unthinned `plans`
- Final scored `plans_final`
- Diagnostic output from `summary(plans)`
- Seed, package versions, and simulation parameters

## References

| Topic | Reference |
|-------|-----------|
| `redist_map`, `redist_smc`, diagnostics, thinning | [01-redist-simulation.md](references/01-redist-simulation.md) |
| All constraint functions and strength calibration | [02-redist-constraints.md](references/02-redist-constraints.md) |
| Compactness metrics | [04-redistmetrics-compactness.md](references/04-redistmetrics-compactness.md) |
| Partisan metrics | [05-redistmetrics-partisan.md](references/05-redistmetrics-partisan.md) |
| Adjacency construction and editing | [09-geomander-adjacency.md](references/09-geomander-adjacency.md) |
