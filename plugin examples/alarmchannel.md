# **RustPlusBot** plugin example: alarmchannel

This plugin example demonstrates how to use the activation of a Smart Alarm to post a Discord message in a specific channel and ping a role every 30 seconds.

This plugin example does not create any team chat commands. Instead, it reacts to the activation of a paired Smart Alarm.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onEntityChanged Event:

```js
console.log('onEntityChanged Event:', obj);
const deviceName = 'raidalarm'; // set the lowercased paired device name here
const roleName = 'alarm'; // set the role name to ping here
const channelID = ''; // set the ID of the channel here; see: https://docs.statbot.net/docs/faq/general/how-find-id/
const message = 'Your base is under attack!'; // set the custom message to post to Discord
if (obj.payload && obj.payload.value && this.app.devices_map.has(obj.entityId + '')) {
    var id = this.app.devices_map.get(obj.entityId + '');
    if (id == deviceName) {
        if (!this.alarmNotifyDate)
            this.alarmNotifyDate = {};
        if (!this.alarmNotifyDate[id] || getTimeDifference(this.alarmNotifyDate[id]) > 30) {
            this.alarmNotifyDate[id] = new Date();
            this.app.postDiscordMessage({
                message: '@' + roleName + ' ' + message,
                channel: channelID,
                tts: false
            });
        }
    }
}
```
