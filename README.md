<p align="center">
<a href="https://bot.rustplus.io/"><img src="./rustplusbot.png" width="400"></a>
</p>

# **RustPlusBot** plugins reference and examples

**RustPlusBot** plugins allow you to develop your own commands to add functionality to the bot. The plugin itself exposes many events for a programmer to attach to and execute code.

The plugins are written in JavaScript and run in a NodeJS environment after they are published. During development, you host the plugin on your client machine and in your web-browser. Your plugin interfaces with the bot via a WebSocket connection and communicates using the RustPlusBot api.

You can load any of the official plugins and use them as a template for getting started in the Plugin Studio. The Plugin Studio can be accessed via a link in the Plugin settings tab on the RustPlusBot settings page for your Discord server.

You can also find complete plugin examples on github in the <a href="plugin examples/">plugin examples</a> folder.

> Plugins are loaded when the bot is starting and lasts for its entire life-cycle. Restarting the bot also restarts all plugins.

## Plugin Storage

For data that persists beyond the bot's instance, use `this.storage`. This object loads with the bot and saves when it stops or restarts.

```
// store the myData variable
if (!this.storage.myData) this.storage.myData = 'Hello World!';
console.log(this.storage.myData);
```

## Plugin Events

<ul>
  <li><code>onConnected()</code> Fires when the bot connects to a server or when the plugin loads</li>
  <li><code>onDisconnected()</code> Fires when the bot disconnects from a server</li>
  <li><code>onEntityChanged(obj)</code> Fires when a paired Smart Device is changed<ul><li><b>obj.entityId</b>: <sup><code>int</code></sup> The entity ID of the Smart device</li><li><b>obj.payload</b>: <sup><code>object</code></sup> The payload data of the event (see <code><a href="#Payload">Payload</a></code> below)</li></ul></li>
  <li><code>onMessageReceive(obj)</code> Fires when a team chat message is received<ul><li><b>obj.message</b>: <sup><code>string</code></sup> The incoming team chat message</li><li><b>obj.name</b>: <sup><code>string</code></sup> The steam name of the sender</li><li><b>obj.steamId</b>: <sup><code>string</code></sup> The steam ID of the sender</li></ul></li>
  <li><code>onMessageSend(obj)</code> Fires when a team chat message is sent<ul><li><b>obj.message</b>: <sup><code>string</code></sup> The outgoing team chat message</li></ul></li>
  <li><code>onNotification(obj)</code> Fires when there is a bot notification (including server events)<ul><li><b>obj.notification</b>: <sup><code>object</code></sup> The notification data of the event (see all <code><a href="#NotificationAlarm">Notification</a></code> below)</li></ul></li>
  <li><code>onTeamChanged(obj)</code> Fires when the team leader changes, or a team member is added or removed from the team<ul><li><b>obj.leaderSteamId</b>: <sup><code>object</code></sup> The steam ID of the team leader</li><li><b>obj.leaderMapNotes</b>: <sup><code>object</code></sup> The leader map notes data of the event (see <code><a href="#MapNotes">MapNotes</a></code> below)</li><li><b>obj.members</b>: <sup><code>object</code></sup> The members list data of the event (see <code><a href="#Members">Members</a></code> below)</li></ul></li>
</ul>

## Plugin Interface

The `app` object exists in the plugin's scope `this`, and exposes the following properties and methods:

### Properties

<ul>
  <li><code>bmData</code> <sup><code>object</code></sup> An object containing the BattleMetrics data of the server (see <code><a href="#BattleMetricsData">BattleMetrics Data</a></code> below)</li>
  <li><code>cameras</code> <sup><code>array</code></sup> An array containing all camera indentifiers saved from the Camera Station</li>
  <li><code>cfg</code> <sup><code>object</code></sup> An object containing the configuration settings for the bot (see <code><a href="#Config">Config</a></code> below)</li>
  <li><code>devices</code> <sup><code>Map</code></sup> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the bot's paired devices (<b>key</b>: device name (lowercase only), <b>value</b>: Array of devices, see <code><a href="#Device">Device</a></code> below)</li>
  <li><code>devices_auto</code> <sup><code>Map</code></sup> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the bot's automatic paired devices function (<b>key</b>: device name, <b>value</b>: An object containing the automatic function config (see <code><a href="#DeviceAuto">DeviceAuto</a></code> below))</li>
  <li><code>devices_icons</code> <sup><code>Map</code></sup> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the bot's paired device icons (<b>key</b>: device name, <b>value</b>: The icon name of the device)</li>
  <li><code>devices_map</code> <sup><code>Map</code></sup> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the bot's paired device names (<b>key</b>: device ID, <b>value</b>: Array of lowercased device names)</li>
  <li><code>event_types</code> <sup><code>Map</code></sup> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing server event names (<b>key</b>: event ID, <b>value</b>: event name)</li>
  <li><code>eventTimers</code> <sup><code>object</code></sup> An object containing the respawn timers for server events in seconds (see <code><a href="#EventTimers">Event Timers</a></code> below)</li>
  <li><code>guild_token</code> <sup><code>string</code></sup> The unique token representing the Discord server</li>
  <li><code>guild_name</code> <sup><code>string</code></sup> The name of the Discord server</li>
  <li><code>itemIds</code> <sup><code>Map</code></sup> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the item names for all item IDs (<b>key</b>: item ID, <b>value</b>: item name)</li>
  <li><code>monuments</code> <sup><code>Map</code></sup> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing all monument tokens  and locations (<b>key</b>: monument token, <b>value</b>: Array of monument locations, see <code><a href="#Point">Point</a></code> below)</li>
  <li><code>tokenMap</code> <sup><code>Map</code></sup> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the monument names for all monument tokens (<b>key</b>: monument token, <b>value</b>: monument name)</li>
  <li><code>player_id</code> <sup><code>string</code></sup> The steam ID of the bot's connected player</li>
  <li><code>player_name</code> <sup><code>string</code></sup> The steam name of the bot's connected player</li>
  <li><code>server_ip</code> <sup><code>string</code></sup> The IP address of the bot's connected server</li>
  <li><code>server_name</code> <sup><code>string</code></sup> The name of the bot's connected server</li>
  <li><code>server_port</code> <sup><code>string</code></sup> The port of the bot's connected server (Rust+ app port)</li>
  <li><code>server_tags</code> <sup><code>string</code></sup> Internal tags used to describe this server</li>
</ul>

```
// get the bot's language setting
console.log(this.app.cfg.lang);
```

### Methods

<ul>
  <li><code>getBattleMetrics(serverId, playerName, success, error)</code> Retrieve the BattleMetrics for a server player<ul><li><b>serverId</b>: <sup><code>string</code></sup> The BattleMetrics ID of the server</li><li><b>playerName</b>: <sup><code>string</code></sup> The name of the player</li><li><b>success(data)</b>: <sup><code>function</code></sup> The function to execute after receiving the BattleMetrics data (optional)</li><li><b>error(err)</b>: <sup><code>function</code></sup> The function to execute when an error occurs (optional)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code></li></ul><p><pre><code>// getBattleMetrics example
var app = this.app;
app.getBattleMetrics('123456', 'Rust Player 2099', (data) => {
    if (data && data.name) {
        app.sendTeamMessage('Server player \'' + data.name + '\' is ' + ((data.online) ? 'ONLINE' : 'OFFLINE') + ' and was last seen ' + getTimeDifference(data.lastseen) + ' ago');
        app.sendTeamMessage('Server player \'' + data.name + '\' was first seen ' + getTimeDifference(data.firstseen) + ' ago and has a time played of ' + getTimeDisplay(data.timeplayed));
    }
    else if (data && data.indexOf('unavailable') > 0) {
        app.sendTeamMessage(data);
    }
    else app.sendTeamMessage('Server player not found');
}, (error) => {
    app.sendTeamMessage('Error obtaining the BattleMetrics data: ' + error);
});
</code></pre></p></li>
  <li><code>getCameraPlayers(id, callback)</code> Get all player names visible on the specified camera<ul><li><b>id</b>: <sup><code>string</code></sup> The identifier of the camera</li><li><b>callback(players, playersDistances)</b>: <sup><code>function</code></sup> The function to execute after getting the players list and players distances list (<code>players</code> is an <code>array</code> of player names and <code>playersDistances</code> is an <code>array</code> of player distances in meters)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// getCameraPlayers example
var app = this.app;
app.getCameraPlayers('CAMERA_ID', (players, playersDistances) => {
    if (players && players.length > 0) {
        for (var i = 0; i < players.length; i++)
            players[i] += ' [' + playersDistances[i] + 'm]';
        app.sendTeamMessage('Detected these players: ' + players.join(', '));
    }
    else {
        app.sendTeamMessage('No players detected on camera');
    }
});
</code></pre></p></li>
  <li><code>getCameraPlayersFiltered(id, callback)</code> Get only non-team member player names visible on the specified camera<ul><li><b>id</b>: <sup><code>string</code></sup> The identifier of the camera</li><li><b>callback(players, playersDistances)</b>: <sup><code>function</code></sup> The function to execute after getting the players list and players distances list (<code>players</code> is an <code>array</code> of player names and <code>playersDistances</code> is an <code>array</code> of player distances in meters)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// getCameraPlayersFiltered example
var app = this.app;
app.getCameraPlayersFiltered('CAMERA_ID', (players, playersDistances) => {
    if (players && players.length > 0) {
        for (var i = 0; i < players.length; i++)
            players[i] += ' [' + playersDistances[i] + 'm]';
        app.sendTeamMessage('Detected these enemy players: ' + players.join(', '));
    }
    else {
        app.sendTeamMessage('No enemy players detected on camera');
    }
});
</code></pre></p></li>
  <li><code>getConnected()</code> Get the date of the server connection<ul><li><b>returns</b>: <sup><code>string</code></sup> a Date string</li></ul><p><pre><code>// getConnected example
var connected = await this.app.getConnected();
console.log(new Date(connected));
</code></pre></p></li>
  <li><code>getCrateTimer()</code> Get the configured crate unlock timer<ul><li><b>returns</b>: <sup><code>int</code></sup> value</li></ul><p><pre><code>// getCrateTimer example
var timer = await this.app.getCrateTimer();
console.log('Crate unlock timer is set to: ' + getTimeDisplay(timer));
</code></pre></p></li>
  <li><code>getDetailedInfo(callback)</code> Get detailed information about the server<ul><li><b>callback(data)</b>: <sup><code>function</code></sup> The function to execute after getting the detailed info (<code>data</code> is <code><a href="#DetailedInfo">DetailedInfo</a></code>)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// getDetailedInfo example
var app = this.app;
app.getDetailedInfo((data) => {
    if (data && data.players) {
        app.sendTeamMessage('Current server population is ' + data.players);
        app.sendTeamMessage('Current server time is ' + data.time);
    }
});
</code></pre></p></li>
  <li><code>getEntityInfo(id, callback)</code> Get data from a Smart device<ul><li><b>id</b>: <sup><code>int</code></sup> The identifier of the Smart device</li><li><b>callback(message)</b>: <sup><code>function</code></sup> The function to execute after getting the entity info (<code>message.response</code> contains <code><a href="#EntityInfo">EntityInfo</a></code>)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// getEntityInfo example
var app = this.app;
app.getEntityInfo(123456, (message) => {
    if (message.response && message.response.entityInfo && message.response.entityInfo.payload) {
        app.sendTeamMessage('This device is: ' + ((message.response.entityInfo.payload.value) ? 'On' : 'Off'));
    }
    else {
        app.sendTeamMessage('This device is inactive');
    }
});
</code></pre></p></li>
  <li><code>getEvents(type)</code> Get the most recent server events (ordered by newest to oldest)<ul><li><b>type</b>: <sup><code>string</code></sup> The event type (optional)<ul><li><code>heli</code> Patrol Helicopter</li><li><code>cargo</code> Cargo Ship</li><li><code>crate</code> Locked Crate</li><li><code>ch47</code> CH-47 Chinook</li><li><code>oil_rig_small</code> Oil Rig (Small)</li><li><code>large_oil_rig</code> Oil Rig (Large)</li></ul></li><li><b>returns</b>: <sup><code>array</code></sup> <code><a href="#Event">Event</a></code> array</li></ul><p><pre><code>// getEvents example
var e = await this.app.getEvents();
console.log(e);
</code></pre></p></li>
  <li><code>getInfo(callback)</code> Get information about the server<ul><li><b>callback(message)</b>: <sup><code>function</code></sup> The function to execute after getting the info (<code>message.response</code> contains <code><a href="#Info">Info</a></code>)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// getInfo example
var app = this.app;
app.getInfo((message) => {
    if (message.response && message.response.info) {
        app.sendTeamMessage('Current server population is ' + message.response.info.players + ' / ' + message.response.info.maxPlayers + ((message.response.info.queuedPlayers > 0) ? ' (' + message.response.info.queuedPlayers + ' in queue)' : ''));
    }
});
</code></pre></p></li>
  <li><code>getMapInfo(callback)</code> Get information about the server's map<ul><li><b>callback(message)</b>: <sup><code>function</code></sup> The function to execute after getting the map info (<code>message</code> is <code><a href="#MapInfo">MapInfo</a></code>)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// getMapInfo example
var app = this.app;
app.getMapInfo((message) => {
    if (message && message.image) {
        app.sendTeamMessage('Link to server map: ' + message.image);
    }
});
</code></pre></p></li>
  <li><code>getMapMarkers(callback)</code> Get information about all map markers<ul><li><b>callback(message)</b>: <sup><code>function</code></sup> The function to execute after getting the map markers (<code>message.response</code> contains <code><a href="#MapMarkers">MapMarkers</a></code>)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// getMapMarkers example
var app = this.app;
app.getMapMarkers((message) => {
    if (message.response && message.response.mapMarkers && message.response.mapMarkers.markers) {
        var cnt = 0;
        for (var i = 0; i < message.response.mapMarkers.markers.length; i++) {
            if (message.response.mapMarkers.markers[i].type == 3) { // 'VendingMachine'
                if (message.response.mapMarkers.markers[i].sellOrders.length > 0) cnt++;
            }
        }
        app.sendTeamMessage('There are this many active vending machines: ' + cnt);
    }
});
</code></pre></p></li>
  <li><code>getMonumentName(x, y)</code> Get the name of the monument at the specified coordinates<ul><li><b>x</b>: <sup><code>int</code></sup> The x-coordinate of where the monument is located</li><li><b>y</b>: <sup><code>int</code></sup> The y-coordinate of where the monument is located</li><li><b>returns</b>: <sup><code>string</code></sup> The name of the monument</li></ul><p><pre><code>// getMonumentName example
var app = this.app,
    x = 1000,
    y = 1000;
app.sendTeamMessage('The monument located at ' + (await app.util.getMapCoords(x, y)) + ' is: ' + (await app.getMonumentName(x, y)));
</code></pre></p></li>
  <li><code>getPrefix(type)</code> Get the command prefix for the bot<ul><li><b>type</b>: <sup><code>string</code></sup> The command type (optional)<ul><li><code>all</code> All Commands</li><li><code>device</code> Device Commands</li></ul></li><li><b>returns</b>: <sup><code>string</code></sup> The selected prefix if it's required</li></ul><p><pre><code>// getPrefix example
var prefix = await this.app.getPrefix('all');
</code></pre></p></li>
  <li><code>getRecyclerItems(items, success, error)</code> Retrieve the recycled items for the input items<ul><li><b>items</b>: <sup><code>object</code></sup> An object containing the item names and item values</li><li><b>success(data)</b>: <sup><code>function</code></sup> The function to execute after receiving recycle data (optional)</li><li><b>error(err)</b>: <sup><code>function</code></sup> The function to execute when an error occurs (optional)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code></li></ul><p><pre><code>// getRecyclerItems example
var app = this.app;
app.getRecyclerItems({'Sheet Metal Door': 1}, (data) => {
    var keys = Object.keys(data),
        recycle = [];
    for (var i = 0; i < keys.length; i++) {
        recycle.push(keys[i] + ' x ' + data[keys[i]]);
    }
    if (recycle.length > 0) app.sendTeamMessage('Recyclables: ' + recycle.join(', '));
}, (error) => {
    app.sendTeamMessage('Error obtaining the recyle items: ' + error);
});
</code></pre></p></li>
  <li><code>getSteamrep(steamId, success, error)</code> Retrieve the Steamrep data for a Steam member<ul><li><b>steamId</b>: <sup><code>string</code></sup> The steam ID of the Steam member</li><li><b>success(data)</b>: <sup><code>function</code></sup> The function to execute after receiving Steamrep data (optional)</li><li><b>error(err)</b>: <sup><code>function</code></sup> The function to execute when an error occurs (optional)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code></li></ul><p><pre><code>// getSteamrep example
var app = this.app;
app.getSteamrep(123456789, (data) => {
    if (data && data.membersince) {
        app.sendTeamMessage('This player has been a member of Steam since ' + data.membersince);
        app.sendTeamMessage('This player has ' + ((data.vacbanned != 'None' || data.communitybanned != 'None' || data.tradebanstate != 'None') ? 'bans: ' + ((data.vacbanned != 'None') ? 'VAC ' : '') + ((data.communitybanned != 'None') ? 'Community ' : '') + ((data.tradebanstate != 'None') ? 'Trade ' : '') : 'NO bans'));
    }
    else app.sendTeamMessage('Unable to lookup Steamrep data');
}, (error) => {
    app.sendTeamMessage('Error obtaining the Steamrep data: ' + error);
});
</code></pre></p></li>
  <li><code>getTeamChat(callback)</code> Get recent team chat messages<ul><li><b>callback(message)</b>: <sup><code>function</code></sup> The function to execute after getting the team chat messages (<code>message.response</code> contains <code><a href="#TeamChat">TeamChat</a></code>)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// getTeamChat example
const messages_max = 5;
var app = this.app;
app.getTeamChat((message) => {
    if (message.response && message.response.teamChat) {
        var cnt = message.response.teamChat.messages.length,
            max = (cnt > messages_max) ? messages_max : cnt;
        app.sendTeamMessage('Showing the last ' + cnt + ' team chat messages:');
        for (var i = 0; i < cnt; i++) {
            app.sendTeamMessage('(' + getFriendlyDate(new Date(message.response.teamChat.messages[cnt - i - 1].time * 1000)) + ') ' + message.response.teamChat.messages[cnt - i - 1].message);
        }
    }
});
</code></pre></p></li>
  <li><code>getTeamData(callback)</code> Get detailed information about the team (leader, members)<ul><li><b>callback(data)</b>: <sup><code>function</code></sup> The function to execute after getting the team data (<code>data</code> is <code><a href="#TeamData">TeamData</a></code>)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// getTeamData example
var app = this.app;
app.getTeamData((data) => {
    if (data && data.members) {
        var info = '';
        for (var i = 0; i < data.members.length; i++) {
            if (data.members[i].steamId == data.leader.steamId) {
                info = data.members[i].status + '; ' + data.members[i].info;
                break;
            }
        }
        app.sendTeamMessage('Team leader \'' + data.leader.name + '\' is ' + info);
    }
});
</code></pre></p></li>
  <li><code>getTeamInfo(callback)</code> Get information about the team (leader, members)<ul><li><b>callback(message)</b>: <sup><code>function</code></sup> The function to execute after getting the team info (<code>message.response</code> contains <code><a href="#TeamInfo">TeamInfo</a></code>)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// getTeamInfo example
var app = this.app;
app.getTeamInfo((message) => {
    if (message.response && message.response.teamInfo) {
        var cnt = 0;
        for (var i = 0; i < message.response.teamInfo.members.length; i++) {
            if (message.response.teamInfo.members[i].isAlive) cnt++;
        }
        app.sendTeamMessage('There are this many alive team members: ' + cnt);
    }
});
</code></pre></p></li>
  <li><code>getTime(callback)</code> Get information about the server time<ul><li><b>callback(message)</b>: <sup><code>function</code></sup> The function to execute after getting the time (<code>message.response</code> contains <code><a href="#Time">Time</a></code>)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// getTime example
var app = this.app;
app.getTime(async (message) => {
    if (message.response && message.response.time) {
        app.sendTeamMessage('Current Rust time is ' + (await app.getTimeInfo(message.response.time)));
    }
});
</code></pre></p></li>
  <li><code>getTimeInfo(time)</code> Get the server time + day/night display text<ul><li><b>time</b>: <sup><code>object</code></sup> The time object from the getTime method</li><li><b>returns</b>: <sup><code>string</code></sup> the server time + day/night display text</li></ul><p><pre><code>// getTimeInfo example
var app = this.app;
app.getTime(async (message) => {
    if (message.response && message.response.time) {
        app.sendTeamMessage('Current Rust time is ' + (await app.getTimeInfo(message.response.time)));
    }
});
</code></pre></p></li>
  <li><code>postDiscordMessage(msg)</code> Post a message to the bot's Main Discord channel or a custom channel<ul><li><b>msg</b>: <sup><code>string</code></sup> The message to post</li><li><b>msg</b>: <sup><code>object</code></sup> The message object containing the <code>message</code><sup><code>string</code></sup> and <code>channel</code><sup><code>string</code></sup>, optional <code>tts</code><sup><code>bool</code></sup> Note: <code>channel</code> is the channel ID</li></ul><p><pre><code>// postDiscordMessage example 1
this.app.postDiscordMessage('This is a message from a bot\'s plugin');
// postDiscordMessage example 2
this.app.postDiscordMessage({
    message: 'This is a message from a bot\'s plugin to a custom channel',
    channel: '966820843924466774',
    tts: false
});
</code></pre></p></li>
  <li><code>postDiscordNotification(title, description, url, img, list)</code> Post a notification to the bot's Notification Discord channel<ul><li><b>title</b>: <sup><code>string</code></sup> The title of the notification</li><li><b>description</b>: <sup><code>string</code></sup> The description of the notification</li><li><b>url</b>: <sup><code>string</code></sup> The url of the notification (optional)</li><li><b>img</b>: <sup><code>string</code></sup> The image url of the notification (optional)</li><li><b>list</b>: <sup><code>array</code></sup> The item list data of the notification (optional; see <code><a href="#NotificationList">NotificationList</a></code> below)</li></ul><p><pre><code>// postDiscordNotification example
var app = this.app;
app.postDiscordNotification('Plugin Alert Title', 'Plugin Alert Message');
</code></pre></p></li>
  <li><code>postDiscordWebhook(url, msg)</code> Post a message to a Discord webhook<ul><li><b>url</b>: <sup><code>string</code></sup> The url of the Discord webhook</li><li><b>msg</b>: <sup><code>string</code></sup> The message to post to the Discord webhook</li></ul><p><pre><code>// postDiscordWebhook example
var app = this.app;
app.postDiscordWebhook('webhook url', 'Webhook Message');
</code></pre></p></li>
  <li><code>runCommand(cmd, steamId, steamName)</code> Run a team chat command as a specific player<ul><li><b>cmd</b>: <sup><code>string</code></sup> The command to run</li><li><b>steamId</b>: <sup><code>string</code></sup> The player's steamId (optional)</li><li><b>steamName</b>: <sup><code>string</code></sup> The player's steam name (optional)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// runCommand example
var app = this.app,
    prefix = await app.getPrefix('all');
app.runCommand(prefix + 'pop');
</code></pre></p></li>
  <li><code>sendDiscordVoiceMessage(msg)</code> Send a voice message to the RustPlusBot voice client<ul><li><b>msg</b>: <sup><code>string</code></sup> The voice message to send</li></ul><p><pre><code>// sendDiscordVoiceMessage example
var app = this.app;
app.sendDiscordVoiceMessage('Hello, this is a test voice message!');
</code></pre></p></li>
  <li><code>sendTeamMessage(msg, callback, noTranslate, sendVoice)</code> Send a team chat message<ul><li><b>msg</b>: <sup><code>string</code></sup> The message to send</li><li><b>callback()</b>: <sup><code>function</code></sup> The function to execute after sending (optional)</li><li><b>noTranslate</b>: <sup><code>bool</code></sup> Set to <code>true</code> to disable message translation (optional)</li><li><b>sendVoice</b>: <sup><code>bool</code></sup> Set to <code>true</code> to send the message to the voice client (optional)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// sendTeamMessage example
var app = this.app;
app.sendTeamMessage('This is a team message');
</code></pre></p></li>
  <li><code>setEntityValue(id, value, callback)</code> Set the value of a Smart Switch<ul><li><b>id</b>: <sup><code>int</code></sup> The identifier of the Smart Switch</li><li><b>value</b>: <sup><code>bool</code></sup> The value (true or false)</li><li><b>callback()</b>: <sup><code>function</code></sup> The function to execute after setting the value (optional)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code> if successful</li></ul><p><pre><code>// setEntityValue example
var app = this.app;
app.setEntityValue(123456, true, () => {
    app.sendTeamMessage('The smart switch is activated');
});
</code></pre></p></li>
  <li><code>translateMessage(msg, lang, success, error)</code> Translate a message from English (default) to another language<ul><li><b>msg</b>: <sup><code>string</code></sup> The message to translate</li><li><b>lang</b>: <sup><code>string</code></sup> The language code to use for translation (see: <a href="https://bot.rustplus.io/languages">Language Codes</a>)</li><li><b>lang</b>: <sup><code>object</code></sup> The lang object containing the <code>from</code><sup><code>string</code></sup> and <code>to</code><sup><code>string</code></sup> language codes</li><li><b>success(res)</b>: <sup><code>function</code></sup> The function to execute after translating (optional)</li><li><b>error(err)</b>: <sup><code>function</code></sup> The function to execute when an error occurs (optional)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code></li></ul><p><pre><code>// translateMessage example 1
this.app.translateMessage('Hello, how are you?', 'es', (res) => {
    app.sendTeamMessage(res);
}, (error) => {
    app.sendTeamMessage('Error: ' + error);
});
// translateMessage example 2
this.app.translateMessage('Hola, como estas?', {
    from: 'es',
    to: 'en'
}, (res) => {
    app.sendTeamMessage(res);
}, (error) => {
    app.sendTeamMessage('Error: ' + error);
});
</code></pre></p></li>
  <li><code>webGet(url, params, headers, success, error)</code> Retrieve data from a url<ul><li><b>url</b>: <sup><code>string</code></sup> The url to access</li><li><b>params</b>: <sup><code>object</code></sup> The parameters of the url (optional)</li><li><b>headers</b>: <sup><code>object</code></sup> The headers to send with the web request (optional)</li><li><b>success(data)</b>: <sup><code>function</code></sup> The function to execute after receiving data (optional)</li><li><b>error(err)</b>: <sup><code>function</code></sup> The function to execute when an error occurs (optional)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code></li></ul><p><pre><code>// webGet example
var app = this.app;
app.webGet('https://rust.facepunch.com/rss/news', null, null, (data) => {
    var link = '',
        pos = data.indexOf('>https://');
    if (pos > 0) link = data.substr(pos + 1, data.indexOf('&lt;', pos) - pos - 1);
    if (link != '')
        app.sendTeamMessage('The newest Rust update link: ' + link);
    else
        app.sendTeamMessage('The newest Rust update link was not found');
}, (error) => {
    app.sendTeamMessage('Error obtaining Rust update link: ' + error);
});
</code></pre></p></li>
  <li><code>webPost(url, data, headers, success, error)</code> Post data to a url<ul><li><b>url</b>: <sup><code>string</code></sup> The url to access</li><li><b>data</b>: <sup><code>string</code></sup> The data to post (optional)</li><li><b>headers</b>: <sup><code>object</code></sup> The headers to send with the web request (optional)</li><li><b>success(data)</b>: <sup><code>function</code></sup> The function to execute after receiving data (optional)</li><li><b>error(err)</b>: <sup><code>function</code></sup> The function to execute when an error occurs (optional)</li><li><b>returns</b>: <sup><code>bool</code></sup> <code>true</code></li></ul><p><pre><code>// webPost example
var app = this.app;
app.webPost('https://httpbin.org/post', 'test data', null, (data) => {
    app.sendTeamMessage('Post result: ' + data);
}, (error) => {
    app.sendTeamMessage('Error posting: ' + error);
});
</code></pre></p></li>
  <li><code>interactiveMap.addMarker(markerId<sup><code>int</code></sup>, steamId<sup><code>string</code></sup>, name<sup><code>string</code></sup>, msg<sup><code>string</code></sup>, x<sup><code>int</code></sup>, y<sup><code>int</code></sup>)</code> Returns <code>true</code> if the custom map marker is added<blockquote>Refer to the <a href="https://github.com/javajuice1337/RustPlusBot/blob/main/plugin%20examples/mapmarker.md">mapmarkers</a> plugin example to see the correct usage for <i>addMarker</i></blockquote></li>
  <li><code>interactiveMap.setMarkerColor(markerId<sup><code>int</code></sup>, color<sup><code>string</code></sup>)</code> Returns <code>true</code> if the custom map marker's color is updated</li>
  <li><code>interactiveMap.removeMarker(markerId<sup><code>int</code></sup>)</code> Returns <code>true</code> if the custom map marker matching markerId is removed</li>
  <li><code>interactiveMap.clearMarkers(steamId<sup><code>string</code></sup>)</code> Returns <code>true</code> if all custom map markers are removed for steamId</li>
  <li><code>interactiveMap.getHeatmapData()</code> Returns an object containing the heatmap data (see <code><a href="#HeatmapData">HeatmapData</a></code> below)</li>
  <li><code>util.collides(x<sup><code>int</code></sup>, y<sup><code>int</code></sup>, rotation<sup><code>int</code></sup>, x1<sup><code>int</code></sup>, y1<sup><code>int</code></sup>, x2<sup><code>int</code></sup>, y2<sup><code>int</code></sup>)</code> Returns <code>true</code> if angled point x,y collides with rectangle x1,y1,x2,y2</li>
  <li><code>util.direction(x1<sup><code>int</code></sup>, y1<sup><code>int</code></sup>, x2<sup><code>int</code></sup>, y2<sup><code>int</code></sup>)</code> Get the direction from the first point facing the second</li>
  <li><code>util.distance(x1<sup><code>int</code></sup>, y1<sup><code>int</code></sup>, x2<sup><code>int</code></sup>, y2<sup><code>int</code></sup>)</code> Get the distance between two points in meters</li>
  <li><code>util.getMapCoords(x<sup><code>int</code></sup>, y<sup><code>int</code></sup>)</code> Get the map coordinates for a point</li>
  <li><code>util.inRect(x<sup><code>int</code></sup>, y<sup><code>int</code></sup>, x1<sup><code>int</code></sup>, y1<sup><code>int</code></sup>, x2<sup><code>int</code></sup>, y2<sup><code>int</code></sup>)</code> Returns <code>true</code> if the point is inside the rectangle</li>
</ul>

```
// get the map coordinates of a specific location
var location = {x: 1000, y: 2000};
console.log('map coordinates: ' + (await this.app.util.getMapCoords(location.x, location.y)));
```

### Other Methods

Note: The following methods exist in the plugin's scope `this` (instead of in `app`).

<ul>
  <li><code>registeredHandlers.add(type<sup><code>string</code></sup>, handler<sup><code>function</code></sup>)</code> Add a handler for a specific update event type: <ul><li><code>camera</code> Fires when the camera list has changed</li><li><code>config</code> Fires when the configuration settings have changed</li><li><code>device</code> Fires when the paired devices has changed</li><li><code>wipe</code> Fires when the server has wiped</li></ul></li>
  <li><code>registeredHandlers.remove(type<sup><code>string</code></sup>, handler<sup><code>function</code></sup>)</code> Remove a handler for a specific update event type (see <code>registeredHandlers.add</code> above)</li>
</ul>

```
// listen for a configuration change event
if (!this.configFunc) {
    var self = this;
    self.configFunc = function() {
        console.log(self.app.cfg);
    };
}
this.registeredHandlers.add('config', this.configFunc);
```

### Data Types

<ul>
  <li>
    <b>BattleMetrics Data</b><a name="BattleMetricsData"></a>
    <pre><code>{     
  "Name": "Server Name",
  "Rank": "#1",
  "Player count": "1/100",
  "Address": "Server Address",
  "Status": "online",
  "Country": "US",
  "Uptime": "1 day 12 hours",
  "Average FPS": "60",
  "Last Wipe": "12/16/2021 - 13 days ago",
  "PVE": "False",
  "Website": "https://",
  "Entity Count": "265,097",
  "Official Server": "True",
  "id": 546784,
  "time": 1640108040219
}</code></pre>
  </li>
  <li>
    <b>Config</b><a name="Config"></a>
    <pre><code>{     
  "lang": "en",
  "gender": "male",
  "cmdPrefix": "!",
  "requirePrefix": "all",
  "teamChatIncoming": "all",
  "teamChatOutgoing": true,
  "teamChatResponses": false,
  "teamChatSilenced": false,
  "teamChatDelay": 0,
  "teamChatTTS": false,
  "teamChatMentions": true,
  "shortTime": false,
  "nextTime": false,
  "altTime": false,
  "dirAngle": true,
  "eventsDiscord": false,
  "broadcastEvents": true,
  "vendingDiscord": false,
  "broadcastVending": true,
  "broadcastVendingName": true,
  "broadcastAmount": 1,
  "respawnAmount": 1,
  "battlemetricsID": 0,
  "battlemetricsDiscord": true,
  "deathDiscord": true,
  "loginDiscord": true,
  "aliasesFullMatch": false,
  "accessOnlyCommands": false,
  "deviceTTS": false,
  "autoCleanDevices": false,
  "autoDeviceCommand": true,
  "showDevicePlayer": false,
  "alwaysPostAlarms": true,
  "alwaysPostDecay": true,
  "decayOffset": 0,
  "streamerMode": false,
  "promoteStrict": false,
  "voiceDevice": true,
  "voiceEvent": true,
  "voiceVending": true,
  "voiceTracking": true,
  "voicePlugin": true,
  "eventsDisplay": "heli,brad,cargo,oil,crate,ch47",
  "subEventsDisplay": "heli_lootable,brad_lootable,cargo_crates,oil_lootable"
}</code></pre>
  </li>
  <li>
    <b>DetailedInfo</b><a name="DetailedInfo"></a>
    <pre><code>{
  "players": "51/200",
  "playersQueue": 0,
  "playersChange": 0,
  "time": "16:23",
  "timeValue": 16.38,
  "wipe": 1147412,
  "nextDay": 1453,
  "nextNight": 775,
  "nextWipe": 22
}</code></pre>
    <pre><code>// wipe usage: getTimeDisplay(wipe * 1000) + ' ago'
// nextDay usage: getTimeDisplay(nextDay)
// nextNight usage: getTimeDisplay(nextNight)
// nextWipe usage: nextWipe + 'd'</code></pre>
  </li>
  <li>
    <b>Device</b><a name="Device"></a>
    <pre><code>{
  name: "MyDevice",
  id: 123456,
  flag: false, // true if op is inverted
  type: "Smart Switch",
  time: 0 // timestamp of last state change
}</code></pre>
  </li>
  <li>
    <b>DeviceAuto</b><a name="DeviceAuto"></a>
    <pre><code>{
  state: true, // true for ON
  time: 60 // auto time in seconds
}</code></pre>
  </li>
  <li>
    <b>EntityInfo</b><a name="EntityInfo"></a>
    <pre><code>{
  "type": "1",
  "payload": {} // see Payload below
}</code></pre>
    <pre><code>// entity types:
// 1 = Smart Switch
// 2 = Smart Alarm
// 3 = Storage Monitor</code></pre>
  </li>
  <li>
    <b>Event</b><a name="Event"></a>
    <pre><code>{
  id: 123456789,
  type: "heli",
  name: "Patrol Helicopter @ A1",
  start: new Date(),
  stop: null // null if active
}</code></pre>
  </li>
  <li>
    <b>Event Timers</b><a name="EventTimers"></a>
    <pre><code>{
  "cargo": {
    "spawn": 14400,
    "spread": 7200
  },
  "crate": {
    "spawn": 7200,
    "spread": 3600
  },
  "heli": {
    "spawn": 7200,
    "spread": 3600
  },
  "oilrig": {
    "spawn": 2700,
    "spread": 2700
  }
}</code></pre>
  </li>
  <li>
    <b>Heatmap Data</b><a name="HeatmapData"></a>
    <pre><code>{
  "deaths": [],
  "brad": [{
    "x": 2667.51416015625,
    "y": 2336.646240234375,
    "count": 1
  }],
  "heli": [{
    "x": 688.2880859375,
    "y": 922.4451904296875,
    "count": 1
  }],
  "crate": [{
    "x": 2047.0020751953125,
    "y": 2884.339111328125,
    "count": 6
  }],
  "cargo": [{
    "x": -1775,
    "y": 2098.646240234375,
    "count": 8
  },{
    "x": -1684.09814453125,
    "y": 2145.852783203125,
    "count": 2
  },{
    "x": -1608.643798828125,
    "y": 2141.641357421875,
    "count": 2
  }]
}</code></pre>
  </li>
  <li>
    <b>Info</b><a name="Info"></a>
    <pre><code>{
  "name": "Rust Server Name",
  "headerImage": "",
  "url": "",
  "map": "Procedure Map",
  "mapSize": 4250,
  "wipeTime": 0,
  "players": 100,
  "maxPlayers": 200,
  "queuedPlayers": 0,
  "seed": 0,
  "salt": 0
}</code></pre>
  </li>
  <li>
    <b>MapInfo</b><a name="MapInfo"></a>
    <pre><code>{
  "width": 3125,
  "height": 3125,
  "oceanMargin": 500,
  "offset": 0,
  "background": "#12404D",
  "image": "",
  "cached": true,
  "info": {
    "name": "Rust Server Name",
    "headerImage": "",
    "url": "",
    "map": "Procedure Map",
    "mapSize": 4250,
    "wipeTime": 0,
    "players": 100,
    "maxPlayers": 200,
    "queuedPlayers": 0,
    "seed": 0,
    "salt": 0
  }
}</code></pre>
  </li>
  <li>
    <b>MapMarkers</b><a name="MapMarkers"></a>
    <pre><code>{
  markers: [{
    "id": 123456,
    "type": 3,
    "x": 1500.958740234375,
    "y": 2551.239990234375,
    "location": "G20",
    "steamId": 0,
    "rotation": 0,
    "radius": 0,
    "name": "",
    "outOfStock": false,
    sellOrders: [{ // sellOrders when type is 3
      "itemId": 123456,
      "quantity": 1,
      "currencyId": 654321,
      "costPerItem": 1,
      "amountInStock": 10,
      "itemIsBlueprint": false,
      "currencyIsBlueprint": false,
      "itemCondition": 42.75,
      "itemConditionMax": 100
    }]
  }]
}</code></pre>
    <pre><code>// marker types:
// 1 = Player
// 2 = Explosion
// 3 = VendingMachine
// 4 = CH47
// 5 = CargoShip
// 6 = Crate
// 8 = PatrolHelicopter</code></pre>
    <blockquote>Find the item name with the itemId using <code>itemIds</code></blockquote>
  </li>
  <li>
    <b>MapNotes</b><a name="MapNotes"></a>
    <pre><code>[{
  "type": 1,
  "x": 1500.958740234375,
  "y": 2551.239990234375
}]</code></pre>
  </li>
  <li>
    <b>Members</b><a name="Members"></a>
    <pre><code>[{
  "steamId": "123456789",
  "name": "RustPlayer1",
  "x": 1497.4344482421875,
  "y": 2522.85546875,
  "isOnline": true,
  "spawnTime": 1638768666,
  "isAlive": true,
  "deathTime": 1638768647
}, {
  "steamId": "123456799",
  "name": "RustPlayer2",
  "x": 1487.4344482421875,
  "y": 2512.85546875,
  "isOnline": true,
  "spawnTime": 1638768866,
  "isAlive": true,
  "deathTime": 1638768847
}]</code></pre>
  </li>
  <li>
    <b>Notification: Alarm</b><a name="NotificationAlarm"></a>
    <pre><code>{
  "notification: {
    "type": "alarm",
    "alarm": {
      "title": "Smart Alarm Title",
      "message": "Smart Alarm Message"
    },
    "server_name": "Rust Server Name"
  }
}</code></pre>
  </li>
  <li>
    <b>Notification: Decaying TC</b><a name="NotificationDecayingTC"></a>
    <pre><code>{
  "notification": {
    "type": "monitor",
    "monitor": {
      "title": "MyDevice",
      "message": "This monitored TC is decaying!",
      "device_name": "MyDevice",
      "device_id": "123456",
      "device_flag": false,
      "device_type": "Storage Monitor"
    },
    "server_name": "Rust Server Name"
  }
}</code></pre>
  </li>
  <li>
    <b>Notification: Entity Pairing</b><a name="NotificationEntityPairing"></a>
    <pre><code>{
  "notification": {
    "type": "entity",
    "server_name": "Rust Server Name",
    "entityName": "Smart Switch",
    "entityId": "123456",
    "entityType": "1"
  }
}</code></pre>
    <pre><code>// entity types:
// 1 = Smart Switch
// 2 = Smart Alarm
// 3 = Storage Monitor</code></pre>
  </li>
  <li>
    <b>Notification: Event</b><a name="NotificationEvent"></a>
    <pre><code>{
  "notification": {
    "type": "event",
    "event": {
      "type": "heli",
      "title": "Patrol Helicopter",
      "message": "The Patrol Helicopter has exploded @ A1",
      "x": 100.543565665,
      "y": 200.345765755
    },
    "server_name": "Rust Server Name"
  }
}</code></pre>
  <li>
    <b>Notification: Inactive Device</b><a name="NotificationInactiveDevice"></a>
    <pre><code>{
  "notification": {
    "type": "inactive",
    "inactive": {
      "title": "MyDevice",
      "message": "This device is inactive.",
      "device_name": "MyDevice",
      "device_id": "123456",
      "device_flag": false,
      "device_type": "Smart Switch"
    },
    "server_name": "Rust Server Name"
  }
}</code></pre>
  </li>
  <li>
    <b>Notification: News</b><a name="NotificationNews"></a>
    <pre><code>{
  "notification: {
    "type": "news",
    "news": {
      "title": "Rust Update News",
      "url": "https://rust.facepunch.com/",
      "message": "A new Rust update blog is available!"
    },
    "server_name": "Rust Server Name"
  }
}</code></pre>
  </li>
  <li>
    <b>Notification: Player Death</b><a name="NotificationPlayerDeath"></a>
    <pre><code>{
  "notification": {
    "type": "death",
    "server_name": "Rust Server Name",
    "targetName": "RustPlayer",
    "targetId": "123456789"
  }
}</code></pre>
  </li>
  <li>
    <b>Notification: Player Tracking</b><a name="NotificationPlayerTracking"></a>
    <pre><code>{
  "notification": {
    "type": "tracking",
    "tracking": {
      "id": "12345",
      "name": "Tracked player \"Rust Player\" has joined the server",
      "status": "joined"
    },
    "server_name": "Rust Server Name"
  }
}</code></pre>
  </li>
  <li>
    <b>Notification: Team Login</b><a name="NotificationTeamLogin"></a>
    <pre><code>{
  "notification": {
    "type": "login",
    "server_name": "Rust Server Name",
    "targetName": "RustPlayer",
    "targetId": "123456789"
  }
}</code></pre>
  </li>
  <li>
    <b>NotificationList</b><a name="NotificationList"></a>
    <pre><code>[{
  name: "Sub Item",
  value: "Sub Value",
  inline: false
}]</code></pre>
  </li>
  <li>
    <b>Payload: Smart Switch, Smart Alarm</b><a name="Payload"></a>
    <pre><code>"payload": {
  "value": false,
  "capacity": 0,
  "hasProtection": false,
  "protectionExpiry": 0
}
</code></pre>
  </li>
  <li>
    <b>Payload: Storage Monitor</b><a name="PayloadStorageMonitor"></a>
    <pre><code>"payload": {
  "value": false,
  "items": [{
    "itemId": 317398316,
    "quantity": 95,
    "itemIsBlueprint": false
  }, {
    "itemId": -151838493,
    "quantity": 998,
    "itemIsBlueprint": false
  }],
  "capacity": 24,
  "hasProtection": true,
  "protectionExpiry": 1638790206
}
</code></pre>
    <blockquote>Find the item name with the itemId using <code>itemIds</code></blockquote>
  </li>
  <li>
    <b>Point</b><a name="Point"></a>
    <pre><code>{
  x: 100,
  y: 200
}</code></pre>
  </li>
  <li>
    <b>Time</b><a name="Time"></a>
    <pre><code>{
  "dayLengthMinutes": 60,
  "timeScale": 1,
  "sunrise": 7,
  "sunset": 20,
  "time": 12
}</code></pre>
  </li>
  <li>
    <b>TeamChat</b><a name="TeamChat"></a>
    <pre><code>{
  "messages": [{
    "steamId": "123456789",
    "name": "RustPlayer1",
    "message": "A Locked Crate has been dropped @ M13 (Airfield)",
    "color": "#5af",
    "time": 1649959932
  }]
}</code></pre>
  </li>
  <li>
    <b>TeamData</b><a name="TeamData"></a>
    <pre><code>{
  "leader": {
    "name": "RustPlayer1",
    "steamId": "123456789"
  },
  "members": [{
    "name": "RustPlayer1",
    "steamId": "123456789",
    "status": "OFFLINE",
    "info": "last seen 5 hours ago",
    "infoText": "last seen",
    "infoValue": 1649959932,
    "location": "G20",
    "x": 1932.2833251953125,
    "y": 2376.96240234375
  }]
}</code></pre>
  </li>
  <li>
    <b>TeamInfo</b><a name="TeamInfo"></a>
    <pre><code>{
  "leaderSteamId": 123456789,
  "members": [], // see Members
  "leaderMapNotes": [], // see MapNotes
  "mapNotes": [] // see MapNotes
}</code></pre>
  </li>
  <li>
    <b>Time</b><a name="Time"></a>
    <pre><code>{
  "dayLengthMinutes": 60,
  "timeScale": 1,
  "sunrise": 7,
  "sunset": 20,
  "time": 12
}</code></pre>
  </li>
</ul>

## Plugin Globals

<ul>
  <li><code>cmdFormat(str)</code> Convert a string into a non-translatable string<ul><li><b>str</b>: <sup><code>string</code></sup> The string to convert</li><li><b>returns</b>: <sup><code>string</code></sup> a non-translatable string</li></ul><p><pre><code>// cmdFormat example
var app = this.app,
    msg = 'Here is the url: ' + cmdFormat('https://www.google.com');
app.sendTeamMessage(msg);
</code></pre></p></li>
  <li><code>cmdFormatUndo(str)</code> Undo the non-translatable string conversion<ul><li><b>str</b>: <sup><code>string</code></sup> The string to undo the non-translatable string conversion</li><li><b>returns</b>: <sup><code>string</code></sup> a string</li></ul><p><pre><code>// cmdFormatUndo example
var app = this.app,
    msg = 'Here is the url: ' + cmdFormat('https://www.google.com');
console.log(cmdFormatUndo(msg));
</code></pre></p></li>
  <li><code>encodeForm(data)</code> Convert an object to form data for a webPost<ul><li><b>data</b>: <sup><code>object</code></sup> The object to convert</li><li><b>returns</b>: <sup><code>string</code></sup> a string of encoded names and values</li></ul><p><pre><code>// encodeForm example
var app = this.app;
app.webPost('https://www.iplocation.net/ip-lookup', encodeForm({ query: app.server_ip }), null, (data) => {
    var regex = new RegExp('&lt;tr&gt;&lt;td&gt;' + app.server_ip + '&lt;/td&gt;&lt;td&gt;([^<]+)<', 'g'),
        match = regex.exec(data);
    if (match)
        app.sendTeamMessage('GeoIP location of the server: ' + match[1]);
    else
        app.sendTeamMessage('Unable to get the GeoIP location of the server');
}, (error) => {
    app.sendTeamMessage('Error posting: ' + error);
});
</code></pre></p></li>
  <li><code>combineItems(items, itemIds)</code> Combine the items from a Storage Monitor payload and resolve the item names<ul><li><b>items</b>: <sup><code>object</code></sup> The items from the payload</li><li><b>itemIds</b>: <sup><code>Map</code></sup> The item ID list to lookup item names</li><li><b>returns</b>: <sup><code>Map</code></sup> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the combined items and item quantities</li></ul><p><pre><code>// combineItems example
var app = this.app,
    device = app.devices.get('BaseTC')[0];
app.getEntityInfo(device.id, (message) => {
    if (message.response && message.response.entityInfo && message.response.entityInfo.payload && message.response.entityInfo.payload.capacity > 0 && message.response.entityInfo.payload.items) {
        var items = combineItems(message.response.entityInfo.payload.items, app.itemIds),
            sorted = [],
            list = [];
        items.forEach((value, key) => {
            sorted.push({ name: key, quantity: value });
        });
        sorted.sort(function (a, b) {
            if (a.name < b.name) return -1;
            if (a.name > b.name) return 1;
        });
        sorted.forEach((item) => {
            list.push(item.name + ' x ' + item.quantity.toLocaleString());
        });
        if (list.length > 0) {
            multiLineFormat('Device \'' + cmdFormat(device.name) + '\' contents: ', list, function(line) {
                app.sendTeamMessage(line);
            });
        }
    }
});
</code></pre></p></li>
  <li><code>findClosestString(str, arr, threshold = 2)</code> Find the closest match for a string in an array using a threshold<ul><li><b>str</b>: <sup><code>string</code></sup> The input string</li><li><b>arr</b>: <sup><code>array</code></sup> The array of possible string matches</li><li><b>threshold</b>: <sup><code>int</code></sup> The threshold used for string matching (optional)</li><li><b>returns</b>: <sup><code>string</code></sup> The closest matched string found in the input array</li></ul><p><pre><code>// findClosestString example
var app = this.app,
    item = 'windturbine',
    closest = findClosestString(item, Array.from(app.itemIds.values()));
if (closest && closest.toLowerCase() != item.toLowerCase()) {
    app.sendTeamMessage('Are you looking for this item? ' + closest);
}
else {
    app.sendTeamMessage('This item name is a direct match: ' + item);
}
</code></pre></p></li>
  <li><code>getFriendlyDate(date)</code> Get a friendly representation of the date<ul><li><b>date</b>: <sup><code>date</code></sup> The date object</li><li><b>returns</b>: <sup><code>string</code></sup> a string containing the friendly representation of the date</li></ul><p><pre><code>// getFriendlyDate example
var app = this.app;
app.getTeamInfo((message) => {
    if (message.response && message.response.teamInfo) {
        var status = '',
            status_time = 0;
        for (var i = 0; i < message.response.teamInfo.members.length; i++) {
            if (message.response.teamInfo.members[i].steamId == message.response.teamInfo.leaderSteamId) {
                status = (message.response.teamInfo.members[i].isAlive) ? 'alive' : 'dead';
                status_time = (message.response.teamInfo.members[i].isAlive) ? message.response.teamInfo.members[i].spawnTime : message.response.teamInfo.members[i].deathTime;
                break;
            }
        }
        app.sendTeamMessage('The team leader has been ' + status + ' since ' + getFriendlyDate(status_time * 1000));
    }
});
</code></pre></p></li>
  <li><code>getTime(timestr, max_val)</code> Convert a time string to seconds<ul><li><b>timestr</b>: <sup><code>string</code></sup> The time string (format: 1d1h1m1s)</li><li><b>max_val</b>: <sup><code>int</code></sup> The maximum allowed time value to return (optional)</li><li><b>returns</b>: <sup><code>int</code></sup> the total seconds of the timestr</li></ul><p><pre><code>// getTime example
var app = this.app;
app.sendTeamMessage('The time in seconds for 1d1h1m1s is ' + getTime('1d1h1m1s'));
</code></pre></p></li>
  <li><code>getTimeDifference(date)</code> Get the time difference for a date<ul><li><b>date</b>: <sup><code>date</code></sup> The date object</li><li><b>returns</b>: <sup><code>int</code></sup> the date difference in seconds</li></ul><p><pre><code>// getTimeDifference example
var app = this.app,
    connected = await app.getConnected();
app.sendTeamMessage('The time in seconds since the bot connected is ' + Math.round(getTimeDifference(new Date(connected))));
</code></pre></p></li>
  <li><code>getTimeDisplay(time)</code> Get the time display for time<ul><li><b>time</b>: <sup><code>int</code></sup> The time in seconds</li><li><b>returns</b>: <sup><code>string</code></sup> a string representing the time</li></ul><p><pre><code>// getTimeDisplay example
var app = this.app,
    connected = await app.getConnected();
app.sendTeamMessage('The bot has been connected for ' + getTimeDisplay(Math.round(getTimeDifference(new Date(connected)))));
</code></pre></p></li>
  <li><code>multiLineFormat(msg, list, callback, all)</code> Format the message + list to fit the Rust message size using multiple lines<ul><li><b>msg</b>: <sup><code>string</code></sup> The message to prepend</li><li><b>list</b>: <sup><code>array</code></sup> The list of items to output</li><li><b>callback(line, msg, data, idx)</b>: <sup><code>function</code></sup> The function to execute for each formatted line (optional)</li><li><b>all</b>: <sup><code>bool</code></sup> Set to true if all lines should include the msg (optional)</li><li><b>returns</b>: <sup><code>array</code></sup> an Array containing the formatted lines</li></ul><p><pre><code>// multiLineFormat example
var app = this.app,
    device = app.devices.get('BaseTC')[0];
app.getEntityInfo(device.id, (message) => {
    if (message.response && message.response.entityInfo && message.response.entityInfo.payload && message.response.entityInfo.payload.capacity > 0 && message.response.entityInfo.payload.items) {
        var items = combineItems(message.response.entityInfo.payload.items, app.itemIds),
            sorted = [],
            list = [];
        items.forEach((value, key) => {
            sorted.push({ name: key, quantity: value });
        });
        sorted.sort(function (a, b) {
            if (a.name < b.name) return -1;
            if (a.name > b.name) return 1;
        });
        sorted.forEach((item) => {
            list.push(item.name + ' x ' + item.quantity.toLocaleString());
        });
        if (list.length > 0) {
            multiLineFormat('Device \'' + cmdFormat(device.name) + '\' contents: ', list, function(line, msg, data, idx) {
                if (idx == 0)
                    app.sendTeamMessage(msg + data);
                else
                    app.sendTeamMessage(data);
            });
        }
    }
});
</code></pre></p></li>
</ul>

## Plugin Publishing
          
You can publish your plugin when you are done making any major changes to it by clicking the Publish tab in the Plugin Studio. You have the option of making it a public plugin for others to use with their bot. There will be a review of your plugin after submitting which could take several days. If your plugin is accepted for publishing, you will then be able to select it in the Plugin settings for your bot and install it. Duplicates of official plugins will not be accepted.

## Links

<ul>
    <li>Website: https://bot.rustplus.io/</li>
    <li>Help & Documentation: https://bot.rustplus.io/help/</li>
</ul>
