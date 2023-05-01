
## Diplomacy client/engine updates to support communication protocols

Updated code can be found in `diplomacy/comm_state`

### Communication state
- added new field to `Power` class to reflect when agent is ready to communicate
- `comm_status`
  - Type: string
  - Values: [strings.INACTIVE, strings.BUSY, strings.READY]
- using `from diplomacy.utils import strings` to access enum for `comm_status` values
- initialized to `strings.INACTIVE` for all powers
- reset to `strings.BUSY` after phase is processed
- can be accessed from `network_game` object, example below:
```python
[pow.comm_status == strings.READY for pow in game.powers.values()]
```

### Updating `comm_status`

```python
await game.set_comm_status(comm_status=strings.READY)
```
- `power_name` is an optional input parameter, otherwise engine determines from context
- server will send out notification to all powers when a power's `comm_status` changes

### Player Type
- add new field to `Power` class to reflect player type
- `player_type`
  - Type: string
  - Values: [strings.HUMAN, strings.NO_PRESS_BOT, strings.PRESS_BOT]
- default value is strings.HUMAN (i.e. 'human')
- player type is specified using an additional input when joining a game
```python
game = await channel.join_game(game_id=game_id, power_name=power_name, player_type=strings.PRESS_BOT)
```

### Using existing methods to bind functions to callbacks
Network_game includes methods to bind user-defined functions to various notifications:
- add_on_cleared_centers(notification_callback)
- add_on_cleared_orders(notification_callback)
- add_on_cleared_units(notification_callback)
- add_on_game_deleted(notification_callback)
- add_on_game_message_received(notification_callback)
- add_on_game_processed(notification_callback)
- add_on_game_phase_update(notification_callback)
- add_on_game_status_update(notification_callback)
- add_on_omniscient_updated(notification_callback)
- add_on_power_orders_flag(notification_callback)
- add_on_power_orders_update(notification_callback)
- add_on_power_vote_updated(notification_callback)
- add_on_power_wait_flag(notification_callback)
- add_on_powers_controllers(notification_callback)
- add_on_vote_count_updated(notification_callback)
- add_on_vote_updated(notification_callback)

The logic for handling incoming messages can be simplified:
```python
def on_message_received(network_game, notification):
    msg = notification.message
    sender = msg.sender
    recipient = msg.recipient
    message = msg.message
    print("({}/{}): {} received the following message from {}: \n\t{}".format(network_game.get_current_phase(), notification.game_role, recipient, sender, message))
    
game = await channel.join_game(game_id=game_id, power_name=power_name, player_type=type)
game.add_on_game_message_received(on_message_received)
```

## Proposed communication protocol
### Lowest common daide-ometer
|Name|Type|Time|Syntax|Semantics / Meaning|Notes|
|-------|-------|-------|-------|-------|-------|
|Accept|Response||YES (press_message)|<ul><li> accept arrangement provided in press_message|
|Reject|Response||REJ (press_message)|<ul><li> reject arrangement provided in press_message|
|Say what|Response||HUH (press_message)|<ul><li> message or proposal includes tokens that receiving agent does not understand|
|Proposal|Proposition ||PRP (arrangement)|<ul><li> indicated sending power is proposing an arrangement|
|Control Z|Proposition |Current turn|NAR (arrangement)|<ul><li> undo previous arrangement|
|Peace|Arrangement|Persistent|PCE ( power1 power2 )|<ul><li>arrange peace between listed powers <li> powers agree not directly attack or support attacks on each others provinces</ul>|Suggestion: limit peace arrangements to two powers only. Decompose multi-party arrangements into pairs|
|Ally|Arrangement|Persistent|ALY ( power1 power2 ) VSS (power power ...)|<ul><li> arrange alliance between power1 and power2 against those powers in the second list <li> includes restrictions of peace agreement and an explicit delcaration that powers in the second list be treated as enemies</ul>|
|Order|Arrangement|Current turn|XDO ( order )|<ul><li> arrangement for the provided order to be executed <li> applies to next turn in which order is valid (primarily current)</ul>|
|Demilitarize |Arrangement|Persistent|DMZ ( power power ...) (province province ...)|<ul><li> arrangement for listed powers to remove all units from, and not order to, support to, convoy to, retreat to, or build any units in any of the listed provinces|
|Multi-order|Arrangement|Current turn|AND ( XDO (order) ) ( XDO (order) )|<ul><li> orders are grouped and should be accepted or rejected as a group|Suggestion: limit multipart arrangements to order type. Achieve other multipart arrangements by issuing individual proposals.|
  
  ### Pseudocode
  ```
  join game and specify player type
  register callback function for on_game_message_received
  
  FOR EACH PHASE
    IF phase != retreat OR phase != adjustment
   
      #comm_status is set to busy after each phase process
      assess board state and query models
      comm_status = strings.READY

      IF comm_status == strings.READY for all player_type == strings.PRESS_BOT
        #proceed to diplomacy sub-phase
        diplomacy = True
        diplomacy_time = 60
        start diplomacy timer

        WHILE diplomacy
          send messages
          received and respond to messages using callback
          IF elapsed time >= diplomacy_time
            diplomacy = False

    #ORDER sub-phase
    select orders
    submit orders
    set wait_flag = False
    IF wait_flag == False for all powers
      process game phase
      engine sets comm_status = strings.BUSY and wait_flag = True
      CONTINUE
  
  ```
