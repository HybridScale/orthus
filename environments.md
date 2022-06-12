---
title: Queues and Parallel Environments
layout: template
filename: environments
---

Content

1. [Job queues]($job-queues)
2. [Parallel environments](#parallel-environments)

## Job queues

On the Orthus cluster there are two job queues:

- all.q
    - the queueu for all jobs without any restrictions
    - for serial and parallel jobs
    - CPU-only jobs
- gpu.q
    - the queue for jobs that require GPU devices

The default queue is _all.q_. The selection of the job queue is achieved by setting the appropriate parallel environment.

To list all the available job qeueues run:
```
qconf -sql
```

And to list the detailed configuration of a particular queueu:
```
qconf -sq <qeueue>
```

## Parallel environments

A parallel environment is a mechanism in SGE that allows multiple processor cores to be occupied for parallel operations. The available parallel environments are:

- mpi
    - SGE will try to allocate all cores on the same worker node
    - the allocation is done on the CPU-only worker nodes
- gpu
    - allocate processor cores on the GPU-based worker nodes

To request more than 1 processor core, add in your submission script option:
```
#$ <parallel environment> <number-of-cores>
```

For example:
```
#$ mpi 12
```

Will request for 12 processor core using mpi parallel environment. In practice it is not required to explicitly set a required queue, only the parallel environment.