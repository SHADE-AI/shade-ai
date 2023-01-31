# Intent Logging Quickstart
The (Paquette) Diplomacy engine has been modified to include the ability to log metadata (e.g. intent associated with messages, general strategy, world observations, etc.). Changes were made in the client and server code to create a dedicated, power-specific logging mechanism. Logging follows the same pattern as game messaging, detailed below. 

## Checkout branch: `intent_log`

Checkout the `intent_log` branch in the diplomacy repo. Note: shade.tacc.utexas.edu will continue to use the `development` branch while the teams transition to the updated codebase. Consequently, client code from the `intent_log` branch is not expected to work with the server code from the `development` branch`.

## Test with development server

To test out the new logging API, you can use the following server, shade-dev.tacc.utexas.edu (UI: port 3000 and engine: port 8432). The `intent_log` branch will be merged with `development` and re-launched on shade.tacc.utexas.edu in the future, but prior to the next evaluation. Note: the UI currently does not display the logs but you can download the game state file to view. 

## New data structures

class **diplomacy.engine.log.Log(kwargs)**

  Bases: diplomacy.utils.jsonable.Jsonable
  Log class. 
  Properties
  - sender: message sender name: power name
  - recipient: OMNICIENT
  - time_sent: log timestamp set by server in microseconds
  - phase: short name of game phase when log was sent
  - message: content of the log message to record

Additional properties added to diplomacy.engine.game.Game

**logs**
  - Logs sent by power during current phase
  - Sorted dict mapping log timestamp (key) to log object (value)
  - Cleared following processing of phase

**log_history**
  - history of logs sent by power up to but not including current phase
  - Sorted dict maps phase name (key) to another sorted dict where timestamp (key2) maps to log object (value)
  - logs (current phase) will be appended to log_history following game processing

## New methods
**Game.new_log_data(body)**
  - creates a new instance of Log class
  - body: String containing the data to be logged
  - returns: Log object
 
**NetworkGame.send_log_data(log)**
  - creats and sends a requests.SendLogData request to server
  - log: Log object containing data to send
  - server returns a timestamp (microseconds) 

## Basic Usage
```python
#assuming Game object has been created
log_data = game.new_log_data(body="Test log")
await game.send_log_data(log=log_data)

#or more compactly
await game.send_log_data(log=game.new_log_data(body="Test log compact"))

#logged data will be appended to game.logs, which can be indexed by timestamp
```

```python
#print content of log_history
for phase in game.log_history:
  logs = game.log_history[phase]
    for t in logs:
      log = logs[t]
      print(log.phase + "\t" + str(log.time_sent) + "\t" + log.sender + "\t" + log.message)
```

## Full example
Game with all dumbbots. Powers will create and send log data and messages with 50% probability during a phase. 

```python

import asyncio
from diplomacy.client.connection import connect
from diplomacy_research.players import RuleBasedPlayer
from diplomacy_research.players.rulesets import dumbbot_ruleset
from random import *

POWERS = ['AUSTRIA', 'ENGLAND', 'FRANCE', 'GERMANY', 'ITALY', 'RUSSIA', 'TURKEY']

async def create_game(game_id, hostname='shade-dev.tacc.utexas.edu', port=8432):
    """ Creates a game on the server """
    connection = await connect(hostname, port)
    channel = await connection.authenticate('random_user', 'password')
    await channel.create_game(game_id=game_id, rules={'REAL_TIME', 'NO_DEADLINE', 'POWER_CHOICE'})

async def play(game_id, power_name, hostname='shade-dev.tacc.utexas.edu', port=8432):
    """ Play as the specified power """
    connection = await connect(hostname, port)
    channel = await connection.authenticate("bot_"+power_name,'password')
    bot = RuleBasedPlayer(dumbbot_ruleset)

    # Waiting for the game, then joining it
    while not (await channel.list_games(game_id=game_id)):
        await asyncio.sleep(1.)
    game = await channel.join_game(game_id=game_id, power_name=power_name)

    # Playing game
    while not game.is_game_done:
        current_phase = game.get_current_phase()

        #log data
        if current_phase != "S1901M" and randint(1,100) > 50:
            msg = "LOG CHECK from " + power_name + "in " + current_phase
            await game.send_log_data(log=game.new_log_data(body=msg))
            await asyncio.sleep(1)
        if current_phase != "S1901M" and randint(1,100) > 50:
            msg = "MESSAGE CHECK from " + power_name + "in " + current_phase
            temp = [rec for rec in POWERS if not rec == power_name]
            recipient = temp[randint(0,3)]
            await game.send_game_message(message=game.new_power_message(recipient, msg))
            await asyncio.sleep(1)

        # Submitting orders
        orders = await bot.get_orders(game, power_name)
        print("{}\t{}\t{}".format(current_phase, power_name, orders))
        await game.set_orders(power_name=power_name, orders=orders, wait=False)

        if current_phase == "F1905M":
            for phase in game.log_history:
                logs = game.log_history[phase]
                for t in logs:
                    log = logs[t]
                    print(log.phase + "\t" + str(log.time_sent) + "\t" + log.sender + "\t" + log.message)
                    
        # Waiting for game to be processed
        while current_phase == game.get_current_phase():
            await asyncio.sleep(0.1)

async def launch(game_id):
    """ Creates and plays a network game """
    await create_game(game_id)
    await asyncio.gather(*[play(game_id, power_name) for power_name in POWERS])

if __name__ == '__main__':
    asyncio.run(launch(game_id=str(randint(1, 1000))))

```
