# SHADE-AI Gym: Corral Quickstart

[Corral](https://portal.tacc.utexas.edu/user-guides/corral) is a collection of storage and data management resources located at TACC. Corral services provide high-reliability, high-performance storage for research requiring persistent access to large quantities of structured or unstructured data. Such data could include data used in analysis or other computational and visualization tasks on other TACC resources, as well as data used in collaborations involving many researchers who need to share large amounts of data.

For the SHADE-AI Gym, we have created workspace on Corral to allow for sharing of data, scripts, etc. within and between performer teams. Currently, there is 5TB provisioned for the program. Additionaly storage can be added as needed. 

## System Access
**Basic command-line access**

Similar to compute systems, Corral can be accessed directly through the login node via SSH:
```
localhost$ ssh username@data.tacc.utexas.edu
```

The login node is provided for use in transferring and organizing data through command-line utilities. The DARPA-SHADE directory is located at

```
/corral/projects/DARPA-SHADE
```

Within this project workspace, there are two subdirectories:

```
Shared/         #intended to store and share data, scripts, etc. across program
Performers/     #intended to store data, scripts, etc. to share within teams
```

Within Performers/ are 7 directories reserved for each performer team. Access to these directories is governed by Access Control Lists. Only users from respective teams will be able to access these directories. 

**Cluster Access**

Corral is mounted on the login and compute nodes of Frontera and Lonestar6. For I/O intensive tasks, it may be more efficient to stage data by copying from Corral to the $SCRATCH or $WORK file systems on specific machines. Data must be staged on Longhorn as Corral is not directly mounted. 

**JupyterHub Access**

The DARPA-SHADE workspace on Corral is mounted to the container running JupyterNotebooks. You will be able to access to the Shared/ and Performers/ directories.


## File Transfer
To transfer a file from your local machine the Shared directory:
```
localhost$ scp filename username@data.tacc.utexas.edu:/corral/projects/DARPA-SHADE/Shared
```

To transfer a file from Corral to your $SCRATCH directory on Lonestar6:

```
login1$ cp /corral-repl/projects/DARPA-SHADE/Shared/myfile.txt $SCRATCH/job_directory/
```
