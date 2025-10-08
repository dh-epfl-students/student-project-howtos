# DHLAB and LHST student projects - Technical specifications  :hammer_and_wrench:

:point_right: For information on generic organisation of projects, see these [slides](https://docs.google.com/presentation/d/1z6iEMMcHeRg0eRZqO0SqqvbVMh0pFAl-WYM3wPpPHRI/edit?usp=sharing) (accessible with EPFL gdrive login).

Sections:
- [GitHub repository](#github-repository)
- [Working with a remote EPFL server](#working-with-a-remote-epfl-server)
- [Managing jobs on the cluster](#managing-jobs-on-the-cluster)
- [GPU usage](#gpu-usage)
- [Data management](#data-management)
- [Best practices for working on a cluster node (DHLAB - `iccluster040`)](#best-practices-for-working-on-a-cluster-node-dhlab---iccluster040)
- [Debugging and troubleshooting](#debugging-and-troubleshooting)
- [Reproducibility and good practices](#reproducibility-and-good-practices)

---

## GitHub repository

### Naming

- use lower case;  
- use hyphens to separate tokens;  
- if related to a larger project, start with the name of this project, followed by the name of your project (e.g. `impresso-image-classification` where `impresso` would be the name of the project);  
- in case of doubt, ask your supervisors.  

### Structure

You are free to structure your repository as you wish, but we advise having:

- a `notebooks` folder, for your working notebook;  
- a `lib` folder, in case you convert your notebook to scripts with a command line interface;  
- a `report` folder, where you put the PDF and LaTex sources of your report;  
- a README, with the information specified below.  

### What should be in your README

- **Basic information**
  - your name  
  - the names of supervisors  
  - the academic year  
- **About**: include a brief introduction of your project.  
- **Research summary**: include a brief summary of your approaches/implementations and an illustration of your results.  
- **Installation and Usage**
  - dependencies: platform, libraries (for Python include a `requirements.txt` file)  
  - compilation (if necessary)  
  - usage: how to run your code  
- **License**    
  We encourage you to choose an open license (e.g. AGPL, GPL, LGPL or MIT).  
  License files are already available in GH (add new file, start typing license, choices will appear).  
  You can also add the following at the end of your README:       

        project_name - Jean Dupont    
        Copyright (c) Year EPFL    
        This program is licensed under the terms of the [license]. 
	    
---

## Working with a remote EPFL server

If necessary, your lab (DHLAB or LHST) can grant you access to a machine on the IC cluster:

- ask your supervisor to gain access;
- you need to be on campus or use VPN to access the machine;
- $USER = your Gaspar username
- login with gaspar credentials: ```ssh $USER@iccluster0XX.iccluster.epfl.ch where 'XX' is the machine number```     

### Resources

- 256GB of RAM  
- 2 GPUs  
- 200GB of disk space on `/`  
- 12TB of disk space under `/scratch` (For DHLAB, THIS has newly changed! to `/rcp-scratch/iccluster040_scratch/`)

⚠️ **Important**: the machine is shared and given the small size of `/`, do not store your data (i.e. datasets, intermediate results, models, etc.) in your `home` but under `/rcp-scratch/iccluster040_scratch/students/$USER/`.

When first connecting via SSH you will see a fingerprint message. Type `yes` to continue.  
```sh
The authenticity of host 'iccluster0XX.iccluster.epfl.ch (XX.XX.XX.XX)' can't be established.
ECDSA key fingerprint is SHA256:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

---

### Moving Data to and from the Remote Server

Efficient data transfer is essential when working with remote cluster machines. You can use either `scp` or `rsync` to copy files or entire folders between your **local machine** and the **cluster**.

#### Using `scp` (Secure Copy)

**From your local machine to the remote server:**
```bash
scp -r /path/to/local/file.txt $USER@iccluster0XX.iccluster.epfl.ch:/scratch/students/$USER/
(DHLAB iccluster040) scp -r /path/to/local/file.txt $USER@iccluster040.iccluster.epfl.ch:/rcp-scratch/iccluster040_scratch/students/$USER/
```

**From the remote server to your local machine:**
```bash
scp $USER@iccluster0XX.iccluster.epfl.ch:/scratch/students/$USER/file.txt /path/to/local/
(DHLAB iccluster040) scp $iccluster040.iccluster.epfl.ch:/rcp-scratch/iccluster040_scratch/students/$USER/file.txt /path/to/local/
```

**Copying a folder recursively:**
```bash
scp -r $USER@iccluster0XX.iccluster.epfl.ch:/scratch/students/$USER/folder /path/to/local/
(DHLAB iccluster040) scp -r $iccluster040.iccluster.epfl.ch:/rcp-scratch/iccluster040_scratch/students/$USER/file.txt /path/to/folder

```

#### Using `rsync` (Recommended for large or repeated transfers)

`rsync` is more efficient for large datasets or syncing folders incrementally.

**From your local machine to the remote server:**
```bash
rsync -avh /path/to/local/folder/ $USER@iccluster0XX.iccluster.epfl.ch:/scratch/students/$USER/folder/
(DHLAB iccluster040) rsync -avh /path/to/local/folder/ $USER@iccluster040.iccluster.epfl.ch:/rcp-scratch/iccluster040_scratch/students/$USER/folder/
```

**From the remote server to your local machine:**
```bash
rsync -avh $USER@iccluster0XX.iccluster.epfl.ch:/scratch/students/$USER/folder/ /path/to/local/folder/
(DHLAB iccluster040) rsync -avh $USER@iccluster040.iccluster.epfl.ch:/rcp-scratch/iccluster040_scratch/students/$USER/folder/ /path/to/local/folder/
```

**Common flags used:**

- `-a`: archive mode (preserves permissions, symbolic links, etc.)  
- `-v`: verbose (shows what’s happening)  
- `-h`: human-readable sizes  
- `--progress`: (optional) shows progress during transfer
  
> ⚠️ **Important:** Always transfer data to and from the `/scratch/students/$USER/` directory, not `/home`, to avoid quota limits and ensure good practices on shared machines.

> ⚠️ **Important:** For DHLAB, on node `iccluster040`, always transfer data to and from the `/rcp-scratch/iccluster040_scratch/students/$USER/` directory, not `/home`, to avoid quota limits and ensure good practices on shared machines.

---

## Best practices for working on a cluster node (DHLAB - `iccluster040`)

The `/home` partition on **iccluster040** is very small and shared across all users. To avoid filling it up and causing problems for everyone, please follow these guidelines.

### Code

- **Where to put it**  
  - For student projects:  
    `/rcp-scratch/iccluster040_scratch/students/YOUR_USERNAME/`  
  - Otherwise:  
    `/rcp-scratch/iccluster040_scratch/YOUR_USERNAME/`

- **Why**  
  - `/home` (root) is small and not meant for heavy use.  
  - Storing in `/rcp-scratch` avoids quota issues and prevents system lock-ups of `iccluster040`.  
  - Keep your code in **Git** and clone it on `iccluster040` or other machines/nodes (best practice everywhere).  
  - (Extra) If you have access to **Run:AI**, you can run the code directly from there.

### Data (datasets, resources, etc.)

- **Where to put it**  
  - Short-term (work in progress):  
    `/rcp-scratch/iccluster040_scratch/students/YOUR_USERNAME/`  
  - Long-term (final datasets or valuable resources, if you have access):  
    `/mnt/u12632_cdh_dhlab_002_files_nfs/` on `cdhvm0002.xaas.epfl.ch` (NAS)

- **Why**  
  - `/scratch` has been copied to `~/rcp-scratch/iccluster040_scratch/` but **may be wiped**.  
  - `/home` is space-limited and **not safe for long-term storage**.  

---

## Practical steps to reduce `/home` usage

- **Move your conda environments**  
  Follow [these instructions](https://github.com/dh-epfl-students/student-project-howtos?tab=readme-ov-file#conda),  
  but instead of `/scratch/students/` use:  
  `/rcp-scratch/iccluster040_scratch/students/YOUR_USERNAME/`

- **Move your `.cache` directory**  
  From `/home/YOUR_USERNAME` to:  
  `/rcp-scratch/iccluster040_scratch/students/YOUR_USERNAME/.cache/`

  Create the folder first:
  ```bash
  mkdir -p ~/rcp-scratch/iccluster040_scratch/students/YOUR_USERNAME/.cache/
  ```
  Then export the following variables:
  ```bash
  export HF_HOME=~/rcp-scratch/iccluster040_scratch/students/YOUR_USERNAME/.cache/
  export HF_BASE=~/rcp-scratch/iccluster040_scratch/students/YOUR_USERNAME/.cache/
  export TRANSFORMERS_CACHE=~/rcp-scratch/iccluster040_scratch/students/YOUR_USERNAME/.cache/
  ```
  You can either run these commands every time or add them permanently to your ~/.bashrc.

---

### How to work with Python on a cluster node

⚠️ **Important:** Do not install environments in `/home`. Always use the scratch space.

- **Generic cluster nodes**:  
  Use `/scratch/students/$USER/`

- **DHLAB-specific node (`iccluster040`)**:  
  Use `/rcp-scratch/iccluster040_scratch/students/$USER/`

---

- Create a **local** Python environment using `conda`, `virtualenv`, or `pipenv`.  
- **Note**: Before creating environments, you need to create your user folder, e.g.:  
  ```sh
  mkdir -p /rcp-scratch/iccluster040_scratch/students/$USER
  ```
  (replace with `/scratch/students/$USER/` on other nodes).  
- To easily code locally and run things remotely, configure your IDE to sync code to the remote server (e.g. [PyCharm](https://www.jetbrains.com/help/pycharm/creating-a-remote-server-configuration.html), [VS Code](https://code.visualstudio.com/docs/remote/ssh-tutorial)).

---

#### conda

Configure conda so environments are stored in the scratch space:

```sh
# On generic nodes
conda config --add envs_dirs /scratch/students/$USER/.conda/envs
conda config --add pkgs_dirs /scratch/students/$USER/.conda/pkgs

# On iccluster040
conda config --add envs_dirs /rcp-scratch/iccluster040_scratch/students/$USER/.conda/envs
conda config --add pkgs_dirs /rcp-scratch/iccluster040_scratch/students/$USER/.conda/pkgs
```

Then create a new environment with Python 3.11:
```sh
export PYTHONUTF8=1
export LANG=C.UTF-8
conda create -n py311 python=3.11 anaconda
```

Activate: `source activate py311`  
Deactivate: `source deactivate`

---

#### virtualenv

Default system Python is 3.8, but we recommend using the latest version.  

```sh
# Generic nodes
virtualenv /scratch/students/$USER/testenv

# iccluster040
virtualenv /rcp-scratch/iccluster040_scratch/students/$USER/testenv

source /.../students/$USER/testenv/bin/activate
```

Avoid memory errors:
```sh
mkdir /.../students/$USER/tmp
export TMPDIR=/.../students/$USER/tmp/
pip install torch
```

(Replace `/.../students/$USER/` with either `/scratch/students/$USER/` or `/rcp-scratch/iccluster040_scratch/students/$USER/` depending on the node.)

---

#### pipenv

To avoid `/home` usage, configure pipenv to use scratch space:

```sh
# Generic nodes
mkdir -p /scratch/students/$USER/.pipenv_tmpdir
export TMPDIR="/scratch/students/$USER/.pipenv_tmpdir"

# iccluster040
mkdir -p /rcp-scratch/iccluster040_scratch/students/$USER/.pipenv_tmpdir
export TMPDIR="/rcp-scratch/iccluster040_scratch/students/$USER/.pipenv_tmpdir"
```

Also add to your `~/.bashrc`:  
```sh
export PIPENV_VENV_IN_PROJECT=1
```

	
Some sources: 
- [temporary directory](https://github.com/pypa/pip/issues/5816)
- [create pipenv in current directory](https://stackoverflow.com/questions/50598220/pipenv-how-to-force-virtualenv-directory)

### How to access a notebook on a remote server

To access an instance of Jupyter notebook or Jupyter Lab running on a remote server, you can either configure Jupyter accordingly or use an SSH tunnel.

#### Configuring Jupyter notebook for remote access

Have a look at the [official Jupyter documentation](https://jupyter-notebook.readthedocs.io/en/stable/public_server.html#running-a-notebook-server). 

In summary, you have to:

- Run `jupyter notebook --generate-config`. This will create a `.jupyter` folder in your home (hidden, use `ls -a`).
- Use `jupyter notebook password` to set a password.
- Edit the `jupyter_notebook_config.py` file in order to set the port where Jupyter will broadcast. The following three lines are needed, all the rest can be commented:

```
c = get_config()
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.port = XXX <= change this port; use the last four digits of your SCIPER number (to avoid colliding with other people on the same port).
```

You can ignore SSL certificates.

Notebook will be accessible at:
```
http://iccluster0XX.iccluster.epfl.ch:XXXX
```
In order to leave it open while you are executing things, you can run the notebook in `screen` (see below).

#### Using an SSH tunnel

You can also use an SSH tunnel, which is easier, but somewhat more brittle (you will need to reconnect the SSH tunnel when you lose the connection, e.g. because you suspended your machine).

- Connect to the remote server by setting up an SSH tunnel. Run the following command from your local machine. As the port number XXXX, use the last four digits of your SCIPER number (to avoid colliding with other people on the same port):  
`ssh -L XXXX:localhost:XXXX [gasparname]@iccluster0NN.iccluster.epfl.ch`
- Launch Jupyter notebook or lab on the node (again replacing XXXX with the same port number):    
`jupyter notebook --no-browser --port=XXXX`

Your notebook is now accessible at `https://localhost:XXXX`. You may need a token, so look at the message given by Jupyter notebook / lab when you run it.

In order to leave it open while you are executing things, you should run the notebook in a `screen` (see below).

---

### How to create another shell session and detach from it: use `screen`     

**Main commands:**

- create a session: `screen -S name_of_the_session`  =>  you are in
- "detach" from screen session: `Ctrl-A D` => session still running, you are out
- reconnect (reattach) to a session: `screen -r name_of_the_session`
- kill a session: from within the session, `Ctrl-A K`
- in case you are reconnecting from inside (by mistake):  `screen -rd name_of_the_session`
- to list all active screens: `screen -ls`

---

## Managing jobs on the cluster

Cluster nodes are shared. To avoid blocking others:

- Always run heavy processes inside `screen` or `tmux`.  
- Monitor resources:  
  ```bash
  top
  htop
  nvidia-smi -l 2
  ```
- Kill runaway processes:  
  ```bash
  ps -u $USER
  kill -9 <PID>
  ```
  
Some sources: 
- Tutorial: [https://linuxize.com/post/how-to-use-linux-screen/](https://linuxize.com/post/how-to-use-linux-screen/) 
- Full documentation: [https://linux.die.net/man/1/screen](https://linux.die.net/man/1/screen)
- Also available via `man screen`

**Steps to work with a notebook in a screen:**

- `cd [your repo]`
- `screen -S work` => you are in a screen named "work" where you will launch the notebook
- activate your env
- start Jupyter notebook (`jupyter notebook`, or `jupyter notebook --no-browser --port=XXXX` if you use a SSH tunnel)
- open the URL in your web browser
  - `http://iccluster0XX.iccluster.epfl.ch:XXXX` if you configured remote access
  - `http://localhost:XXXX` if you use an SSH tunnel
- if everything is ok, then detach the screen (`Ctr-a d`). You can now work in the notebook, open and close your browser as you want, it will keep running. 

---

## GPU usage

- Default precision: **FP32 (float32)**  
- FP16/bfloat16 may not be supported on some GPUs.

Example (PyTorch):
```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model.to(device)
```

Free memory when done:
```python
del model
torch.cuda.empty_cache()
```

⚠️ Don’t keep idle notebooks holding GPUs.

---

## Data management

- Always store under:
  ```
  /scratch/students/$USER/data/
  ```
- Compress large corpora with `.tar.gz` or `.bz2`.  
- Coordinate with supervisors before duplicating datasets.  
- Clean up unused checkpoints and logs.  
- Use `rsync --progress` for efficient large transfers.  

---

## Debugging and troubleshooting

- **Environment issues**  
  - Check: `which python`  
  - If `pip` fails:  
    ```sh
    mkdir /rcp-scratch/iccluster040_scratch/students/$USER/tmp
    export TMPDIR=/rcp-scratch/iccluster040_scratch/students/$USER/tmp
    ```

- **GPU errors**  
  - `CUDA out of memory`: reduce batch size, sequence length, or use gradient accumulation.  
  - Driver mismatch: restart session, check with `nvidia-smi`.  

- **SSH disconnects**  
  - Use `screen` or `tmux`  

---

## Reproducibility and good practices

- Version code with Git  
- Save environment files (`requirements.txt`, `environment.yml`)  
- Fix random seeds for experiments  
- Document experiments in README or notebooks  
- Clean up unused files on the cluster  
- Use open licenses (AGPL, GPL, LGPL, MIT)  

---
