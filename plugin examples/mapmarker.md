# **RustPlusBot** plugin example: mapmarker

This plugin example demonstrates how to pin a custom map marker to the Interactive Map.

In this plugin example, the following team chat commands are implemented:

- `!pin` sets a map marker at the player's current location with a custom message
- `!pin-clear` clears all of the player's custom markers

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onConnected Event:

```js
console.log('onConnected Event');
if (!this.mapInfo) {
    var self = this;
    setTimeout(function() {
        if (self.app && self.app.getMapInfo) {
            self.app.getMapInfo((message) => {
                if (message) {
                    self.mapInfo = message;
                }
            });
        }
    }, 5000);
}
```

> [!NOTE]
> After placing the *onConnected* code block in the plugin editor, you will need to press the Play button :arrow_forward: to simulate the event.

#### onMessageReceive Event:

```js
console.log('onMessageReceive Event:', obj);
var msg = obj.message,
    m = msg.toLowerCase(),
    prefix = await this.app.getPrefix('all');
if (m.indexOf(prefix + 'pin ') == 0) {
    var pinmsg = msg.substr(msg.indexOf(' ') + 1);
    if (pinmsg.length > 0) {
        var self = this,
            id = (new Date()).getTime() + '';
        self.app.getTeamInfo((message) => {
            if (message.response && message.response.teamInfo) {
                for (var i = 0; i < message.response.teamInfo.members.length; i++) {
                    if (message.response.teamInfo.members[i].steamId == obj.steamId) {
                        var margin = self.mapInfo.oceanMargin,
                            diff = (self.mapInfo.width - (margin * 2)) / self.mapInfo.info.mapSize,
                            x = margin + (message.response.teamInfo.members[i].x * diff),
                            y = self.mapInfo.height - (margin + (message.response.teamInfo.members[i].y * diff));
                        self.app.interactiveMap.addMarker(id, obj.steamId, obj.name, pinmsg, x, y);
                        self.app.sendTeamMessage('A marker was pinned at your location with message: ' + cmdFormat(pinmsg));
                        break;
                    }
                }
            }
        });
    }
    else
        this.app.sendTeamMessage('Please enter a message for the pinned marker after the command');
}
else if (m == prefix + 'pin') {
    this.app.sendTeamMessage('Example usage: ' + cmdFormat(prefix + 'pin my marker message'));
}
else if (m == prefix + 'pin-clear') {
    this.app.interactiveMap.clearMarkers(obj.steamId);
    this.app.sendTeamMessage('Your pinned markers have been cleared');
}
```
