# DHLAB and LHST student projects — Technical specifications 🔧

> 📌 For information on generic organisation of projects, see these [slides](https://docs.google.com/presentation/d/1z6iEMMcHeRg0eRZqO0SqqvbVMh0pFAl-WYM3wPpPHRI/edit?usp=sharing) (accessible with EPFL gdrive login).

---

## Table of Contents

1. [GitHub repository](#1-github-repository)
2. [Connecting to the remote server](#2-connecting-to-the-remote-server)
3. [IDE setup for remote development](#3-ide-setup-for-remote-development)
4. [Environment setup](#4-environment-setup)
5. [Code and data storage](#5-code-and-data-storage)
6. [Security](#6-security)
7. [Working on the remote server](#7-working-on-the-remote-server)
8. [Managing CPU and GPU resources](#8-managing-cpu-and-gpu-resources)
9. [Debugging and troubleshooting](#9-debugging-and-troubleshooting)
10. [Reproducibility and good practices](#10-reproducibility-and-good-practices)
11. [End-of-project checklist](#11-end-of-project-checklist)

---

## 1. GitHub repository

### Creating your repository

Before starting any coding, you must set up a GitHub repository **in concertation with your supervisor** in order to:

- confirm the GitHub organisation or account under which the repository should be created;
- define naming conventions consistent with the project;
- assign the appropriate license (see [Licenses](#licenses) below).

Working repositories are set to private, they are made public at the end of the project.

### Repository Naming conventions

- Use lower case.
- Use hyphens to separate tokens.
- If related to a larger project, prefix with that project's name (e.g. `impresso-image-classification`).
- In case of doubt, ask your supervisor.

### Repository structure

You are free to structure your repository as you wish, but we advise:

```
project-name/
├── notebooks/      # Working notebooks
├── lib/            # Scripts with CLI interfaces (converted from notebooks)
├── report/         # PDF and LaTeX sources of your report
├── requirements.txt
└── README.md
```

### README content

Your README must include:

- **Basic information**: your name, supervisors' names, academic year.
- **About**: a brief introduction to the project.
- **Research summary**: a brief summary of your approaches/implementations and an illustration of your results.
- **Installation and Usage**: dependencies (platform, libraries), compilation if necessary, and how to run the code.
- **License**: as decided with your supervisor (see below).

### Licenses

The license is **chosen by your supervisor**, not by you. Your supervisor will indicate which open license applies (typically AGPL, GPL, LGPL, or MIT). Once confirmed, add the license file via GitHub (add new file → start typing "license" → pre-filled choices will appear) and add the following at the end of your README:

```
project_name - Jean Dupont
Copyright (c) Year Jean Dupont / EPFL
This program is licensed under the terms of the [license].
```

---

## 2. Connecting to the remote server

Your lab (DHLAB or LHST) can grant you access to a machine on the IC cluster. Ask your supervisor to obtain access.

### Requirements

- You must be on the EPFL campus or connected via the **EPFL VPN**. If you have not set it up yet, follow the [official EPFL VPN instructions](https://www.epfl.ch/campus/services/en/it-services/network-services/remote-connection/vpn/).
- `$USER` = your Gaspar username.

### Login

```sh
ssh $USER@iccluster0XX.iccluster.epfl.ch
```
Replace `XX` with the machine number provided by your supervisor. The DHLAB node is `iccluster028`.

When connecting for the first time, type `yes` to accept the fingerprint:
```
The authenticity of host 'iccluster0XX.iccluster.epfl.ch (XX.XX.XX.XX)' can't be established.
ECDSA key fingerprint is SHA256:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

### Setting up SSH keys (recommended)

Using SSH keys avoids typing your password on every connection and is required by some IDE integrations.

```sh
# On your LOCAL machine — generate a key pair if you don't have one
ssh-keygen -t ed25519 -C "your_email@epfl.ch"

# Copy your public key to the remote server
ssh-copy-id $USER@iccluster028.iccluster.epfl.ch
```

You can now connect without a password. Keep your private key secure and never share it.

### Available resources on the DHLAB node (`iccluster028`)

- 256 GB RAM
- 2 GPUs
- 200 GB disk on `/home` (shared — do not use for data storage)
- Several TB under `/rcp-scratch` (primary storage for all work)

### Checking your disk usage

```sh
du -sh ~                                    # your /home usage
du -sh /rcp-scratch/students/$USER/         # your scratch usage
df -h /rcp-scratch                          # total scratch space available
```

### Run:AI — alternative compute platform

If the cluster node is heavily loaded or you need more GPU resources, your lab may have access to **Run:AI**, EPFL's managed GPU scheduling platform. See [RCP portal](https://portal.rcp.epfl.ch) for documentation and access instructions. Ask your supervisor whether Run:AI is available for your project. 

> ⚠️ **The machine is shared by all lab members and students.** Before running anything intensive, always check current resource usage (see [Section 8](#8-managing-cpu-and-gpu-resources)).

---

## 3. IDE setup for remote development

When working on a cluster, your code can exist in three places: your **local machine**, the **remote server**, and **GitHub**. Before writing any code, decide on a workflow and stick to it.

> ⚠️ **Edit code in one place only.** Never edit the same file simultaneously on your local machine and on the server — this leads to conflicts and lost work. GitHub is your single source of truth; commit and push regularly from whichever side you are working on.

### Workflow options

#### Option A — Edit directly on the server 

Your IDE connects to the remote server over SSH. You edit files as if they were local, but everything runs and is saved on the cluster. You push to GitHub from the server.

**This is the simplest setup**: one copy of the code, no sync to configure, no risk of local/remote divergence.

##### VS Code — Remote SSH

1. Install the [Remote - SSH extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh).
2. Open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`) → **Remote-SSH: Connect to Host** → enter:
   ```
   $USER@iccluster028.iccluster.epfl.ch
   ```
3. Once connected, open your project folder: `/rcp-scratch/students/$USER/your-project/`.
4. Set the Python interpreter via the Command Palette → **Python: Select Interpreter** → point it to your remote conda or virtualenv binary (e.g. `/rcp-scratch/students/$USER/.conda/envs/py311/bin/python`).
5. Open the integrated terminal (`Ctrl+`` ` ``) — it runs directly on the server.

📖 [VS Code Remote SSH tutorial](https://code.visualstudio.com/docs/remote/ssh-tutorial)

##### PyCharm — SSH interpreter

1. Go to **Settings → Project → Python Interpreter → Add Interpreter → On SSH**.
2. Enter the host (`iccluster028.iccluster.epfl.ch`) and authenticate with your Gaspar credentials or SSH key.
3. Point the interpreter to the Python binary in your remote environment (e.g. `/rcp-scratch/students/$USER/.conda/envs/py311/bin/python`).

📖 [PyCharm SSH interpreter documentation](https://www.jetbrains.com/help/pycharm/creating-a-remote-server-configuration.html)

---

#### Option B — Edit locally, auto-sync to the server

You write code on your local machine. Your IDE automatically uploads every saved change to the server. You push to GitHub from your local machine.

**Use this if** you need to work offline, or if you prefer your local tooling (linters, plugins, etc.). Be disciplined: always let the IDE sync before running anything on the server, and never edit files on the server directly while in this mode.

##### PyCharm — Automatic deployment

PyCharm can mirror your local project to the server on every save:

1. Go to **Settings → Build, Execution, Deployment → Deployment → Add** → choose **SFTP**.
2. Configure the connection to `iccluster028.iccluster.epfl.ch` with your credentials or SSH key.
3. Under **Mappings**, set the remote path to `/rcp-scratch/students/$USER/your-project/`.
4. Enable **Tools → Deployment → Automatic Upload** — PyCharm will sync every saved file immediately.
5. Set the remote interpreter as in Option A above.

📖 [PyCharm deployment documentation](https://www.jetbrains.com/help/pycharm/deploying-applications.html)

##### VS Code — alternative

VS Code does not have built-in auto-sync, but the [SFTP extension](https://marketplace.visualstudio.com/items?itemName=Natizyskunk.sftp) provides similar functionality. Configure it with the server address and your remote project path.

---

> 💡 With either option, the integrated terminal in your IDE opens directly on the remote server, letting you run scripts, manage environments, and monitor jobs without a separate SSH window.

---

## 4. Environment setup

> ⚠️ **Never install environments or store packages in `/home`.** The `/home` partition is small and shared. Always use `/rcp-scratch/students/$USER/`.

### Create your user folder first

```sh
mkdir -p /rcp-scratch/students/$USER
```

### Setting up a Python environment

#### conda (recommended)

**Step 1 — Configure conda to store environments on scratch space (do this before creating any environment):**

```sh
conda config --add envs_dirs /rcp-scratch/students/$USER/.conda/envs
conda config --add pkgs_dirs /rcp-scratch/students/$USER/.conda/pkgs
```

**Step 2 — Create a new environment:**

```sh
export PYTHONUTF8=1
export LANG=C.UTF-8
conda create -n projectxyz-py311 python=3.11
```

Activate: `source activate projectxyz-py311`  
Deactivate: `source deactivate`

---

**If you already created a conda environment in `/home` by mistake**, move it to scratch space:

```sh
# 1. Export the existing environment
conda activate your_env_name
conda env export > environment.yml
conda deactivate

# 2. Remove the old environment
conda env remove -n your_env_name

# 3. Make sure conda is configured for scratch (see Step 1 above)

# 4. Recreate the environment in scratch
conda env create -f environment.yml
```

---

#### virtualenv

```sh
virtualenv /rcp-scratch/students/$USER/testenv
source /rcp-scratch/students/$USER/testenv/bin/activate
```

To avoid memory errors during package installation:
```sh
mkdir /rcp-scratch/students/$USER/tmp
export TMPDIR=/rcp-scratch/students/$USER/tmp/
pip install torch
```

#### pipenv

```sh
mkdir -p /rcp-scratch/students/$USER/.pipenv_tmpdir
export TMPDIR="/rcp-scratch/students/$USER/.pipenv_tmpdir"
```

Also add to your `~/.bashrc`:
```sh
export PIPENV_VENV_IN_PROJECT=1
```

---

### Moving your `.cache` directory out of `/home`

Large model caches (e.g. from Hugging Face) can quickly fill `/home`. Move them to scratch:

```sh
mkdir -p /rcp-scratch/students/$USER/.cache/
```

Add this line to your `~/.bashrc` so they are set automatically on each login:
```sh
export HF_HOME=$SCRATCH/.cache/huggingface
```

Then reload: `source ~/.bashrc`

> 💡 `.bashrc` is not always sourced automatically at login. If your environment variables seem missing, run `source ~/.bashrc` manually.

### Where to put things

| Content | Location |
|---|---|
| Code / repositories | `/rcp-scratch/students/$USER/` |
| Datasets | `/rcp-scratch/students/$USER/data/` |
| Model checkpoints | `/rcp-scratch/students/$USER/checkpoints/` |
| conda environments | `/rcp-scratch/students/$USER/.conda/` |
| Hugging Face cache | `/rcp-scratch/students/$USER/.cache/` |

> ⚠️ Do **not** store any of the above under `/home`.


### Transferring data

#### Using `scp`

```sh
# Local → remote
scp -r /path/to/local/file.txt $USER@iccluster0XX.iccluster.epfl.ch:/rcp-scratch/students/$USER/

# Remote → local
scp $USER@iccluster0XX.iccluster.epfl.ch:/rcp-scratch/students/$USER/file.txt /path/to/local/
```

#### Using `rsync` (recommended for large or repeated transfers - but use with caution!)

```sh
# Local → remote
rsync -avh --progress /path/to/local/folder/ $USER@iccluster0XX.iccluster.epfl.ch:/rcp-scratch/students/$USER/folder/

# Remote → local
rsync -avh --progress $USER@iccluster0XX.iccluster.epfl.ch:/rcp-scratch/students/$USER/folder/ /path/to/local/folder/
```

> ⚠️ Be very careful about source and target paths with `rsync`. Ask your supervisor if unsure.

**Common flags:**
- `-a`: archive mode (preserves permissions, symlinks, etc.)
- `-v`: verbose
- `-h`: human-readable sizes
- `--progress`: shows transfer progress

### Data management best practices

- Compress large corpora: `.tar.gz` or `.bz2`.
- Coordinate with your supervisor before duplicating large datasets that may already be on the cluster.
- Clean up unused checkpoints, logs, and intermediate files regularly.

---

## 6. Security

> 🔴 **Never put passwords, API tokens, or any credentials in your code or in a file tracked by Git.**

### Rules

- Store secrets (API keys, tokens, passwords) in environment variables or in a `.env` file — never hardcoded in scripts or notebooks.
- Add `.env` to your `.gitignore` immediately when you create the repository.
- Use a password manager for personal credentials.

### Using `python-dotenv`

```sh
pip install python-dotenv
```

Create a `.env` file at the root of your project (never committed to Git):
```
HF_TOKEN=your_token_here
OPENAI_API_KEY=your_key_here
```

Load it in your code:
```python
from dotenv import load_dotenv
import os

load_dotenv()
token = os.getenv("HF_TOKEN")
```

### If you accidentally push a secret to Git

**Act immediately** — do not just delete the file in a new commit, as the secret remains visible in the Git history.

1. **Revoke/rotate the credential** straight away (API key, token, password) — assume it is compromised.
2. Notify your supervisor.
3. Remove the secret from the Git history using `git filter-repo` or BFG Repo-Cleaner, then force-push.
4. If the repository is public, treat the secret as fully compromised regardless of how quickly you acted.

📖 [GitHub guide: removing sensitive data](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)

---

## 7. Working on the remote server

### Keeping sessions alive with `screen`

Always run long-running processes inside `screen` (or `tmux`) so they survive SSH disconnects.

**Main `screen` commands:**

| Command | Effect |
|---|---|
| `screen -S session_name` | Create and enter a new session |
| `Ctrl-A D` | Detach from the current session (it keeps running) |
| `screen -r session_name` | Reattach to a running session |
| `screen -rd session_name` | Force reattach if already attached |
| `screen -ls` | List all active sessions |
| `Ctrl-A K` | Kill the current session |

📖 References: [linuxize tutorial](https://linuxize.com/post/how-to-use-linux-screen/) · [full documentation](https://linux.die.net/man/1/screen) · `man screen`

---

### Accessing a Jupyter notebook on the remote server

Configure Jupyter for direct remote access by following the [official documentation](https://jupyter-notebook.readthedocs.io/en/stable/public_server.html#running-a-notebook-server). Key steps:

```sh
jupyter notebook --generate-config
jupyter notebook password
```

Then edit `~/.jupyter/jupyter_notebook_config.py`:
```python
c = get_config()
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.port = XXXX  # use the last 4 digits of your SCIPER to avoid port conflicts
```

Your notebook will then be accessible at `http://iccluster028.iccluster.epfl.ch:XXXX`.

### Recommended workflow: notebook in a screen

```sh
cd /rcp-scratch/students/$USER/your-project
screen -S work          # create a screen session
source activate py311   # activate your environment
jupyter notebook
# Open http://iccluster028.iccluster.epfl.ch:XXXX in your browser
# Then detach: Ctrl-A D
```

You can now close your terminal and reconnect later — the notebook keeps running.

---

## 8. Managing CPU and GPU resources

> 🔴 **The cluster is a shared machine.** Before launching any compute-intensive job, you are required to check current resource usage. Failing to do so may block other users' work.

### Checking current usage

```sh
# CPU and memory usage by all users
htop

# GPU usage (refreshes every 2 seconds)
nvidia-smi -l 2

# Your own running processes
ps -u $USER
```

### Running jobs considerately

- Always run heavy jobs inside `screen` or `tmux` (never in a raw SSH session).
- Use `nice` to lower the scheduling priority of non-urgent jobs, leaving resources available to others:
  ```sh
  nice -n 10 python train.py        # lower CPU priority
  ```
- Close idle Jupyter notebooks that are holding GPU memory.
- Do not run multiple GPU-intensive jobs simultaneously without checking with your supervisor.

### If the machine is already heavily loaded

Do not launch additional intensive jobs. **Contact your supervisor** to coordinate — they can advise on scheduling, off-hours execution, or whether Run:AI (see [Section 2](#2-connecting-to-the-remote-server)) is a better option for your workload.

### Killing runaway processes

```sh
ps -u $USER           # list your processes and their PIDs
kill -9 <PID>         # force-kill a specific process
```

### GPU best practices (PyTorch)

**Default precision is FP32.** FP16/bfloat16 may not be supported on all GPUs.

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model.to(device)
```

Free GPU memory when done:
```python
del model
torch.cuda.empty_cache()
```

If you encounter `CUDA out of memory`: reduce batch size, sequence length, or use gradient accumulation.

---

## 9. Debugging and troubleshooting

### Python environment issues

```sh
which python          # verify you are using the right interpreter
which pip
```

If `pip install` fails with a temp-directory error:
```sh
mkdir /rcp-scratch/students/$USER/tmp
export TMPDIR=/rcp-scratch/students/$USER/tmp
pip install <package>
```

### GPU / CUDA errors

| Error | Solution |
|---|---|
| `CUDA out of memory` | Reduce batch size or sequence length; use gradient accumulation |
| Driver mismatch | Restart your session; verify with `nvidia-smi` |
| GPU not found | Check `torch.cuda.is_available()`; confirm device is free with `nvidia-smi` |

### SSH disconnects

Use `screen` or `tmux` — your session persists on the server regardless of your connection.

### `/home` quota exceeded

Move conda envs, `.cache`, and all data to `/rcp-scratch/students/$USER/` (see [Section 4](#4-environment-setup) and [Section 5](#5-code-and-data-storage)).

---

## 10. Reproducibility and good practices

- **Version all code with Git** and push to your GitHub repository regularly.
- **Save environment files**: `requirements.txt` (`pip freeze > requirements.txt`) or `environment.yml` (`conda env export > environment.yml`).
- **Fix random seeds** for all experiments to ensure reproducibility.
- **Document experiments** in your README or in notebooks (hyperparameters, dataset version, results).
- **Clean up** unused checkpoints, logs, and intermediate files on the cluster.
- **License**: apply the license chosen by your supervisor (see [Section 1](#1-github-repository)).

---

## 11. End-of-project checklist

Before considering your project complete, go through the following steps **with your supervisor**.

### Code and documentation

- [ ] All code is committed and pushed to the GitHub repository.
- [ ] The README is complete and up to date (see [Section 1](#1-github-repository)).
- [ ] The final report (PDF) is added to the `report/` folder in the repository.
- [ ] Dependencies are fully documented (`requirements.txt` or `environment.yml`).
- [ ] Experiments are documented (hyperparameters, dataset versions, key results).

### Data on the cluster

- [ ] Decide with your supervisor what data should be **kept** (e.g. final model weights, processed datasets) and what can be deleted.
- [ ] Remove unnecessary intermediate files, checkpoints, cached downloads, and temporary files from `/rcp-scratch/students/$USER/`.
- [ ] Confirm with your supervisor that no data needs to be archived or transferred elsewhere before your access is removed.

### Listing your project

Once the repository is finalised, add your project to the appropriate lab page so it is discoverable by future students and researchers:

- **DHLAB**: open a PR to add your project to [dh-epfl-students/EPFL-DHLAB-student-projects](https://github.com/dh-epfl-students/EPFL-DHLAB-student-projects)
- **LHST**: open a PR to add your project to [dh-epfl-students/EPFL-LHST-student-projects](https://github.com/dh-epfl-students/EPFL-LHST-student-projects)

Your entry should include your name, project title, academic year, a link to the GitHub repository, and a link to the report. Do this **in coordination with your supervisor**.
