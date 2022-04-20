# Diplomacy benchmark bots: quickstart

We have compiled and containerized several existing Diplomacy bots to assist in the training and development of SHADE AI bots. The table below provides additional details. All bots are set to auto-play after joining a game. Containers can be launched with `docker` or `singularity`. Dockerfile and build instructions can be found in [SHADE-AI/diplomacy-playground/bots repo](https://github.com/SHADE-AI/diplomacy-playground), currently in the dev branch.

|        Bot       |     Image (Docker) |       Description          |          Refs         |
|:-----------------|:-------------------|:---------------------------|----------------------|
|Albert v6         | tacc/albert-ai:v1  | -Capable of DAIDE level 30 <br /> -Windows exe run with wine within docker <br /> -randomly assigned power when join game engine| Developer:[Jason van Hal](https://sites.google.com/site/diplomacyai/home?authuser=0) <br /> Implementation: [Paquette et. al](https://github.com/SHADE-AI/research/tree/master/diplomacy_research/containers/albert-ai)|
|Dumbbot (python)  | tacc/dumbbot:v1    | -Python implementation of David Norman's dumbbot<br />-uses rule_based_player.py and ruleset in [diplomacy_research](https://github.com/SHADE-AI/research/tree/master/diplomacy_research/players) repo| Implemented by Paquette et al.|
|DipNetSL          | tacc/dipnet_sl:v1  | -No-press bot developed by Paquette et. al<br />-runs a tensorflow model server which provides orders via DipNetSLPlayer class | [Paquette et al.](https://arxiv.org/abs/1909.02128)|
|DipNetSL TF model server| tacc/dipnet_sl_tf_server:v1| -Tensorflow model server for DipNetSL<br />-A single running instance can support numerous DipNetSLPlayers<br />-separate code required to create/connect player and join game|[Paquette et al.](https://arxiv.org/abs/1909.02128)|
|Searchbot/DORA    | TBD                | -No-press bot developed by Facebook<br />-requires a translation layer to Paquette's game engine| [Bakhtin et al.](https://arxiv.org/pdf/2110.02924.pdf)|
|Deepmind          | TBD                | -No-press bot developed by Deepmind<br />-requires a translation layer which they provide to Paquette's game engine| [Anthony et al.](https://arxiv.org/abs/2006.04635)|

## Albert V6
Dockerfile, build instructions, and other information can be found [here](https://github.com/SHADE-AI/diplomacy-playground/blob/dev/bots/albert/BUILD.md). Currently, it appears that Paquette's game engine does not allow the explicit assignment of powers to DAIDE players. That is, the game engine will assign any empty power following the NME or HLO DAIDE message. 

```shell
# Get usage
% docker run -it albert-ai --help

Usage: run.sh [options]
   -s | --host	HOSTNAME
   -p | --port	DAIDE_PORT
   -u | --power	POWER
   -i 		IP_ADDRESS
   -n		set never ally mode
   -g		set gunboat mode
   -t		set tournament mode
```
Note: we've included a --power option for consistency across bots but it is ignored here.

Example: create local game and launch 7 Albert bots
```shell
#create game, retrieve daide_port. Running with dev branch of diplomacy-playground. 
$ python diplomacy-playground/scripts/create_game.py --game_id all_alberts
{
    "id": "all_alberts",
    "deadline": 0,
    "map_name": "standard",
    "registration_password": null,
    "rules": [
        "REAL_TIME",
        "POWER_CHOICE"
    ],
    "n_controls": 7,
    "status": "forming",
    "daide_port": 8547
}

#launch albert bots
for i in {0..6}
do
	singularity run docker://tacc/albert-ai:v1 --host localhost --port 8547 &
	sleep 2
done

#to run with docker use the following:
# docker run -it tacc/albert-ai:v1 --host host.docker.internal --port 8547
```
## Dumbbot (python)
Dockerfile, build instructions, and other information can be found [here](https://github.com/SHADE-AI/diplomacy-playground/blob/dev/bots/dumbbot).
This implementation of David Norman's [dumbbot]http://www.daide.org.uk/s0003.html uses the RuleBasedPlayer class from diplomacy_research.players attributed to Paquette et al. More information on the 'easy' ruleset can be found [here](https://github.com/SHADE-AI/research/blob/master/diplomacy_research/players/rulesets/easy_ruleset.py). 

Usage:
```shell
$ docker run -it dumbbot-python --help
options
  --game_id GAME_ID
  --power POWER
  --host HOST [default localhost]
  --port PORT [default 8432]
  --ruleset RULESET [(default) dumbbot | easy]
```

Easy rulest:
```
Easy Ruleset
    Movement phase:
        1) - Hold if unit is on a foreign SC and SC is not captured
        2) - Attack unoccupied enemy SC
        3) - Move to unoccupied enemy territory
        4) - Attack occupied enemy SC
        5) - Attack occupied enemy unit
        6) - Move in direction of closest foreign SC
        7) - Otherwise hold
    Retreat phase:
        - Move to state having most friendly surrounding units
        - Disband if no retreat locations possible
    Adjustement phase:
        - If build, maintain a 60% land, 40% fleet ratio, build in location closest to closest enemy SC first
        - If disband, disband units that are further from enemy territory
```

Example: 7 dumbbots
```shell
GAME_ID="all_dumbbots"

singularity run docker://tacc/dumbbot:v1 --game_id all_dumbbots --power AUSTRIA &
singularity run docker://tacc/dumbbot:v1 --game_id all_dumbbots --power ENGLAND &
singularity run docker://tacc/dumbbot:v1 --game_id all_dumbbots --power GERMANY &
singularity run docker://tacc/dumbbot:v1 --game_id all_dumbbots --power FRANCE &
singularity run docker://tacc/dumbbot:v1 --game_id all_dumbbots --power RUSSIA &
singularity run docker://tacc/dumbbot:v1 --game_id all_dumbbots --power TURKEY &
singularity run docker://tacc/dumbbot:v1 --game_id all_dumbbots --power ITALY &

#docker
#docker run -d tacc/dumbbot:v1 --host host.docker.internal --game_id [GAME_ID] --power [POWER]

#run with remote game engine at TACC
docker run -d tacc/dumbbot:v1 --host shade.tacc.utexas.edu --game_id [GAME_ID] --power [POWER]

```

## DipNetSL
Dockerfile, build instructions, and other information can be found [here](https://github.com/SHADE-AI/diplomacy-playground/tree/dev/bots/dipnet_sl). This container includes the tensorflow model server and a python script that will create a DipNetSLPlayer, which will join and play a game. 

Usage:
```shell
$ docker run -it dipnet_sl --help
--host 		HOST [default localhost]
--port 		PORT [default 8432]
--game_id 	GAME_ID
--power		POWER
```

Example: 1 DipNetSL vs. 6 dumbbots running on remote game engine
```shell
GAME_ID="dipnet_v_dumbbots"
HOST="shade.tacc.utexas.edu"

singularity run docker://tacc/dipnet_sl:v1 --host $HOST  --game_id $GAME_ID --power AUSTRIA &
singularity run docker://tacc/dumbbot:v1 --host $HOST  --game_id $GAME_ID --power ENGLAND &
singularity run docker://tacc/dumbbot:v1 --host $HOST  --game_id $GAME_ID --power RUSSIA &
singularity run docker://tacc/dumbbot:v1 --host $HOST  --game_id $GAME_ID --power GERMANY &
singularity run docker://tacc/dumbbot:v1 --host $HOST  --game_id $GAME_ID --power TURKEY &
singularity run docker://tacc/dumbbot:v1 --host $HOST  --game_id $GAME_ID --power ITALY &
singularity run docker://tacc/dumbbot:v1 --host $HOST  --game_id $GAME_ID --power FRANCE &
```

We have not extensively tested various combinations of bots benchmark bots playing one another. If you find that some configuration fails, please let us know. Also if you have a bot (e.g. Searchbot/DORA) that is able to connect and play a game with Paquette's game engine and would like to include that in the benchmark suite, please let the TACC team know. 
