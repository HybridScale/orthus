---
title: Applications
layout: template
filename: applications
---

Content

1. [Spack](#spack)
    - [Find applications and packages](#find-applications-and-packages)
    - [Loading and unloading packages](#loading-and-unloading-packages)

2. [Installed applications](#installed-applications)

## Spack

The users' applications are installed, maintained, and loaded/unloaded from the shell on the cluster using [Spack](https://spack.readthedocs.io/en/latest/). Spack is a package management tools designed to support multiple versions and configurations of software on a wide variety of platforms and environments.

### Find applications and packages

To get a list of all installed software packages:

```
spack find
```

which will print a long list of all installed packaged. To list only explicitly installed packages (without listing their dependencies) run:

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
When the package is loaded, all its dependencies and auxiliary packages are also loaded automatically. The user does not have to take care of the dependency list.

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