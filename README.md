# .github

Shared GitHub configuration for all [cube-m4st3r](https://github.com/cube-m4st3r) repositories: reusable CI workflows and,
later, default community files (`SECURITY.md`, issue templates, ...). GitHub applies those to every repo that doesn't
have its own.

## Reusable workflows

| Workflow | What it does | Status |
| --- | --- | --- |
| [`pre-commit.yml`](.github/workflows/pre-commit.yml) | Runs all pre-commit hooks, plus a gitleaks scan of the full git history | ready |
| `container-image.yml` | Build, Trivy scan, push to GHCR, SBOM + provenance, cosign keyless signing | planned |

### Usage

Fix bugs here once, and every repo picks them up. A repo's whole CI file is just:

```yaml
---
name: ci

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read

jobs:
  lint:
    uses: cube-m4st3r/.github/.github/workflows/pre-commit.yml@main
```

The doubled `.github/.github/` is not a typo: the repo is called `.github`, and workflows always live in a `.github/workflows/`
folder inside it.

### Why gitleaks runs twice

The gitleaks pre-commit hook only scans **staged** changes, which catches a secret before it becomes a commit. In CI nothing
is staged, so the hook is skipped there, and a separate job scans **every commit in the history** instead.

## License

[MIT](LICENSE)
