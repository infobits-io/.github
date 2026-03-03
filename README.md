# .github

Shared CI/CD infrastructure for the [infobits-io](https://github.com/infobits-io) organization.

This is a GitHub [organization-level `.github` repository](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file). It provides reusable workflows, composite actions, and org-wide defaults (CODEOWNERS, Dependabot) available to all repositories.

## Composite Actions

### `actions/setup-pnpm`

Consolidates the common ~20-line pnpm/Node.js setup into a single step. Handles pnpm installation, Node.js setup with caching, npm registry configuration (GitHub Packages + FontAwesome Pro), and dependency installation.

**Usage:**

```yaml
steps:
  - uses: actions/checkout@v6
  - uses: infobits-io/.github/actions/setup-pnpm@master
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      fontawesome-token: ${{ secrets.FONTAWESOME_NPM_AUTH_TOKEN }}
```

**Inputs:**

| Input | Default | Description |
|---|---|---|
| `node-version` | `24.x` | Node.js version |
| `github-token` | *(required)* | GitHub token for GitHub Packages registry |
| `fontawesome-token` | `""` | FontAwesome Pro registry token (optional) |
| `working-directory` | `.` | Working directory for monorepos |
| `install` | `true` | Whether to run `pnpm install` |

## Reusable Workflows

### `.github/workflows/website-ci.yml`

Standard CI pipeline for Next.js websites. Runs lint, test (with coverage), build (PR only), and deploy (push to default branch).

**Usage:**

```yaml
name: CI
on:
  push:
    branches: [master]
  pull_request:
    branches: [master]
  workflow_dispatch:

jobs:
  ci:
    uses: infobits-io/.github/.github/workflows/website-ci.yml@master
    with:
      node-version: "24.x"
    secrets:
      FONTAWESOME_NPM_AUTH_TOKEN: ${{ secrets.FONTAWESOME_NPM_AUTH_TOKEN }}
      CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_KEY }}
      CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

**Inputs:**

| Input | Default | Description |
|---|---|---|
| `node-version` | `24.x` | Node.js version |
| `lint-command` | `pnpm run lint` | Lint command |
| `test-command` | `pnpm run test:ci` | Test command |
| `build-command` | `pnpm run build` | Build command |
| `deploy-command` | `pnpm run deploy` | Deploy command |
| `working-directory` | `.` | Working directory for monorepos |
| `coverage` | `true` | Upload coverage artifacts and PR report |

**Secrets:** `FONTAWESOME_NPM_AUTH_TOKEN`, `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`

### `.github/workflows/go-ci.yml`

Standard CI pipeline for Go backend services. Runs lint (golangci-lint), optional buf lint, test (with race detection + Codecov), and build.

Deployment is intentionally excluded — it varies significantly between Go services (different Scaleway containers, Docker registries, MaxMind databases). Each project keeps its own deploy job.

**Usage:**

```yaml
name: CI/CD
on:
  push:
    branches: [master]
  pull_request:
    branches: [master]
  workflow_dispatch:

jobs:
  ci:
    uses: infobits-io/.github/.github/workflows/go-ci.yml@master
    with:
      buf-lint: true
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}

  deploy:
    # Keep project-specific deploy job
    needs: [ci]
    if: github.event_name == 'push' || github.event_name == 'workflow_dispatch'
    runs-on: ubuntu-latest
    steps:
      # ... project-specific deploy steps
```

**Inputs:**

| Input | Default | Description |
|---|---|---|
| `go-version` | `1.26` | Go version |
| `golangci-lint-version` | `v2` | golangci-lint version |
| `buf-lint` | `false` | Run buf lint for protobuf projects |
| `codecov` | `true` | Upload coverage to Codecov |

**Secrets:** `CODECOV_TOKEN` (optional)

## Org-Level Defaults

- **`CODEOWNERS`** — Default code ownership: `@infobits-io/maintainers`
- **`dependabot.yml`** — Weekly updates for npm, GitHub Actions, and Go modules
