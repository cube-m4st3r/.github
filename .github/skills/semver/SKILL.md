---
name: semver
description: >-
  Versioning and commit-message rules for the .github repository (shared reusable GitHub Actions workflows). Use when writing or
  reviewing a commit message or PR title, deciding whether a change is breaking, or predicting
  the next release version.
---

# Semantic versioning for `.github`

This repo holds reusable workflows (`on: workflow_call`) that all other cube-m4st3r repos call.
Its public API is each workflow's file path, its `inputs`, `secrets` and `outputs`, the
`permissions` a caller must grant, and the jobs it runs. Callers should pin a major tag
(`@v1`) instead of `@main`, so a breaking change only reaches them when they choose to upgrade.

## How versions are made here

- Versions are **never chosen by hand**. release-please reads the Conventional Commit messages on
  `main`, keeps a release PR open with the next version and the generated `CHANGELOG.md`, and
  merging that PR creates the tag (e.g. `v1.4.0`); the moving major tag (`v1`) is updated by the release workflow.
- Never edit `.release-please-manifest.json`, `CHANGELOG.md`, or create git tags yourself. Your job is to write
  commit messages that make release-please compute the right version.
- If a pull request is squash-merged, its title becomes the commit message, so PR titles follow
  the same format.
- Versions follow SemVer `MAJOR.MINOR.PATCH`. While the latest release is still `0.x`, a breaking
  change is marked exactly the same way; release-please (configured with `bump-minor-pre-major`)
  turns it into a MINOR bump until `1.0.0`.

## Commit message format

```text
<type>(<scope>)<!>: <summary>

<body: why the change was made, not what the diff shows>

BREAKING CHANGE: <what breaks, and the exact steps to migrate>
```

- `summary`: imperative mood, lowercase start, no trailing period, at most 72 characters.
- `scope` (optional but preferred): the workflow name without extension (`pre-commit`, `container-image`), or `ci`, `deps`.
- `!` after the type/scope **and** a `BREAKING CHANGE:` footer for every breaking change. The footer
  is what users read in the changelog, so it must explain the migration, not just repeat the summary.

## Type → version bump

| Type | Use for | Bump |
| --- | --- | --- |
| `feat` | Something new a user of this repo can use | MINOR |
| `fix` | A bug fix users would notice | PATCH |
| `perf` | Faster or lighter, same behaviour | PATCH |
| `fix(deps)` | Updating a pinned action or tool version (e.g. gitleaks) used by a workflow | PATCH |
| `docs`, `refactor`, `test`, `ci`, `build`, `style`, `chore` | Everything with no effect on what users get | none |
| any type + `!` / `BREAKING CHANGE:` | See the checklist below | MAJOR |

Pick the type by the **effect on users**, not by the size of the diff.

## Breaking change checklist for `.github`

A change is breaking (`!` + footer) if **any** of these is true:

- A reusable workflow is removed, renamed or moved.
- An input, secret or output is removed or renamed, or an optional input becomes required.
- An input's default changes in a way callers notice, e.g. a different Python version that
  breaks their hooks.
- Callers must grant more `permissions` than before (e.g. now `id-token: write`). Their runs
  fail until they do.
- A job or check that callers rely on is removed, or a check becomes strict enough to fail runs
  that passed before (e.g. Trivy now fails on HIGH instead of only CRITICAL).

## MINOR in this repo

- A new reusable workflow.
- A new optional input, secret or output.
- A new optional job or check that is off by default.

## PATCH in this repo

- A bug fix in a workflow that keeps inputs and behaviour.
- `fix(deps)`: bumping a pinned action or tool version without changing behaviour for callers.

## No release in this repo

- README and comments: `docs`.
- This repo's own CI (`ci.yml`) and pre-commit config: `ci`, `chore`.

## Procedure

1. Look at the change: `git diff --staged` (or the PR diff).
2. Go through the breaking checklist above, item by item. If one matches, use `!` and write a
   `BREAKING CHANGE:` footer with migration steps.
3. When it is unclear whether something breaks users, **ask** instead of guessing. Hiding a break
   in a `feat` or `fix` is worse than flagging one too many.
4. Pick the type and scope by user-visible effect.
5. One logical change per commit. If the diff mixes unrelated changes (e.g. a feature and an
   unrelated fix), suggest splitting it into separate commits.
6. Write the message. Keep the body short and focused on *why*.

## Predicting the next version

```bash
last=$(git describe --tags --abbrev=0)
git log "$last"..HEAD --format='%s%n%b'
```

The highest bump among those commits wins: any breaking change → MAJOR (MINOR while `0.x`),
else any `feat` → MINOR, else any `fix`/`perf`, `fix(deps)` → PATCH, else no release.

## Examples

```text
feat(container-image): add optional platforms input for multi-arch builds
```

```text
fix(deps): update gitleaks to 8.31.0
```

```text
feat(pre-commit)!: require python 3.14 by default

ansible-lint's hook now pins python3.14.

BREAKING CHANGE: callers relying on the old default of 3.13 must pass
`python-version: "3.13"` explicitly, or install Python 3.14 locally
for their hooks.
```
