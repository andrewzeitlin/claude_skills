---
name: bibtex
description: Conventions for creating and editing BibTeX entries. Use when adding references to .bib files or converting inline citations to proper \cite commands.
---

# /bibtex — BibTeX Conventions

Follow these rules when creating or editing BibTeX entries.

## Verification (HIGH PRIORITY)

**Never add a reference to a .bib file without first verifying it.** LLMs
hallucinate citation details — wrong years, wrong journals, wrong page numbers,
and sometimes entirely fabricated papers. Before incorporating any new entry:

1. Search Google Scholar via the `scholarly` Python package:

   ```bash
   python3 -c "
   from scholarly import scholarly
   results = scholarly.search_pubs('AUTHOR SURNAME TITLE KEYWORDS')
   pub = next(results)
   bib = pub['bib']
   print('Title:', bib.get('title'))
   print('Authors:', bib.get('author'))
   print('Year:', bib.get('pub_year'))
   print('Venue:', bib.get('venue'))
   print('Volume:', bib.get('volume'))
   print('Number:', bib.get('number'))
   print('Pages:', bib.get('pages'))
   # For DOI, check the pub_url or eprint_url
   print('URL:', pub.get('pub_url'))
   print('eprint:', pub.get('eprint_url'))
   "
   ```

   If `scholarly` is not installed: `pip install scholarly`

2. Confirm: author names, year, journal/outlet, volume, number, pages, and DOI.
   If `scholarly` does not return volume/pages/DOI, try fetching the publisher
   page URL with `WebFetch` to extract those fields.
3. If the search returns no match, flag the reference to the user — do not
   guess or fill in details from memory.

This applies to every new entry, even for well-known papers.

### Approved sources for verification

Two kinds of anchor are acceptable as ground truth:

1. **An approved `.bib` file** maintained by the user (lab-wide bibliographies,
   prior project bibs). If the entry already exists there, reuse the approved
   fields verbatim rather than re-deriving them.
2. **A credible citation source on the web**: the journal or publisher page,
   a DOI resolver, the issuing institution's working-paper repository, or
   Google Scholar via `scholarly`. **The source document itself** (a PDF
   stored in the project, an arXiv preprint, the journal landing page) is the
   strongest anchor — if the paper sits one folder away in the repo, open it
   before writing the entry.

Memory, plausibility, association, and "well-known paper" do **not** count
as verification.

### Never guess first names from initials

If a source provides only initials — e.g. `A. Mackintosh`, `Mackintosh, A.`,
or just `A.` in a co-author list — preserve the initial (with a period) in
the BibTeX `author` field. **Never expand initials into guessed full first
names from memory.** Filling in "Anna" because the surname looks like it
should go with one of the Annas you've seen is the same class of error as
fabricating a year or a journal: it produces wrong, attributable text in a
publishable document, and the mistake is easy to make and hard to catch.

To convert initials to full first names, verify them via one of the approved
sources above — the source document is usually fastest if it lives in the
project. If no approved source confirms the full first name, **keep the
initial**.

## Entry types

| Publication type | BibTeX type |
|-----------------|-------------|
| Journal article | `@article` |
| Working paper / preprint / report series | `@misc` |
| Book | `@book` |
| Book chapter | `@incollection` |

**Working papers** (World Bank, NBER, IZA, CEPR, etc.) use `@misc` with the
series name and number in the `howpublished` field.

## Field order and inclusion

The following fields must appear **first**, in this order. Remaining fields
(`volume`, `number`, `pages`, `month`, `doi`, etc.) follow after:

| `@article` | `@misc` (working paper) | `@book` | `@incollection` |
|-------------|------------------------|---------|-----------------|
| `title` | `title` | `title` | `title` |
| `author` | `author` | `author` | `author` |
| `year` | `year` | `year` | `year` |
| `journal` | `howpublished` | `publisher` | `booktitle` |
| `volume` | `month` | | `publisher` |
| `number` | `doi` | | |
| `pages` | | | |
| `month` | | | |
| `doi` | | | |

**Always omit:**
- `file`, `note`, `urldate`, `language`, `abstract`, `keywords`, `shorttitle`
- `series` (put series info in `howpublished` instead)
- `_eprint` fields

Templates:

```bibtex
@article{EggHauMigNieWal22ecta,
    title = {General equilibrium effects of cash transfers: Experimental evidence from {Kenya}},
    author = {Egger, Dennis and Johannes Haushofer and Edward Miguel and Paul Niehaus and Michael Walker},
    year = {2022},
    journal = {Econometrica},
    volume = {90},
    number = {6},
    pages = {2603--2643},
    doi = {10.3982/ECTA17945}
}

@misc{PapGonGolFri25wb,
    title = {Cash is queen: Local economy effects of transfers to women in {West} {Africa}},
    author = {Papineni, Sreelakshmi and Paula Gonzalez and Markus Goldstein and Jed Friedman},
    year = {2025},
    howpublished = {World Bank Policy Research Working Paper 11112},
    month = may
}
```

## Author formatting

The **first author** is listed as `Surname, Forenames` (inverted, for
alphabetical sorting). All subsequent authors are listed as
`Forenames Surname` (natural order). Authors are separated by `and`
(no commas between authors):

```bibtex
author = {Egger, Dennis and Johannes Haushofer and Edward Miguel and Paul Niehaus and Michael Walker},
```

For a single author: `author = {Lin, Winston},`

**Initials must include periods**: `Kenneth E.` not `Kenneth E`;
`Guido W.` not `Guido W`.

**No Unicode characters** in author names (or anywhere in the entry).
Use LaTeX equivalents: `Garc{\'i}a` not `García`, `{\"O}zler` not `Özler`,
`S{\'a}nchez` not `Sánchez`.

## Title capitalization

BibTeX styles control capitalization. Write titles in **sentence case** (only
the first word and proper nouns capitalized). Protect proper nouns that must
always be capitalized with braces:

```bibtex
title = {Cash is queen: Local economy effects of transfers to women in {West} {Africa}},
```

Do NOT use Zotero-style forced capitalization like `{Every} {Word} {Braced}`.
Only brace:
- Country and region names: `{Kenya}`, `{West} {Africa}`, `{U.S.}`
- Proper nouns: `{Bayesian}`, `{Fisher}`, `{Wald}`
- Acronyms: `{RCT}`, `{GE}`, `{SUTVA}`

## DOI field

- Every entry SHOULD have a `doi` field if one exists.
- **Never guess or fabricate a DOI.** Each article has a unique DOI — do not
  copy a DOI from one entry to another, even within the same journal or
  working paper series.
- If you don't know the DOI, omit the field entirely. Do not leave it blank.

## Citation keys

Keys use **CamelCase with three letters per author surname**, a two-digit year,
and a short abbreviation for the publication outlet.

Format: `AaaAaaAaa00outlet`

Examples:
- Egger, Haushofer, Miguel, Niehaus, Walker (2022) in Econometrica →
  `EggHauMigNieWal22ecta`
- Berger and Boos (1994) in JASA → `BerBoo94jasa`
- Chung and Romano (2013) in Annals of Statistics → `ChuRom13aos`
- Ding, Feller, Miratrix (2016) in JRSS-B → `DinFelMir16jrssb`
- Lin (2013) in Annals of Applied Statistics → `Lin13aoas`

Common outlet abbreviations: `ecta` (Econometrica), `jasa` (JASA),
`qje` (QJE), `aer` (AER), `res` (Review of Economic Studies),
`jrssb` (JRSS-B), `aos` (Annals of Statistics), `aoas` (Annals of
Applied Statistics), `wp` (working paper), `cup` (Cambridge UP).

For working papers without a journal, use a series abbreviation:
`PapGonGolFri25wb` (World Bank).

## Converting inline references

When converting plain-text citations to `\cite` commands:
- Parenthetical: `(Author et al., 2022)` → `\citep{key}`
- Textual: `Author et al. (2022)` → `\citet{key}`
- Multiple in one parenthetical: `(A, 2022; B, 2023)` → `\citep{key_a, key_b}`
- Do not convert a reference unless its bib key exists. If the key is missing,
  add the entry to the `.bib` file first.