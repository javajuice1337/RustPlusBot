# **RustPlusBot** plugin example: eventstimestamp

This plugin example demonstrates how to add timestamps to server event messages sent to team chat.

Be sure to disable the option to send server event notifications to team chat in **RustPlusBot** settings, otherwise the messages will be duplicated.

> :x: Send server event notifications to team chat

This plugin example does not create any team chat commands. Instead, it reacts to receiving an event notification.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onNotification Event:

```js
console.log('onNotification Event:', obj);
const locale = 'en-US'; // change this to match your locale (see: https://www.w3schools.com/jsref/jsref_tolocalestring.asp)
const timestamp = 'America/New_York'; // change this to match your timezone (see: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
const oil_rig_keys = ['oil_rig_small', 'large_oil_rig'];
if (obj.notification && obj.notification.event && this.app.cfg.eventsDisplay) {
    if ((obj.notification.event.type == 'heli' && this.app.cfg.eventsDisplay.indexOf('heli') >= 0) ||
        (obj.notification.event.type == 'cargo' && this.app.cfg.eventsDisplay.indexOf('cargo') >= 0) ||
        (oil_rig_keys.indexOf(obj.notification.event.type) >= 0 && this.app.cfg.eventsDisplay.indexOf('oil') >= 0) ||
        (obj.notification.event.type == 'crate' && this.app.cfg.eventsDisplay.indexOf('crate') >= 0) ||
        (obj.notification.event.type == 'ch47' && this.app.cfg.eventsDisplay.indexOf('ch47') >= 0) ||
        (obj.notification.event.type == 'vendor' && this.app.cfg.eventsDisplay.indexOf('vendor') >= 0) ||
        (obj.notification.event.type == 'deepsea' && this.app.cfg.eventsDisplay.indexOf('deepsea') >= 0)) {
        if (!obj.notification.event.subtype || (this.app.cfg.subEventsDisplay && this.app.cfg.subEventsDisplay.indexOf(obj.notification.event.subtype) >= 0)) {
            var date = new Date(new Date().toLocaleString(locale, {timeZone: timestamp})),
                hour = date.getHours(),
                min = date.getMinutes();
            if (hour < 10) hour = '0' + hour;
            if (min < 10) min = '0' + min;
            this.app.sendTeamMessage('[' + hour + ':' + min + '] ' + obj.notification.event.message);
        }
    }
}
```

> [!TIP]
> Modify the `locale` and `timestamp` const variables to match your physical location.
