# DHLAB and LSHT student projects - Technical specifications  :hammer_and_wrench:

:point_right: For information on generic organisation of projects, see these [slides]() (link broken at the moment, refer to email).

Sections:
- [GitHub repository](#github-repository)
- [Working with a remote EPFL server](#working-with-a-remote-EPFL-server)


## GitHub repository:

**Naming:**

- use lower case;
- use hyphens to separate tokens;
- if related to a larger project, start with the name of this project, followed by the name of your project (e.g. `impresso-image-classification`);
- in case of doubt, ask your supervisors;

**Structure:**    
You are free to structure your repository as you wish, but we advise to have:

- a `notebooks` folder, for your working notebook;
- a `lib` folder, in case you convert your notebook in scripts with a command line interface;
- a `report` folder, where you put the PDF and latex sources of your report;
- a README, with the information specified below.


**What should be in your README:**

- **Basic information**
    - your name
    - the names of supervisors
    - the academic year
- **About**: include a brief introduction of your project.
- **Research summary**: include a brief summary of your approaches/implementations and an illlustration of your results.
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
	    


## Working with a remote EPFL server

If necessary your lab (DHLAB or LHST) can grant you access to a machine on the IC cluster:

- ask your supervisor to gain access;
- you need to be on campus or use VPN to access the machine;
- login with gaspar credentials: ```ssh [gasparname]@iccluster0XX.iccluster.epfl.ch where 'XX' is the machine number```     
- the node usually has 
	- 256GB of RAM, 
	- 2 GPUs
	- 200GB of disk space on `/`
	- 12TB of disk space under `scratch`
-  :warning: **important**: the machine is shared and given the small size of `/`, do not store your data (i.e. the data you work with, intermediary results, models, various resources, etc.) in your `home` but under `scratch/students` where you can create your own folder. 


### For people using Python on a cluster node

- create a **local** python environment using conda or pipenv. You need to ensure your environment is on the `/scratch/`, not `/home/`, to do so check guides just below.
- to easily code locally and run things remotely, configure your IDE to save your code on the remote server as you code (e.g. with [PyCharm](https://www.jetbrains.com/help/pycharm/creating-a-remote-server-configuration.html), [Visual Studio Code](https://code.visualstudio.com/docs/remote/ssh-tutorial)).
     
#### pipenv

Note: the following procedure for pipenv has not been thoroughly tested, if you test it and there are other steps, please update this document via a pull request

To ensure you are not using `/home`, there are two things to do: 1) ensure your pipenv environments are not installed in `/home`, 2) ensure pip's temporary directory (where pip downloads the files before loading them into the environment) is not on `/home`.

1) Create the temporary directory `mkdir /scratch/<your-scracth-folder>/.pipenv_tmpdir`	
2) Add the following lines to your `/home/<user>/.bashrc/``file:
```sh
export PIPENV_VENV_IN_PROJECT=1 # tells pip to create the environment in the folder where you're creating it.
export TMPDIR="/scratch/<your-scracth-folder>/.pipenv_tmpdir" # tells pip to use this folder as the temporary directory
```
	
Some sources: [temporary directory](https://github.com/pypa/pip/issues/5816), [create pipenv in current directory](https://stackoverflow.com/questions/50598220/pipenv-how-to-force-virtualenv-directory)
	
#### conda

Before creating conda environments, you need to execute the following two commands. They configure conda so that environments (and packages) are stored on the `/scratch/` instead of the `/home/`:
```sh
conda config --add envs_dirs /scratch/$USER/.conda/envs
conda config --add pkgs_dirs /scratch/$USER/.conda/pkgs
```

Moreover, it is good practice to regularly clean your conda packages: `clean conda --all`

Documentation: [envs_dirs](https://conda.io/projects/conda/en/latest/user-guide/configuration/use-condarc.html#specify-environment-directories-envs-dirs), [pkgs_dirs](https://conda.io/projects/conda/en/latest/user-guide/configuration/use-condarc.html#specify-package-directories-pkgs-dirs ), [clean](https://conda.io/projects/conda/en/latest/commands/clean.html)

### How to access a notebook on a remote server

In order to access an instance of Jupyter notebook or Jupyter Lab running on a remote server, you can either configure Jupyter accordingly or use a SSH tunnel.

#### Configuring Jupyter notebook for remote access

Have a look at the [official Jupyter documentation](https://jupyter-notebook.readthedocs.io/en/stable/public_server.html#running-a-notebook-server). 

In summary, you have to:

- Run `jupyter notebook --generate-config`. This will create a `.jupyter` folder in your home (hidden, use `ls -a`).
- Use `jupyter notebook password` to set a password.
- Edit the `jupyter_notebook_config.py` file in order to set the port where jupyter will broadcast. The following three lines are needed, all the rest can be commented:

```
c = get_config()
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.port = XXX <= change this port; use the last four digits of your SCIPER number (to avoid colliding with other people on the same port).
```

You can ignore SSL certificates.

If you run `jupyter notebook` the notebook will start and be accessible at http://iccluster0XX.iccluster.epfl.ch:XXXX (you need to enter your password)

In order to leave it open while you are executing things, you can run the notebook in `screen` (see below).

#### Using a SSH tunnel

You can also use a SSH tunnel, which is easier, but somewhat more brittle (you will need to reconnect the SSH tunnel when you lose the connection, e.g. because you suspended your machine).

1. Connect to the remote server setting up a SSH tunnel. Run the following command from your local machine. As the port number XXXX, use the last four digits of your SCIPER number (to avoid colliding with other people on the same port):    
`ssh -L XXXX:localhost:XXXX [gasparname]@iccluster0NN.iccluster.epfl.ch`
2. Launch Jupyter notebook or lab on the node (again replacing XXXX with the same port number):    
`jupyter notebook --no-browser --port=XXXX`

Your notebook is now accessible at `https://localhost:XXXX`. You may need a token, so look at the message given by Jupyter notebook / lab when you run it.

In order to leave it open while you are executing things, you should run the notebook in a `screen` (see below).

### How to create another shell session and detach from it: use `screen`     

**Main commands:**

- create a session: `screen -S name_of_the_session`  =>  you are in
- go out of a session: `Ctrl-A D` => session still running, you are out
- reconnect to a session: `screen -r name_of_the_session`
- kill a session: from within the session, `Ctrl-A K`
- in case you are reconnecting from inside (by mistake):  `screen -rd name_of_the_session`
- to list all active screens: `screen -ls`

**Documentation:**

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
  - `http://localhost:XXXX` if you use a SSH tunnel
- if everything is ok then exit the screen (`Ctr-a d`). You can now work in the notebook, open and close your browser as you want, it will keep running. 
