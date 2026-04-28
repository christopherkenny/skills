---
name: redistricting-optimization
description: 'Find redistricting plans that are extreme with respect to a defined objective using redist_shortburst(). Use when the question is what can be achieved, not what is typical. Covers objective definition, scorer functions, multiple restarts, score path diagnostics, and comparison to a neutral ensemble.'
---

# Redistricting Optimization

Optimization answers a different question than simulation: not "what does a typical fair plan look like?" but "what is achievable when plans are deliberately pushed toward an objective?"

`redist_shortburst()` implements the short-burst method (Cannon et al. 2023): repeatedly run a short Markov chain, keep the best plan found, restart from there. The result is a local maximum with respect to the objective — contiguous and within population tolerance, but not a representative draw.

**Optimized plans should always be interpreted as local maxima, not global optima. There are no theoretical guarantees of global optimality.**

## Define the Objective

For every optimization, record:

- Objective in prose
- Score function code
- Whether the objective is maximized or minimized
- Constraints and population tolerance
- Number of bursts, burst size, seeds, and starting plans
- Best score from each restart
- Comparison to a neutral simulation ensemble when available
- Substantive warning that the optimized plan is not a neutral draw

## When to Use Optimization

| Question | Use |
|----------|-----|
| What do neutral plans typically look like? | Simulation (`redist_smc`) |
| Is the enacted plan an outlier? | Simulation + comparison |
| What is the most compact achievable plan? | Optimization |
| How many seats *can* a party win? | Optimization |
| What is the least-splits achievable plan? | Optimization |

Optimization is most useful when paired with a neutral simulation ensemble — the optimized plan's score only becomes interpretable when compared to the distribution of typical scores.

## Core Usage

```r
library(redist)

plans_opt <- redist_shortburst(
  map       = map,
  score_fn  = scorer_frac_kept(map),  # objective to maximize
  max_bursts = 200,                    # number of bursts to run
  return_all = TRUE                    # keep all burst-best plans, not just the final
)
```

`return_all = TRUE` is needed for score path diagnostics. Use `return_all = FALSE` (default) when you only need the final plan.

### Multiple Restarts

Run several restarts from different seeds or starting plans before interpreting the best score. Similar best scores across restarts increase confidence that the optimizer is not reporting an idiosyncratic local maximum.

```r
seeds <- 1:10

opt_runs <- lapply(seeds, function(s) {
  set.seed(s)
  redist_shortburst(
    map = map,
    score_fn = scorer_frac_kept(map),
    max_bursts = 200,
    return_all = TRUE
  )
})

best_by_restart <- tibble(
  seed = seeds,
  best_score = vapply(opt_runs, function(x) max(x$score, na.rm = TRUE), numeric(1))
)
```

### Stopping Early

```r
plans_opt <- redist_shortburst(
  map, score_fn = scorer_frac_kept(map),
  stop_at   = 0.85,   # stop once score reaches this threshold
  max_bursts = 500
)
```

## Built-in Scorers

```r
scorer_frac_kept(map)           # fraction of edges kept (compactness)
scorer_polsby_popper(map)       # Polsby-Popper compactness
scorer_splits(map, admin)       # minimize administrative splits
scorer_group_pct(map, group_pop, total_pop, k)  # k-th highest group share
```

Pass `maximize = FALSE` to `redist_shortburst()` to minimize instead of maximize.

## Custom Scorers

A scorer is a function that takes a plans matrix and returns one numeric score per plan. District-level scores must be aggregated to a single plan-level value.

```r
# Advanced example: maximize Democratic seats, with vote share in next-closest district
#          as a tiebreaker (score of 7.34 = 7 Dem seats, next seat at 34% Dem)
scorer_dem_seats <- function(map, dvote, rvote) {
  dvote <- rlang::eval_tidy(rlang::enquo(dvote), map)
  rvote <- rlang::eval_tidy(rlang::enquo(rvote), map)
  nd <- attr(map, "ndists")

  fn <- function(plans) {
    dcounts <- redistmetrics:::agg_p2d(vote = dvote, dm = plans, nd = nd)
    rcounts <- redistmetrics:::agg_p2d(vote = rvote, dm = plans, nd = nd)
    dseats  <- redistmetrics:::dseats(rcounts = rcounts, dcounts = dcounts)
    dvs     <- redistmetrics:::DVS(dcounts = dcounts, rcounts = rcounts)
    # tiebreak by vote share in next most winnable district
    next_best <- apply(dvs, 2, function(x) max(x[x < 0.5], min(x)))
    dseats + next_best
  }

  class(fn) <- c("redist_scorer", "function")
  fn
}

score_fn <- scorer_dem_seats(map, dvote = ndv, rvote = nrv)

plans_opt <- redist_shortburst(map, score_fn = score_fn,
                               max_bursts = 200, maximize = TRUE)
```

The custom scorer above uses internal `redistmetrics` helpers and should be treated as an advanced pattern. Prefer exported package functions when they can express the objective.

## Diagnostics: Score Path

Plot the best score after each burst to check whether the optimizer is still improving or has plateaued:

```r
plans_opt |>
  as_tibble() |>
  group_by(draw) |>
  summarize(score = first(score)) |>
  mutate(burst = row_number()) |>
  ggplot(aes(burst, score)) +
  geom_line() +
  theme_bw() +
  labs(x = "Burst", y = "Best score")
```

**Interpreting the path:** A flat plateau with no improvement suggests the optimizer may be stuck at a local maximum from the current starting plan. There is no formal stopping rule; use multiple restarts and compare the best scores.

## Comparing to a Neutral Ensemble

Optimized plans are most interpretable against a neutral simulation. Run `redist_smc()` separately, score those plans with the same scorer, then compare:

```r
# Score the neutral ensemble
neutral_scores <- tibble(
  score = score_fn(get_plans_matrix(plans_sim))
)

# Best score from optimizer
best_score <- max(plans_opt$score, na.rm = TRUE)

ggplot(neutral_scores, aes(x = score)) +
  geom_histogram(bins = 30) +
  geom_vline(xintercept = best_score, color = "red", linetype = "dashed") +
  theme_bw() +
  labs(x = "Score", y = "Count",
       title = "Optimized plan vs. neutral ensemble")
```

## Tradeoffs Along the Optimization Path

As optimization proceeds, other metrics often degrade (e.g., maximizing compactness may increase splits). Plot these to characterize the tradeoff:

```r
plans_opt |>
  mutate(
    splits = splits_admin(map, county),
    comp   = comp_edge(map)
  ) |>
  as_tibble() |>
  group_by(draw) |>
  summarize(score = first(score), splits = first(splits), comp = first(comp)) |>
  mutate(burst = row_number()) |>
  pivot_longer(c(score, splits, comp), names_to = "metric", values_to = "value") |>
  ggplot(aes(burst, value)) +
  geom_line() +
  facet_wrap(~metric, scales = "free_y") +
  theme_bw()
```

## Interpretation Guardrails

- Do not describe the result as "the optimal plan" unless exact enumeration proves it.
- Do not use optimized partisan-seat plans as neutral evidence of typicality.
- Always report the objective because different objectives can produce very different maps.
- Check that optimized plans still satisfy the same legal and data constraints used for the comparison ensemble.
- When optimizing a sensitive criterion, report tradeoffs in compactness, splits, population deviation, and minority representation diagnostics.

## References

| Topic | Reference |
|-------|-----------|
| `redist_shortburst`, `scorer_*`, `redist_cyclewalk` | [03-redist-advanced.md](references/03-redist-advanced.md) |
