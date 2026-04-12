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
- Organize targets in **dependency order** — final outputs first, upstream
  inputs below — so a reader can follow the pipeline top-down.

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

## Don't

- Don't use Make variables (`TABLES = ...`, `FIGS = ...`) to group prerequisites.
- Don't use automatic variables (`$<`, `$@`, `$^`) in recipes.
- Don't use `$(shell ...)` or computed paths.
- Don't add unnecessary complexity — a Makefile is documentation of the build
  graph, not a program.

## Related skills

- **`/latex`** — conventions for `.tex` files and preambles.
- **`/quarto`** — conventions for `.qmd` documents rendered to PDF or Beamer.