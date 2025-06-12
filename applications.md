---
title: Software and Applications
layout: template
filename: applications
order: 3
---

## Content
1. [Overview](#overview)
2. [Lmod software modules](#lmod-software-modules)
   - [Basic Module Commands](#basic-module-commands)
   - [Understanding the Module Hierarchy](#understanding-the-module-hierarchy)
   - [Common Workflow](#common-workflow)
   - [Module Collections](#module-collections)
   - [Useful Tips](#useful-tips)
   - [Environment Variables](#environment-variables)
   - [Getting Help](#getting-help)
3. [Charliecloud Containers](#charliecloud-containers)
    - [Key Features for Scientific Computing](#key-features-for-scientific-computing)
    - [Basic Workflow](#basic-workflow)
    - [Container Management](#container-management)
    - [Running Applications](#running-applications)
    - [Integration with Slurm](#integration-with-slurm)
    - [Building Scientific Containers](#building-scientific-containers)
    - [Performance Considerations](#performance-considerations)
    - [Best Practices](#best-practices)
    - [Debugging and Troubleshooting](#debugging-and-troubleshooting)
    - [Advanced Features](#advanced-features)
    - [Documentation and Resources](#documentation-and-resources)
4. [Spack](#spack)
    - [Find applications and packages](#find-applications-and-packages)
    - [Loading and unloading packages](#loading-and-unloading-packages)
    - [Install using Spack](#install-using-spack)
    - [Users's applications](#users-applications)

## Overview
The Orthus cluster uses components from [OpenHPC](https://openhpc.community/) to provide much of the HPC functionality. The user interface to OpenHPC software system is provide by [Lmod](https://lmod.readthedocs.io/en/latest/), please see [this](https://openhpc.github.io/cloudwg/tutorials/sc20/exercise3.html) tutorial for more details. In addition the Lmod modules [Spack](https://spack.readthedocs.io/en/latest/) HPC package manager and the [Charliecloud](https://charliecloud.io/latest/) container system are also avalible on the cluster.

## Lmod Environment Modules

OpenHPC uses Lmod to manage software environments. Lmod provides a hierarchical module system that automatically manages dependencies and conflicts between different compilers, MPI libraries, and applications. You can see the openHPC package manifest in the [installation guides](Install_guide.pdf#page=36) for an overview of software avalable.

### Basic Module Commands

```bash
# List currently loaded modules
module list
ml list                    # Short form

# Show all available modules
module avail
ml av                      # Short form

# Load a module
module load gcc
ml gcc                     # Short form

# Unload a module
module unload gcc
ml -gcc                    # Short form with minus sign

# Get help for a module
module help gcc
module whatis gcc          # Brief description
```

### Understanding the Module Hierarchy

OpenHPC organizes software in a three-tier hierarchy:

1. **Core modules** - Basic tools and compilers (gcc, intel, etc.)
2. **Compiler-dependent** - Libraries built with specific compilers
3. **MPI-dependent** - Applications requiring both compiler and MPI

```bash
# Load a compiler to see compiler-dependent modules
ml gcc
ml av                      # Shows additional modules now available

# Load MPI to see MPI-dependent applications  
ml openmpi
ml av                      # Shows even more modules
```

### Common Workflow

```bash
# Typical development environment setup
ml gcc                     # Load compiler
ml openmpi                 # Load MPI library
ml boost                   # Load libraries as needed
ml list                    # Verify loaded modules

# Switch to different compiler (automatic cleanup)
ml intel                   # Lmod swaps gcc→intel, rebuilds stack
```

### Module Collections

Save and restore entire module environments:

```bash
# Save current modules as default collection
module save

# Save with custom name
module save myproject

# Restore saved collection
module restore
module restore myproject

# List saved collections
module savelist
```

### Useful Tips

- Use `ml` instead of `module` - it's shorter and context-aware
- Lmod automatically handles conflicts and dependencies
- Module names are case-sensitive
- Use tab completion for module names
- Check `module help <name>` for module-specific usage notes

### Environment Variables

Key variables set by OpenHPC modules:
- `$CC` - C compiler
- `$CXX` - C++ compiler  
- `$FC` - Fortran compiler
- `$MPICC` - MPI C compiler wrapper

### Getting Help

```bash
module --help              # Full Lmod help
module spider <name>       # Search for modules containing 'name'
module keyword <term>      # Search module descriptions
```

## Charliecloud Containers

Charliecloud is an unprivileged container runtime designed for high-performance computing that enables user-defined software stacks (UDSS). It enables unprivileged container execution on HPC systems while maintaining performance and security. For complete documentation, see the [official Charliecloud documentation](https://charliecloud.io/latest/).

### Key Features for Scientific Computing

- **Unprivileged execution** - No root required on compute nodes
- **HPC-optimized** - Native performance with minimal overhead
- **MPI support** - Full integration with HPC message passing
- **GPU acceleration** - CUDA support

### Basic Workflow

```bash
# 1. Pull or build image
ch-image pull ubuntu:20.04               # Pull from registry
# OR
ch-image build -t myapp .                # Build from Dockerfile

# 2. Convert to Charliecloud format
ch-convert ubuntu:20.04 /path/to/images/
# OR
ch-convert myapp /path/to/images/

# 3. Run on compute nodes
ch-run /path/to/images/ubuntu+20.04 -- /bin/bash
```

### Container Management

```bash
# List available images
ch-image list

# Pull from registry
ch-image pull ubuntu:20.04
ch-image pull nvcr.io/nvidia/pytorch:22.03-py3

# Build from Dockerfile
ch-image build -t mycode:latest .

# Convert and manage
ch-convert mycode:latest ./images/
ls ./images/
```

### Running Applications

```bash
# Interactive shell
ch-run ./images/mycode -- /bin/bash

# Execute specific command
ch-run ./images/mycode -- python script.py

# With bind mounts
ch-run -b /scratch:/scratch ./images/mycode -- ./myapp

# MPI applications
mpirun ch-run ./images/mycode -- ./mpi_program
```

### Integration with Slurm

```bash
#!/bin/bash
#SBATCH --job-name=container_job
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=16

# Load required modules
ml charliecloud

# Run MPI application in container
mpirun ch-run -b /scratch:/scratch ./images/myapp -- ./parallel_code
```

### Building Scientific Containers

Example Dockerfile for scientific software:

```dockerfile
FROM ubuntu:20.04

# Install base dependencies
RUN apt-get update && apt-get install -y \
    gcc gfortran python3 python3-pip \
    libopenmpi-dev openmpi-bin

# Install Python packages
RUN pip3 install numpy scipy matplotlib

# Copy application code
COPY . /app
WORKDIR /app

# Compile if needed
RUN make

CMD ["./myapp"]
```

### Performance Considerations

```bash
# Use tmpfs for temporary data
ch-run --tmpfs=/tmp ./images/myapp -- ./compute_intensive

# Bind mount high-performance storage
ch-run -b /lustre:/lustre -b /gpfs:/gpfs ./images/myapp -- ./io_intensive

# Access GPUs
ch-run --nvidia ./images/cuda_app -- ./gpu_program
```

### Debugging and Troubleshooting

```bash
# Verbose output
ch-run -v ./images/myapp -- ./problematic_program

# Interactive debugging
ch-run ./images/myapp -- /bin/bash

# Check image contents
ch-run ./images/myapp -- ls -la /
ch-run ./images/myapp -- env
```

### Advanced Features

```bash
# Custom user namespace mapping
ch-run --uid=1000 --gid=1000 ./images/myapp -- id

# Multiple bind mounts
ch-run -b /data1:/data1 -b /data2:/data2 ./images/myapp -- ./analysis

# Environment variables
ch-run -e CUDA_VISIBLE_DEVICES=0,1 ./images/gpu_app -- ./gpu_code
```

### Documentation and Resources

- **Official Documentation**: [Charliecloud Docs](https://charliecloud.io/latest/)
- **Best Practices**: [Charliecloud Best Practices](https://charliecloud.io/latest/best_practices.html)
- **Scientific Containerization**: [Grüning et al. (2018)](https://f1000research.com/articles/7-742/v2) provides a valuable editorial with eleven specific recommendations for containerizing scientific software


## Spack

The users' applications are installed, maintained, and loaded/unloade  from the shell on the cluster using [Spack](https://spack.readthedocs.io/en/latest/). Spack is a package management tools designed to support multiple versions and configurations of software on a wide variety of platforms and environments.

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

### Install using Spack

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

### Users's applications

List of installed user's applications (multiple versions are possible):

- [Boost](https://www.boost.org/): 1.78.0
- [Bowtie2](https://bowtie-bio.sourceforge.net/bowtie2/index.shtml): 2.4.2
- [CMake](https://cmake.org/): 3.22.0
- [CUDA](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html): 11.6.0
- [git](https://git-scm.com/about): 2.31.1
- [Intel OneAPI MKL](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/api-based-programming/intel-oneapi-math-kernel-library-onemkl.html): 2022.0.1
- [Magma](https://icl.utk.edu/magma/): 2.6.1
- [Miniconda3](https://docs.conda.io/en/latest/miniconda.html): 4.10.3
- [Octave](https://octave.org): 7.1.0
- [OpenJDK](https://openjdk.org/): 17.0.3 11.0.12
- [OpenMPI](https://www.open-mpi.org/): 4.1.2, 4.1.3
- [Python](https://www.python.org/): 3.9.10
- [PyTorch](https://pytorch.org/): 1.11.0
- [R](https://www.r-project.org/): 4.2.1
- [R-Gviz](https://www.bioconductor.org/packages/release/bioc/html/Gviz.html): 1.40.1
- [RStudio](https://www.rstudio.com/): 1.4.1717
- [Slate](https://icl.utk.edu/slate/): 2021.05.02

