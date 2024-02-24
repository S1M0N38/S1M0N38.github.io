+++
title = 'Minimal Server Configuration'
date = 2023-12-04T22:55:58+01:00
draft = false
+++

For my [thesis](https://github.com/S1M0N38/master-thesis) at [CENTAI](https://centai.eu), I had to set up a server user for running Python. They gave me a username and password for SSH for a non-sudo user. The server is running Ubuntu 20.04 LTS.

## Enhanced SSH

SSH is the communication tunnel established between your local machine and the server.
To avoid manually entering the username and password every time, you can use [SSH keys](https://chat.openai.com/share/4caacbdb-60d0-478c-b5a6-b9782ec89b05) to authenticate yourself.

1. [Check for existing SSH keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)

1. If none exist, [generate a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).

1. Finally, [add the SSH key to the server](https://askubuntu.com/questions/4830/easiest-way-to-copy-ssh-keys-to-another-machine).

Then you can establish a connection to the server with:

```
ssh user@server \
  -L 8377:localhost:8377 \
  -L 8378:localhost:8378 \
  -L 8379:localhost:8379 \
```

I've also opened some ports for Jupyter Notebook/Lab, TensorBoard, Ollama, or other applications that expose an HTTP interface. It's practical to export this command as an alias in your `.bashrc` or `.zshrc` file.

If you plan to use git for version control, consider signing your commits using [SSH keys](https://calebhearth.com/sign-git-with-ssh).

## Familiarize with the Server

Using `cd` and `ls` commands is an effective way to explore the server and find if there are folders or partitions for specific uses (e.g., a partition to store a large amount of data). [Symbolic links](https://en.wikipedia.org/wiki/Symbolic_link) are handy for creating shortcuts to folders, even between different partitions.

[Tmux](https://github.com/tmux/tmux/wiki) is a terminal multiplexer pre-installed on many Linux distributions. It allows you to create multiple terminal sessions to start background processes, keep them running after closing the SSH connection, and re-attach to them later.

If you need to exchange files between your local machine and the server, you can use the [rsync](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories) command. For instance, you can upload a codebase to the server and download the produced artifacts.

## Install Python

My preferred way to install different Python versions is to use [pyenv](https://github.com/pyenv/pyenv), but it requires installing [some dependencies](https://github.com/pyenv/pyenv/wiki#suggested-build-environment) using sudo. An alternative is to use [miniconda](https://docs.conda.io/projects/miniconda/en/latest), which is a Python distribution.

When running the miniconda installation script, it creates a virtual environment called `base`. I use that to install programs that I want to be available system-wide in all virtual environments, such as *Jupyter Notebook/Lab* and *nvitop*, so I keep this environment always active.

### Virtual Environments

When working on a project, it is good practice to create a virtual environment to avoid conflicts with other projects. Miniconda offers its own way to manage Python virtual environments, but I prefer to stick to the more simple built-in [venv](https://docs.python.org/3/library/venv.html) module.

When I enter the project directory, I immediately activate the corresponding virtual environment so that every time I use the `python` command, it refers to the Python version of the virtual environment with all the installed dependencies.

For instance, if you need to install dependencies from a `requirements.txt` file, you can do:

```
python -m pip install -r requirements.txt
```

The default prompt in Ubuntu provides information about the active virtual environment, that is

```
(my-project-venv) (base) user@servername:~$
```

So when I type the command `python`, it uses the executable contained in `my-project-venv` virtual environment. When I use the `jupyter` command, it uses the executable contained in the `base` virtual environment because there is no such executable in `my-project-venv`.

### Jupyter Notebook

You can [install Jupyter](https://anaconda.org/anaconda/jupyter) in `base` but use it to run notebooks from a project with dependencies installed in `my-project-venv`. This can be done using Python kernels, which is the way to tell Jupyter which Python executable to use.

To do so, you need to pip install `ipykernel` in `my-project-venv` and register it as a kernel in Jupyter.

```
python -m pip install ipykernel
python -m ipykernel install --user --name=my-project-venv
```

When you open a notebook, you can select the kernel to use from the menu *Kernel > Change kernel*. This method allows for a single installation of Jupyter and multiple kernels for different projects.

When running Jupyter on a remote server, remember to choose one of the open SSH ports, for instance:

```
jupyter notebook --port=8377 --no-browser
```

## Monitoring Server Resources

Monitoring server resources is crucial for debugging purposes, restricting, or fully utilizing resources.

- [df](https://www.man7.org/linux/man-pages/man1/df.1.html): command to check disk partition usage.

- [du](https://man7.org/linux/man-pages/man1/du.1.html): command to check the size of a directory.

- [htop](https://htop.dev): a pre-installed process viewer for monitoring CPU and RAM usage.

- [nvitop](https://github.com/XuehaiPan/nvitop): NVIDIA-GPU process viewer. Install with:

```
conda install -n base -c conda-forge nvitop
```

Some common pitfalls in resource usage include:

- **Running out of GPU memory**. This can be caused by too large a batch size or too large a model to fit in memory.

- **Leaving a Jupyter notebook running**. If the notebook allocates RAM or GPU memory, it keeps the resources allocated until the kernel is restarted or the notebook is closed.

- **Using all the CPU cores**. This can be caused by libraries that use multiprocessing under the hood. For instance, some routines in Python scientific libraries ([scikit-learn](https://scikit-learn.org/stable/computing/parallelism.html), [umap](https://umap-learn.readthedocs.io/en/latest/faq.html#umap-is-eating-all-my-cores-help)) can make use of multiple threads. In those cases, you can limit the number of threads by setting environment variables, i.e.,

```python
import os

# limit computation to 4 threads
os.environ['NUMBA_NUM_THREADS'] = '4'
os.environ['OMP_NUM_THREADS'] = '4'
os.environ['MKL_NUM_THREADS'] = '4'
os.environ['OPENBLAS_NUM_THREADS'] = '4'
os.environ['BLIS_NUM_THREADS'] = '4'

# your code here ...
```
