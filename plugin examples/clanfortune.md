# **RustPlusBot** plugin example: clanfortune

This plugin example demonstrates how to automatically set your clan's message of the day (MOTD) to a random fortune cookie quote.

In this plugin example, the following team chat commands are implemented:  

- `!clanfortune-interval` / `!clanfortuneinterval` will show or set the interval for how often to update the clan's MOTD. (set to `0` to disable)

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onConnected Event:

```js
console.log('onConnected Event');
const clanfortune_interval = 60; // default interval (in minutes)
if (this.storage.interval == null) this.storage.interval = clanfortune_interval;
if (!this.clanfortuneFunc) {
    var app = this.app,
        fortuneFunc = function() {
        app.webGet("https://www.fortunecookiemessage.com/api/fortune", null, {
            Referer: 'https://www.fortunecookiemessage.com'
        }, function (response) {
            if (response) {
                if (response.message && response.message.length > 0)
                    app.setClanMotd(response.message);
                else
                    app.setClanMotd('No fortune cookie quote available at this time');
            }
        }, (error) => {
            app.setClanMotd('There was an error reading the fortune cookie quote');
        });
    };
    if (!this.clanfortuneFunc)
        setTimeout(fortuneFunc, 5000);
    this.clanfortuneFunc = function() {
        if (this.clanfortuneTask)
            clearInterval(this.clanfortuneTask);
        this.clanfortuneTask = setInterval(fortuneFunc, this.storage.interval * 60000);
    };
}
if (this.storage.interval > 0)
    this.clanfortuneFunc();
```

> [!NOTE]
> After placing the *onConnected* code block in the plugin editor, you will need to press the Play button :arrow_forward: to simulate the event.

#### onDisconnected Event:

```js
console.log('onDisconnected Event');
if (this.clanfortuneTask) {
    clearInterval(this.clanfortuneTask);
    this.clanfortuneTask = null;
}
```

#### onMessageReceive Event:

```js
console.log('onMessageReceive Event:', obj);
var m = obj.message.toLowerCase(),
    prefix = await this.app.getPrefix('all');
if (m.indexOf(prefix + 'clanfortune-interval ') == 0 || m.indexOf(prefix + 'clanfortuneinterval ') == 0) {
    var time = m.substr(m.indexOf(' ') + 1);
    if (time.length > 0 && !isNaN(time) && parseInt(time) >= 0 && parseInt(time) <= 1440) {
        this.storage.interval = parseInt(time);
        if (this.storage.interval > 0) {
            this.clanfortuneFunc();
            this.app.sendTeamMessage('The clan MOTD fortune update interval is now configured for: ' + getTimeDisplay(this.storage.interval * 60));
        }
        else {
            if (this.clanfortuneTask) {
                clearInterval(this.clanfortuneTask);
                this.clanfortuneTask = null;
            }
            this.app.sendTeamMessage('The clan MOTD fortune update interval has been disabled');
        }
    }
    else
        this.app.sendTeamMessage('Please enter a valid interval in minutes 0-1440 (0 to disable)');
}
else if (m == prefix + 'clanfortune-interval' || m == prefix + 'clanfortuneinterval') {
    if (this.storage.interval > 0)
        this.app.sendTeamMessage('The clan MOTD fortune update interval is configured for: ' + getTimeDisplay(this.storage.interval * 60));
    else
        this.app.sendTeamMessage('The clan MOTD fortune update interval has been disabled');
}
```
