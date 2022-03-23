<p align="center">
<a href="https://bot.rustplus.io/"><img src="./rustplusbot.png" width="400"></a>
</p>

# **RustPlusBot** plugins reference and examples

**RustPlusBot** plugins allow the individual to develop their own commands to add functionality to the bot. The plugin itself exposes many events for a programmer to attach to and execute code.

The plugins are written in JavaScript and run in a NodeJS environment after they are published. During development, you host the plugin on your client machine and in your web-browser. Your plugin interfaces with the bot via a WebSocket connection.

You can load any of the official plugins and use them as a template for getting started in the Plugin Studio. The Plugin Studio can be accessed via a link in the Plugin settings tab on the RustPlusBot settings page for your Discord server.

> Plugins are loaded when the bot is starting and lasts for its entire life-cycle. Restarting the bot also restarts all plugins.

## Plugin Storage

For data that persists beyond the bot's instance, use `this.storage`. This object loads with the bot and saves when it shuts down or restarts.

```
// store the playerName variable
if (!this.storage.playerName) this.storage.playerName = 'Player';
console.log(this.storage.playerName);
```

## Plugin Events

<ul>
  <li><code>onConnected()</code> Fires when the bot connects to a server or when the plugin loads</li>
  <li><code>onDisconnected()</code> Fires when the bot disconnects from a server</li>
  <li><code>onEntityChanged(obj)</code> Fires when a paired Smart Device is changed<ul><li><b>obj.entityId</b>: The entity ID of the Smart device</li><li><b>obj.payload</b>: The payload data of the event (see <code>Payload</code> below)</li></ul></li>
  <li><code>onMessageReceive(obj)</code> Fires when a team chat message is received<ul><li><b>obj.message</b>: The incoming team chat message</li><li><b>obj.name</b>: The steam name of the sender</li><li><b>obj.steamId</b>: The steam ID of the sender</li></ul></li>
  <li><code>onMessageSend(obj)</code> Fires when a team chat message is sent<ul><li><b>obj.message</b>: The outgoing team chat message</li></ul></li>
  <li><code>onNotification(obj)</code> Fires when there is a bot notification (including game events)<ul><li><b>obj.notification</b>: The notification data of the event (see <code>Notification</code> below)</li></ul></li>
  <li><code>onTeamChanged(obj)</code> Fires when the team leader changes, or a team member is added or removed from the team<ul><li><b>obj.leaderSteamId</b>: The steam ID of the team leader</li><li><b>obj.leaderMapNotes</b>: The leader map notes data of the event (see <code>LeaderMapNotes</code> below)</li><li><b>obj.members</b>: The members list data of the event (see <code>Members</code> below)</li></ul></li>
</ul>

## Plugin Interface

The `app` object exists in the plugin's scope `this`, and exposes the following properties and methods:

### Properties

<ul>
  <li><code>bmData</code> An object containing the BattleMetrics data of the server (see <code>BattleMetrics Data</code> below)</li>
  <li><code>cfg</code> An object containing the configuration settings for the bot (see <code>Config</code> below)</li>
  <li><code>devices</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the bot's paired devices (<b>key</b>: device name (lowercase only), <b>value</b>: Array of devices, see <code>Device</code> below)</li>
  <li><code>devices_map</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the bot's paired device names (<b>key</b>: device ID, <b>value</b>: Array of lowercased device names)</li>
  <li><code>event_types</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing game event names (<b>key</b>: event ID, <b>value</b>: event name)</li>
  <li><code>guild_token</code> The unique token representing the Discord server</li>
  <li><code>guild_name</code> The name of the Discord server</li>
  <li><code>itemIds</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the item names for all item IDs (<b>key</b>: item ID, <b>value</b>: item name)</li>
  <li><code>monuments</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing all monument tokens  and locations (<b>key</b>: monument token, <b>value</b>: monument location, see <code>Point</code> below)</li>
  <li><code>tokenMap</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the monument names for all monument tokens (<b>key</b>: monument token, <b>value</b>: monument name)</li>
  <li><code>player_id</code> The steam ID of the bot's connected player</li>
  <li><code>player_name</code> The steam name of the bot's connected player</li>
  <li><code>player_token</code> The server token of the bot's connected player</li>
  <li><code>server_ip</code> The IP address of the bot's connected server</li>
  <li><code>server_name</code> The name of the bot's connected server</li>
  <li><code>server_port</code> The port of the bot's connected server (Rust+ app port)</li>
</ul>

```
// get the bot's language setting
console.log(this.app.cfg.lang);
```

### Methods

<ul>
  <li><code>getBattleMetrics(serverId, playerName, success, error)</code> Retrieve the BattleMetrics for a server player<ul><li><b>serverId</b>: The BattleMetrics ID of the server</li><li><b>playerName</b>: The name of the player</li><li><b>success(data)</b>: The function to execute after receiving the BattleMetrics data (optional)</li><li><b>error(err)</b>: The function to execute when an error occurs (optional)</li><li><b>returns</b>: <code>true</code></li></ul><p><pre><code>// getBattleMetrics example
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
  <li><code>getConnected()</code> Get the date of the server connection<ul><li><b>returns</b>: a Date string</li></ul><p><pre><code>// getConnected example
var connected = await this.app.getConnected();
console.log(new Date(connected));
</code></pre></p></li>
  <li><code>getEvents(type)</code> Get the most recent game events (ordered by newest to oldest)<ul><li><b>type</b>: The event type (optional)<ul><li><code>heli</code> Patrol Helicopter</li><li><code>brad</code> Bradley Tank</li><li><code>cargo</code> Cargo Ship</li><li><code>crate</code> Locked Crate</li><li><code>ch47</code> CH-47 Chinook</li><li><code>oil_rig_small</code> Oil Rig (Small)</li><li><code>large_oil_rig</code> Oil Rig (Large)</li></ul></li><li><b>returns</b>: <code>Event</code> array</li></ul><p><pre><code>// getEvents example
var e = await this.app.getEvents();
console.log(e);
</code></pre></p></li>
  <li><code>getEntityInfo(id, callback)</code> Get data from a Smart device<ul><li><b>id</b>: The identifier of the Smart device</li><li><b>callback(message)</b>: The function to execute after getting the entity info (<code>message.response</code> contains <code>EntityInfo</code>)</li><li><b>returns</b>: <code>true</code> if successful</li></ul><p><pre><code>// getEntityInfo example
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
  <li><code>getInfo(callback)</code> Get information about the server<ul><li><b>callback(message)</b>: The function to execute after getting the info (<code>message.response</code> contains <code>Info</code>)</li><li><b>returns</b>: <code>true</code> if successful</li></ul><p><pre><code>// getInfo example
var app = this.app;
app.getInfo((message) => {
    if (message.response && message.response.info) {
        app.sendTeamMessage('Current server population is ' + message.response.info.players + ' / ' + message.response.info.maxPlayers + ((message.response.info.queuedPlayers > 0) ? ' (' + message.response.info.queuedPlayers + ' in queue)' : ''));
    }
});
</code></pre></p></li>
  <li><code>getItemName(item, success, error)</code> Lookup the full item name for a partial item name search<ul><li><b>item</b>: The item name to search for</li><li><b>success(data)</b>: The function to execute after receiving ItemName data (optional)</li><li><b>error(err)</b>: The function to execute when an error occurs (optional)</li><li><b>returns</b>: <code>true</code></li></ul><p><pre><code>// getItemName example
var app = this.app;
app.getItemName("metal", (data) => {
    if (data) {
        app.sendTeamMessage('Item \'metal\' matches: ' + data);
    }
    else app.sendTeamMessage('Unable to lookup ItemName data');
}, (error) => {
    app.sendTeamMessage('Error obtaining the ItemName data: ' + error);
});
</code></pre></p></li>
  <li><code>getMapMarkers(callback)</code> Get information about all map markers<ul><li><b>callback(message)</b>: The function to execute after getting the map markers (<code>message.response</code> contains <code>MapMarkers</code>)</li><li><b>returns</b>: <code>true</code> if successful</li></ul><p><pre><code>// getMapMarkers example
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
  <li><code>getPrefix(type)</code> Get the command prefix for the bot<ul><li><b>type</b>: The command type (optional)<ul><li><code>all</code> All Commands</li><li><code>device</code> Device Commands</li></ul></li><li><b>returns</b>: The selected prefix if it's required</li></ul><p><pre><code>// getPrefix example
var prefix = await this.app.getPrefix('all');
</code></pre></p></li>
  <li><code>getRecycleItems(items, success, error)</code> Retrieve the recycled items for the input items<ul><li><b>items</b>: An object containing the item names and item values</li><li><b>success(data)</b>: The function to execute after receiving recycle data (optional)</li><li><b>error(err)</b>: The function to execute when an error occurs (optional)</li><li><b>returns</b>: <code>true</code></li></ul><p><pre><code>// getRecycleItems example
var app = this.app;
app.getRecycleItems({'Sheet Metal Door': 1}, (data) => {
    var keys = Object.keys(data),
        recycle = [];
    for (var i = 0; i < keys.length; i++) {
        recycle.push(keys[i] + ' x ' + data[keys[i]].toLocaleString());
    }
    if (recycle.length > 0) app.sendTeamMessage('Recyclables: ' + recycle.join(', '));
}, (error) => {
    app.sendTeamMessage('Error obtaining the recyle items: ' + error);
});
</code></pre></p></li>
  <li><code>getSteamrep(steamId, success, error)</code> Retrieve the Steamrep data for a Steam member<ul><li><b>steamId</b>: The steam ID of the Steam member</li><li><b>success(data)</b>: The function to execute after receiving Steamrep data (optional)</li><li><b>error(err)</b>: The function to execute when an error occurs (optional)</li><li><b>returns</b>: <code>true</code></li></ul><p><pre><code>// getSteamrep example
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
  <li><code>getTeamInfo(callback)</code> Get information about the team (leader, members)<ul><li><b>callback(message)</b>: The function to execute after getting the team info (<code>message.response</code> contains <code>TeamInfo</code>)</li><li><b>returns</b>: <code>true</code> if successful</li></ul><p><pre><code>// getTeamInfo example
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
  <li><code>getTime(callback)</code> Get information about the server time<ul><li><b>callback(message)</b>: The function to execute after getting the time (<code>message.response</code> contains <code>Time</code>)</li><li><b>returns</b>: <code>true</code> if successful</li></ul><p><pre><code>// getTime example
var app = this.app;
app.getTime((message) => {
    if (message.response && message.response.time) {
        var timeDisplay = function(value) {
            let hours = Math.floor(value),
                minutes = Math.floor((value - hours) * 60);
            hours = (hours < 10) ? "0" + hours : hours;
            minutes = (minutes < 10) ? "0" + minutes : minutes;
            return hours + ":" + minutes;
        };
        app.sendTeamMessage('Current Rust time is ' + timeDisplay(message.response.time.time) +
            ' | Sunrise: ' + timeDisplay(message.response.time.sunrise) +
            ' | Sunset: ' + timeDisplay(message.response.time.sunset) +
            ' | Day length: ' + getTimeDisplay(message.response.time.dayLengthMinutes * message.response.time.timeScale * 60));
    }
});
</code></pre></p></li>
  <li><code>postDiscordMessage(msg)</code> Post a message to the bot's communication Discord channel<ul><li><b>msg</b>: The message to post</li></ul><p><pre><code>// postDiscordMessage example
this.app.postDiscordMessage('This is a message from a bot\'s plugin');
</code></pre></p></li>
  <li><code>postDiscordNotification(title, description, url, img, list)</code> Post a notification to the bot's communication Discord channel<ul><li><b>title</b>: The title of the notification</li><li><b>description</b>: The description of the notification</li><li><b>url</b>: The url of the notification (optional)</li><li><b>img</b>: The image url of the notification (optional)</li><li><b>list</b>: The item list data of the notification (optional; see <code>NotificationList</code> below)</li></ul><p><pre><code>// postDiscordNotification example
this.app.postDiscordNotification('Plugin Alert Title', 'Plugin Alert Message');
</code></pre></p></li>
  <li><code>postDiscordWebhook(url, msg)</code> Post a message to a Discord webhook<ul><li><b>url</b>: The url of the Discord webhook</li><li><b>msg</b>: The message to post to the Discord webhook</li></ul><p><pre><code>// postDiscordWebhook example
this.app.postDiscordWebhook('webhook url', 'Webhook Message');
</code></pre></p></li>
  <li><code>sendTeamMessage(msg, callback, noTranslate)</code> Send a team chat message<ul><li><b>msg</b>: The message to send</li><li><b>callback()</b>: The function to execute after sending (optional)</li><li><b>noTranslate</b>: Set to <code>true</code> to disable message translation (optional)</li><li><b>returns</b>: <code>true</code> if successful</li></ul><p><pre><code>// sendTeamMessage example
var app = this.app;
app.sendTeamMessage('This is a team message');
</code></pre></p></li>
    <li><code>setEntityValue(id, value, callback)</code> Set the value of a Smart Switch<ul><li><b>id</b>: The identifier of the Smart Switch</li><li><b>value</b>: The value (true or false)</li><li><b>callback()</b>: The function to execute after setting the value (optional)</li><li><b>returns</b>: <code>true</code> if successful</li></ul><p><pre><code>// setEntityValue example
var app = this.app;
app.setEntityValue(123456, true, () => {
    app.sendTeamMessage('The smart switch is activated');
});
</code></pre></p></li>
  <li><code>translateMessage(msg, lang, success, error)</code> Translate a message to another language<ul><li><b>msg</b>: The message to translate</li><li><b>lang</b>: The language code to use for translation (see: <a href="https://bot.rustplus.io/languages">Language Codes</a>)</li><li><b>success(res)</b>: The function to execute after translating (optional)</li><li><b>error(err)</b>: The function to execute when an error occurs (optional)</li><li><b>returns</b>: <code>true</code></li></ul><p><pre><code>// translateMessage example
var app = this.app;
app.translateMessage('Hello, how are you?', 'es', (res) => {
    app.sendTeamMessage(res);
}, (error) => {
    app.sendTeamMessage('Error: ' + error);
});
</code></pre></p></li>
  <li><code>webGet(url, params, headers, success, error)</code> Retrieve data from a url<ul><li><b>url</b>: The url to access</li><li><b>params</b>: The parameters of the url (optional)</li><li><b>headers</b>: The headers to send with the web request (optional)</li><li><b>success(data)</b>: The function to execute after receiving data (optional)</li><li><b>error(err)</b>: The function to execute when an error occurs (optional)</li><li><b>returns</b>: <code>true</code></li></ul><p><pre><code>// webGet example
var app = this.app;
app.webGet('https://rust.facepunch.com/rss/news', null, null, (data) => {
    var link = '',
        pos = data.indexOf('>https://');
    if (pos > 0) link = data.substr(pos + 1, data.indexOf('<', pos) - pos - 1);
    if (link != '')
        app.sendTeamMessage('The newest Rust update link: ' + link);
    else
        app.sendTeamMessage('The newest Rust update link was not found');
}, (error) => {
    app.sendTeamMessage('Error obtaining Rust update link: ' + error);
});
</code></pre></p></li>
  <li><code>webPost(url, data, headers, success, error)</code> Post data to a url<ul><li><b>url</b>: The url to access</li><li><b>data</b>: The data to post (optional)</li><li><b>headers</b>: The headers to send with the web request (optional)</li><li><b>success(data)</b>: The function to execute after receiving data (optional)</li><li><b>error(err)</b>: The function to execute when an error occurs (optional)</li><li><b>returns</b>: <code>true</code></li></ul><p><pre><code>// webPost example
var app = this.app;
app.webPost('https://httpbin.org/post', 'test data', null, (data) => {
    app.sendTeamMessage('Post result: ' + data);
}, (error) => {
    app.sendTeamMessage('Error posting: ' + error);
});
</code></pre></p></li>
  <li><code>util.collides(x, y, rotation, x1, y1, x2, y2)</code> Returns true if angled point x,y collides with rectangle x1,y1,x2,y2</li>
  <li><code>util.direction(x1, y1, x2, y2)</code> Get the direction from the first point facing the second</li>
  <li><code>util.distance(x1, y1, x2, y2)</code> Get the distance between two points in meters</li>
  <li><code>util.getMapCoords(x, y)</code> Get the map coordinates for a point</li>
  <li><code>util.inRect(x, y, x1, y1, x2, y2)</code> Returns true if the point is inside the rectangle</li>
</ul>

### Data Types

<ul>
  <li>
    <b>BattleMetrics Data</b>
    <pre><code>{     
  "Name": "Server Name",
  "Rank": "#1",
  "Player count": "1/100",
  "Address": "Server Address",
  "Status": "online",
  "Distance": "1 km",
  "Country": "United States",
  "Uptime": "7 Days: 100%, 30 Days: 100%",
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
    <b>Config</b>
    <pre><code>{     
  "lang": "en",
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
  "eventsDiscord": false,
  "broadcastEvents": true,
  "vendingDiscord": false,
  "broadcastVending": true,
  "broadcastVendingName": true,
  "broadcastAmount": 1,
  "battlemetricsID": 0,
  "battlemetricsDiscord": true,
  "deathDiscord": true,
  "loginDiscord": true,
  "autoCleanDevices": true,
  "autoDeviceCommand": true,
  "showDevicePlayer": false,
  "alwaysPostAlarms": true,
  "alwaysPostAlarms": true,
  "decayOffset": 0,
  "eventsDisplay": "heli,brad,cargo,oil,crate,ch47",
  "subEventsDisplay": "heli_lootable,brad_lootable,cargo_crates,oil_lootable"
}</code></pre>
  </li>
  <li>
    <b>Device</b>
    <pre><code>{
  name: "MyDevice",
  id: 123456,
  flag: false, // true if op is inverted
  type: "Smart Switch",
  time: 0 // timestamp of last state change
}</code></pre>
  </li>
  <li>
    <b>EntityInfo</b>
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
    <b>Event</b>
    <pre><code>{
  id: 123456789,
  type: "heli",
  name: "Patrol Helicopter @ A1",
  start: new Date(),
  stop: null // null if active
}</code></pre>
  </li>
  <li>
    <b>Info</b>
    <pre><code>[{
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
}]</code></pre>
  </li>
  <li>
    <b>LeaderMapNotes</b>
    <pre><code>[{
  "type": 1,
  "x": 1500.958740234375,
  "y": 2551.239990234375
}]</code></pre>
  </li>
  <li>
    <b>MapMarkers</b>
    <pre><code>{
  markers: [{
    "id": 123456,
    "type": 3,
    "x": 1500.958740234375,
    "y": 2551.239990234375,
    "steamId": 0,
    "rotation": 0,
    "radius": 0,
    "name": "",
    sellOrders: [{ // sellOrders when type is 3
      "itemId": 123456,
      "quantity": 1,
      "currencyId": 654321,
      "costPerItem": 1,
      "amountInStock": 10,
      "itemIsBlueprint": false,
      "currencyIsBlueprint": false
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
    <b>Members</b>
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
    <b>Notification: Alarm</b>
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
    <b>Notification: Decaying TC</b>
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
    <b>Notification: Entity Pairing</b>
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
    <b>Notification: Event</b>
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
    <b>Notification: Inactive Device</b>
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
    <b>Notification: News</b>
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
    <b>Notification: Player Death</b>
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
    <b>Notification: Player Tracking</b>
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
    <b>Notification: Team Login</b>
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
    <b>NotificationList</b>
    <pre><code>[{
  name: "Sub Item",
  value: "Sub Value",
  inline: false
}]</code></pre>
  </li>
  <li>
    <b>Payload: Smart Switch, Smart Alarm</b>
    <pre><code>{
  "broadcast": {
    "entityChanged": {
      "entityId": 29408388,
      "payload": {
        "value": false,
        "capacity": 0,
        "hasProtection": false,
        "protectionExpiry": 0
      }
    }
  }
}</code></pre>
  </li>
  <li>
    <b>Payload: Storage Monitor</b>
    <pre><code>{
  "broadcast": {
    "entityChanged": {
      "entityId": 29408388,
      "payload": {
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
    }
  }
}</code></pre>
    <blockquote>Find the item name with the itemId using <code>itemIds</code></blockquote>
  </li>
  <li>
    <b>Point</b>
    <pre><code>{
  x: 100,
  y: 200
}</code></pre>
  </li>
  <li>
    <b>TeamInfo</b>
    <pre><code>{
  "leaderSteamId": 123456789,
  "members": [], // see Members
  "leaderMapNotes": [] // see LeaderMapNotes
}</code></pre>
  </li>
  <li>
    <b>Time</b>
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
  <li><code>cmdFormat(str)</code> Convert a string into a non-translatable string<ul><li><b>str</b>: The string to convert</li><li><b>returns</b>: a non-translatable string</li></ul><p><pre><code>// cmdFormat example
var app = this.app,
    msg = 'Here is the url: ' + cmdFormat('https://www.google.com');
app.sendTeamMessage(msg);
</code></pre></p></li>
  <li><code>cmdFormatUndo(str)</code> Undo the non-translatable string conversion<ul><li><b>str</b>: The string to undo the non-translatable string conversion</li><li><b>returns</b>: a string</li></ul><p><pre><code>// cmdFormatUndo example
var app = this.app,
    msg = 'Here is the url: ' + cmdFormat('https://www.google.com');
console.log(cmdFormatUndo(msg));
</code></pre></p></li>
  <li><code>encodeForm(data)</code> Convert an object to form data for a webPost<ul><li><b>data</b>: The object to convert</li><li><b>returns</b>: a string of encoded names and values</li></ul><p><pre><code>// encodeForm example
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
  <li><code>combineItems(items, itemIds)</code> Combine the items from a Storage Monitor payload<ul><li><b>items</b>: The items from the payload</li><li><b>itemIds</b>: The item ID list to lookup item names</li><li><b>returns</b>: A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the combined items and item quantities</li></ul><p><pre><code>// combineItems example
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
  <li><code>getFriendlyDate(date)</code> Get a friendly representation of the date<ul><li><b>date</b>: The date object</li><li><b>returns</b>: a string containing the friendly representation of the date</li></ul><p><pre><code>// getFriendlyDate example
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
  <li><code>getTime(timestr)</code> Convert a time string to seconds<ul><li><b>timestr</b>: The time string (format: 1d1h1m1s)</li><li><b>returns</b>: the total seconds of the timestr</li></ul><p><pre><code>// getTime example
var app = this.app;
app.sendTeamMessage('The time in seconds for 1d1h1m1s is ' + getTime('1d1h1m1s'));
</code></pre></p></li>
  <li><code>getTimeDifference(date)</code> Get the time difference for a date<ul><li><b>date</b>: The date object</li><li><b>returns</b>: the date difference in seconds</li></ul><p><pre><code>// getTimeDifference example
var app = this.app,
    connected = await app.getConnected();
app.sendTeamMessage('The time in seconds since the bot connected is ' + Math.round(getTimeDifference(new Date(connected))));
</code></pre></p></li>
  <li><code>getTimeDisplay(time)</code> Get the time display for time<ul><li><b>time</b>: The time in seconds</li><li><b>returns</b>: a string representing the time</li></ul><p><pre><code>// getTimeDisplay example
var app = this.app,
    connected = await app.getConnected();
app.sendTeamMessage('The bot has been connected for ' + getTimeDisplay(Math.round(getTimeDifference(new Date(connected)))));
</code></pre></p></li>
  <li><code>multiLineFormat(msg, list, callback)</code> Format the message + list to fit the Rust message size using multiple lines<ul><li><b>msg</b>: The message to prepend</li><li><b>list</b>: The list of items to output</li><li><b>callback(line, msg, data)</b>: The function to execute for each formatted line (optional)</li><li><b>returns</b>: an Array containing the formatted lines</li></ul><p><pre><code>// multiLineFormat example
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
</ul>

## Plugin Publishing
          
You can publish your plugin when you are done making any major changes to it by clicking the Publish tab in the Plugin Studio. You have the option of making it a public plugin for others to use with their bot. There will be a review of your plugin after submitting which could take several days. If your plugin is accepted for publishing, you will then be able to select it in the Plugin settings for your bot and install it.

## Links

<ul>
    <li>Website: https://bot.rustplus.io/</li>
    <li>Help & Documentation: https://bot.rustplus.io/help/</li>
</ul>
