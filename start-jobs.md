## Introduction

For scheduling and maintainig the jobs on Orthus cluster the **SGE** (Sun of Grid Engine) is used. 

## Running jobs

User's applications (in following text names *jobs*) are run/submitted via SGE and have to be described with the **start shell script**. Inside each shell script, before standard shell command, a special SGE command has to be defined that describe describe to the SGE the resources the user's job requires.

To start a job, the following command is required:

    squb <SGE PARAMETERS> <name of the start script>

### Job description

The SGE language is used to describe jobs, and the job description file (start script) is a standard shell script. The header of each script contains the SGE parameters that describe the job in details, followed by normal commands to execute user applications.

The structure of the job start script:

    my_job.sge
    
    #!/bin/bash
    
    #$ -<parameter1> <value1>
    #$ -<parameter2> <value2>
     
    <command1>
    <command2>

The job is submitted to cluster for execution by running the command:

    qsub my_job.sge


