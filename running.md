---
title: Running jobs
layout: template
filename: running
---

## Content

1. [Introduction](#introduction)
2. [File system](#file-system)
    - [Storage folder](#storage-folder)
2. [Running job](#running-jobs)
    - [Job description](#job-description)
3. [Types of jobs](#types-of-jobs)
    - [Serial](#serial-jobs)
    - [Parallel](#parallel-jobs)
    - [Interactive](#interactive-jobs)
    - [GPU](#gpu-jobs)

## Introduction

For scheduling and maintainig the jobs on Orthus cluster the **[SGE](http://star.mit.edu/cluster/docs/0.93.3/guides/sge.html)** (Sun of Grid Engine) queuing system is used. 

## File system

The available file systems on Orthus are combination of local (per node) file systems and mounted (NFS) storage locations.
The table below gives an overview over the available file systems:

| Mount point | Accesibility | Description |
|-------------|--------------|-------------|
| /home/<username> | Login+Compute | NFS user's home folder visible on frontend and compute nodes |
| /apps | Login+Compute | NFS shared folder in which packages are installed |
| /storage | Login+Compute | NFS shared folder for sharing
| /scratch | Compute | Local and fast (SSD) storage on each compute node |

### Storage folder
The _/storage_ folder is used for storing and sharing large amount of data between the user. To each registered project a shared  folder is created in which members of the project can share data.

### Scratch folder
The _/scratch_ folder is a fast (SSD) disc attached to the compute node. The main purpose of the folder is to be a working folder for all active jobs on the compute node. Also, the users can store their temporary data. 

**==CAUTION== All the data in the /scratch folder will be deleted once the job is finished!**

## Running jobs

User's applications (in following text names *jobs*) are run/submitted via SGE and have to be described with the **start shell script**. Inside each shell script, before standard shell command, a special SGE command has to be defined that describe to the SGE the resources the job requires.

To start a job run command:
```
qsub <SGE PARAMETERS> <name of the start script>
```

The command _qsub_ returns the job ID which can be used later for monitoring the status of the job.
```
Your job <JobID> ("my_job") has been submitted
```

Check the status of submitted jobs:
```
qstat
```
which will list all the jobs submitted by the current user. To check the status of all jobs submitted by all users / particular user run:
```
qstat -u '*'
qstat -u <username>
```

The possible states of the jobs are: r (running), p (pending), qw (queuing). 

A more detailed description on how to describe and monitor jobs can be found in the [documentation](https://wiki.srce.hr/display/RKI/Pokretanje+i+upravljanje+poslovima) of the [Isabella](https://www.srce.unizg.hr/isabella/) cluster at SRCE as both systems have the same queuing management.

### Job description

The SGE language is used to describe jobs, and the job description file (start script) is a standard shell script. The header of each script contains the SGE parameters that describe the job in details, followed by normal commands to execute user applications.

The structure of the job start script (file: _my_job.sh_):
```
    #!/bin/bash

    #$ -<parameter1> <value1>
    #$ -<parameter2> <value2>

    <command1>
    <command2>
```

The job is submitted to cluster for execution by running the command:

```
qsub my_job.sh
```

The basic SGE parameters for describing the jobs are:
```
    -N <job name>: name of the job that will be printed when checking the job status
    -o <file name>: file to which the standard output will be redirected (the standard output to the terminal will be printed into this file)
    -e <file name>: file to which the standard error messaages will be redirected
    -cwd: define that the directory with the submission script is the working directory
    -j y|n: enable joining standard output and error in the same file (default is: n)
    -pe <parallel environment> <range>: define parallel jobs, name of the parallel environment and number of processors required
    -q <queueu>: name of the queueu in which the job will be submitted
    -V: SGE will transfer all current environment variable to the job
```

## Types of jobs

### Serial jobs

An example of the a simple script to start a serial job (requiring only 1 CPU core) which prints the _data_ and _hostname_:
```
    #!/bin/bash

    #$ -N example-serial
    #$ -o example-serial.out
    #$ -e example-serial.err

    data
    hostname
```
### Interactive jobs

To start an interactive job, a `qrsh` command is required. The direct access to the command line of the worker node can be done simply by running:
```
qrsh
```

For example, to run a parallel interactive job that requires _RStudio_ package and 4 processor cores:
```
spack load RStudio
qrsh -pe mpi 4 -V -display 10.1.1.1:0.0 rstudio
```

### Parallel jobs

To start parallel jobs it is necessary to specify the desire parallel environment and the number of CPU cores required for the application. An example of the script requiring _mpi_ parallel environment with 12 compute cores and running a simple _Hello_World_ program:

```
    #!/bin/bash
    
    #$ -N example-mpi
    #$ -o example-mpi.out
    #$ -e example-mpi.err
    #$ -pe mpi 12
    #$ -V
    
    mpirun -np 12 ./hello_world.exe
```

The concrete script requires that an MPI package is loaded (using [Spack](https://hybridscale.github.io/orthus/applications#loading-and-unloading-packages)) in the user's environment before the job is submitted.

### GPU jobs
