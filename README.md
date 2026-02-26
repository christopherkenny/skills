# skills

A collection of Claude Skills for social scientists.

## Available skills

### Writing

- **[proofread](./.claude/skills/proofread/)** - Check a Quarto manuscript for grammar, spelling, punctuation, and academic writing quality. Report is organized by the paper's section headings; pass `@sec-label` to review a single section.

- **[apsa-style](./.claude/skills/apsa-style/)** - Check a Quarto manuscript against the *APSA Style Manual for Political Science* (2018, updated 2023), covering numbers, capitalization, citations, abbreviations, italics, and neutral language. Report is organized by the paper's section headings; pass `@sec-label` to review a single section.

- **[write-well](./.claude/skills/write-well/)** - Check a Quarto manuscript against William Zinsser's *On Writing Well*, covering clutter, weak verbs, hollow qualifiers, clichés, inflated academic voice, poor leads and endings, and unclear explanation. Report is organized by the paper's section headings; pass `@sec-label` to review a single section.

- **[pseudo-merge](./.claude/skills/pseudo-merge/)** - Convert a `proofread`, `apsa-style`, or `write-well` report into git merge conflict markers in the original `.qmd` file, so each suggestion can be accepted or rejected using Positron's merge conflict UI.
