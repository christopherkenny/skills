# R analyst

## Analysis posture

- Correctness before cleverness or volume. Code should work on real inputs, not just toy cases or happy-path examples.
- Do not declare an analysis done while the script, report, model, figure, table, or workflow still fails in practice.
- Stay inside the requested analysis. Do not invent extra scope or wander into adjacent tasks.
- When reimplementing an existing method, compare carefully against the reference and identify meaningful behavioral differences.
- Prefer readable, explicit analysis steps over dense abstractions that hide what the analysis is doing.

## Reproducible workflows

- Keep portable entrypoints, such as a top-level `run.R`, when the workflow should be runnable directly.
- Use `here::here()` for file paths in analysis scripts. Never use `setwd()`.
- Call `set.seed()` before any randomness.
- If a script queries an API or external source, save the output rather than forcing every run to recompute it.

## Style

- Write idiomatic R. Prefer well-tested tidyverse, r-lib, and Posit ecosystem functions over custom wrappers.
- Use the native `|>` pipe. Do not use `%>%` or import `magrittr`.
- Use single quotes for strings.
- Always use braces on `if` statements, even for single-line bodies.
- Single-line anonymous functions use `\(x) x + 1`. Multi-line anonymous functions use `function(x) { ... }` with braces.
- Use `<-` for assignment throughout.
- Use `snake_case` for all variable and column names.
- Use `seq_len(n)`, not `1:n`. `1:n` breaks when `n = 0`.
- Use integer literals with the `L` suffix where the value is logically an integer (`1L`, `8L`).
- Use `glue::glue()` for string construction. Avoid `sprintf()` unless there is a clear formatting reason.
- Use `glue::glue_sql()` when constructing SQL strings.
- Use `fs` for file system operations in packages or when path manipulation is nontrivial.
- Avoid deprecated tidyverse functions: `recode()`, `transmute()`, and purrr's `_dfr`/`_dfc` variants. Use `imap() + list_rbind()` instead of `imap_dfr()`.
- Use `anyNA(x)` not `any(is.na(x))`. Use `nlevels(x)` not `length(levels(x))`.
- Never use the `.by` arugment of `summarize()` as this obfuscates the grouping. Always use explicit `group_by()` beforehand and handle `.groups` inside `summarize()`.

## Functions and abstractions

- Use `match.arg()` with a character vector default for arguments that select between named options. Booleans are for flags that modify a single algorithm, not for choosing between algorithms.
- Parameter names should mirror base R conventions: `method`, not `algorithm_mode`.
- Follow a consistent verb-object naming pattern when writing helpers.
- Internal functions do not need pseudo-private prefixes like `.pkg_*`. Undocumented is sufficient.
- Name files for the primary function or responsibility they contain.
- Put nontrivial shared helpers in clearly named utility files.
- Do not hide an entire analysis behind layers of helpers when the steps matter.
- Do not invent wrappers whose main effect is hiding real behavior or swallowing errors.
- Remove unreachable branches and duplicate functions.

## Null, NA, and missing values

- Use base R `%||%` for null fallbacks.
- `%||%` is not NA-coalescing: `NA %||% 0.05` returns `NA`. Guard NAs separately.

## Messages and errors

- Use `cli::cli_warn()` and `cli::cli_inform()` instead of `warning()`, `message()`, or `cat()` for user-facing output.
- `cli_warn()` is for actual problems only. Use `cli_inform()` for status updates.
- Add `.frequency = 'once'` to `cli_inform()` calls inside loops.
- Do not use wrappers that suppress errors needed to diagnose failed analyses.

## Data, SQL, and performance

- Keep as much work as possible in SQL rather than materializing data into R.
- Treat `DBI::dbGetQuery()`, `collect()`, and tibble construction from SQL output as materialization points that deserve scrutiny.
- If R immediately groups, counts, filters, or joins rows that came from SQL, that work probably belongs in SQL.
- Avoid unnecessary materialization, copying, or conversion of data.
- Prefer vectorized or backend-aware operations when working with large data.
- Keep performance fixes grounded in measurement, not speculation.
- Respect the native data structure being used, such as tibbles, `sf` objects, database tables, or package-specific objects.

## Modeling and summaries

- Models such as `glm()` should run on very small summary tables where possible, not full data pulls.

## ggplot2

- Do not put `aes()` inside the initial `ggplot()` call.
- Reshape data with `pivot_*()` before plotting rather than mapping awkwardly in `aes()`.
- Use facets when comparing related quantities instead of awkward rescaling.
- Use `scale_*()` for axis and legend labels. `labs()` is for titles and subtitles.
- Keep themes and scales minimal unless they serve a clear purpose.
- In exported package plot functions, do not set a default theme or call `theme_minimal()`. That is the user's prerogative.
- Do not hardcode aesthetic values that override ggplot2 defaults without reason.
- Extract repeated theme code into a named `theme_*()` helper rather than duplicating it.

## Script execution

- On Windows, do not use `Rscript -e "..."` for multi-line or complex R code. It segfaults. Write the code to a `.R` file and run `Rscript path/to/file.R` instead.
- Simple one-liners (`Rscript -e "devtools::test()"`) are fine.

## Reports, tables, and prose

- README and vignette examples should show real workflows, not fake data or function listings.
- Markdown prose should use one sentence per line, with line breaks only at sentence boundaries.

## Package-specific fallback

- If the task is inside an R package, keep `DESCRIPTION` clean. `Imports` are for packages used unconditionally. `Suggests` are for packages used only in tests, examples, or documentation.
- Once a package is in `Imports`, remove `requireNamespace()` guards for it. They are dead code.
- Do not list base or recommended packages such as `stats` or `utils` in `DESCRIPTION` unless there is a real reason.
- Use `devtools::check()` rather than raw commands that leave `Rcheck` directories behind.
- If a package is not public yet, prefer cleaning up the API directly over adding compatibility shims.
- Write individual roxygen blocks per function. Do not use `@describeIn` or `@name` group patterns.
- Do not use `@inheritParams`. Write the relevant parameter documentation explicitly for each function.
- Each exported function should have its own title, `@description`, `@param`, `@return`, and useful runnable `@examples`.
- Never use `\dontrun{}` or `\donttest{}` in examples. Use `tempfile()`, small real examples, and explicit `requireNamespace()` guards for optional packages.
- Keep roxygen, pkgdown, NEWS, vignettes, and code in sync.
- In pre-release packages, avoid release-note framing such as "legacy", "new", "now", or "fixed". Describe current feature coverage instead.
- Tests should verify package behavior, not basic facts about R or fixture data.
- Do not put `expect_*()` calls inside loops.
- Consolidate near-duplicate tests.
- Do not clutter tests with explicit default arguments when those are already the defaults.
- Put shared test helpers in a common test setup file instead of copying them around.
- Add regression tests for real failure modes and edge cases. Do not add regression tests for code that was removed. That is not a regression.
- Use `skip_if()` when a test depends on a suggested package.
- Delete tests that add runtime without testing anything meaningful.

## Comments

- Comments explain why: a constraint, an invariant, an analytic decision, or a workaround.
- Do not narrate what the code already says.
- Keep comments that explain algorithmic reasoning or how code maps to a reference implementation.
- In analysis scripts, use `# Section Name ----` headers to delineate top-level sections. That is the maximum decoration; no further nesting or embellishment.
- In package code, do not use section headers at all.
