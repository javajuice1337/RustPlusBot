# RustPlusBot
RustPlusBot plugins reference and examples

**RustPlusBot** plugins allow the user to develop their own commands and functionality for the bot. The plugin itself exposes many events for a programmer to attach to and execute code.

Plugins are written in JavaScript and run in a NodeJS environment.

##Plugin Events:

**onConnected** Fires when the bot connects to a server
```Arguments: none```
**onDisconnected** Fires when the bot disconnects from a server
```Arguments: none```
**onEntityChanged** Fires when a paired Smart Device is changed
```Arguments: obj.entityId, obj.payload```
**onMessageReceive** Fires when a team chat message is received
```Arguments: obj.message, obj.name, obj.steamId```
**onMessageSend** Fires when a team chat message is sent
```Arguments: obj.message```
**onNotification** Fires when a there is a bot notification
```Arguments: obj.data```
**onTeamChanged** Fires when a team member is added or removed from the team
```Arguments: obj.leaderSteamId, obj.leaderMapNotes, obj.members```
