# static-pkgsite-action

GitHub Action that generates static Go package documentation using [static-pkgsite](https://github.com/wow-look-at-my/static-pkgsite) and deploys it to GitHub Pages.

## Usage

### Reusable workflow (simplest)

Create `.github/workflows/docs.yml` in your Go project:

```yaml
name: Docs

on:
  push:
    branches: [main]

permissions:
  pages: write
  id-token: write

jobs:
  docs:
    uses: wow-look-at-my/static-pkgsite-action/.github/workflows/pages.yml@master
```

This checks out your repo, generates static documentation, and deploys it to GitHub Pages in one step.

### Composite action (more control)

If you need to customise the workflow (e.g. run extra steps before/after generation):

```yaml
name: Docs

on:
  push:
    branches: [main]

permissions:
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: wow-look-at-my/static-pkgsite-action@master
        with:
          path: '.'
          go-version: 'stable'

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/deploy-pages@v4
        id: deployment
```

## Inputs

| Input | Description | Default |
|---|---|---|
| `path` | Path to the Go module (or space-separated list of paths) to document | `.` |
| `go-version` | Go version to use | `stable` |
| `pkgsite-ref` | Git ref of `wow-look-at-my/static-pkgsite` to build | `master` |
| `out` | Output directory for the generated static site | `_site` |

## Outputs

| Output | Description |
|---|---|
| `path` | Path to the generated static site directory |

## How it works

1. Sets up Go
2. Builds the `pkgsite` binary from [wow-look-at-my/static-pkgsite](https://github.com/wow-look-at-my/static-pkgsite)
3. Runs `pkgsite -out <dir> <paths>` to generate static HTML documentation
4. Uploads the output as a GitHub Pages artifact via `actions/upload-pages-artifact`

The reusable workflow additionally handles the `actions/deploy-pages` step.
