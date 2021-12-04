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

###### Properties:

<ul>
  <li><code>devices</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the bot's paired devices (<b>key</b>: device name, <b>value</b>: Array of devices, see <code>Device</code>)</li>
  <li><code>itemIds</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the item names for all item IDs (<b>key</b>: item ID, <b>value</b>: item name)</li>
  <li><code>monuments</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing all monument names and locations (<b>key</b>: monument name, <b>value</b>: monument location, see <code>Point</code>)</li>
  <li><code>tokenMap</code> A <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map">Map</a> object containing the monument names for all monument tokens (<b>key</b>: monument token, <b>value</b>: monument name)</li>
  <li><code>player_id</code> The steam ID of the bot's connected player</li>
  <li><code>player_name</code> The steam name of the bot's connected player</li>
  <li><code>player_token</code> The server token of the bot's connected player</li>
  <li><code>server_ip</code> The IP address of the bot's connected server</li>
  <li><code>server_name</code> The name of the bot's connected server</li>
  <li><code>server_ip</code> The port of the bot's connected server (Rust+ app port)</li>
</ul>

###### Methods:

<ul>
  <li><code>sendTeamMessage(msg, callback)</code> Send a team chat message<ul><li><b>msg</b>: The message to send</li><li><b>callback()</b>: The function to execute after sending (optional)</li></ul></li>
  <li><code>getEntityInfo(id, callback)</code> Get data from a Smart device<ul><li><b>id</b>: The identifier of the Smart device</li><li><b>callback(message)</b>: The function to execute after getting the info (<b>message</b>: contains <code>Payload</code>)</li></ul></li>
  <li><code>getInfo(callback)</code> Get information about the server<ul><li><b>callback(message)</b>: The function to execute after getting the info (<b>message</b>: contains <code>Info</code>)</li></ul></li>
  <li><code>getMapMarkers(callback)</code> Get information about all map markers<ul><li><b>callback(message)</b>: The function to execute after getting the info (<b>message</b>: contains <code>Markers</code>)</li></ul></li>
  <li><code>getTeamInfo(callback)</code> Get information about the team (leader, members)<ul><li><b>callback(message)</b>: The function to execute after getting the info (<b>message</b>: contains <code>Team</code>)</li></ul></li>
  <li><code>getTime(callback)</code> Get information about the server time<ul><li><b>callback(message)</b>: The function to execute after getting the info (<b>message</b>: contains <code>Time</code>)</li></ul></li>
  <li><code>postDiscordMessage(msg)</code> Post a message to the bot's communication Discord channel<ul><li><b>msg</b>: The message to post</li></ul></li>
  <li><code>postDiscordNotification(title, description, url, img, list)</code> Post a notification to the bot's communication Discord channel<ul><li><b>title</b>: The title of the notification</li><li><b>description</b>: The description of the notification</li><li><b>url</b>: The url of the notification (optional)</li><li><b>img</b>: The image url of the notification (optional)</li><li><b>list</b>: The item list data of the notification (optional; see <code>NotificationList</code>)</li></ul></li>
    <li><code>setEntityValue(id, value, callback)</code> Set the value of a Smart Switch<ul><li><b>id</b>: The identifier of the Smart Switch</li><li><b>value</b>: The value (true or false)</li><li><b>callback()</b>: The function to execute after setting the value (optional)</li></ul></li>
  <li><code>webGet(url, params, headers, success, error)</code> Retrieve data from a url<ul><li><b>url</b>: The url to access</li><li><b>params</b>: The parameters of the url (optional)</li><li><b>headers</b>: The headers to send with the web request (optional)</li><li><b>success(data)</b>: The function to execute after receiving data</li><li><b>error(msg)</b>: The function to execute when an error occurs (optional)</li></ul></li>
  <li><code>webPost(url, data, headers, success, error)</code> Post data to a url<ul><li><b>url</b>: The url to access</li><li><b>data</b>: The data to post (optional)</li><li><b>headers</b>: The headers to send with the web request (optional)</li><li><b>success(data)</b>: The function to execute after receiving data</li><li><b>error(msg)</b>: The function to execute when an error occurs (optional)</li></ul></li>
  <li><code>util.getMapCoords(x, y)</code> Get the map coordinates for a Point</li>
  <li><code>util.inRect(x, y, x1, y1, x2, y2)</code> Check if a Point is inside a rectangle</li>
  <li><code>util.direction(x1, y1, x2, y2)</code> Get the direction of one Point versus the other</li>
  <li><code>util.distance(x1, y1, x2, y2)</code> Get the distance between two Points in meters</li>
</ul>
