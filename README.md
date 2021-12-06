<p align="center">
<a href="https://bot.rustplus.io/"><img src="./rustplusbot.png" width="400"></a>
</p>

# **RustPlusBot** plugins reference and examples

**RustPlusBot** plugins allow the user to develop their own commands to add functionality to the bot. The plugin itself exposes many events for a programmer to attach to and execute code.

The plugins are written in JavaScript and run in a NodeJS environment.

## Plugin Events:

<ul>
  <li><code>onConnected()</code> Fires when the bot connects to a server</li>
  <li><code>onDisconnected()</code> Fires when the bot disconnects from a server</li>
  <li><code>onEntityChanged(obj)</code> Fires when a paired Smart Device is changed<ul><li><b>obj.entityId</b>: The entity ID of the Smart device</li><li><b>obj.payload</b>: The payload data of the event (see <code>Payload</code> below)</li></ul></li>
  <li><code>onMessageReceive(obj)</code> Fires when a team chat message is received<ul><li><b>obj.message</b>: The incoming team chat message</li><li><b>obj.name</b>: The steam name of the sender</li><li><b>obj.steamId</b>: The steam ID of the sender</li></ul></li>
  <li><code>onMessageSend(obj)</code> Fires when a team chat message is sent<ul><li><b>obj.message</b>: The outgoing team chat message</li></ul></li>
  <li><code>onNotification(obj)</code> Fires when there is a bot notification<ul><li><b>obj.notification</b>: The notification data of the event (see <code>Notification</code> below)</li></ul></li>
  <li><code>onTeamChanged(obj)</code> Fires when the team leader changes, or a team member is added or removed from the team<ul><li><b>obj.leaderSteamId</b>: The steam ID of the team leader</li><li><b>obj.leaderMapNotes</b>: The leader map notes data of the event (see <code>LeaderMapNotes</code> below)</li><li><b>obj.members</b>: The members list data of the event (see <code>Members</code> below)</li></ul></li>
</ul>

> Plugins are loaded when the bot is starting and lasts for its entire life-cycle. Restarting the bot also restarts all plugins.

## Plugin Interface:

The `app` object exists in the plugin's scope `this`, and exposes the following properties and methods:

### Properties:

<ul>
  <li><code>devices</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the bot's paired devices (<b>key</b>: device name, <b>value</b>: Array of devices, see <code>Device</code> below)</li>
  <li><code>itemIds</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the item names for all item IDs (<b>key</b>: item ID, <b>value</b>: item name)</li>
  <li><code>monuments</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing all monument names and locations (<b>key</b>: monument name, <b>value</b>: monument location, see <code>Point</code> below)</li>
  <li><code>tokenMap</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the monument names for all monument tokens (<b>key</b>: monument token, <b>value</b>: monument name)</li>
  <li><code>player_id</code> The steam ID of the bot's connected player</li>
  <li><code>player_name</code> The steam name of the bot's connected player</li>
  <li><code>player_token</code> The server token of the bot's connected player</li>
  <li><code>server_ip</code> The IP address of the bot's connected server</li>
  <li><code>server_name</code> The name of the bot's connected server</li>
  <li><code>server_ip</code> The port of the bot's connected server (Rust+ app port)</li>
</ul>

### Methods:

<ul>
  <li><code>sendTeamMessage(msg, callback)</code> Send a team chat message<ul><li><b>msg</b>: The message to send</li><li><b>callback()</b>: The function to execute after sending (optional)</li></ul><p><pre><code>// sendTeamMessage example
var app = this.app;
app.sendTeamMessage('This is a team message');
</code></pre></p></li>
  <li><code>getEntityInfo(id, callback)</code> Get data from a Smart device<ul><li><b>id</b>: The identifier of the Smart device</li><li><b>callback(message)</b>: The function to execute after getting the info (<code>message.response</code> contains <code>EntityInfo</code>)</li></ul><p><pre><code>// getEntityInfo example
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
  <li><code>getInfo(callback)</code> Get information about the server<ul><li><b>callback(message)</b>: The function to execute after getting the info (<code>message.response</code> contains <code>Info</code>)</li></ul><p><pre><code>// getInfo example
var app = this.app;
app.getInfo((message) => {
    if (message.response && message.response.info) {
        app.sendTeamMessage('Current server population is ' + message.response.info.players + ' / ' + message.response.info.maxPlayers + ((message.response.info.queuedPlayers > 0) ? ' (' + message.response.info.queuedPlayers + ' in queue)' : ''));
    }
});
</code></pre></p></li>
  <li><code>getMapMarkers(callback)</code> Get information about all map markers<ul><li><b>callback(message)</b>: The function to execute after getting the info (<code>message.response</code> contains <code>MapMarkers</code>)</li></ul><p><pre><code>// getMapMarkers example
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
  <li><code>getTeamInfo(callback)</code> Get information about the team (leader, members)<ul><li><b>callback(message)</b>: The function to execute after getting the info (<code>message.response</code> contains <code>TeamInfo</code>)</li></ul><p><pre><code>// getTeamInfo example
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
  <li><code>getTime(callback)</code> Get information about the server time<ul><li><b>callback(message)</b>: The function to execute after getting the info (<code>message.response</code> contains <code>Time</code>)</li></ul><p><pre><code>// getTime example
var app = this.app;
app.getTime((message) => {
    if (message.response && message.response.time) {
        app.sendTeamMessage('Current Rust time is ' + (Math.floor(message.response.time.time) + (Math.round((message.response.time.time % 1) * (message.response.time.dayLengthMinutes * message.response.time.timeScale)) / 100)).toFixed(2));
    }
});
</code></pre></p></li>
  <li><code>postDiscordMessage(msg)</code> Post a message to the bot's communication Discord channel<ul><li><b>msg</b>: The message to post</li></ul><p><pre><code>// postDiscordMessage example
this.app.postDiscordMessage('This is a message from a bot\'s plugin');
</code></pre></p></li>
  <li><code>postDiscordNotification(title, description, url, img, list)</code> Post a notification to the bot's communication Discord channel<ul><li><b>title</b>: The title of the notification</li><li><b>description</b>: The description of the notification</li><li><b>url</b>: The url of the notification (optional)</li><li><b>img</b>: The image url of the notification (optional)</li><li><b>list</b>: The item list data of the notification (optional; see <code>NotificationList</code> below)</li></ul><p><pre><code>// postDiscordNotification example
this.app.postDiscordNotification('Plugin Alert Title', 'Plugin Alert Message');
</code></pre></p></li>
    <li><code>setEntityValue(id, value, callback)</code> Set the value of a Smart Switch<ul><li><b>id</b>: The identifier of the Smart Switch</li><li><b>value</b>: The value (true or false)</li><li><b>callback()</b>: The function to execute after setting the value (optional)</li></ul><p><pre><code>// setEntityValue example
var app = this.app;
app.setEntityValue(123456, true, () => {
    app.sendTeamMessage('The smart switch is activated');
});
</code></pre></p></li>
  <li><code>webGet(url, params, headers, success, error)</code> Retrieve data from a url<ul><li><b>url</b>: The url to access</li><li><b>params</b>: The parameters of the url (optional)</li><li><b>headers</b>: The headers to send with the web request (optional)</li><li><b>success(data)</b>: The function to execute after receiving data (optional)</li><li><b>error(msg)</b>: The function to execute when an error occurs (optional)</li></ul><p><pre><code>// webGet example
var app = this.app;
app.webGet('https://rust.facepunch.com/rss/news', null, null, (data) => {
    var link = '',
        pos = data.indexOf('>https://');
    if (pos > 0) link = data.substr(pos, data.indexOf('<', pos) - pos);
    if (link != '')
        app.sendTeamMessage('The newest Rust update link: ' + link);
    else
        app.sendTeamMessage('The newest Rust update link was not found');
}, (error) => {
    app.sendTeamMessage('Error obtaining Rust update link: ' + error);
});
</code></pre></p></li>
  <li><code>webPost(url, data, headers, success, error)</code> Post data to a url<ul><li><b>url</b>: The url to access</li><li><b>data</b>: The data to post (optional)</li><li><b>headers</b>: The headers to send with the web request (optional)</li><li><b>success(data)</b>: The function to execute after receiving data (optional)</li><li><b>error(msg)</b>: The function to execute when an error occurs (optional)</li></ul><p><pre><code>// webPost example
var app = this.app;
app.webPost('https://httpbin.org/post', 'test data', null, (data) => {
    app.sendTeamMessage('Post result: ' + data);
}, (error) => {
    app.sendTeamMessage('Error posting: ' + error);
});
</code></pre></p></li>
  <li><code>util.getMapCoords(x, y)</code> Get the map coordinates for a Point</li>
  <li><code>util.inRect(x, y, x1, y1, x2, y2)</code> Check if a Point is inside a rectangle</li>
  <li><code>util.direction(x1, y1, x2, y2)</code> Get the direction of one Point versus the other</li>
  <li><code>util.distance(x1, y1, x2, y2)</code> Get the distance between two Points in meters</li>
</ul>

### Data Types:

<ul>
  <li>
    <b>Device</b>
    <pre><code>device.name, device.id, device.flag, device.type</code></pre>
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
    <b>Info</b>
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
    sellOrders: [{
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
    <pre><code>
// marker types:
// 1 = Player
// 2 = Explosion
// 3 = VendingMachine
// 4 = CH47
// 5 = CargoShip
// 6 = Crate</code></pre>
    <blockquote>Find the item name with the itemId using <code>itemIds</code></blockquote>
  </li>
  <li>
    <b>Members</b>
    <pre><code>[{
  "steamId":"123456789",
  "name":"RustPlayer1",
  "x":1497.4344482421875,
  "y":2522.85546875,
  "isOnline":true,
  "spawnTime":1638768666,
  "isAlive":true,
  "deathTime":1638768647
}, {
  "steamId":"123456799",
  "name":"RustPlayer2",
  "x":1487.4344482421875,
  "y":2512.85546875,
  "isOnline":true,
  "spawnTime":1638768866,
  "isAlive":true,
  "deathTime":1638768847
}]</code></pre>
  </li>
  <li>
    <b>Notification: Alarm</b>
    <pre><code>{
  "notification: {
    "type": "alarm",
    "server_name": "Rust Server Name",
    "alarm": {
      "title": "Smart Alarm Title",
      "message": "Smart Alarm Message"
    }
  }
}</code></pre>
  </li>
  <li>
    <b>Notification: Decaying TC</b>
    <pre><code>{
  "notification": {
    "type": "monitor",
    "server_name": "Rust Server Name",
    "monitor": {
      "title": "MyDevice",
      "message": "This monitored TC is decaying!",
      "device_name": "MyDevice",
      "device_id": "123456",
      "device_flag": false,
      "device_type": "Storage Monitor"
    }
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
    <b>Notification: Inactive Device</b>
    <pre><code>{
  "notification": {
    "type": "inactive",
    "server_name": "Rust Server Name",
    "inactive": {
      "title": "MyDevice",
      "message": "This device is inactive.",
      "device_name": "MyDevice",
      "device_id": "123456",
      "device_flag": false,
      "device_type": "Smart Switch"
    }
  }
}</code></pre>
  </li>
  <li>
    <b>Notification: News</b>
    <pre><code>{
  "notification: {
    "type": "news",
    "server_name": "Rust Server Name",
    "news": {
      "title": "Rust Update News",
      "url": "https://rust.facepunch.com/",
      "message": "A new Rust update blog is available!"
    }
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
    "server_name": "Rust Server Name",
    "tracking": {
      "id": "12345",
      "name": "Tracked player \"Rust Player\" has joined the server",
      "status": "joined"
    }
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
  title: "Sub Item",
  value: "Sub Value"
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
    <pre><code>point.x, point.y</code></pre>
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
