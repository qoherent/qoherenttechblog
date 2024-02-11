---
title: "How to Run GNU Radio in a Python Virtual Environment."
date: 2024-02-10T01:16:34-04:00
draft: false
cover:
    # image img/example.jpg
    alt: 'Pull an External GNU Radio Installation into your Python Virtual Environment.'
    caption: 'Pull an External GNU Radio Installation into your Python Virtual Environment'
tags: ['GNU Radio','python','ubuntu', 'venv']
categories: ['how-to']
---



## Pull an External GNU Radio Installation into your Python Virtual Environment

This article explains how to access your system-level GNU Radio installation from within a pre-existing Python virtual environment. After completing these steps, you'll be able to import GNU Radio or any of its subpackages into your Python modules using simple import statements, such as: `from gnuradio import gr`. 

We provide an explanation and a recommendation before the solution, but to [jump straight to the solution, click here.](#h-example-solution)

## Motivation

[GNU Radio](https://wiki.gnuradio.org/index.php?title=Main_Page) is a free and open-source software development toolkit for software-defined radios. The easiest way to get started with GNU Radio is thought the GNU Radio Companion graphical user interface. The GNU Radio Companion allows you to create and execute signal processing applications by simply dragging, dropping, and connecting together predefined radio blocks. 

One of the key strengths of GNU Radio is its Python API, which allows users to control and interact with GNU Radio functions programmatically. Instead of designing signal processing flowgraphs graphically, you can use the power and flexibility of Python to manage GNU Radio flowgraphs. The benefits of using gnu radio in python are [outlined in this presentation from GR Con 2023.](https://github.com/TheWylieStCoyote/writing_GNURadio_blocks_python)

For more information on getting started with Python, please refer to W3Schools' online [Python Tutorial](https://www.w3schools.com/python/default.asp) or Replit’s [100 days of python course](https://replit.com/learn/100-days-of-python).

## Virtual Environments

Python dependencies are external libraries on which your project depends. There are typically many, each with their own respective dependencies. In Python, we use virtual environments to facilitate project-specific dependency management. Virtual environments helps us avoid inter-project dependency conflicts by providing isolated, sandboxed environments, each with their own, project-specific set of dependencies. This means we can use one version of a Python package for one project and a different version for another.

However, for virtual environments to be effective, you need to be able to install all project dependencies into the project's virtual environment. For this, we use [Pip](https://pypi.org/project/pip/). Pip is the standard package manager for Python, and allows users to easily install libraries and utilities from the Python Package Index ([PyPI](https://pypi.org/)) and other indexes. For instance, to install NumPy into our virtual environment, we would execute `pip install numpy` from the command line. Then, we would be able to import and use NumPy and its dependencies in our project.

In general, it's advisable to conduct all Python development within isolated virtual environments.

For as accessible introduction to creating, using, and managing Python virtual environments, check out this Medium article by Jake Manger: [Getting Started with Python Virtual Environments](https://towardsdatascience.com/getting-started-with-python-virtual-environments-252a6bd2240).

## The Problem

Because it has many non-Python dependencies, GNU Radio is not part of the Python ecosystem. This means GNU Radio is not listed on the PyPI, and is not installable with pip. This can make it challenging to install GNU Radio into an isolated environment. But it is a very common use case to want to run GNU Radio code within a larger python project. The deviation from conventional Python package installations poses a challenge for users accustomed to this approach.

## The Recommended Solution

For most use cases, the recommend solution is to install GNU Radio into a Conda environment. [Conda](https://conda.io/projects/conda/en/latest/user-guide/getting-started.html#managing-python) is a system package manager specifically designed for installing complex software stacks, such as GNU Radio, into sandboxed Python environments, called Conda environment. Conda environments are very similar to traditional virtual environments, except Conda environments support both Python and non-Python packages, while virtual environments are specifically tailored for managing Python packages.

[Conda-forge](https://conda-forge.org/), a community effort to provide conda packages for a wide range of software, makes it easy to install GNU Radio, its dependencies, and out-of-tree modules in a self-contained environment. Please refer to [CondaInstall](https://wiki.gnuradio.org/index.php/CondaInstall) for more information on installing GNU Radio with Conda.

If you're new to Conda and Python package management, it is recommended to start with [radioconda](https://github.com/ryanvolz/radioconda), a complete Conda distribution that comes pre-installed with GNU Radio.

## Our Workaround

Should your use case require a virtual environment or you encounter difficulties with radioconda, we present a workaround to link an external GNU Radio installation into an already existing virtual environment.

This workaround, involving [symbolic links](https://en.wikipedia.org/wiki/Symbolic_link), can be used to pull your system-level GNU Radio installation into a standard Python virtual environment.

> **Caveat:** This is not the recommended way to use GNU Radio. GNU Radio is built against a specific Python installation, and using it with a different Python installation—such as the one in your virtual environment—may result in unexpected behavior.

In a Python virtual environment, user-installed packages are typically installed in the `site-packages` directory within the module directory of the Python installation. For example, if I created a virtual environment for Python 3.10 using `venv`, user-installed packages would be installed to `venv/lib/python3.10/site-packages`.

When you try to import a GNU Radio module, Python searches for it in all locations listed on `PYTHONPATH`, which includes the virtual environment's `site-packages` directory. However, if GNU Radio is not installed within this environment, this will result in the following error: `ModuleNotFoundError: No module named 'gnuradio'`

You could update `PYTHONPATH` to include the directory where GNU Radio is installed, but this is not a good idea, as doing so may introduce dependencies that conflict with those already installed in your virtual environment.

A better solution is to create a link in your project's `site-packages` directory to the location where the `gnuradio` module is installed. Then, when Python searches for `gnuradio` in `site-packages`, it will find a link to a functional GNU Radio installation.

### Symbolic Links

Symbolic links, also known as symlinks or soft links, are a type of file that serves as a reference to another file or directory in the file system. Symbolic links contain the path of the target file or directory, unlike hard links, which directly point to the index node of the target file or directory.

They are essentially shortcuts or pointers to other files or directories. They allow you to create a reference to a file or directory in a different location without duplicating the content. 

Symbolic links can enable the integration of GNU Radio's functionality into Python virtual environments. By creating symbolic links, developers can interact with GNU Radio seamlessly within their Python projects.

### Creating a Symbolic Link

On Debian based distributions such as Ubuntu, we can install a system-level GNU Radio installation with:

```bash
sudo apt-get install gnuradio
```

System level packages are installed in `dist-packages`, which we can verify with:

```bash
python3 -c "import gnuradio; print(gnuradio.__path__[0])"
```

This should return `/usr/lib/python3/dist-packages/gnuradio`, or similar.

We can create a link from our virtual environment's `site-package` directory to the `dist-packages` directory where GNU Radio is installed using `ln`, the standard Unix command utility used to create links to existing files or directories.

The `ln` command can be used to create both hard links and symbolic links (symlinks). Here, we use the `-s` option to create symlinks, which point to the path of the target file or directory rather than its inode. You can think of symlinks as shortcuts.

## Example solution:

Assuming GNU Radio is installed, here is the complete bash script we use to generate the symlinks, intended to be executed within the Python virtual environment:

```bash
#!/bin/bash

# Find the system-level GNU Radio installation

GNURADIO_PATH=$(python3 -c "import gnuradio; print(gnuradio.__path__[0])")
PMT_PATH=$(python3 -c "import pmt; print(pmt.__path__[0])")

# Identify the Python version used within the virtual environment

PYTHON_VERSION=$(python3 -c "import sys; print('.'.join(map(str, sys.version_info[:2])))")

# Create a symbolic link to system-level GNU Radio installation

ln -s $GNURADIO_PATH venv/lib/python${PYTHON_VERSION}/site-packages/
ln -s $PMT_PATH venv/lib/python${PYTHON_VERSION}/site-packages/

# Print a success message
echo "Successfully created symbolic link for gnuradio in venv."

deactivate
```

The specific process we use to  deploy it (called `gr_setup.sh` below) from a project’s root folder to a new virtual environent includes:


1. Create the venv then run the bash script

```bash
python3 -m venv venv
./gr_setup.sh
```


2. Activate the virtual environment then test in the python console

```bash
source venv/bin/activate
python3
import gnuradio
```

If there is no error, you are successful! Exit the python window.

You are now ready to install the rest of your dependencies to the virtual environment- e.g.:

```javascript
pip install -r requirements.txt
```


**A Few Things to Note:**

* This does not only need to occur from the system-level, it can be used to link a conda-installed version of GNU Radio to a python virtual environment.
* Since we're using symlinks, if the system-level GNU Radio installation is moved or removed, these links will break, resulting in a `ModuleNotFoundError`.
* If we were to modify or recreate this link, we would need to also use the `-f` argument to remove existing destination files.
* Here, we assume our virtual environment was created using `venv`. However, if you come across a virtual environment, it is more likely to have been created using `virtualenv`, which is more popular, faster, and has more features than `venv`. This workaroun works for all environments, including `virtualenv`, `pyenv`, and `conda`, but the `site-apckages` directory might be in a slightly different location depending on the specifics of the environment.
* We also pull in GNU Radio's [polymorphic types](https://wiki.gnuradio.org/index.php/Polymorphic_Types_(PMTs)) module (pmt), which is required to carry data from one block/thread to another.
* Any other Python packages, including other GNU Radio subpackages can also be pulled in using this same workaround.


