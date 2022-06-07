### Start an interactive session on Lonestar6: single node, normal queue

By default, `idev` will submit jobs to the development queue, which imposes a 2 hour run time limit and a maximum of one job per user. Interactive sessions can be launched on other queues with the `-p` option and the requested amount of time with the `-m` option. The command below launches an interactive session on the normal queue with an allocated time of 240 minutes. 

```shell
login1.ls6(1038)$ idev -p normal -m 240
```

### Launching the containerized diplomacy game engine on Lonestar6 compute node
The docker container running the diplomacy game engine writes the server log files to the /logs directory. Because singularity imposes a read-only filesystem, /logs needs to be mapped or mounted to the host filesystem. 

```shell
#first create a directory to store log files
c307-014.ls6(1008)$ pwd
/scratch/01262/jadrake/demo_06_07_2022

c307-014.ls6(1009)$ mkdir logs

#run container: note container is running in the foreground
c307-014.ls6(1010)$ singularity run -B /scratch/01262/jadrake/demo_06_07_2022/logs:/logs docker://tacc/diplomacy_server
INFO:    Using cached SIF image

#in another terminal, check server logs to confirm running properly
login2.ls6(1007)$ cd $SCRATCH/demo_06_07_2022/
login2.ls6(1008)$ ls
data  logs  maps

login2.ls6(1011)$ tail -f diplomacy_server_run.log

100%|██████████| 4950/4950 [00:26<00:00, 184.78it/s]
2022-06-07 12:45:42 diplomacy.server.server[441671] INFO Server loaded.
2022-06-07 12:45:42 diplomacy.server.server[441671] DEBUG Ping        : 30
2022-06-07 12:45:42 diplomacy.server.server[441671] DEBUG Backup delay: 600
2022-06-07 12:45:42 diplomacy.server.server[441671] INFO Running on port 8432
2022-06-07 12:45:42 diplomacy.server.server[441671] INFO Serving DAIDE on ports 8434:8600
2022-06-07 12:45:42 diplomacy.server.server[441671] INFO Writing server data to /scratch/01262/jadrake/demo_06_07_2022
2022-06-07 12:45:42 diplomacy.server.server[441671] INFO Waiting for save events.
2022-06-07 12:45:42 diplomacy.server.server[441671] INFO Waiting for notifications to send.
```

### Launching game engine server directly on Lonestar6 compute node

```shell
#git clone diplomacy repo
#add to PYTHONPATH
c307-014.ls6(1016)$ export PYTHONPATH=/work/01262/jadrake/ls6/shade/diplomacy

#activate virtual environment with necessary dependencies installed
c307-014.ls6(1021)$ source /home1/01262/jadrake/Pyenv/diplo2/bin/activate
(diplo2) c307-014.ls6(1023)$ cd $WORK/shade/diplomacy

#run the server
(diplo2) c307-014.ls6(1023)$ python3 -m diplomacy.server.run
/work/01262/jadrake/ls6/shade/diplomacy/maps/convoy_paths_cache.pkl
2022-06-07 13:03:39 diplomacy.server.server[448148] INFO Loading database.
2022-06-07 13:03:39 diplomacy.server.server[448148] INFO Loading server.json.
2022-06-07 13:03:39 diplomacy.server.server[448148] INFO Server loaded.
2022-06-07 13:03:39 diplomacy.server.server[448148] DEBUG Ping        : 30
2022-06-07 13:03:39 diplomacy.server.server[448148] DEBUG Backup delay: 600
2022-06-07 13:03:39 diplomacy.server.server[448148] INFO Running on port 8432
2022-06-07 13:03:39 diplomacy.server.server[448148] INFO Serving DAIDE on ports 8434:8600
2022-06-07 13:03:39 diplomacy.server.server[448148] INFO Writing server data to /work/01262/jadrake/ls6/shade/diplomacy
2022-06-07 13:03:39 diplomacy.server.server[448148] INFO Waiting for save events.
2022-06-07 13:03:39 diplomacy.server.server[448148] INFO Waiting for notifications to send.
```

### Launch game engine on LS6 and connect WebUI running locally
To connect the WebUI running locally to the game engine server running remotely on LS6, we need to set up an ssh tunnel from the local machine to the compute node via the login node. Thanks to Michael C. at LM for help working this out. 

```shell
#launch game engine server (assuming idev session running)
#on LS6 compute node
(diplo2) c307-014.ls6(1029)$ singularity run -B /scratch/01262/jadrake/demo_06_07_2022/logs:/logs docker://tacc/diplomacy_server
```

#open terminal on local machine, forward localhost port 8432 to compute node
```shell
jadrake$ ssh -L 8432:c307-014:8432 jadrake@ls6.tacc.utexas.edu
```

#on local machine, launch diplomacy webui container
```shell
jadrake$ docker run -p 3000:3000 tacc/diplomacy_webui
> web@0.1.0 start
> react-scripts start

Starting the development server...

Browserslist: caniuse-lite is outdated. Please run next command `npm update`
Compiled successfully!

You can now view web in the browser.

  Local:            http://localhost:3000/
  On Your Network:  http://172.17.0.2:3000/

Note that the development build is not optimized.
To create a production build, use npm run build.
```

Then open a browser and enter webui url - http://localhost:3000/. Under server settings, set host and port to localhost and 8432, respectively. 
