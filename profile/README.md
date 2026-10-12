# JupyterHub User Guide

**BIH Center of Digital Health**

This guide is for students and collaborators using the JupyterHub instance at [dh-jhub.bihealth.org](https://dh-jhub.bihealth.org/), hosted on the [de.NBI Cloud](https://www.denbi.de/cloud).

Important contacts:
- [Naveed Ishaque](naveed.ishaque@bih-charite.de) for access and scientific issues.
- [Martin Braun](martin.braun@bih-charite.de) for cloud related questions.

## Contents

- [Quick start](#quick-start)
- [Getting access](#getting-access)
- [Logging in](#logging-in)
- [Managing your server](#managing-your-server)
- [The JupyterLab interface](#the-jupyterlab-interface)
- [First-time setup (required)](#first-time-setup-required)
- [Conda environments and notebook kernels](#conda-environments-and-notebook-kernels)
- [For RStudio users](#for-rstudio-users)
- [File system and storage](#file-system-and-storage)
- [Customisation](#customisation)
- [Data policy](#data-policy)
- [Getting help](#getting-help)

## Quick start

If you have access and just need the essential first-time setup, run this in a terminal on your server:

```bash
# Switch to bash (the default terminal shell is sh)
/bin/bash

# Initialise conda for your shell
conda init

# Make bash the default terminal shell
echo 'c.ServerApp.terminado_settings = { "shell_command": ["/bin/bash"] }' >> ~/.jupyter/jupyter_server_config.py
echo 'c.NotebookApp.terminado_settings = { "shell_command": ["/bin/bash"] }' >> ~/.jupyter/jupyter_notebook_config.py

# Store conda environments in your home directory so they persist
conda config --add envs_dirs ~/conda_envs/
```

> [!WARNING]
> If you skip the `envs_dirs` step, conda environments are written to `/opt/conda`, which is deleted when your server stops. **Your environments will disappear.** See [First-time setup](#first-time-setup-required) for details.

## Getting access

Access is managed through this GitHub organisation:

1. Email [naveed.ishaque@bih-charite.de](mailto:naveed.ishaque@bih-charite.de) with your GitHub username and a one-line description of your intended use.
2. Accept the invitation to join the organisation.
3. Log in at [dh-jhub.bihealth.org](https://dh-jhub.bihealth.org/) using GitHub authentication.

## Logging in

1. Go to [https://dh-jhub.bihealth.org/](https://dh-jhub.bihealth.org/).
2. Sign in with your GitHub account and authorise the application when prompted.

<!-- TODO: screenshot of the login page -->

## Managing your server

### Choosing a machine profile

After logging in you are asked to select a machine profile before your server starts.

| Profile | When to use |
|---|---|
| Default | Most tasks. Start here. |
| Large | Memory- or CPU-intensive jobs. Request access first. |
| GPU | GPU-accelerated workloads. Request access first. |

To request the large or GPU profiles, email [naveed.ishaque@bih-charite.de](mailto:naveed.ishaque@bih-charite.de) with a brief justification. Resources are shared, so please use the smallest profile that fits your task.

<!-- TODO: screenshot of the server options page -->

### Starting your server

Starting takes 1 to 2 minutes. Once running, you are dropped into the JupyterLab interface.

### Your username is jovyan

Every user on the hub has the username `jovyan`. This is a Jupyter convention ("Jovian" means Jupiter-like). Although the username is shared, each user has their own separate home directory. Nobody else can see your files.

### Stopping your server

When you are finished, stop your server to free resources for others:

**File → Hub Control Panel → Stop My Server**

Your home directory persists between sessions. Anything outside it (including `/opt/conda`) does not.

## The JupyterLab interface

JupyterLab is a browser-based working environment combining notebooks, terminals, and a file browser.

<!-- TODO: annotated screenshot of the JupyterLab interface -->

### Navigation pane

The left sidebar contains the file browser. Use it to navigate your home directory and the shared storage, upload and download files, and manage running sessions.

### Launcher pane

The launcher (the main panel when you open a new tab) lists available notebook kernels, consoles, and other tools. Each conda environment you register as a kernel appears here as a separate tile.

### Python notebooks

Open a Python notebook from the launcher. Code cells and markdown cells can be mixed freely. Useful keyboard shortcuts:

| Shortcut | Action |
|---|---|
| `A` | Insert cell above |
| `B` | Insert cell below |
| `M` | Convert cell to markdown |
| `DD` | Delete cell |
| `Ctrl`/`Cmd` + `Enter` | Run current cell |

### R notebooks

R notebooks work the same way. The kernel selector in the top right of the notebook switches between registered R versions and environments. See [For RStudio users](#for-rstudio-users) if you are used to RStudio.

### Terminal

Open a terminal from the launcher for command-line work.

> [!IMPORTANT]
> The default terminal shell is `sh`, in which `conda activate` does not work. Type `/bin/bash` at the start of each terminal session, or complete the [first-time setup](#first-time-setup-required) once to make bash the default.

## First-time setup (required)

Complete these steps once, in a terminal, before doing any real work.

### 1. Start bash and initialise conda

```bash
/bin/bash
conda init
```

### 2. Make bash the default terminal shell

```bash
echo 'c.ServerApp.terminado_settings = { "shell_command": ["/bin/bash"] }' >> ~/.jupyter/jupyter_server_config.py
echo 'c.NotebookApp.terminado_settings = { "shell_command": ["/bin/bash"] }' >> ~/.jupyter/jupyter_notebook_config.py
```

New terminals will now open in bash automatically.

### 3. Set a persistent conda environment directory

```bash
conda config --add envs_dirs ~/conda_envs/
```

> [!WARNING]
> This step is essential. By default conda installs environments to `/opt/conda`, which lives inside the server image and is **deleted every time your server stops**. Only `/home/jovyan` persists. With `envs_dirs` set to `~/conda_envs/`, your environments survive server restarts. This folder will be created when you first create an environment in conda.

### 4. Verify

Create a test environment and check where it was installed:

```bash
conda create -n test_env -c conda-forge scanpy
conda env list
```

The environment path must begin with `/home/jovyan/conda_envs/`. If it begins with `/opt/conda`, the `envs_dirs` setting was not applied; repeat step 3 and recreate the environment.

### 5. Do not work in base

Always create and activate a named environment for your work:

```bash
conda create -n my_project_env
conda activate my_project_env
```

The `base` environment lives in `/opt/conda` and is reset when your server stops.

## Conda environments and notebook kernels

Creating a conda environment does not automatically make it available in the launcher. You must register it as a Jupyter kernel.

### Registering a Python environment as a kernel

```bash
/bin/bash

# Activate the environment and install ipykernel
conda activate test_env
conda install -c conda-forge ipykernel

# Register it as a Jupyter kernel
python -m ipykernel install --user --name=test_env
```

The environment now appears as a notebook option in the launcher (reload the page if it does not show immediately).

### Registering an R environment as a kernel

```bash
/bin/bash

# Create an environment with R and the IRkernel package
conda create -n r44_env -c conda-forge r-base=4.4 r-irkernel
conda activate r44_env

# Enter R and register the kernel
R
```

Then, inside R:

```r
IRkernel::installspec(name = 'ir44', displayname = 'R 4.4')
```

### Listing and removing kernels

```bash
# List registered kernels
jupyter kernelspec list

# Remove a kernel (this does not delete the conda environment)
jupyter kernelspec uninstall test_env
```

## For RStudio users

There is no RStudio on the hub, but JupyterLab covers the same ground: notebooks with R code and markdown cells, a file browser, plots inline, and a terminal. A short orientation for R users coming from RStudio: [Jupyter notebooks for R users](https://www.datacamp.com/blog/notebooks-for-r-users).

If you want Python and R in the same notebook, see [this tutorial](https://anderfernandez.com/en/blog/how-to-program-with-python-and-r-in-the-same-jupyter-notebook/).

## File system and storage

| Path | Size | Visibility | Purpose |
|---|---|---|---|
| `/home/jovyan/` (`~`) | 20 GB | Only you | Your personal working area. Persists between sessions. |
<!--| `/home/jovyan/shared/` | 40 TB | All hub users | Shared reference data, conda recipes, and project folders. |

Suggested structure within the shared folder:

- `conda_recipes/` – shared conda environment YAML files
- `data/` – reference and project data
- `projects/` – create a subfolder for your project or team
- `user_folders/` – create a personal folder here to share files with others -->

> [!NOTE]
> If you require more space, please discuss this with your supevisor who will make the request.

## Customisation

### Dark mode

**Settings → Theme → JupyterLab Dark**

### Colours in `ls`

Add the following to `~/.bashrc`:

```bash
alias ls='ls --color=auto'
eval "$(dircolors -b)"
```

## Data policy

Data hosted on this platform may be subject to access and use agreements. **Do not copy data off the platform without written permission from the data owner or project lead.** If in doubt, ask before transferring anything.

## Getting help

- Email: [naveed.ishaque@bih-charite.de](mailto:naveed.ishaque@bih-charite.de)
- Or open an issue in this repository

When reporting a problem, include what you were trying to do, the exact command or action, and the full error message.

---

*This JupyterHub instance is hosted on the [de.NBI Cloud](https://www.denbi.de/cloud) and operated by the [Health Data Group](https://www.hidih.org/research/health-data) of the [BIH Center of Digital Health](https://www.hidih.org/). Please acknowledge de.NBI in publications that made use of this resource.*
