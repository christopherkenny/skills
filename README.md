# skills

A collection of Claude Skills for social scientists.
This is a work-in-progress.

I am still of the opinion that we should not have LLMs writing for us.
However, there are many basic tasks that are tedious and time consuming relative to their research value.
Things like proofreading, checking style guide compliance, or analyzing for bad prose are all great writing tasks for a copyeditor named Claude.
The below writing tasks all take the same approach: they create a report on your text with details about how to "fix" it.
If you like what you see and are using Positron or VSCode, you can use the `pseudo-merge` skill to make the suggests clickable in your document, rather than copy-pasting the ones you like.

For typsetting, I prefer [Quarto](https://quarto.org/) above all else and [Typst](https://typst.app) to [LaTeX](https://www.latex-project.org/).
To this end, there are tasks for cleaning up Quarto docs to avoid old LaTeX syntax where Quarto has a better approach.
The `latex-to-typst` skill has distilled Typst documentation to capture the necessary LaTeX to Typst conversions and does a pretty good job based on my testing.
Please open an issue if you use these and find an example that does not translate faithfully.
Remember that Typst is much more powerful, but LaTeX has a longer history, so there may be cases that need more examples or prompting.

## Available skills

### Writing

- **[proofread](./.claude/skills/proofread/)** - Check a Quarto manuscript for grammar, spelling, punctuation, and academic writing quality. Report is organized by the paper's section headings; pass `@sec-label` to review a single section.

- **[apsa-style](./.claude/skills/apsa-style/)** - Check a Quarto manuscript against the *APSA Style Manual for Political Science* (2018, updated 2023), covering numbers, capitalization, citations, abbreviations, italics, and neutral language. Report is organized by the paper's section headings; pass `@sec-label` to review a single section.

- **[write-well](./.claude/skills/write-well/)** - Check a Quarto manuscript against William Zinsser's *On Writing Well*, covering clutter, weak verbs, hollow qualifiers, clichés, inflated academic voice, poor leads and endings, and unclear explanation. Report is organized by the paper's section headings; pass `@sec-label` to review a single section.

- **[pseudo-merge](./.claude/skills/pseudo-merge/)** - Convert a `proofread`, `apsa-style`, or `write-well` report into git merge conflict markers in the original `.qmd` file, so each suggestion can be accepted or rejected using Positron's merge conflict UI.

### Typesetting

- **[latex-to-typst](./.claude/skills/latex-to-typst/)** - Translate a LaTeX article (`.tex`) to Typst (`.typ`). Covers document structure, text formatting, page layout, sectioning, lists, math equations and symbols, figures, tables, bibliography, cross-references, TikZ-to-CeTZ diagrams, and code blocks. Includes comprehensive LaTeX-to-Typst symbol mapping tables.

- **[quarto-clean](./.claude/skills/quarto-clean/)** - Clean a Quarto manuscript (`.qmd`) by replacing LaTeX holdovers with proper Quarto/Pandoc syntax, so the document renders correctly to HTML, Word, and Typst PDF. Covers citations (`\citep` → `[@key]`), cross-references (`\ref` → `@label`), figures, tables, text formatting, inline math, hyperlinks, footnotes, list environments, section headings, and R Markdown chunk options.

- **[markdown-unwrap](./.claude/skills/markdown-unwrap/)** - Reflow a hard-wrapped markdown or Quarto file so that each sentence occupies its own line. Joins mid-sentence line breaks and re-breaks at sentence boundaries, while preserving blank lines, headings, fenced code blocks, block quotes, and list items.


## License

These are licensed under the [MIT license](LICENSE).

## Reminder

Naturally, Claude skills are written with help from Claude by giving it examples and starter prompts.
The resulting skills are the product of iterating with Claude Code until I'm happy with the output.
You remain the researcher; don't delegate everything to a predictive text model.
Always use human judgement when deciding if the suggestions remain truthful and accurate descriptions of the underlying research.
