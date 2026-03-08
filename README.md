<!-- PROJECT SHIELDS -->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MS-PL License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/Lisp-Stat/ls-dev-image">
    <img src="https://lisp-stat.dev/images/stats-image.svg" alt="Logo" width="80" height="80">
  </a>
  <h3 align="center">Lisp-Stat Development Container</h3>
  <p align="center">
    An OCI container image for Common Lisp and Lisp-Stat development<br />
    <a href="https://lisp-stat.dev/"><strong>Learn more at lisp-stat.dev »</strong></a><br />
    <a href="https://github.com/Lisp-Stat/ls-dev-image/issues">Report Bug</a>
    ·
    <a href="https://github.com/Lisp-Stat/ls-dev-image/issues">Request Feature</a>
  </p>
</p>

<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary><h2 style="display: inline-block">Table of Contents</h2></summary>
  <ol>
    <li>
      <a href="#about-the-project">About the Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#run-locally-with-docker">Run Locally with Docker</a></li>
        <li><a href="#run-in-a-devcontainer">Run in a devcontainer</a></li>
        <li><a href="#run-on-the-cloud">Run on the Cloud</a></li>
      </ul>
    </li>
    <li><a href="#developer-workflow">Developer Workflow</a></li>
    <li><a href="#building-an-image">Building an Image</a></li>
    <li><a href="#resources">Resources</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->
## About the Project

This repository provides a ready-to-use, containerized development environment for Common Lisp and [Lisp-Stat](https://lisp-stat.dev/). It is ideal for:
- Evaluating Lisp-Stat without local installation.
- Quickly starting new Lisp-Stat projects in consistent, reproducible environments.
- Contributing to Lisp-Stat and its ecosystem.

It is suited for both **experimenters/learners** and **contributors**.

### Built With

* [SBCL](https://www.sbcl.org/) (Steel Bank Common Lisp)
* [Quicklisp](https://www.quicklisp.org/)
* [Lisp-Stat](https://lisp-stat.dev/)
* [devcontainer Features](https://containers.dev/features)
* [OpenBLAS](https://www.openblas.net/)
* [Docker](https://www.docker.com/)
* [VS Code Devcontainers](https://code.visualstudio.com/docs/devcontainers/containers)

---

## Getting Started

A prebuilt image is published at:

```
ghcr.io/lisp-stat/ls-dev:latest
```

You can use this image locally with Docker, in a Devcontainer, or in the Cloud (e.g. Codespaces).

Supported runtimes:
- Docker (Linux/Mac/Windows)
- Podman
- containerd, Lima, Rancher Desktop, Colima  
- VS Code Dev Containers
- GitHub Codespaces

---

### Run Locally with Docker

#### Quick Experimentation

1. **Pull the latest image:**

   ```sh
   docker pull ghcr.io/lisp-stat/ls-dev:latest
   ```

2. **Start an interactive Lisp-Stat REPL:**

   ```sh
   docker run --rm -it --user vscode -w /home/vscode ghcr.io/lisp-stat/ls-dev:latest /bin/bash -c "ls-init.sh --mode developer && ls-repl"
   ```

   This will start a script that will ask you how to proceed:

   ```sh
   [ls-init.sh] SHELL: /bin/bash                  
   [ls-init.sh] Bash version: 5.2.21(1)-release   
   [ls-init.sh] Args: --mode developer            
   [ls-init.sh] TTY: /dev/pts/0                   
   HOME=/home/vscode                              
   PWD=/home/vscode                               
   SHLVL=2                                        
   TERM=xterm                                     
   Select setup mode:                             
   1) Experimenter (default, upstream only)       
   2) Contributor (fork and relink)               
   #?
   ```                                          
  Choose your option and the script will configure and compile the source and start a Lisp-Stat REPL. From here you can experiment with Lisp-Stat or Common-Lisp (Lisp-Stat is a superset of Common Lisp).  If in future you wish to use only Common-Lisp, modify your `~/.sbclrc` file to remove the loading of `.ls-init.sh`.

  When you start `ls-repl`, the ~/.ls-init.lisp file configures lisp-stat to load a few data sets, like `mtcars`, and also configures an instance of [ls-server](https://github.com/Lisp-Stat/ls-server) on port 20202 by default.  Data frames and plots that you create will show up in this web interface, and you can edit the data-frames from there in an excel like interface.  You must have port forwarding configured for your container, after which you can open http://localhost:20202.

<img width="1038" height="704" alt="image" src="https://github.com/user-attachments/assets/98c0e707-b259-48e4-93c1-1ba7f62e3f09" />

#### Editing Code
Emacs and slime are also configured in this container.  This is the typical lisp coding environment.  To edit with emacs:

1. Start your container with a shell:

```sh
docker run --rm -it --user vscode -w /home/vscode ghcr.io/lisp-stat/ls-dev:latest bash
```

2. Run `ls-init.sh` to obtain the source if you have not already done so
3. From within the container, run `emacs`
4. From emacs, run `M-x slime` (the 'M' stands for the 'Meta' key, and is often Alt in PC style keyboards)

<img width="1306" height="762" alt="image" src="https://github.com/user-attachments/assets/1abaeea4-1bee-494b-9dd4-c590c9a66f5c" />

  
---

#### Setting Up a Persistent Development Workspace

To persist source code and settings between container restarts, use Docker volumes or bind mounts. This is essential if you want to do any code editing, save your work, or contribute changes.

##### Option A: Named Docker Volume (Recommended)

1. **Create or reuse a named volume for Lisp-Stat developer workspace:**

   ```sh
   docker volume create lisp-stat-workspace
   ```

2. **Start the container with persistent workspace:**

   ```sh
   docker run --rm -it \
     --user vscode \
     -v lisp-stat-workspace:/workspaces/ls-dev \
     ghcr.io/lisp-stat/ls-dev:latest bash
   ```

##### Option B: Host Directory Mount

1. **Create a directory on your host for Lisp-Stat projects:**
   ```sh
   mkdir -p ~/lisp-stat-workspace
   ```

2. **Start the container mounting this directory:**
   ```sh
   docker run --rm -it \
     --user vscode \
     -v ~/lisp-stat-workspace:/workspaces/ls-dev \
     ghcr.io/lisp-stat/ls-dev:latest bash
   ```

---

#### (Optional) Pre-configure the Container with ls-init.sh

The included **ls-init.sh** script sets up your developer workspace. You should run this INSIDE the container after starting it with a persistent workspace as above. See the [Developer Workflow](#developer-workflow) for details.

---

### Run in a Devcontainer

If you use [Visual Studio Code](https://code.visualstudio.com/) or similar tools that support devcontainers, you can use the image directly.

1. **Create a `devcontainer.json` file** in your project's root directory:

   ```json
   {
     "name": "My Lisp-Stat Project",
     "image": "ghcr.io/lisp-stat/ls-dev:latest",
     "mounts": [
       "source=lisp-stat-workspace,target=/workspaces/ls-dev,type=volume"
     ]
   }
   ```

2. Open the folder in VS Code and "Reopen in Container" when prompted.

You can also use the [devcontainercli](https://code.visualstudio.com/docs/devcontainers/devcontainer-cli) as a docker replacement for working with these images.  This is our recommendation.
---

### Run on the Cloud (GitHub Codespaces)

1. Go to your repository on GitHub.
2. Click on the **Code** button and choose **"Open with Codespaces"**.
3. Use the [devcontainer.json](#run-in-a-devcontainer) example above for the fastest start.
4. Wait for the Codespace to initialize and you can begin using VS Code in the browser.

---

## Developer Workflow

This section focuses on helping **new developers** onboard, regardless of familiarity with Common Lisp, Docker, or GitHub. All essential setup is automated by the `ls-init.sh` script included in the image.

> **There are two main modes:**
> - **Experimenter**: Want to try out/dev or learn? Get read-only access to all Lisp-Stat software with no need to set up GitHub.
> - **Contributor**: Want to contribute and submit pull requests? Fork, clone, and set up your workspace for development and PRs on GitHub.

### 1. Open Your Container

Start the container using one of the methods above:
- Docker (with a persistent volume or directory, see "Run Locally")
- VS Code Devcontainer
- Codespaces

> For all modes, commands below should be run **inside the container shell as the vscode user**.

---

### 2. Set Up Your Development Environment with ls-init.sh

#### About `ls-init.sh`

`ls-init.sh` sets up your Lisp-Stat workspace in two ways:

- **Experimenter mode** (default): Clones official Lisp-Stat repositories to a workspace directory (e.g., `/workspaces/ls-dev`), making them available read-only. Safe, fast, and ideal for exploring or learning.
- **Contributor mode**: Forks the official repositories to your GitHub account and clones your forks to your workspace (e.g., `/workspaces/ls-dev`) so you can commit and push changes, submit PRs, and track upstream changes.

#### Prerequisites (for Contributor mode only)

- You must have [GitHub CLI (`gh`)](https://cli.github.com/) installed in the container (preinstalled in image).
- You must be authenticated to GitHub:  
  ```
  gh auth login
  ```

---

#### Experimenter Workflow: Fastest Start without GitHub

1. **(Inside container) Run ls-init.sh in experimenter mode (default):**

   ```sh
   /usr/local/bin/ls-init.sh
   ```

   Or, explicitly:

   ```sh
   /ls-init.sh --mode experimenter
   ```

   - This clones the latest official Lisp-Stat repositories to `/workspaces/lisp-stat-upstream` (or `$HOME/lisp-stat-upstream`), then symlinks them into:
     - `/workspaces/ls-dev` (your workspace mount for editor access)
     - `~/quicklisp/local-projects` (so SBCL/Quicklisp can discover them)

   - **You can immediately open and work with all Lisp-Stat source code.**
   - Code is **read-only** with respect to pushing changes upstream or submitting PRs.

**To update to the latest upstream after a while:**

```sh
ls-init.sh --mode experimenter --refresh
```
This fetches and updates all repos to their newest state.

---

#### Contributor Workflow: Fork, Edit, and Submit Pull Requests

1. **Authenticate to GitHub (if not already):**

   ```sh
   gh auth login
   ```

2. **Run ls-init.sh in contributor mode:**

   ```sh
   ls-init.sh --mode contributor
   ```

   - This does all experimenter steps BUT
     - Forks each Lisp-Stat repo to your GitHub account (using the `gh` command).
     - Clones your forks to `/workspaces/ls-dev` (the persistent workspace).
     - Adds the official Lisp-Stat repo as `upstream` so you can stay up-to-date.
     - Symlinks all your clones into `~/quicklisp/local-projects` (so Lisp can load your forks by default).

   - Now, any edits you make are in your own GitHub fork. You can commit, push, and create pull requests directly.

---

#### Workflow Quick Reference Table

| Mode           | Workspace source            | Pushing changes           | PRs?     | Suitable for                  |
|----------------|----------------------------|---------------------------|----------|-------------------------------|
| Experimenter   | Official repos (read-only) | No                        | No       | Exploration, learning         |
| Contributor    | Your GitHub forks (w/upstream) | Yes                     | Yes      | Active development, contributing |

**To switch modes later:**  
You can always re-run ls-init.sh in contributor mode later to become a contributor.

---

##### What Gets Set Up

- `/workspaces/ls-dev/<repo>`: Your working tree (fork or upstream clone)
- `~/quicklisp/local-projects/<repo>`: Symlink to above (for SBCL/Quicklisp)
- For contributors: `origin` remote is your fork, `upstream` is the official repo.

---

#### Example: Full Contributing Process (Step-by-Step)

1. **Start the container with a persistent workspace.**
2. **Inside the container:**
    ```sh
    gh auth login         # if needed
    ls-init.sh --mode contributor
    ```
3. **Make and commit code changes:**
    ```sh
    cd /workspaces/ls-dev/data-frame
    git checkout -b my-cool-feature
    # ...edit/code/test...
    git add .
    git commit -m "Add my cool feature"
    git push origin my-cool-feature
    ```

4. **Open a pull request on GitHub as usual.**
5. **To update your fork with latest from upstream:**
    ```sh
    git fetch upstream
    git merge upstream/master
    git push origin master
    ```

---

## Building an Image

To customize or rebuild the image yourself, you need the [devcontainer CLI](https://github.com/devcontainers/cli):

```sh
npm install -g @devcontainers/cli
devcontainer build --workspace-folder . --image-name my-ls-dev:latest
```

To push to a registry:
```sh
devcontainer build --workspace-folder . --image-name ghcr.io/youruser/ls-dev:latest --push
```

Standard `docker build` is not supported (due to devcontainer features composition).

---

## Resources

- [Lisp-Stat Documentation](https://lisp-stat.dev/)
- [devcontainer features source](https://github.com/Symbolics/devcontainer-features)
- [cl-jupyter-image](https://github.com/Lisp-Stat/cl-jupyter-image)

---

## Contributing

Contributions are welcome—see [CONTRIBUTING](CONTRIBUTING.md) for guidelines.

---

## License

Distributed under the MS-PL License. See [LICENSE](LICENSE) for details.

---

## Contact

Project Link: [https://github.com/Lisp-Stat/ls-dev-image](https://github.com/Lisp-Stat/ls-dev-image)

<!-- MARKDOWN LINKS & IMAGES -->
[contributors-shield]: https://img.shields.io/github/contributors/Lisp-Stat/ls-dev-image.svg?style=for-the-badge
[contributors-url]: https://github.com/Lisp-Stat/ls-dev-image/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/Lisp-Stat/ls-dev-image.svg?style=for-the-badge
[forks-url]: https://github.com/Lisp-Stat/ls-dev-image/network/members
[stars-shield]: https://img.shields.io/github/stars/Lisp-Stat/ls-dev-image.svg?style=for-the-badge
[stars-url]: https://github.com/Lisp-Stat/ls-dev-image/stargazers
[issues-shield]: https://img.shields.io/github/issues/Lisp-Stat/ls-dev-image.svg?style=for-the-badge
[issues-url]: https://github.com/Lisp-Stat/ls-dev-image/issues
[license-shield]: https://img.shields.io/github/license/Lisp-Stat/ls-dev-image.svg?style=for-the-badge
[license-url]: https://github.com/Lisp-Stat/ls-dev-image/blob/master/LICENSE
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/company/symbolics/
