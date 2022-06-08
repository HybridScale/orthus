---
title: Running jobs
layout: template
filename: running
---

## Content

1. [Introduction](#introduction)
2. [Running job](#running-jobs)
    - [Job description](#job-description)
3. [Types of jobs](#types-of-jobs)
    - [Serial](#serial-jobs)
    - [Parallel](#parallel-jobs)
    - [GPU](#gpu-jobs)

## Introduction

For scheduling and maintainig the jobs on Orthus cluster the **[SGE](http://star.mit.edu/cluster/docs/0.93.3/guides/sge.html)** (Sun of Grid Engine) queuing system is used. 

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

Check the status of all submitted jobs:
```
qstat
```
Check the status of jobs submitted by user:
```
qstat -u $USER
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
