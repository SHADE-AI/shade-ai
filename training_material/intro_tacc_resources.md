# TACC Compute and Storage Quickstart

## Prerequisites 

Sign up for a TACC user account here: https://portal.tacc.utexas.edu/. After your account has been approved, enable Multi-factor Authentication by following the instructions [here](https://portal.tacc.utexas.edu/tutorials/multifactor-authentication). 

Individual users have been added to the DARPA-SHADE project. You can view a list of projects associated with your account by logging into the [portal](https://portal.tacc.utexas.edu/) and selecting 'Projects and Allocations' from the 'Allocations' drop-down in the main menu bar. For most users who have not used TACC resources, you will only see the DARPA-SHADE project. Selecting the DARPA-SHADE project will provide additional details, including a list of resource allocations and users associated with the project. Current compute allocations include [Frontera](https://frontera-portal.tacc.utexas.edu/user-guide/), [Longhorn](https://portal.tacc.utexas.edu/user-guides/longhorn), and [Lonestar6](https://portal.tacc.utexas.edu/user-guides/lonestar6). In addition to the 1TB of user/account-specific storage on the shared, Lustre filesystem, Stockyard (mounted on Frontera, Longhorn, and Lonestar6), we have also provisioned storage on Corral where you can share data within and among teams. Below we will discuss access and basic usage. 

## Compute
### Access the Systems

Secure shell ("ssh" command) is the standard way to connect to the login nodes on TACC systems. To initiate a session:

```
#Frontera
localhost$ ssh username@frontera.tacc.utexas.edu

#Lonestar6
localhost$ ssh username@ls6.tacc.utexas.edu

#Longhorn
localhost$ ssh username@longhorn.tacc.utexas.edu
```

You will be prompted to enter your password followed by the multi-factor authentication (MFA) code. Welcome text, project/allocation information, and disk quotas will be presented upon successful login. By default, you will be located in your home directory (`echo $HOME` or `pwd` to see path), which is specific to each machine (i.e. it is not shared across systems). Login nodes are shared resources and must accommodate hundreds of other user simultaneously. They are intended to be used for editing, compiling code, submiting jobs, file transfer, among other low impact tasks. Please do not run applications on the login nodes. 

### File Systems
Each machine contains 3 standard file systems. Environment variables `$HOME`, `$WORK` and `$SCRATCH` store the paths to directories that you own on each filesystem. 

| File System | Quota | Key Features|
| ----------- | ----------------| --------------------------------- |
| $HOME       | Frontera: 25GB<br />LS6: 10GB<br />Longhorn: 10GB| Not intended for parallel or high-intensity file operations.<br />Best for cron jobs, small scripts, envrionment settings.<br />Backed up regularly, not purged.   |
| $STOCKYARD | 1TB across all TACC systems | Global Shared Filesystem<br />Not intended for high-intesnity file operations or jobs involving very large files. <br /> Good for storing software installations, original datasets, job scripts and templates.<br /> Not backed up, not purged. |
| $SCRATCH | no quota | All job I/O activity, temporary storage.<br /> Files that have not been accessed in 10 or more days are subject to purge.|

Note: different systems offer additional resources. For example, Frontera includes two additonal file systems, $SCRATCH2 and $SCRATCH3, which can accommodate intensive parallel I/O operations. Consult user guides for additional details. 

### Stockyard
Stockyard is the Global Shared File System at TACC. It is mounted on all major TACC clusters. The `$STOCKYARD` environment variable points to the highest level directory that you own on the shared file system. This variable is consistent across all TACC resources that mount Stockyard. The `$WORK` environment variable, on the other hand, is resource specific and varies across systems. `$WORK` is a subdirectory of `$STOCKYARD`. 
```
/work/12345/bjones/      #$STOCKYARD on all systems
|
|---> /frontera          #$WORK on frontera
|
|---> /lonestar6         #$WORK on LS6
|
|---> /longhorn          #$WORK on longhorn
```

### Transferring Files
For Linux-based systems, `scp` or `rsync` can be used to transfer files to TACC systems. Windows SSH clients typically include `scp`-based file transfer capabilities. To transfer a file to your home directory on Frontera:

```
localhost$ scp myfile.txt jadrake@frontera.tacc.utexas.edu:
```

To transfer a file directly to your work directory on LS6, first retrieve the path to your work directory:

```
login1.ls6(11)$ echo $WORK
/work/01262/jadrake/ls6
```
then use the path to upload file:
```
localhost$ scp myfile.txt jadrake@ls6.tacc.utexas.edu:/work/01262/jadrake/ls6
```
                 
### Using Modules
**Lmod** is a module system developed and maintained at TACC that makes it easy to manage your environment so you have access to software packages. Loading a module amounts to choosing a specific package by defining or modifying environment variables. To list available modules run the following on the login node:
```
login1.ls6(14)$ module av
```

To load a specific module:
```
login1.ls6(17)$ module load python3/3.9.7
```

To see which modules are currently loaded:
```
login1.ls6(18)$ module list
```

### Running Jobs on Compute Nodes
**Batch Mode**
TACC systems run a job scheduler, [Slurm WWorkload Manager](https://schedmd.com), which provides commands to submit, manage, monitor, and control jobs. Jobs submitted to the scheduler are queued, then run on the compute nodes. TACC does not implement node-sharing. Your queue wait times will be less if you request only the time you need: the scheduler will have a much easier time finding a slot or the 2 hours you really need than, for example, if you requested 12 hours in your job script. Specific details, like the names of each queue and job request limits, can be found on each machine's user guide. Below we provide an overview of important concepts.

Jobs are are submitted from the login node. To submit a batch job (i.e. an unattended job), use the command `sbatch` followed by your job script (discussed below).  

```
login1.ls6(20)$ sbatch myjobscript
```

Until your batch job begins it will wait in the queue. You do not need to remain connected while the job is waiting or executing. 

**Interactive Mode**
An interactive session can be launched on a compute node using `idev`, functionality developed at TACC which submits a batch script requesting access to a compute node, after which the user is automatically ssh'd to that specific node. To launch a thirty-minute interactive session on a single node in the `development` queue:

```
login1.ls6(20)$ idev
```

To launch an interactive job on the normal queue, with 2 nodes, for 120 minutes:
```
login1.ls6(20)$ idev -p normal -N 2 -m 120
```

**Job Scripts**
Slurm's `sbatch` command is used to submit a batch job script. #SBATCH directives are used within the script to specifiy a number of parameters/options. The user guides provide various example job scripts. Basic example is provided below.

Example: serial job, small queue, on Frontera.
```
#!/bin/bash
#----------------------------------------------------
# Sample Slurm job script
#   for TACC Frontera CLX nodes
#
#   *** Serial Job in Small Queue***
# 
# Last revised: 22 June 2021
#
# Notes:
#
#  -- Copy/edit this script as desired.  Launch by executing
#     "sbatch clx.serial.slurm" on a Frontera login node.
#
#  -- Serial codes run on a single node (upper case N = 1).
#       A serial code ignores the value of lower case n,
#       but slurm needs a plausible value to schedule the job.
#
#  -- Use TACC's launcher utility to run multiple serial 
#       executables at the same time, execute "module load launcher" 
#       followed by "module help launcher".
#----------------------------------------------------

#SBATCH -J myjob           # Job name
#SBATCH -o myjob.o%j       # Name of stdout output file
#SBATCH -e myjob.e%j       # Name of stderr error file
#SBATCH -p small           # Queue (partition) name
#SBATCH -N 1               # Total # of nodes (must be 1 for serial)
#SBATCH -n 1               # Total # of mpi tasks (should be 1 for serial)
#SBATCH -t 01:30:00        # Run time (hh:mm:ss)
#SBATCH --mail-type=all    # Send email at begin and end of job
#SBATCH -A myproject       # Project/Allocation name (req'd if you have more than 1)
#SBATCH --mail-user=username@tacc.utexas.edu

# Any other commands must follow all #SBATCH directives...
module list
pwd
date

# Launch serial code...
# python3 myscript.py
./mycode.exe         # Do not use ibrun or any other MPI launcher
```

### Monitoring Job Status
Slurm's `squeue` command allows you to monitor jobs in the queues, whether pending or running:
```
login1$ squeue             # show all jobs in all queues
login1$ squeue -u bjones   # show all jobs owned by bjones
login1$ man squeue         # more info
```
Excerpt from the default output of `squeue` command:
```
 JOBID   PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
25618      normal   SP256U   connor PD       0:00      1 (Dependency)
25944      normal  MoTi_hi   wchung  R      35:13      1 c112-203
25945      normal WTi_hi_e   wchung  R      27:11      1 c113-131
25606      normal   trainA   jackhu  R   23:28:28      1 c119-152
```
To cancel a job you submitted, use `squeue` to find the JOBID. Then use `scancel JOBID` to cancel the job. Use `squeue` again to confirm that the job was successfully terminated. 
