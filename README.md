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
docker run --rm -it --user vscode -w /home/vscode ghcr.io/lisp-stat/ls-dev:latest ls-repl
```

This will give you a Common Lisp REPL with Lisp-Stat loaded. Exiting from the REPL will exit the container.

If you want to edit files with emacs use:

```sh
docker run --rm -it --user vscode -w /home/vscode ghcr.io/lisp-stat/ls-dev:latest
```

## Read documentation

The [lisp-stat](https://lisp-stat.dev/) website is available locally.  Use this to test your documentation additions or edits.  To start it the server:

```sh
cd ~/quicklisp/local-projects/documentation
hugo server
```
This will start a webserver on the local host at port 1313 that you can connect to.  If you're running remotely, you'll probably want to use an SSH tunnel.  If running under VS Code, the port forwarding will happen automatically. (If running on VS Code or Code Spaces, the /workspaces directory will be automatically populated with the Lisp-Stat source).

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

The upstream Lisp-Stat repositories are cloned into
`~/quicklisp/local-projects/` during the image build. SBCL and Lisp-Stat work
are available no setup required. The repos are read-only copies of the upstream Lisp-Stat organisation. They are
sufficient for using and exploring Lisp-Stat, but you cannot push changes to them.

### Setting up a contributor workspace

`ls-fork` clones your forks into `/workspaces/lisp-stat/`.  That directory must
be backed by **persistent storage** so your work survives container rebuilds.
After a rebuild, the container automatically re-links `~/quicklisp/local-projects/`
to your surviving fork clones — no manual steps required.

Choose the setup that matches your environment before running `ls-fork`:

#### GitHub Codespaces

`/workspaces` is automatically a persistent volume in Codespaces — no
configuration needed.  Just authenticate and run:

```sh
gh auth login
ls-fork
```

#### VS Code Dev Containers

Add a `mounts` entry to your `devcontainer.json` to attach persistent storage
to `/workspaces/lisp-stat`, then rebuild the container before running `ls-fork`.

**Named Docker volume** (recommended — no host-side setup):

```jsonc
{
  "image": "ghcr.io/lisp-stat/ls-dev:latest",
  "mounts": [
    "source=lisp-stat-forks,target=/workspaces/lisp-stat,type=volume"
  ]
}
```

**Host bind mount** (forks visible in your host filesystem):

```jsonc
{
  "image": "ghcr.io/lisp-stat/ls-dev:latest",
  "mounts": [
    "source=${localWorkspaceFolder}/../lisp-stat,target=/workspaces/lisp-stat,type=bind,consistency=cached"
  ]
}
```

After adding the mount, **Rebuild Container**, then:

```sh
gh auth login
ls-fork
```

#### Local `docker run`

Pass a volume flag so `/workspaces/lisp-stat` persists between container runs:

```sh
# Named volume (simplest)
docker run --rm -it --user vscode \
  -v lisp-stat-forks:/workspaces/lisp-stat \
  ghcr.io/lisp-stat/ls-dev:latest bash

# Or bind to a host directory
docker run --rm -it --user vscode \
  -v ~/lisp-stat:/workspaces/lisp-stat \
  ghcr.io/lisp-stat/ls-dev:latest bash
```

Then inside the container:

```sh
gh auth login
ls-fork
```

Always pass the same `-v` flag on subsequent runs to access your fork clones.

#### What `ls-fork` does

For every repo in `~/quicklisp/local-projects/`:

1. Forks the repo to your GitHub account (via `gh repo fork`)
2. Clones your fork to `/workspaces/lisp-stat/<repo>` (on the persistent mount)
3. Adds an `upstream` remote pointing back to the Lisp-Stat origin
4. Replaces the in-container `~/quicklisp/local-projects/<repo>` directory with
   a symlink to the fork clone

After running `ls-fork`:

| Location | Remote | Purpose |
|---|---|---|
| `/workspaces/lisp-stat/<repo>` | `origin` → your fork | Push your changes here |
| `/workspaces/lisp-stat/<repo>` | `upstream` → Lisp-Stat | Fetch official updates |
| `~/quicklisp/local-projects/<repo>` | symlink → above | SBCL/Quicklisp load path |

`ls-fork` is idempotent — repos already present in `/workspaces/lisp-stat/` are
skipped, so it is safe to re-run if the process is interrupted.

#### After a rebuild

When the container is rebuilt or restarted, the startup scripts automatically
re-link `~/quicklisp/local-projects/<repo>` to your fork clones on the
persistent volume.  SBCL continues to load your fork code without any manual
intervention.

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
