# ls-dev Image

An OCI image for Common Lisp development with [Lisp-Stat](https://lisp-stat.dev/).

Built from `mcr.microsoft.com/devcontainers/base:ubuntu-24.04` using
[devcontainer features](https://containers.dev/features). The image includes:

- **SBCL** – Steel Bank Common Lisp, built from source
- **Quicklisp** – the de-facto package manager for Common Lisp
- **ACLREPL** – enhanced REPL for SBCL
- **Lisp-Stat** – statistical computing environment with OpenBLAS

The published image is at:

```
ghcr.io/lisp-stat/ls-dev:latest
```

## Usage in a devcontainer

Instead of listing individual features, point your `devcontainer.json` directly at
this pre-built image to get fast container startup (no feature install phase):

```jsonc
{
  "name": "My Lisp-Stat Project",
  "image": "ghcr.io/lisp-stat/ls-dev:latest"
}
```

## Run locally

```sh
docker run --rm -it --user vscode ghcr.io/lisp-stat/ls-dev:latest sbcl
```

## Building the image

Building requires the [devcontainer CLI](https://github.com/devcontainers/cli).
Standard `docker build` cannot be used because the image is composed from
devcontainer features.

```sh
npm install -g @devcontainers/cli

# Build locally for testing
devcontainer build --workspace-folder . --image-name ls-dev:latest

# Build and push to a registry
devcontainer build --workspace-folder . --image-name ghcr.io/lisp-stat/ls-dev:latest --push
```

## Publishing

The GitHub Actions workflow in `.github/workflows/publish.yml` builds and pushes
the image automatically on every push to `main` and on version tags (`v*`).

## Versioning

| Tag | Description |
|-----|-------------|
| `latest` | Most recent build from `main` |
| `1.0.0`, `1.1.0`, … | Pinned release builds (from `v*` git tags) |

## BLAS variant

This image bundles **OpenBLAS**. An Intel MKL variant can be produced by
creating a second `devcontainer.json` (e.g. in `.devcontainer-mkl/`) with
`"blas": "intel-mkl"` and tagging the resulting image `ls-dev:latest-mkl`.

## Developer Workflow

### Out of the box

The upstream Lisp-Stat repositories are pre-cloned into
`~/quicklisp/local-projects/` during the image build. SBCL and Lisp-Stat work
immediately with no setup required:

```sh
sbcl --eval "(ql:quickload :lisp-stat)" --eval "(quit)"
```

The repos are read-only copies of the upstream Lisp-Stat organisation. They are
sufficient for using and exploring Lisp-Stat, but you cannot push changes to them.

### Setting up a contributor workspace

When you are ready to contribute, authenticate with GitHub and run `ls-fork`:

```sh
gh auth login
ls-fork
```

`ls-fork` does the following for every repo in `~/quicklisp/local-projects/`:

1. Forks the repo to your GitHub account (via `gh repo fork`)
2. Clones your fork to `/workspaces/lisp-stat/<repo>` — this is a host bind
   mount that **persists across container rebuilds**
3. Adds an `upstream` remote pointing back to the Lisp-Stat origin
4. Replaces the in-container `~/quicklisp/local-projects/<repo>` directory with
   a symlink to the new host-mount clone

After running `ls-fork`:

| Location | Remote | Purpose |
|---|---|---|
| `/workspaces/lisp-stat/<repo>` | `origin` → your fork | Push your changes here |
| `/workspaces/lisp-stat/<repo>` | `upstream` → Lisp-Stat | Fetch official updates |
| `~/quicklisp/local-projects/<repo>` | symlink → above | SBCL/Quicklisp load path |

`ls-fork` is idempotent — repos already present in `/workspaces/lisp-stat/` are
skipped, so it is safe to re-run if the process is interrupted.

### Keeping your forks up to date

```sh
cd /workspaces/lisp-stat/<repo>
git fetch upstream
git merge upstream/master
```

Or for all repos at once:

```sh
for repo in /workspaces/lisp-stat/*/; do
  echo "Updating $(basename $repo)..."
  git -C "$repo" fetch upstream
  git -C "$repo" merge upstream/master
done
```

## Resources

- [Lisp-Stat](https://lisp-stat.dev/)
- [devcontainer features source](https://github.com/Symbolics/devcontainer-features)
- [cl-jupyter-image](https://github.com/Lisp-Stat/cl-jupyter-image) – companion Jupyter image

## License

MS-PL
