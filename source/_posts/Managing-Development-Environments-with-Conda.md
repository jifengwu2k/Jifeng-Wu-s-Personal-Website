---
title: Managing Development Environments with Conda
date: 2025-07-10
categories:
  - DevOps, SysAdmin
tags:
  - Reference
---

When juggling multiple data science or machine learning projects, maintaining isolated, reproducible environments is crucial. **conda** has emerged as the *de facto standard* in Python for managing such environments. But *why* should you use conda over alternatives like `venv`, `poetry`, `pipenv`, system package managers like `apt`, or container tools like Docker? Let's find out - and get you set up!

## Why Conda?

Conda offers several key advantages:

- Beyond Python Packages: Unlike `venv`, `pipenv`, or `poetry`, conda handles both Python packages *and* complex system-level dependencies, such as CUDA toolkits for GPU acceleration, C/C++ compiler toolchains, and other command-line utilities.
- No Admin Required: With system-wide package managers like `apt`, you'll often need root privileges to install software. In comparison, Conda lets you install everything in user-space.
- Better than Docker for Rapid Iteration: While Docker excels at packaging for production, it often means verbose Dockerfiles, long build times, and a less pleasant development experience. Conda offers a lightweight, rapid way to set up and switch between environments on your local machine.

## Installing Conda (Miniconda)

The preferred way to get started is with Miniconda, a minimal conda installer. Here's a step-by-step guide:

Create an Installation Directory:

```bash
mkdir -p ~/miniconda3
```

Download the Miniconda Installer:

- Linux x86_64: `wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh`
- Linux aarch64: `wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh -O ~/miniconda3/miniconda.sh`
- Linux s390x: `wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-s390x.sh -O ~/miniconda3/miniconda.sh`
- Linux ppc64le: `wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-ppc64le.sh -O ~/miniconda3/miniconda.sh`
- Linux x86: `wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86.sh -O ~/miniconda3/miniconda.sh`
- Linux armv7l: `wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-armv7l.sh -O ~/miniconda3/miniconda.sh`
- macOS arm64: `curl https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh -o ~/miniconda3/miniconda.sh`
- macOS x86_64: `curl https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh -o ~/miniconda3/miniconda.sh`
- macOS x86: `curl https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86.sh -o ~/miniconda3/miniconda.sh`

Run the Installer

```bash
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3 && rm ~/miniconda3/miniconda.sh 
```

What does this do?

- `-b`: Run in batch (no interactive prompts)
- `-u`: Unpack the installer
- `-p ~/miniconda3`: Install into this directory
- `&& rm ...`: Clean up the installer file

## Activate Conda

```bash
source ~/miniconda3/bin/activate
```

You'll need to run this every time you log into your shell.

## Managing Conda Environments

List all environments:

```bash
conda env list
```

Remove an environment:

```bash
conda env remove --name ENV_NAME
```

Replace `ENV_NAME` with the actual name (e.g., `myenv`). This will delete the environment and all its files.
