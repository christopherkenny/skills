---
name: redistricting-visualization
description: 'Create redistricting figures and tables using ggredist and rict. Covers district maps, metric distributions, compactness and split plots, minority representation diagnostics, evidence-supported figure selection, and gt summary tables.'
---

# Redistricting Visualization

## Figure Evidence

Choose figures that match the estimand and known diagnostics/caveats. Every visual claim should be supported by the plotted data or a companion table.

Common figure types include district maps, metric distributions with reference plans marked, administrative-split distributions, compactness distributions, minority-population share diagnostics, minority electoral-performance diagnostics, and constraint-sensitivity displays.

## District Maps

Use built-in redist plotting helpers for standard plan displays. Reach for custom `ggplot2`/`ggredist` code only when the built-in plot does not express the comparison.

```r
# Grid of sampled or reference plans
redist.plot.plans(plans, draws = 1:4, shp = map)
```

For custom maps, never put `aes()` in the top-level `ggplot()` call. Put mappings in the explicit layer that uses them. This avoids accidental inheritance and common failures when later layers use different data.

```r
library(ggredist)

# Partisan score fill by precinct
ggplot(map) +
  geom_sf(
    aes(fill = ndv / (ndv + nrv)),
    color = "white", linewidth = 0.1
  ) +
  scale_fill_party_c(name = "Dem. share") +
  theme_map()

# District identity fill with dissolved districts
ggplot(map) +
  geom_district(aes(group = district, fill = factor(district))) +
  scale_fill_penn82() +
  theme_map()
```

For dense precinct maps, simplify geometry only for display, keep thin or suppressed unit boundaries, and use insets or cropped panels when statewide maps hide the relevant geography.

## Metric Distributions

Use `redist_plans` plotting methods for standard ensemble distributions. They know how to evaluate plan metrics and are less error-prone than recreating histograms by hand.

```r
# Histogram of a plan-level metric
hist(plans, part_egap(pl(), ndv, nrv), bins = 40)

# District-level metric distribution
plot(plans, comp_polsby(pl(), map), style = "box")

# Two-metric comparison
plot(
  plans,
  x = part_egap(pl(), ndv, nrv),
  y = splits_admin(pl(), map, county)
)
```

Use consistent axis direction for partisan metrics and state which party positive values favor. When comparing an enacted plan to an ensemble, report the enacted plan's percentile/rank alongside the plot.

If a custom `ggplot2` distribution is still needed, keep aesthetics inside layers:

```r
ggplot(as_tibble(plans)) +
  geom_histogram(
    aes(x = efficiency_gap),
    bins = 40, fill = "steelblue", alpha = 0.8
  ) +
  geom_vline(
    xintercept = reference_value,
    color = "red", linetype = "dashed"
  ) +
  theme_bw() +
  labs(x = "Efficiency Gap", y = "Count")
```

## Summary Tables

```r
library(rict)

rict_table(plans, map)                          # population, demographics, elections
rict_table(plans, map, type = "compactness")    # compactness metrics
rict_table(plans, map, type = "splits")         # split counts
```

## Saving Figures

```r
ggsave("figures/{name}.png", width = 8, height = 6, dpi = 300)
```

Use draft settings while iterating and publication settings for final output. Wrap `ggsave()` calls in `if (interactive()) { }` when figures should not be regenerated every time the script is sourced.

## Figure Consistency Risks

Common figure problems include text, legends, or captions that do not match metric definitions; inconsistent reference-plan labels across figures and tables; colors that imply party for nonpartisan metrics; map projections or extents that distort the comparison; and saved figure paths that do not match the analysis script.

## References

| Topic | Reference |
|-------|-----------|
| Full ggredist API — color scales, geoms, themes | [07-ggredist.md](references/07-ggredist.md) |
| Full rict API — table types and options | [16-rict.md](references/16-rict.md) |
