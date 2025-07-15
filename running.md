---
title: Job scheduling
layout: template
filename: running
order: 3
---

## Content

1. [Introduction](#introduction)
2. [File system](#file-system)
    - [Storage folder](#storage-folder)
    - [Scratch folder](#scratch-folder)
3. [Running jobs](#running-jobs)
    - [Job description](#job-description)
    - [Slurm environment variables](#slurm-environment-variables)
4. [Types of jobs](#types-of-jobs)
    - [Serial jobs](#serial-jobs)
    - [Interactive jobs](#interactive-jobs)
    - [Array jobs](#array-jobs)
    - [Parallel jobs](#parallel-jobs)
        - [MPI + OpenMP](#mpi-with-openmp-threads)
    - [GPU jobs](#gpu-jobs)
5. [Monitoring and management of jobs](#monitoring-and-management-of-jobs)
    - [Host information](#host-information)
    - [Job management](#job-management)
    - [Get statistics of finished jobs](#get-statistics-of-finished-jobs)

## Introduction
 
The Orthus cluster uses **[Slurm](https://slurm.schedmd.com/overview.html)** (Simple Linux Utility for Resource Management) job scheduling system to enable users to run and administer batch compute jobs.

When you ssh to the Orthus cluster via `$ ssh orthus.cir.irb.hr` you are logging into a login node. When you submit a batch job with `sbatch`, the login node schedules your job to run on compute nodes. The login node has all the same software installed as the compute nodes.

## File system

The available file systems on Orthus are combination of local (per node) file systems and mounted (NFS) storage locations.
The table below gives an overview over the available file systems:

| Mount point | Accessibility | Description |
|-------------|--------------|-------------|
| /home | Login+Compute | NFS user's home folder visible on frontend and compute nodes |
| /apps | Login+Compute | NFS shared folder in which packages are installed |
| /storage | Login+Compute | NFS shared folder for sharing data |
| /scratch | Compute | Local and fast (SSD) storage on each compute node |

### Storage folder
The _/storage_ folder is used for storing and sharing large amounts of data between users. To each registered project a shared folder is created in which members of the project can share data.

### Scratch folder
The _/scratch_ folder is a fast (SSD) disc attached to the compute node. The main purpose of the folder is to be a working folder for all active jobs on the compute node. Users can also store their temporary data here.

**==CAUTION== All the data in the /scratch folder will be deleted once the job is finished!**

## Running jobs

Jobs are run/submitted via Slurm and have to be described with a **batch script**. Inside each script, before standard shell commands, special Slurm directives have to be defined that describe to Slurm the resources the job requires.

To start a job run command:
```bash
sbatch <name of the batch script>
```

The command `sbatch` returns the job ID which can be used later for monitoring the status of the job:
```
Submitted batch job <JobID>
```

Check the status of submitted jobs:
```bash
squeue
```
which will list all jobs in the queue. To check only your jobs:
```bash
squeue -u $USER
```

To see detailed information about a specific job:
```bash
scontrol show job <JobID>
```

The possible states of jobs include: PD (pending), R (running), CG (completing), CD (completed), F (failed), CA (cancelled).

### Job description

Slurm batch scripts are standard shell scripts with special directives that describe the job requirements. The header of each script contains the Slurm parameters, followed by normal commands to execute user applications.

The structure of a job batch script (file: _my_job.sh_):
```bash
#!/bin/bash

#SBATCH --<parameter1>=<value1>
#SBATCH --<parameter2>=<value2>

<command1>
<command2>
```

Here's a basic example batch script:
```bash
#!/bin/bash
#SBATCH --job-name=test_job
#SBATCH --output=test_job_%j.out
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --time=00:05:00

echo "Job started at $(date)"
echo "Running on host: $(hostname)"
echo "CPU info:"
lscpu | grep "Model name"
echo "Memory info:"
free -h
echo "Sleeping for 30 seconds..."
sleep 30
echo "Job completed at $(date)"
```

The job is submitted to the cluster for execution by running:

```bash
sbatch my_job.sh
```

The basic Slurm parameters for describing jobs are:
```bash
--job-name=<name>          # Name of the job
--output=<filename>        # File for standard output
--error=<filename>         # File for standard error
--time=<time>              # Maximum runtime (e.g., 1:30:00 for 1.5 hours)
--nodes=<count>            # Number of nodes required
--ntasks=<count>           # Number of tasks (processes)
--ntasks-per-node=<count>  # Tasks per node
--cpus-per-task=<count>    # CPU cores per task
--mem=<size>               # Memory per node (e.g., 4G, 1000M)
--mem-per-cpu=<size>       # Memory per CPU core
--partition=all            # Partition name (use "all")
--gres=gpu:<count>         # GPU resources
```

### Slurm environment variables
Inside the batch script it is possible to use Slurm environment variables. Some of the most commonly used are:
```bash
$SLURM_JOB_ID          # Unique job identifier
$SLURM_JOB_NAME        # Job name
$SLURM_SUBMIT_DIR      # Directory from which job was submitted
$SLURM_JOB_NODELIST    # List of nodes assigned to job
$SLURM_NTASKS          # Number of tasks
$SLURM_CPUS_PER_TASK   # CPU cores per task
$SLURM_PROCID          # Process rank
$SLURM_LOCALID         # Local task ID on node
$SLURM_ARRAY_JOB_ID    # Job array ID
$SLURM_ARRAY_TASK_ID   # Task ID within job array
$SLURM_TMPDIR          # Temporary directory (typically /scratch)
```

## Types of jobs

### Serial jobs

An example of a simple script to start a serial job (requiring only 1 CPU core) which prints some system information:
```bash
#!/bin/bash

#SBATCH --job-name=example-serial
#SBATCH --output=example-serial.out
#SBATCH --error=example-serial.err
#SBATCH --partition=all
#SBATCH --ntasks=1
#SBATCH --time=00:10:00

echo "Starting serial job at $(date)"
echo "Running on node: $(hostname)"
echo "Working directory: $(pwd)"

# Your application commands here
./my_serial_program
```

### Interactive jobs

To start an interactive job use the `qalloc srun` command with the `--pty` flag. This gives you direct access to a compute node:
```bash
salloc srun --pty bash
```

For example, to run an interactive job that requires 4 CPU cores for 2 hours:
```bash
salloc --partition=all --ntasks=4 --time=02:00:00 srun --pty bash
```

It is also possible to ssh to the compute node (`ssh compute01`) but this is only allowed when there is an active job for your user active on the node.

### Array jobs
Slurm enables multiple submissions of the same job with different parameters, called job arrays. Each job inside the array is called a *task* and has its unique identifier.

At submission time, specify the array range using the `--array` parameter:
```bash
--array=<start>-<end>:<step>
```

The task identifier is stored in the environment variable **$SLURM_ARRAY_TASK_ID**. Tasks can be both serial or parallel jobs.

#### Example of usage

An example script that starts 10 serial jobs, each processing a different input file:

```bash
#!/bin/bash

#SBATCH --job-name=job_array_serial
#SBATCH --output=output/job_%A_%a.out
#SBATCH --error=output/job_%A_%a.err
#SBATCH --partition=all
#SBATCH --ntasks=1
#SBATCH --time=01:00:00
#SBATCH --array=1-10

echo "Processing task $SLURM_ARRAY_TASK_ID"
./myexec inputFile.$SLURM_ARRAY_TASK_ID
```
    
An example script starting 10 parallel jobs:
```bash
#!/bin/bash

#SBATCH --job-name=job_array_parallel
#SBATCH --output=output/job_%A_%a.out
#SBATCH --error=output/job_%A_%a.err
#SBATCH --partition=all
#SBATCH --ntasks=4
#SBATCH --time=02:00:00
#SBATCH --array=1-10

echo "Running parallel task $SLURM_ARRAY_TASK_ID"
srun ./myexec inputFile.$SLURM_ARRAY_TASK_ID
```

### Parallel jobs

To start parallel jobs, specify the number of tasks and optionally the number of nodes. An example script requiring 12 compute cores and running a simple _Hello World_ MPI program:

```bash
#!/bin/bash

#SBATCH --job-name=example-mpi
#SBATCH --output=example-mpi.out
#SBATCH --error=example-mpi.err
#SBATCH --partition=all
#SBATCH --ntasks=12
#SBATCH --time=01:00:00

# Load required modules
module load openmpi

echo "Starting MPI job with $SLURM_NTASKS tasks"
echo "Nodes allocated: $SLURM_JOB_NODELIST"

srun ./hello_world.exe
```

The script requires that an MPI package is loaded using the module system before the job is submitted.

#### MPI with OpenMP threads
When running hybrid parallel applications that combine both MPI and OpenMP (thread) parallelism, you need to properly configure the resource allocation and thread binding to achieve optimal performance.

```bash
#!/bin/bash

#SBATCH --job-name=test-mpi-openmp
#SBATCH --output=test.out
#SBATCH --error=test.err
#SBATCH --partition=all
#SBATCH --nodes=1
#SBATCH --ntasks=4
#SBATCH --cpus-per-task=12
#SBATCH --time=02:00:00

# Set OpenMP environment variables
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
export OMP_PROC_BIND=close
export OMP_PLACES=cores

# Load required modules
module load openmpi

echo "Running hybrid MPI+OpenMP job"
echo "MPI tasks: $SLURM_NTASKS"
echo "OpenMP threads per task: $OMP_NUM_THREADS"

srun --cpu-bind=cores ./my-hybrid-application
```

### GPU jobs

GPU jobs require special resource allocation using the `--gres` flag. An example GPU job script that requires 1 CPU core and 2 GPU devices:

```bash
#!/bin/bash

#SBATCH --job-name=test-gpu
#SBATCH --output=test_gpu_%j.out
#SBATCH --error=test_gpu_%j.err
#SBATCH --partition=all
#SBATCH --ntasks=1
#SBATCH --gres=gpu:2
#SBATCH --time=01:00:00

echo "Job started at $(date)"
echo "CUDA_VISIBLE_DEVICES = $CUDA_VISIBLE_DEVICES"
echo "SLURM_JOB_GPUS = $SLURM_JOB_GPUS"

# Display GPU information
nvidia-smi

# Load CUDA if needed
module load cuda

# Run your GPU application
./my_gpu_program
```

## Monitoring and management of jobs

### Host information

To print information about nodes in the cluster:
```bash
sinfo                    # Show partition and node state information
sinfo -N                 # Show node-oriented format
scontrol show nodes      # Detailed node information
```

### Job management

Jobs can be managed after submission using various Slurm commands:

Cancel a job:
```bash
scancel <JobID>
```

Cancel all your jobs:
```bash
scancel -u $USER
```

Hold/suspend a job (prevent it from running):
```bash
scontrol hold <JobID>
```

Release a held job:
```bash
scontrol release <JobID>
```

Suspend a running job:
```bash
scontrol suspend <JobID>
```

Resume a suspended job:
```bash
scontrol resume <JobID>
```

### Get statistics of finished jobs

To get information about finished jobs use the `sacct` command:
```bash
sacct                    # Show accounting information for your recent jobs
sacct -j <JobID>         # Detailed info for specific job
sacct -u <username>      # Jobs for specific user
```

For a quick efficiency report of a completed job:
```bash
seff <JobID>             # Shows CPU and memory efficiency
```

Common `sacct` usage examples:
```bash
# Detailed job information with custom format
sacct -j <JobID> --format=JobID,JobName,State,ExitCode,CPUTime,MaxRSS

# Jobs from a specific time period
sacct --starttime=2024-01-01 --endtime=2024-01-31

# Your jobs from today
sacct -u $USER --starttime=today
```

The `sacct` command provides extensive information about completed jobs including runtime, memory usage, CPU efficiency, and exit codes, which is useful for optimizing future job submissions.
