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
docker run --rm -it ghcr.io/lisp-stat/ls-dev:latest sbcl
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

To make the package publicly accessible after the first push, go to the GitHub
package settings for `ghcr.io/lisp-stat/ls-dev` and set visibility to **Public**.

## Versioning

| Tag | Description |
|-----|-------------|
| `latest` | Most recent build from `main` |
| `1.0.0`, `1.1.0`, … | Pinned release builds (from `v*` git tags) |

## BLAS variant

This image bundles **OpenBLAS**. An Intel MKL variant can be produced by
creating a second `devcontainer.json` (e.g. in `.devcontainer-mkl/`) with
`"blas": "intel-mkl"` and tagging the resulting image `ls-dev:latest-mkl`.

## Resources

- [Lisp-Stat](https://lisp-stat.dev/)
- [devcontainer features source](https://github.com/Symbolics/devcontainer-features)
- [cl-jupyter-image](https://github.com/Lisp-Stat/cl-jupyter-image) – companion Jupyter image

## License

MS-PL
