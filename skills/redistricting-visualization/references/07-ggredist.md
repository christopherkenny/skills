# ggredist — Visualization

## Overview

`ggredist` extends `ggplot2` with geoms, scales, and themes designed specifically for redistricting maps and election data. The key innovation is `geom_district()`, which dissolves precinct polygons into district regions and handles adjacent-district coloring automatically.

## Core Geom: `geom_district()`

Dissolves precinct-level `sf` geometries into district polygons and fills them by a variable. The `group` aesthetic specifies the district assignment column.

```r
library(ggredist)
library(ggplot2)

# Basic district map — color by district number
ggplot(map) +
  geom_district(aes(fill = cd_2020, group = cd_2020)) +
  theme_map()

# Partisan vote share (fill + denom computes the ratio automatically)
ggplot(map) +
  geom_district(aes(fill = ndv, group = cd_2020, denom = ndv + nrv)) +
  scale_fill_party_c() +
  theme_map()
```

### Key Aesthetics

| Aesthetic | Description |
|-----------|-------------|
| `group` | District assignment column (required) |
| `fill` | Numerator for fill; if `denom` is provided, computes `fill/denom` |
| `denom` | Denominator for ratio fill (optional) |
| `color` | Border color (default: black) |
| `linewidth` | Border width (default: 0.3) |

### Auto-coloring Adjacent Districts

`ggredist` uses `adj_color()` from the `adj` package to automatically assign colors so adjacent districts differ. Pass `color_adj = TRUE` (or use the `adj` from the map):

```r
ggplot(map) +
  geom_district(aes(fill = adj_color(map))) +
  scale_fill_distiller() +
  theme_map()
```

## District Labels

Add district numbers or names at district centroids:

```r
ggplot(map) +
  geom_district(aes(fill = ndv / (ndv + nrv), group = cd_2020,
                    denom = ndv + nrv)) +
  geom_district_text(aes(label = cd_2020), size = 3, color = 'white') +
  scale_fill_party_c() +
  theme_map()
```

## Place Labels

Overlay Census-designated places (cities, towns) for geographic context:

```r
ggplot(map) +
  geom_district(aes(fill = cd_2020, group = cd_2020)) +
  geom_places(state = 'MD', size = 2) +   # 'MD' or FIPS code
  theme_map()
```

## Color Scales

### Partisan Scales

```r
scale_fill_party_c(midpoint = 0.5, limits = c(0, 1))   # continuous red-white-blue
scale_fill_party_b(breaks = c(0.4, 0.5, 0.6))          # binned version
scale_color_party_c()                                    # for color= aesthetic
```

The midpoint (default 0.5) maps to white; values above are blue (Democratic), below are red (Republican).

### Cartographic Palettes

Discrete palettes for coloring districts by number:

```r
scale_fill_penn82()         # Penn82 atlas palette
scale_fill_natgeo()         # National Geographic style
scale_fill_randmcnally()    # Rand McNally atlas palette
scale_fill_fivethirtyeight() # FiveThirtyEight electoral palette
scale_fill_wiki()           # Wikipedia map palette
```

## Clean Map Theme

```r
theme_map()    # removes axes, gridlines, background; keeps legend
```

Combine with `coord_sf(datum = NA)` to suppress the lat/lon grid:

```r
ggplot(map) +
  geom_district(aes(fill = cd_2020, group = cd_2020)) +
  scale_fill_penn82() +
  theme_map() +
  coord_sf(datum = NA)
```

## Visualizing the Ensemble

### Distribution Plots

Use base `redist` functions to plot metric distributions:

```r
# Histogram of efficiency gap with enacted plan highlighted
hist(plans, part_egap(pl(), ndv, nrv))

# Box/jitter plot of district-level Polsby-Popper
plot(plans, comp_polsby(pl(), map, perim_df), style = 'box')

# Scatter plot of two metrics across plans
plot(plans, x = part_egap(pl(), ndv, nrv),
            y = splits_admin(pl(), map, map$county))
```

### Mapping a Specific Plan

```r
# Extract one plan's assignments into the map sf
one_plan <- get_plans_matrix(plans)[, 1]   # plan 1, 0-indexed
map$district <- one_plan + 1               # 1-indexed

ggplot(map) +
  geom_district(aes(fill = ndv / (ndv + nrv), group = district,
                    denom = ndv + nrv)) +
  scale_fill_party_c() +
  theme_map()
```

### Grid of Plans

```r
library(patchwork)

# Map multiple plans side by side
plot_plan <- function(k) {
  map$district <- get_plans_matrix(plans)[, k] + 1
  ggplot(map) +
    geom_district(aes(fill = district, group = district)) +
    scale_fill_penn82() +
    theme_map() +
    labs(title = paste('Plan', k))
}

plot_plan(1) + plot_plan(2) + plot_plan(3) + plot_plan(4)
```

