---
title: Applications
layout: template
filename: applications
---

## Content

1. [Spack](#spack)
    - [Find applications and packages](#find-applications-and-packages)
    - [Loading and unloading packages](#loading-and-unloading-packages)

2. [Installed applications](#installed-applications)
    - [Compilers](#compilers)
    - [MPI](#mpi)
    - [Users' applications](#users-applications)
3. [How to install new software](#how-to-install-new-software)
    - [System-wide installation](#system-wide-installation)
    - [Local installation](#local-installation)

## Spack

The users' applications are installed, maintained, and loaded/unloaded from the shell on the cluster using [Spack](https://spack.readthedocs.io/en/latest/). Spack is a package management tools designed to support multiple versions and configurations of software on a wide variety of platforms and environments.

### Find applications and packages

To get a list of all installed packages:

```
spack find
```

The command will print a long list of all installed packaged. To list only explicitly installed packages (without listing their dependencies) run:

```
spack find -x
```

To show the install path of a specific installed package:

```
spack find -p <package_name>
```

for example, to find the installation path of the **PyTorch** package run:
```
spack find -p py-torch
```

Get all other _find_ optional argument: `spack find -h`

To list all available _spack_ command run:

```
spack -h
```

You can get info on each command by running: `spack help <command>`

### Loading and unloading packages

Loading is the process of adding the desired pre-installed package in the user's environment. The load process set all the required environmental variables and paths to libraries and binaries in the user's environment. The unloading is the process of removing the all the variables and paths set by the _load_ command.

To load the packages into environment run:

```
spack load <package> <package>
```
Use _tab_ to autocomplete command and thus avoid unnecessary typing :)
When the package is loaded, all its dependencies and auxiliary packages are loaded automatically. The user does not have to take care of the dependency list.

If multiple version of the same package are installed then provide version number when loading by running:
```
spack load <package>@<version>
```
For example, currently on the system are installed 2 versions of OpenMPI package (4.1.2 and 4.1.3). To load latter run:
```
spack load openmpi@4.1.3
```

To unload (remove) the loaded package from the user's environment run:

```
spack unload <package> <package>
```

To unload all the packages at once run:
```
spack unload -a
```

List all loaded packages with either of the below commands:
```
spack find --loaded
spack find --loaded -x
spack load --list
```

## Installed applications

### Compilers

The list of compilers and their corresponding Spack packages are listed in the table:

| Version               | package                          |
|-----------------------|----------------------------------|
| GCC 4.8.5             |                             |
| GCC 9.4.0             | gcc@9.4.0                        |
| GCC 11.2.0            | gcc@11.2.0                       |
| Intel OneAPI 2022.0.1 | intel-openapi-compilers@2022.0.1 |
| NVHPC 2022.1          | nvhpc@22.1                       |

### MPI

### Users's applications

List of installed user's applications:

- PyTorch: 1.11.0
- R: 4.1.2
- RStudio: 1.4.1717
- Python: 3.9.10
- OpenMPI: 4.1.2, 4.1.3
- Octave: 7.1.0
- Magma: 2.6.1
- Miniconda3: 4.10.3
- git: 2.31.1
- cuda: 11.6.0
- boost: 1.78.0
- cmake: 3.22.0
- Intel OneAPI MKL: 2022.0.1
- Bowtie2: 2.4.2

## How to install new software

### System-wide installation

If you want to install new software system-wide, so that the application is available to all users of the Orthus cluster, send your request on the email: [orthus-users@irb.hr](mailto:orthus-users@irb.hr).

### Local installation

If you want to install application locally there are two way to do so. In both cases, the application will be installed in your user's home folder and therefor will be **visible and accessible only to you**. Other users will now have an access to the application nor its dependencies.

#### Manual installation

The instalation can be done manually by first downloading the source code of the application in your home folder and then by following the installation instructions provided with the software. If the installation is done using *cmake* don't forget first to load *CMake* module using `spack load cmake`.

#### Install using Spack

If the software installer is available via Spack (you can check by running the command `spack list <name-of-application>` you can install it using **Spack**. We suggest to use this method since it allows automatic reuse of the dependency software already installed globally via Spack. A detailed instructions on how to setup your own Spack environment and install your application within it read the following [manual](https://spack-tutorial.readthedocs.io/en/latest/tutorial_environments.html).

To create a Spack environment as a system user add the flag `-d`:
```
spack env create -d /path/to/directory/<env-name>
```

The folder with the environment name will be stored on the given path (if not provided, the environment will be created in the current directory).

To activate the environment run:
```
spack env activate -d -p /path/to/directory/<env-name>
```

Deactivate environment by typing:
```
despacktivate
```