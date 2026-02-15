---
title: Using Jupyter Lab as a Lightweight Remote Desktop
date: 2025-10-04
categories:
  - DevOps, SysAdmin
tags:
  - Reference
---

# Using Jupyter Lab as a Lightweight Remote Desktop

Jupyter Lab isn't just for data science - it's a powerful all-in-one environment offering:

- Programming (Python, R, etc.)
- Shell Access
- File Browser & Editor
- Plugins

All within your browser, without heavy windowing systems or VNC, making it a great alternative to traditional remote desktops!

## Setup Instructions

### Server Side

First, [install Conda and set up a Conda environment](/2025/07/10/Managing-Development-Environments-with-Conda/).

#### Install Jupyter Lab

```bash
conda install -c conda-forge jupyterlab
```

Install kernels for Jupyter Lab:

Python (IPyKernel):

```bash
conda install -c conda-forge ipykernel
```

R (IRKernel):

```bash
conda install -c conda-forge r-irkernel
```

#### Launch Jupyter Lab and Make it accessible

Start a `tmux` session.

Start JupyterLab in it, listening on localhost:8888, with no password/token:

```bash
jupyter lab \
    --ip=127.0.0.1 --port 8888 \
    --allow-root \
    --no-browser \
    --IdentityProvider.token='' \
    --ServerApp.password='' \
    --ServerApp.allow_remote_access=True
```

Then open another `tmux` pane, set up [push-pull-port](https://github.com/jifengwu2k/push-pull-port), and push port 8888 to port `$PRIVATE_PORT` (localhost only) of remote public server `$PUBLIC_HOST`:

```
sh push-local-port.sh -l 8888 -r $PRIVATE_PORT -u $PUBLIC_USER -h $PUBLIC_HOST -x
```

Then detach from the `tmux` session.

### Client Side

Set up [push-pull-port](https://github.com/jifengwu2k/push-pull-port).

Pull the remote port and forward it to your local `8888` port:

```
sh pull-remote-port.sh -r $PRIVATE_PORT -l 8888 -u $PUBLIC_USER -h $PUBLIC_HOST
```

Access in Browser: Open [http://localhost:8888/lab](http://localhost:8888/lab).

## Exporting Jupyter Notebooks

You can export a Jupyter Notebook to a Python script or Markdown file by using `nbconvert` to convert notebooks:

### Export to Python

```bash
jupyter nbconvert --to script your_notebook.ipynb
```

### Export to Markdown

```bash
jupyter nbconvert --to markdown your_notebook.ipynb
```

This will create `your_notebook.py` or `your_notebook.md` in the current directory.
