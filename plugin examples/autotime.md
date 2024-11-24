# **RustPlusBot** plugin example: autotime

This plugin example demonstrates how to automatically send until daytime / until nighttime team chat announcements.

You can change the *until daytime* and *until nighttime* offsets using the implemented plugin commands below.

In this plugin example, the following team chat commands are implemented:

- `offset-daytime` will show or set the until daytime announcement message offset.
- `offset-nighttime` will show or set the until nighttime announcement message offset.
- `autotime-voice` will enable / disable voice feedback for the announcement message.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onConnected Event:

```js
console.log('onConnected Event');
const offset_daytime = 3; // default offset until daytime in minutes
const offset_nighttime = 3; // default offset until nighttime in minutes
if (this.storage.offset_daytime == null) this.storage.offset_daytime = offset_daytime;
if (this.storage.offset_nighttime == null) this.storage.offset_nighttime = offset_nighttime;
if (this.storage.voice_feedback == null) this.storage.voice_feedback = true;
if (!this.timeData) this.timeData = { daytime: false, nighttime: false };
if (!this.timeTask) this.timeTask = null;
if (!this.timeFunc) {
    this.timeFunc = function() {
        var self = this;
        self.timeTask = setInterval(function() {
            self.app.getDetailedInfo((data) => {
                if (data.time) {
                    if ((self.storage.offset_daytime > 0 && !self.timeData.daytime && data.nextDay < self.storage.offset_daytime * 60) ||
                        (self.storage.offset_nighttime > 0 && !self.timeData.nighttime && data.nextNight < self.storage.offset_nighttime * 60)) {
                        if (self.storage.offset_daytime > 0 && !self.timeData.daytime && data.nextDay < self.storage.offset_daytime * 60) {
                            self.timeData.daytime = true;
                            self.timeData.nighttime = false;
                        }
                        else if (self.storage.offset_nighttime > 0 && !self.timeData.nighttime && data.nextNight < self.storage.offset_nighttime * 60) {
                            self.timeData.nighttime = true;
                            self.timeData.daytime = false;
                        }
                        var nexttime = (data.nextDay >= data.nextNight) ? data.nextNight : data.nextDay,
                            nexttext = (data.nextDay >= data.nextNight) ? 'night' : 'day';
                        self.app.sendTeamMessage('The current server time is ' + cmdFormat(data.time) + ', with ' + formatTimeLong(nexttime, false, true) + ' remaining until ' + nexttext + 'time', null, false, self.storage.voice_feedback);
                    }
                }
            });
        }, 30000);
    };
}
this.timeFunc();
```

> **Note:** After placing the *onConnected* code block in the plugin editor, you will need to press the Play button :arrow_forward: to simulate the event.

#### onDisconnected Event:

```js
console.log('onDisconnected Event');
if (this.timeTask) {
    clearInterval(this.timeTask);
    this.timeTask = null;
}
```

#### onMessageReceive Event:

```js
console.log('onMessageReceive Event:', obj);
var m = obj.message.toLowerCase(),
    prefix = await this.app.getPrefix('all');
if (m.indexOf(prefix + 'offset-daytime ') == 0 || m.indexOf(prefix + 'offsetdaytime ') == 0) {
    var time = m.substr(m.indexOf(' ') + 1);
    if (time.length > 0 && !isNaN(time) && parseInt(time) >= 0 && parseInt(time) <= 10) {
        this.storage.offset_daytime = parseInt(time);
        if (this.storage.offset_daytime > 0)
            this.app.sendTeamMessage('The until daytime announcement is offset: ' + getTimeDisplay(this.storage.offset_daytime * 60));
        else
            this.app.sendTeamMessage('The until daytime announcement has been disabled');
    }
    else
        this.app.sendTeamMessage('Please enter a valid until daytime offset in minutes 0-10 (0 to disable)');
}
else if (m == prefix + 'offset-daytime' || m == prefix + 'offsetdaytime') {
    if (this.storage.offset_daytime > 0)
        this.app.sendTeamMessage('The until daytime announcement is offset: ' + getTimeDisplay(this.storage.offset_daytime * 60));
    else
        this.app.sendTeamMessage('The until daytime announcement offset has been disabled');
}
else if (m.indexOf(prefix + 'offset-nighttime ') == 0 || m.indexOf(prefix + 'offsetnighttime ') == 0) {
    var time = m.substr(m.indexOf(' ') + 1);
    if (time.length > 0 && !isNaN(time) && parseInt(time) >= 0 && parseInt(time) <= 10) {
        this.storage.offset_nighttime = parseInt(time);
        if (this.storage.offset_nighttime > 0)
            this.app.sendTeamMessage('The until nighttime announcement is offset: ' + getTimeDisplay(this.storage.offset_nighttime * 60));
        else
            this.app.sendTeamMessage('The until nighttime announcement has been disabled');
    }
    else
        this.app.sendTeamMessage('Please enter a valid until nighttime offset in minutes 0-10 (0 to disable)');
}
else if (m == prefix + 'offset-nighttime' || m == prefix + 'offsetnighttime') {
    if (this.storage.offset_nighttime > 0)
        this.app.sendTeamMessage('The until nighttime announcement is offset: ' + getTimeDisplay(this.storage.offset_nighttime * 60));
    else
        this.app.sendTeamMessage('The until nighttime announcement offset has been disabled');
}
else if (m == prefix + 'autotimevoice' || m == prefix + 'autotime-voice' || m == prefix + 'auto-time-voice') {
    this.storage.voice_feedback = !this.storage.voice_feedback;
    this.app.sendTeamMessage('The auto time voice feedback is ' + ((this.storage.voice_feedback) ? 'enabled' : 'disabled'));
}
```
