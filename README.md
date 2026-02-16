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
| `base-path` | Base URL path prefix for the generated site (see below) | auto-detected |

## Outputs

| Output | Description |
|---|---|
| `path` | Path to the generated static site directory |

## Base path handling

When deploying to GitHub Pages, project repositories are served under a subpath
(e.g., `https://user.github.io/repo-name/`). The action automatically detects the
correct base path from the repository name so that all CSS, JS, image, and
internal link references work correctly.

- **Project pages** (`user/repo-name`): base path is auto-set to `/repo-name/`
- **User/org pages** (`user/user.github.io`): base path is auto-set to `/`
- **Custom**: pass `base-path: /custom/prefix/` to override auto-detection

## How it works

1. Sets up Go
2. Builds the `pkgsite` binary from [wow-look-at-my/static-pkgsite](https://github.com/wow-look-at-my/static-pkgsite)
3. Runs `pkgsite -base-path <path> -out <dir> <paths>` to generate static HTML documentation with correct URL paths
4. Uploads the output as a GitHub Pages artifact via `actions/upload-pages-artifact`

The reusable workflow additionally handles the `actions/deploy-pages` step.
