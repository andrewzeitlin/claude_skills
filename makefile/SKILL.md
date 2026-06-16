---
name: makefile
description: Conventions for writing Makefiles in academic research projects. Use when creating a new Makefile, adding targets, or reviewing an existing Makefile.
---

# /makefile — Makefile Conventions

Follow these conventions when creating or editing Makefiles in research projects.

## Structure

- Start with a comment: `#  Makefile for <description>`
- Primary target is the output file (e.g., `Document.pdf`, `analysis.arrow`)
- Use `.PHONY` for convenience aliases (`paper`, `all`, `clean`, etc.)
- **Follow the DAG, not a flat list.** A `.PHONY` target should depend on
  the most-downstream real file (e.g., the PDF), not enumerate every
  intermediate output. Make will chase the dependency chain automatically.
  Only list intermediates as prerequisites of the rules that actually
  consume them.

### Rule ordering (defined by file position — avoid the words "top-down" / "bottom-up")

The terms "top-down" and "bottom-up" are ambiguous for Makefiles because they
collide on two different axes — *reading direction* (down the file) versus
*DAG-arrow direction* (ingredients → outputs). Do not use them. Define the
layout strictly by **line position** instead:

- **Final outputs go near the top of the file** (lowest line numbers, just
  under the `.PHONY` header). **Raw ingredients / upstream inputs go near the
  bottom** (highest line numbers).
- Equivalently: **each rule depends on rules written *below* it.** The DAG's
  dependency arrows point *upward* as line numbers decrease. The most
  downstream artifact (the final PDF / `.arrow`) is the first real rule;
  primitives are last.
- Reading the file from line 1 downward therefore presents the goal first,
  then progressively more basic inputs. This matches the GNU Make manual's own
  examples (the final executable rule first, object-file rules below) and makes
  the default goal fall naturally at the top.

Note: this is functionally irrelevant to Make — textual rule order never
affects the build (the only exception: the first non-`.PHONY` target is the
default goal). It is purely a readability convention. The authoring friction
(you often compose against data flow, knowing ingredients before the final
rule) is a one-time write cost and is accepted in favor of read clarity.

This is the convention the user refers to as "bottom-up" (meaning the DAG
flows *up* the file). Keep using exactly this layout.

## Prerequisites

List every prerequisite directly on the target rule using `\` continuation.
**Avoid using Make variables** (`FOO = ...`) to group prerequisites, even when
the list is long. Repeat a file in multiple targets rather than factoring it
into a variable. 

Break the line immediately after the target colon, then indent each
prerequisite with **two tabs**, and list one prerequisite per line for readability:

```makefile
main_paper.pdf: \
		main_paper.tex \
		preamble.tex \
		appendix.tex \
		references.bib \
		tables/balance_tables/balance_table.tex \
		figures/results_visuals/main_coefficients.jpg
	latexmk -pdf -interaction=nonstopmode -halt-on-error main_paper.tex
```

For the `.qmd` source, list it first among the prerequisites.

## Recipes

Use **explicit script names** in recipes — never automatic variables like
`$<`, `$@`, or `$^`. A reader should see the exact command without decoding
Make syntax.

| Output type | Recipe |
|------------|--------|
| PDF via Quarto | `quarto render File.qmd --to pdf` |
| Beamer via Quarto | `quarto render File.qmd --to beamer` |
| PDF via LaTeX | `latexmk -pdf -interaction=nonstopmode -halt-on-error File.tex` |
| R script | `Rscript path/to/script.R` |

## Indentation

- **Recipes**: one tab (Make requirement).
- **Continuation prerequisites**: two tabs for visual alignment.

## Comments

Use `#  ` (hash + two spaces) for section comments that group related targets:

```makefile
#  Import source data
#  Draw feasible assignments
#  Create neighborhood-level strata
```

## Phony targets

Provide short convenience aliases:

```makefile
.PHONY: all paper clean
all: kenya uganda
paper: main_paper.pdf
```

## Clean target

For LaTeX projects, use `latexmk -C` to clean:

```makefile
clean:
	latexmk -C main_paper.tex
```

For R/data projects, a `clean` target is optional — only include it if
intermediate files are genuinely disposable.

## Copying exhibits into a submodule (e.g., Overleaf)

When the paper lives in a git submodule (Overleaf), **do not batch all `cp`
commands into the LaTeX build recipe**. Instead, make each submodule copy a
separate target with its own `cp` recipe. The LaTeX target then lists the
submodule copies as prerequisites — Make only re-copies when the source
changes, and the recipe stays clean:

```makefile
paper: 4_paper/main_paper.pdf

4_paper/main_paper.pdf: \
		4_paper/main_paper.tex \
		4_paper/figures/coefficient_plot.pdf \
		4_paper/figures/balance_table.tex
	cd 4_paper && latexmk -pdf -interaction=nonstopmode -halt-on-error main_paper.tex

4_paper/figures/coefficient_plot.pdf: 2_build/figures/coefficient_plot.pdf
	cp 2_build/figures/coefficient_plot.pdf 4_paper/figures/coefficient_plot.pdf

4_paper/figures/balance_table.tex: 2_build/tables/balance_table.tex
	cp 2_build/tables/balance_table.tex 4_paper/figures/balance_table.tex
```

## Cross-directory dependencies

Use relative paths for dependencies in sibling directories:

```makefile
2_build/output.arrow: \
		../Registration/2_build/eligibles_uganda.arrow \
		0_scripts/4.0.PotentialDraws.Uganda.R
	Rscript 0_scripts/4.0.PotentialDraws.Uganda.R
```

## Full example (LaTeX paper)

```makefile
#  Makefile for main paper (LaTeX build of main_paper.tex)

.PHONY: paper clean

paper: main_paper.pdf

main_paper.pdf: \
		main_paper.tex \
		preamble.tex \
		appendix.tex \
		references.bib \
		tables/balance_tables/balance_table.tex \
		tables/results_tables/effects_main.tex \
		figures/results_visuals/main_coefficients.jpg \
		figures/design_visuals/sample_division.jpg
	latexmk -pdf -interaction=nonstopmode -halt-on-error main_paper.tex

clean:
	latexmk -C main_paper.tex
```

## Full example (R data pipeline)

```makefile
#  Makefile for sampling pipeline

.PHONY: all kenya uganda
all: kenya uganda
kenya: 2_build/potential_assignments_kenya.arrow
uganda: 2_build/potential_assignments_uganda.arrow

#  Draw feasible assignments
2_build/potential_assignments_uganda.arrow: \
		../Registration/2_build/eligibles_uganda.arrow \
		0_scripts/lib/groupings.R \
		0_scripts/4.0.PotentialDraws.Uganda.R
	Rscript 0_scripts/4.0.PotentialDraws.Uganda.R

#  Create neighborhood-level strata
2_build/neighborhood_strata_uganda.csv: \
		0_scripts/2.2.Strata.Neighborhoods.Uganda.R \
		1_source/neighborhood_frame_uganda.csv
	Rscript 0_scripts/2.2.Strata.Neighborhoods.Uganda.R
```

## Grouped targets (`&:`)

When a single script produces multiple output files, prefer structuring the
Makefile so each target has its own rule (one output per rule). This avoids
the multi-target problem entirely and keeps the DAG clean.

When that is not practical (e.g., one R script that writes 5 files in a
single run), use **grouped targets** (`&:` instead of `:`) so Make knows
the recipe produces all listed outputs together. Without `&:`, Make treats
a multi-output rule as N separate rules sharing a recipe and may show
duplicate invocations in `--dry-run`:

```makefile
# Good: grouped target — recipe runs once for all outputs
2_build/table_a.tex \
2_build/table_b.tex &: \
		0_scripts/make_tables.R \
		1_source/data.csv
	Rscript 0_scripts/make_tables.R
```

Requires GNU Make 4.3+. Check with `make --version`.

## Don't

- Don't use Make variables (`TABLES = ...`, `FIGS = ...`) to group prerequisites.
- Don't use automatic variables (`$<`, `$@`, `$^`) in recipes.
- Don't use `$(shell ...)` or computed paths.
- Don't add unnecessary complexity — a Makefile is documentation of the build
  graph, not a program.

## Related skills

- **`/latex`** — conventions for `.tex` files and preambles.
- **`/quarto`** — conventions for `.qmd` documents rendered to PDF or Beamer.