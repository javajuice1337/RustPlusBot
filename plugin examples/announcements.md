# **RustPlusBot** plugin example: announcements

This plugin example demonstrates how to automatically send pre-defined team chat announcements.

You can define the announcements in the bot's config on the Commands tab, in the Alias section. Create an alias with the input `%ANNOUNCEMENT_0%` and output will be the message you want to display. Do this for announcements `0` - `9`.

In this plugin example, the following team chat commands are implemented:

- `!announcement-interval` / `!announcementinterval` will show or set the interval for the announcement message.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onConnected Event:

```
console.log('onConnected Event');
const announcements_amount = 9; // total number of announcements
const announcements_interval = 20; // default interval (in minutes)
if (this.storage.interval == null) this.storage.interval = announcements_interval;
if (!this.announcementFunc) {
    this.announcementFunc = function() {
        if (this.announcementTask)
            clearInterval(this.announcementTask);
        var self = this,
            last_idx = 0;
        self.announcementTask = setInterval(function() {
            var idx = 0;
            while (idx == 0 || idx == last_idx)
                idx = Math.floor(Math.random() * announcements_amount) + 1;
            self.app.sendTeamMessage(`%ANNOUNCEMENT_${idx}%`);
            last_idx = idx;
        }, this.storage.interval * 60000);
    };
}
if (this.storage.interval > 0)
    this.announcementFunc();
```

> **Note:** After placing the *onConnected* code block in the plugin editor, you will need to press the Play button :arrow_forward: to simulate the event.

#### onDisconnected Event:

```
console.log('onDisconnected Event');
if (this.announcementTask) {
    clearInterval(this.announcementTask);
    this.announcementTask = null;
}
```

#### onMessageReceive Event:

```
console.log('onMessageReceive Event:', obj);
var m = obj.message.toLowerCase(),
    prefix = await this.app.getPrefix('all');
if (m.indexOf(prefix + 'announcement-interval ') == 0 || m.indexOf(prefix + 'announcementinterval ') == 0) {
    var time = m.substr(m.indexOf(' ') + 1);
    if (time.length > 0 && !isNaN(time) && parseInt(time) >= 0 && parseInt(time) <= 60) {
        this.storage.interval = parseInt(time);
        if (this.storage.interval > 0) {
            this.announcementFunc();
            this.app.sendTeamMessage('The Announcement Message interval is now configured for: ' + getTimeDisplay(this.storage.interval * 60));
        }
        else {
            if (this.announcementTask) {
                clearInterval(this.announcementTask);
                this.announcementTask = null;
            }
            this.app.sendTeamMessage('The Announcement Message interval has been disabled');
        }
    }
    else
        this.app.sendTeamMessage('Please enter a valid interval in minutes 0-60 (0 to disable)');
}
else if (m == prefix + 'announcement-interval' || m == prefix + 'announcementinterval') {
    if (this.storage.interval > 0)
        this.app.sendTeamMessage('The Announcement Message interval is configured for: ' + getTimeDisplay(this.storage.interval * 60));
    else
        this.app.sendTeamMessage('The Announcement Message interval has been disabled');
}
```
