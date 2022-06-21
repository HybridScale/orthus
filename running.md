---
title: Running jobs
layout: template
filename: running
---

## Content

1. [Introduction](#introduction)
2. [File system](#file-system)
    - [Storage folder](#storage-folder)
    - [Scratch folder](#scratch-folder)
2. [Running job](#running-jobs)
    - [Job description](#job-description)
    - [SGE environment variables](#sge-environment-variables)
3. [Types of jobs](#types-of-jobs)
    - [Serial](#serial-jobs)
    - [Interactive](#interactive-jobs)
    - [Array](#array-jobs)
    - [Parallel](#parallel-jobs)
    - [GPU](#gpu-jobs)
4. [Monitoring and management of jobs](#monitoring-and-management-of-jobs)
    - [Host information](#host-information)
    - [Job management](#job-management)
    - [Get statistics of the finished job](#get-statistics-of-the-finished-job)

## Introduction

For scheduling and maintainig the jobs on Orthus cluster the **[SGE](http://star.mit.edu/cluster/docs/0.93.3/guides/sge.html)** (Sun of Grid Engine) queuing system is used. 

## File system

The available file systems on Orthus are combination of local (per node) file systems and mounted (NFS) storage locations.
The table below gives an overview over the available file systems:

| Mount point | Accessibility | Description |
|-------------|--------------|-------------|
| /home | Login+Compute | NFS user's home folder visible on frontend and compute nodes |
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

### SGE environment variables
Inside the submission script it is possible to use SGE environment variables. Some of the most commonly used are:
```
$TMPDIR :  The absolute path to the job's temporary working directory (e.g. /scratch).
$JOB_ID : A unique identifier assigned by the SGE when the job was submitted.
$SGE_TASK_ID : The task identifier in the array job represented by this task.
$SGE_O_HOST : The host from which the job was submitted.
$SGE_O_PATH : The content of the PATH environment variable in the context of the job submission command.
$SGE_O_WORKDIR : The working directory of the job submission command.
$SGE_STDOUT_PATH : The path name of the file to which the standard output stream of the job is diverted.
$SGE_STDERR_PATH : The path name of the file to which the standard error stream of the job is diverted.
$HOSTNAME : The host name of the node on which the job is running.
$JOB_NAME : The job name, which is built from the file name provided with the qsub command
$PE_HOSTFILE : The path of a file that contains the definition of the virtual parallel machine that is assigned to a parallel job by the grid engine system. This variable is used for parallel jobs only.
$QUEUE : The name of the queue in which the job is running.
```

## Types of jobs

### Serial jobs

An example of a simple script to start a serial job (requiring only 1 CPU core) which prints the current _date_ and _hostname_ of the computing host where the job is executed:
```
#!/bin/bash

#$ -N example-serial
#$ -o example-serial.out
#$ -e example-serial.err

date
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
### Array jobs
SGE enables multiple submissions of the same job, called parametric jobs or an array of jobs. Each job inside of the array of jobs is called a *task* and has its unique identifier.

At the submission time the user has to specific the value range of the identifier using the parameter **-t**:
```
-t  <start>:<end>:<step>
```
The value of *`<start>`* is the identifier of the first task, the *`<end>`* is identifier of the last task, and the *`<step>`* is an increment value for the next task.  The idetifier of each task is stored in the environemt variable **$SGE_TASK_ID**. Task can be both serial or parallel jobs.

#### Example of usage

1. An example of the script that starts 10 serial jobs:

```
#! /bin/bash

#$ -N job_array_serial
#$ -cwd
#$ -o output/
#$ -j y
#$ t 1:10

./myexec inputFile.$SGE_TASK_ID
```

2. An example of the script starting 10 parallel jobs:
```
#! /bin/bash

#$ -N job_array_parallel
#$ -cwd
#$ -o output/
#$ -j y
#$ -pe mpi 10
#$ t 1:10

mpirun -np 10 ./myexec inputFile.$SGE_TASK_ID

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

The GPU jobs are type of parallel jobs that can use one or more CPU core and 1 or more GPUs. An example of a GPU job script that requires 1 CPU core and 2 GPU devices and prints the visible devices allocated to the job and status of the devices.
```
#!/bin/bash

#$ -N test-gpu
#$ -o test_gpu_$JOB_ID.out
#$ -e test_gpu_$JOB_ID.err
#$ -cwd
#$ -pe gpu 1
#$ -l gpu=2
#$ -V

export `cat $TMPDIR/gpus`

echo "CUDA_VISIBLE_DEVICES = $CUDA_VISIBLE_DEVICES"

nvidia-smi
```
To succesfully run the job on the allocated GPU devices (and to avoid different users to run on the same GPU devices at the same time) the command **`export 'cat $TMPDIR/gpus'`** has to be executed before any other command in the submission script.

## Monitoring and management of jobs

### Host information

To print the number of procesors, computing cores and amount of the main memory per node use the command
```
qhost
```

### Job management
It is possible to manage the jobs even after it is submitted.
While the jobs is still in the waiting queueu, it is possible to temporarily suspend its execution with the command
```
qhold <JobID>
```
The suspended job can be returned to the waiting queueu with:
```
qrls <JobID>
```
A better control over the job execution is possible to have with the command **qmod**. The command can temporarily stop active jobs by sending the *SIGSTOP* signal. The job will enter in inactive state (T) but will not free the allocated resources (memory, CPUs). With the command it is also possible to save current states of the jobs on disc (*checkpointing*= for the jobs that have this functionality. Furthermore, the with **qmod** the user can stop an active jobs and return it to the submitting queueu. The specific parameters are:
```
-c : save the state of the job.

-f : force the execution of the command (useful when returning into the submission queueu the jobs that are run with  *-r n*).

-r : stop the running job and return it the submission queue.

-s : stop/suspend the execution of the running job.

-us : continue the execution of the previously stopped/suspended job.
```

### Get statistics of the finished jobs

To get the information of the finished jobs use the command:
```
qacct <parameters>
```
The most common examples of using the command is:
```
qacct -j <JobID>
```
which will print all the information of the finished job with the given ID.

The other options are:
```
-j <JobID> : detailed description of the given job

-h <hostname> : statistics of usage for a specific computing node

-q <queue> : statistics for a specific queue

-i <username> : resource usage for a user

-pe <parallel_environment> : statistics of usage for a parallel environment
```