---
name: quarto
description: Conventions for Quarto (.qmd) documents — YAML headers, figure/table captions and notes, appendices, R setup chunks. Use when creating or editing .qmd files.
---

# /quarto — Quarto Document Conventions

Follow these conventions when creating or editing Quarto (`.qmd`) documents.

## PDF document YAML header

```yaml
---
title: "Title"
author: "<Your Name>"
date: today
date-format: long
format:
  pdf:
    number-sections: true
    fig-cap-location: top
    include-in-header:
      text: |
        \usepackage{booktabs}
        \usepackage{siunitx}
        \sisetup{table-align-text-post=false}
        \sisetup{parse-numbers=false}
        \newcommand{\fignote}[1]{\par\vspace{0.3em}\begin{minipage}{\linewidth}\footnotesize\textit{Notes:} #1\end{minipage}\vspace{1em}}
        \newcommand{\tblnote}[1]{\nopagebreak\par\vspace{0.3em}\begin{minipage}{\linewidth}\footnotesize\textit{Notes:} #1\end{minipage}\vspace{1em}}
---
```

Always include `\fignote` and `\tblnote` commands in the header — they are
used for figure and table notes throughout.

## Beamer presentation YAML header

```yaml
---
title: "Title"
author: "<Your Name>"
date: today
date-format: long
format:
  beamer:
    section-titles: true
    aspectratio: 1610
    include-in-header:
      text: |
        \usepackage{booktabs}
        \usepackage{multirow}
        \usepackage{siunitx}
        \sisetup{table-align-text-post=false}
        \sisetup{parse-numbers=false}
        \newcolumntype{C}[1]{>{\centering\arraybackslash}b{#1}}
        \usepackage{makecell}
        \usepackage{tikz}
---
```

## R setup chunk

Always use `include=FALSE` on the initial setup chunk:

````
```{r setup, include=FALSE}
library(here)
library(data.table)
library(knitr)
library(kableExtra)
library(ggplot2)
library(scales)
```
````

Common setup libraries: `here`, `data.table`, `fst`, `knitr`, `kableExtra`,
`ggplot2`, `scales`. Only load what the document actually uses.

## Figure captions and notes

**Keep captions short (one sentence).** Move explanatory details, definitions,
and methodological notes into `\fignote{}`.

### Chunk-generated figures (R output)

Use Quarto's `fig.cap=` chunk option for the short caption. Place
`\fignote{...}` on a line after the closing `` ``` ``:

````
```{r}
#| label: fig-main-results
#| fig-cap: "Treatment effects on primary outcomes."

ggplot(dt, aes(x = estimate, y = outcome)) + geom_point()
```

\fignote{Each point shows the ITT estimate from equation (1). Bars show
95\% confidence intervals. Standard errors clustered at the group level.}
````

Note: very tall figures may need `fig.height` reduced to keep the note on
the same page.

### Hand-written figures (TikZ, `\input{}`, raw LaTeX `figure` blocks)

When the figure is a hand-written `\begin{figure}...\end{figure}` block
(TikZ diagram, external `.tex` via `\input{}`, or raw LaTeX image), the
float can drift away from its anchor in the source. Place `\fignote{...}`
**inside** the figure environment, between `\label{}` and `\end{figure}`,
so the note travels with the float rather than being left behind.

For strict placement (so the float does not drift at all), load `float`
in the header:

```yaml
include-in-header:
  text: |
    \usepackage{float}
```

and use `[H]` instead of `[h]`:

```latex
\begin{figure}[H]
\centering
\begin{tikzpicture}
...
\end{tikzpicture}
\caption{Short one-sentence caption.}
\label{fig-mylabel}
\fignote{Longer explanatory notes. They sit inside the figure
environment so they stay with the float.}
\end{figure}
```

## Table captions and notes

Wrap each table in a Quarto div for cross-referencing and to keep the table,
caption, and note together as a single float:

````
::: {#tbl-balance}
```{r}
kb(dt)   # no caption argument
```

\tblnote{Sample restricted to respondents interviewed at both baseline and
endline. Column 1 reports control-group means.}

Balance on baseline characteristics.
:::
````

Rules:
- Do **not** pass `caption =` to `kb()` — the caption goes as the last text
  line before `:::`.
- Do **not** use `kableExtra::footnote()` with `threeparttable` — it constrains
  note width to the table and causes escaping issues.
- The `\tblnote` approach gives page-width justified notes with correct LaTeX
  rendering.

## Inputting pre-built .tex tables

When tables are produced by standalone R scripts (not inline R chunks), use
`\input{}` inside the Quarto div — the same wrapper pattern as for inline
tables, preserving cross-referencing, `\tblnote{}`, and the short caption:

````
::: {#tbl-balance}

\input{tables/balance_table.tex}

\tblnote{Sample restricted to respondents interviewed at both baseline and
endline. Column 1 reports control-group means.}

Balance on baseline characteristics.
:::
````

The `\input{}` path is relative to the `.qmd` file's directory. This pattern
ensures consistency: the same .tex file is used in both the Quarto summary
document and the LaTeX paper (via `\input{}` in both places).

The same approach works for TikZ diagrams produced by scripts — wrap the
`\input{}` in a `\begin{figure}...\end{figure}` environment with
`\caption{}`, `\label{}`, and `\fignote{}` (see the hand-written figures
subsection above for placement inside the float):

```latex
\begin{figure}[H]
\centering
\input{figures/policy_tree.tex}
\caption{Learned policy tree (depth 2).}
\label{fig-policy-tree}
\fignote{Interior nodes show splitting variable and threshold;
leaves show recommended treatment.}
\end{figure}
```

## The `kb()` helper

Always pass `format = "latex"` and `booktabs = TRUE` in `knitr::kable()` calls
(include in the `kb()` definition so all tables get it automatically). This
prevents Quarto's Pandoc pipeline from re-escaping LaTeX math in column
headers. With `escape = FALSE`, use standard R double-backslash escaping for
LaTeX commands (e.g., `"$\\pi_0$"`).

Do **not** use `kable_styling(latex_options = "hold_position")` — Quarto
manages float placement via the div.

## Appendices in PDF documents

Insert the following block between the main body and the first appendix
section:

```latex
\clearpage
\thispagestyle{empty}
\vspace*{0.25\textheight}
\begin{center}
{\Huge\bfseries\sffamily Appendices}
\end{center}
\clearpage

\appendix
\renewcommand{\sectionformat}{Appendix~\thesection\autodot\enskip}
\renewcommand{\thetable}{\thesection\arabic{table}}
\renewcommand{\thefigure}{\thesection\arabic{figure}}
\setcounter{table}{0}
\setcounter{figure}{0}
```

This produces:
- A standalone separator page with "Appendices" in large bold sans-serif.
- Section headings prefixed with "Appendix" (e.g. "Appendix A ...").
- Table and figure counters reset and prefixed with the appendix letter.

Note: `\sectionformat` is a KOMA-Script command (`scrartcl`/`scrreprt`). For
standard document classes, use
`\renewcommand{\thesection}{Appendix~\Alph{section}}` instead.

## R language conventions (within .qmd)

- Prefer `data.table` over tidyverse for data manipulation.
- Always use the `here` package with relative file paths from the project root.
- Chain data.table operations using either:
  - The magrittr pipe (`%>%`) with dot notation:
    `dt %>% .[condition] %>% .[, col := val]`
  - The base pipe (`|>`) with `_[` notation:
    `dt |> _[condition] |> _[, col := val]`

## File output from R chunks

Prefer open, interoperable formats:
- **Small tabular data** -> `.csv` (`fwrite` / `fread`)
- **Large tabular data** -> `.parquet` (`write_parquet` / `read_parquet`)
- **`.rds`** only for objects that cannot be represented as a flat table.

## Don't

- Don't use `kable(caption = ...)` — use the Quarto div pattern above.
- Don't use `kable_styling(latex_options = "hold_position")`.
- Don't use `kableExtra::footnote()` with `threeparttable = TRUE`.
- Don't put long explanatory text in the `fig-cap` or `tbl-cap` — use
  `\fignote{}` or `\tblnote{}` for detail.
- Don't use `$ ... $` for display math — use `$$ ... $$` or fenced environments.

## Related skills

- **`/latex`** — conventions for `.tex` documents and preambles.
- **`/bibtex`** — conventions for `.bib` entries and citation keys.
- **`/makefile`** — conventions for Makefiles that build these documents.