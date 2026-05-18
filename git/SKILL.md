---
name: git
description: This skill should be used when the user asks to "initialise a git repo", "set up a new repository", "move a tracked file", "rename a file with git", "add Git LFS", "configure .gitattributes", "work with a submodule", "squash merge", "merge a feature branch into main", or mentions Overleaf-hosted repos, `origin` vs `github` remote naming, or `.gitignore`/`.gitattributes` templates. Covers local git conventions only; for GitHub issues and Projects, use `/github-projects` instead.
---

# Git version control conventions

Apply this skill when initialising a repository, relocating tracked files, configuring Git LFS, or performing file operations inside a submodule. For GitHub issue and project management, defer to `/github-projects`.

---

## Remote naming

Name the default remote `github`, not `origin`. Before pushing or pulling in an unfamiliar repo, inspect the configured remotes:

```bash
git remote -v
```

If the remote is named `origin`, rename it:

```bash
git remote rename origin github
```

Reference `github` in subsequent commands, e.g. `git push github HEAD`.

---

## Moving tracked files — use `git mv`

Use `git mv` rather than plain `mv` when relocating tracked files. This preserves version history and keeps the index consistent with the working tree.

Check `.gitignore` first to confirm whether the file is tracked. For untracked or ignored files, plain `mv` is acceptable — git history is irrelevant there.

```bash
# Tracked file — use git mv
git mv old/path/file.R new/path/file.R

# Untracked or ignored file — plain mv is fine
mv .DS_Store /tmp/
```

### File operations inside submodules

When moving or renaming files inside a submodule, `cd` into the submodule directory first and run `git mv` from there. Do not run plain `mv` from the parent repo — this loses history in the submodule and leaves its index inconsistent with the working tree.

```bash
cd path/to/submodule
git mv old.tex subdir/old.tex
git commit -m "Move old.tex into subdir/"
cd -
git add path/to/submodule
git commit -m "Update submodule pointer"
```

---

## Merging feature branches into `main` — squash by default

Prefer a squash merge when merging a feature branch into `main`. This keeps `main`'s history linear and produces one commit per merged feature.

### Commit-message format

Format the squash commit as:

- **Subject line**: a short description of what the feature branch accomplishes (often the PR title).
- **Body**: one line per commit on the feature branch since it diverged from `main`, each formatted as `<7-char-hash> <subject>`.

Generate the body with:

```bash
git log --pretty=format:'%h %s' main..<feature-branch>
```

`%h` yields the 7-character (abbreviated) short hash; `%s` yields the commit subject.

Example squash commit message:

```
Add widget rendering pipeline

9012345 Initial widget implementation
def5678 Add widget tests
abc1234 Fix widget rendering edge case
```

### Before merging — check that the target branch is up to date

Always `git fetch` the target branch's remote before starting a squash merge, and check whether the remote is ahead of the local copy:

```bash
git fetch <remote> <target-branch>
git log --oneline <target-branch>..<remote>/<target-branch>
```

If the remote is ahead, **stop and ask the user** whether to:
1. Fast-forward the local target branch to the remote first, then squash-merge into the updated branch (usually the right choice — it lands the squash on top of latest), or
2. Squash-merge into the current local state and reconcile later.

Do not silently pull, and do not start the squash merge first and then back out of it after discovering the divergence — backing out a partially-applied squash is messy and easy to get wrong.

### Performing the merge

**With `gh` (preferred when merging via a PR):**

```bash
# Compose the body first so it can be passed into the merge command
BODY=$(git log --pretty=format:'%h %s' main..<feature-branch>)

gh pr merge <PR_NUMBER> --squash \
  --subject "Short description of the feature" \
  --body "$BODY"
```

**With plain `git` (local-only merge):**

```bash
git checkout main
git merge --squash <feature-branch>
git commit   # Compose the squash message using the format above
```

---

## Git LFS

### Machine-level prerequisite (install once per machine)

Before LFS will work for any repo, the `git-lfs` binary has to be on the
system AND its filters must be wired into git's global config. **Without
this, `git clone` of an LFS-using repo silently produces tiny pointer
files (~130 bytes each) in place of the real binary content** — no
error at clone time, but every program that tries to read those files
fails with cryptic errors like "could be corrupt or not supported" or
"invalid magic bytes."

Run on every new machine that will clone LFS-tracked repos:

```bash
sudo apt install -y git-lfs
sudo git lfs install --system   # writes /etc/gitconfig; system-wide
```

`--system` enables LFS filters for every user on the machine (so they
work for `shiny`, daemon users, cron jobs, etc.). On a single-user
machine, `git lfs install` (no flag, current user only) is also fine.

For provisioning scripts, install both in the base-packages step so
later clones Just Work. Existing clones that pre-date the LFS install
still have pointer files in their working tree — hydrate them
retroactively with `git lfs pull` inside each affected repo. Pattern:

```bash
if ! command -v git-lfs >/dev/null 2>&1; then
  sudo apt-get install -y -qq git-lfs
fi
sudo git lfs install --system >/dev/null
# ... clone or pull ...
git lfs pull    # hydrate any pointer files left over from a pre-LFS clone
```

GitHub's LFS auth piggybacks on the same SSH credential used for `git
clone`/`git pull`, so per-repo deploy keys handle LFS downloads
transparently. No separate token needed.

### Per-repo setup (when creating a new LFS-using repo)

In any non-Overleaf repo, after the machine-level prerequisite is in
place, initialise LFS filters in the local clone:

```bash
git lfs install
```

Then add tracking rules via `.gitattributes` (see the template below).

**LFS is not supported on Overleaf-hosted repositories** — see the next section for details.

---

## Overleaf-hosted repositories

Overleaf repos (remotes on `git.overleaf.com`) are subject to platform constraints that differ from ordinary GitHub-hosted repos. They are commonly used as submodules inside a larger GitHub-hosted project — for LaTeX paper drafts, beamer presentations, and similar documents that Overleaf's web editor needs to open.

Detect an Overleaf repo with:

```bash
git remote -v
```

If any remote hostname is `git.overleaf.com`, treat the repo as Overleaf-bound and apply the rules below. When operating inside a submodule that is Overleaf-hosted, these rules override any conflicting conventions from the parent repo.

### Primary branch is `master`

Overleaf's primary branch is `master`, not `main`. Commit and push to `master` when updating an Overleaf submodule.

### No additional branches

Overleaf does not accept pushes to branches other than `master`. Feature branches cannot be used against an Overleaf remote. The squash-merge convention above does not apply — commits land on `master` directly.

Local branches may still be created for work-in-progress staging, but any branch intended to reach Overleaf must be merged back to `master` before pushing.

### No LFS

Do not:

- run `git lfs install` in an Overleaf-bound repo,
- run `git lfs track <pattern>`,
- add LFS filter rules to `.gitattributes`,
- copy the standard `assets/gitattributes.template` (which contains LFS rules).

This applies even when the parent repo uses LFS for other files.

### `.gitignore` — exclude local LaTeX/BibTeX build artefacts

Copy `assets/gitignore.overleaf.template` into the Overleaf repo root. It is a minimal file covering only macOS nuisances, LaTeX/BibTeX build outputs (`*.aux`, `*.log`, `*.bbl`, `*.blg`, `*synctex*`, `*.fdb_latexmk`, `*.fls`, `*.xdv`, etc.), and VSCode files — nothing else. Compiling locally with `latexmk`, `pdflatex`, or `biber` will then not produce files that get pushed back to Overleaf.

---

## New repository setup — checklist

Follow this checklist when starting a new project or initialising a git repo:

1. Copy the `.gitignore` template into the repo root.
2. Copy the `.gitattributes` template into the repo root. **Skip for Overleaf repos** — its LFS rules are incompatible with Overleaf.
3. Run `git lfs install`. **Skip for Overleaf repos.**
4. If the remote is named `origin`, rename it: `git remote rename origin github`. Overleaf remotes may stay under any name.

### Templates

Three template files live in `assets/`:

- **`assets/gitignore.template`** — default `.gitignore`. Ignores macOS, R/RStudio, Quarto/Rmd render artefacts (`*_files/`), TeX/LaTeX, Make, VSCode, Sublime, Excel lock files, and Python.
- **`assets/gitignore.overleaf.template`** — minimal `.gitignore` for Overleaf-hosted repos. Covers macOS, LaTeX/BibTeX build artefacts, and VSCode files — nothing else.
- **`assets/gitattributes.template`** — `.gitattributes` for non-Overleaf repos. Enforces LF line endings and tracks common binary/data formats (PDF, PNG, Stata, parquet, xlsx, etc.) via Git LFS.

Copy them with:

```bash
SKILL_DIR="$HOME/.claude/skills/git"

# Standard (non-Overleaf) repo:
cp "$SKILL_DIR/assets/gitignore.template"    .gitignore
cp "$SKILL_DIR/assets/gitattributes.template" .gitattributes

# Overleaf-hosted repo — no .gitattributes:
cp "$SKILL_DIR/assets/gitignore.overleaf.template" .gitignore
```

---

## Related skills

- `/github-projects` — creating GitHub issues, linking them to GitHub Projects, and setting workstream/dates/blocked-by via the `gh` CLI and GraphQL API.

---

## Updating this file

After completing a task, if a git convention or gotcha worth preserving comes up, propose a specific addition. Prefer refining existing entries over adding new ones, and keep detailed or long-form material out of SKILL.md — move it into `references/` if needed.