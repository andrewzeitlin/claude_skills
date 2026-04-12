---
name: latex
description: Conventions for setting up LaTeX documents — preamble, tables, math, figures, appendices, bibliography. Use when creating a new .tex document, adding or reviewing a preamble, or reviewing table/figure formatting in an existing document.
---

# /latex — LaTeX Document Conventions

Use these conventions when creating or editing LaTeX documents. For BibTeX
entries and `.bib` maintenance, see the separate **`/bibtex`** skill.

## Document class

Default to `\documentclass[11pt]{article}`. Use `scrartcl` only if the project
already uses a KOMA-Script class.

## Preamble template (article)

Drop the blocks you don't need. The grouping and comment style below is the
one to follow — short section headers in `%---  name  ---%` or `%  name` form,
packages that always travel together kept together.

```latex
\documentclass[11pt]{article}

%---  page setup  ---%
\usepackage[letterpaper, margin=1.125in]{geometry}
\usepackage{titling}                      % fine-grained title control
\usepackage{setspace}                     % \onehalfspacing, \doublespacing

%---  math  ---%
\usepackage{amsmath}
\usepackage{amssymb}
\usepackage{amsthm}
\DeclareMathOperator{\E}{E}
\DeclareMathOperator{\Var}{Var}
\DeclareMathOperator{\Cov}{Cov}

%---  figures  ---%
\usepackage{graphicx}
\usepackage{float}                        % enables [H] placement
\usepackage[position=bottom]{subfig}      % subfig with bottom-captioned panels
\usepackage{tikz}
\usetikzlibrary{backgrounds,fit,positioning}

%---  tables  ---%
\usepackage{booktabs}                     % \toprule \midrule \bottomrule
\usepackage{multirow}
\usepackage{makecell}                     % \makecell[]{line1\\line2} inside cells
\usepackage{array}                        % \newcolumntype
\usepackage{siunitx}                      % S-columns for decimal alignment
\sisetup{
  table-align-text-post = false,
  parse-numbers         = false,
  detect-all            % inherit surrounding font
}
\usepackage{longtable}
\usepackage{rotating}                     % sidewaystable / sideways figures
\usepackage{pdflscape}                    % landscape pages in portrait PDF
\usepackage{dcolumn}                      % older decimal-alignment (rarely needed if using siunitx)
\newcolumntype{d}[1]{D{.}{.}{#1}}
\newcolumntype{C}[1]{>{\centering\arraybackslash}b{#1}}   % wrap-centred column

%---  lists  ---%
\usepackage{enumitem}                     % list spacing / custom labels

%---  cross-references  ---%
\usepackage{hyperref}                     % always LAST among ref packages
\hypersetup{colorlinks=true, linkcolor=blue!50!black,
            citecolor=blue!50!black, urlcolor=blue!50!black}

%---  bibliography  ---%
\usepackage[authoryear, sort&compress, round]{natbib}
\bibliographystyle{aer}

%---  editing notes  ---%
\usepackage[textwidth=0.9in]{todonotes}
\setuptodonotes{color=green!20, size=footnotesize}

%---  figure and table notes  ---%
\newcommand{\fignote}[1]{%
  \par\vspace{0.3em}%
  \begin{minipage}{\linewidth}\footnotesize\textit{Notes:} #1\end{minipage}%
  \vspace{1em}}
\newcommand{\tblnote}[1]{%
  \nopagebreak\par\vspace{0.3em}%
  \begin{minipage}{\linewidth}\footnotesize\textit{Notes:} #1\end{minipage}%
  \vspace{1em}}
```

Only include what's used. A short, tidy preamble is better than a dumping
ground of commented-out packages.

## Tables — `booktabs` + `siunitx`

**Always** use `booktabs` rules. Never `\hline`.

```latex
\begin{tabular}{l S[table-format=2.3] S[table-format=2.3]}
\toprule
              & {OLS}   & {Poisson} \\
\midrule
Treatment     &  0.142  &  0.153    \\
              & (0.031) & (0.034)   \\
Control mean  &  1.240  &  1.240    \\
\bottomrule
\end{tabular}
```

- **Decimal alignment**: use `siunitx` `S` columns with `table-format=X.Y`.
  `X` is digits to the left of the decimal, `Y` to the right. Headings and
  non-numeric cells in an `S` column must be wrapped in `{...}`.
- `parse-numbers=false` lets cells contain pre-formatted numbers or stars
  for significance, at the cost of automatic alignment — the S-column still
  aligns on the explicit decimal point in the source.
- Use `\addlinespace` for visual grouping within a single table rather than
  extra rules.
- **Avoid vertical rules.** If a reader needs a separator, reach for
  `\cmidrule(lr){2-4}` or a grouped `\addlinespace`.

### Captions and notes
- Captions go **above** tables (set once with `\captionsetup[table]{position=top}`
  or use `fig-cap-location: top` in Quarto).
- Keep captions to one sentence. Put explanatory detail in a `\tblnote{...}`
  immediately after the tabular environment but before `\end{table}`.
- For long/landscape tables, `sidewaystable` (from `rotating`) is usually
  enough; use `pdflscape` only when you also need the page itself rotated.

## Figures

- `\usepackage{graphicx}`; `\usepackage{float}` if you need `[H]` to force
  in-place placement.
- Captions go **above** figures (`\captionsetup[figure]{position=top}`) when
  the venue allows — otherwise below.
- Put detail in a `\fignote{...}` directly after the `\includegraphics` line,
  inside the float environment.

## Math

- `amsmath`, `amssymb`, `amsthm` are the minimum.
- Use `\DeclareMathOperator{\OpName}{OpName}` rather than redefining
  primitives like `\exp`, `\log`, `\Pr`. Collect operator declarations in
  one block near the top of the preamble.
- Define project-specific shortcuts (e.g. `\E`, `\Var`, `\Cov`) as operators
  or via `\newcommand` — don't redefine existing math primitives.

## Bibliography

- **Default to `natbib` with `aer` style**:
  `\usepackage[authoryear, sort&compress, round]{natbib}` and
  `\bibliographystyle{aer}`.
- Use `\citet{}` for textual citations (`Author et al. (2022)`) and
  `\citep{}` for parenthetical citations (`(Author et al., 2022)`).
- For two bibliographies (e.g. main paper + online appendix), use
  `multibib`:
  ```latex
  \usepackage{multibib}
  \newcites{App}{Online Appendix References}
  \bibliographystyle{aer}
  \bibliographystyleApp{aer}
  ```
- For `.bib` file conventions — entry types, field order, author formatting,
  citation keys — **see the separate `/bibtex` skill.** That skill is the
  authoritative source for anything inside a `.bib` file.

## Appendices

Appendix top-level sections should display as **"Appendix A"**, **"Appendix B"**,
etc., but subsections should remain **"A.1"**, **"A.2"** (no "Appendix" prefix).
Figures and tables are numbered within each appendix and counters reset at each
new appendix.

```latex
\cleardoublepage
\bibliography{refs}

\cleardoublepage
\appendix
%  Display section as "Appendix A", subsections as "A.1", "A.2", ...
%  Tables and figures numbered within appendix (A1, A2, ...).
\renewcommand{\thesection}{\Alph{section}}
\renewcommand{\thetable}{\thesection\arabic{table}}
\renewcommand{\thefigure}{\thesection\arabic{figure}}
\makeatletter
\renewcommand\@seccntformat[1]{%
  \@ifundefined{my@appendix@#1}{}{Appendix~}%
  \csname the#1\endcsname\quad}
\@namedef{my@appendix@section}{}
\makeatother
\setcounter{table}{0}
\setcounter{figure}{0}

\section{First appendix title}
\label{app:first}
...

\cleardoublepage
\setcounter{table}{0}
\setcounter{figure}{0}
\section{Second appendix title}
\label{app:second}
...
```

Notes:
- Put a `\cleardoublepage` **between** appendices and reset the `table` /
  `figure` counters so numbering restarts as B1, B2, …
- `\thesection` is just `\Alph{section}` — the "Appendix~" prefix is added
  only to the displayed section heading, via `\@seccntformat`. This keeps
  in-text references clean: write `Appendix~\ref{app:first}` and it
  renders as "Appendix A", not "Appendix Appendix A".
- The `\@ifundefined` guard ensures the prefix is applied to sections only,
  not subsections. (If you later add a named sub-environment that should
  also get the prefix, `\@namedef{my@appendix@<envname>}{}` adds it to the
  allow-list.)
- If the document has a single appendix, still use `\@ifundefined` +
  `\@namedef` — that keeps subsections like "A.1" unprefixed.

## Editing-phase packages

- `todonotes` for inline editing marks. Set a small textwidth
  (`[textwidth=0.9in]`) and a muted colour so todos don't dominate the
  margin. Remove all `\todo{...}` calls before a ship-ready build.
- `comment` if you need large commented-out blocks (safer than `%` on
  every line).

## Spacing and page setup

- `titling` for fine-grained title positioning (`\setlength{\droptitle}{-6em}`
  to pull the title closer to the top).
- `setspace` for line spacing: `\onehalfspacing` for drafts, `\singlespacing`
  for tables / footnotes / abstract, `\doublespacing` only when a venue
  requires it.
- Standard margins: `margin=1.125in` (conference / working-paper) or
  `margin=1.5in` (double-spaced manuscript style).

## Don't

- Don't use `\hline` — use `booktabs` rules.
- Don't use `\begin{center}...\end{center}` inside floats — use `\centering`.
- Don't load `subfigure` (obsolete) — use `subfig` or `subcaption`.
- Don't load both `subfig` and `subcaption` in the same document.
- Don't load `hyperref` before `cleveref` / `natbib` — `hyperref` goes last.
- Don't redefine `\exp`, `\log`, `\Pr`, etc. — use `\DeclareMathOperator`.
- Don't use `$ ... $` for display math — use `\[ ... \]` or an
  `equation`/`align` environment.

## Cross-references

- `\ref{}` for numbered things, `\eqref{}` for equations, `\citet{}` /
  `\citep{}` for citations.
- Label prefixes by type: `sec:`, `subsec:`, `app:`, `tbl:`, `fig:`, `eq:`,
  `assn:`, `thm:`. One consistent prefix scheme makes `\label{}`/`\ref{}`
  pairs scannable.

## Related skills

- **`/bibtex`** — conventions for `.bib` entries (entry types, field order,
  author formatting, citation keys, DOI handling). Use this skill whenever
  editing or creating `.bib` entries.
- **`/quarto`** — conventions for `.qmd` documents rendered to PDF or Beamer.
  Those conventions mirror the LaTeX conventions here but are expressed
  through the Quarto YAML header rather than a preamble.
- **`/makefile`** — conventions for Makefiles that build LaTeX and Quarto
  documents.
