# **RustPlusBot** plugin example: turretrotation

The plugins works by rotating auto turret segments, or sections, aka turret flip flop, with a total of 9 segments supported. Each turret segment is controlled by a single smart switch. Each turret in the segment has their Has Target output combined to a single smart alarm. When a turret segment is active, and if its related smart alarm is activated from the combined Has Target output, then the turret segment timer is reset keeping the segment active for longer.

You will need to pair the following named devices to the bot:

- Smart Switches named `TurretSwitch1` through `TurretSwitch9`
- Smart Alarms named `TurretAlarm1` through  `TurretAlarm9`

> [!NOTE]
> For a setup that has less than 9 switches and alarms, use team chat command `!seqmax` to set the amount.

The plugin implements the following team chat commands allowing you to configure it:

- `!seqactive` to disable or enable the plugin's functionality (default: enabled)
- `!seqmax` to change the maximum turret segments to rotate (default: 9; max: 9)
- `!seqtimer` to change the duration of the active turret segment in seconds (default: 60; max: 3600)

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onConnected Event:

```js
console.log('onConnected Event');
const seq_max = 9,
    seq_timer = 60; // 1 minute
if (this.storage.seq_max == null) this.storage.seq_max = seq_max;
if (this.storage.seq_timer == null) this.storage.seq_timer = seq_timer;
if (this.storage.seq_active == null) this.storage.seq_active = true;
if (!this.seq_id) this.seq_id = 1;
if (!this.task) this.task = null;
if (!this.func) this.func = async function() {
    var self = this,
        app = self.app,
        dprefix = '!';
    if (!self.task)
        app.runCommand(dprefix + 'on* TurretSwitch' + self.seq_id);
    else
        clearInterval(self.task);
    self.task = setInterval(function() {
        app.runCommand(dprefix + 'off* TurretSwitch' + self.seq_id);
        if (self.seq_id + 1 <= self.storage.seq_max)
            self.seq_id++;
        else
            self.seq_id = 1;
        app.runCommand(dprefix + 'on* TurretSwitch' + self.seq_id);
    }, self.storage.seq_timer * 1000);
};
if (this.storage.seq_active) this.func();
```

> [!NOTE]
> After placing the *onConnected* code block in the plugin editor, you will need to press the Play button :arrow_forward: to simulate the event.

#### onDisconnected Event:

```js
console.log('onDisconnected Event');
if (this.func) this.func = null;
if (this.task) clearInterval(this.task);
```

#### onEntityChanged Event:

```js
console.log('onEntityChanged Event:', obj);
if (this.storage.seq_active && this.storage.seq_timer > 0 && this.app.devices_map.has(obj.entityId + '') && obj.payload && obj.payload.value) {
    var devices = this.app.devices_map.get(obj.entityId + '');
    if (devices.length > 0 && devices[0] == 'turretalarm' + this.seq_id) {
        if (this.task) clearInterval(this.task);
    }
}
else if (this.storage.seq_active && this.storage.seq_timer > 0 && this.app.devices_map.has(obj.entityId + '') && obj.payload && !obj.payload.value) {
    var devices = this.app.devices_map.get(obj.entityId + '');
    if (devices.length > 0 && devices[0] == 'turretalarm' + this.seq_id) {
        this.func();
    }
}
```

#### onMessageReceive Event:

```js
console.log('onMessageReceive Event:', obj);
const seq_max = 9,
    timer_max = 3600; // 1 hour
var m = obj.message.toLowerCase(),
    prefix = await this.app.getPrefix('all');
if (m.indexOf(prefix + 'seqmax ') == 0 || m.indexOf(prefix + 'seq-max ') == 0) {
    var max = m.substr(m.indexOf(' ') + 1);
    if (!isNaN(max) && parseInt(max) > 0 && parseInt(max) <= seq_max) {
        this.storage.seq_max = parseInt(max);
        this.func();
        this.app.sendTeamMessage('The turret sequence max has been set to: ' + this.storage.seq_max);
    }
    else
        this.app.sendTeamMessage('Please enter a valid turret sequence max amount (1-9)');
}
else if (m == prefix + 'seqmax' || m == prefix + 'seq-max') {
    this.app.sendTeamMessage('The turret sequence max is set to: ' + this.storage.seq_max);
}
else if (m.indexOf(prefix + 'seqtimer ') == 0 || m.indexOf(prefix + 'seq-timer ') == 0) {
    var time = m.substr(m.indexOf(' ') + 1);
    if (time.length > 0)
        time = getTime(time, timer_max);
    else
        time = 0;
    if (time >= 0 && time <= timer_max) {
        this.storage.seq_timer = time;
        if (this.storage.seq_timer > 0) {
            this.func();
            this.app.sendTeamMessage('The turret sequence timer has been set to: ' + getTimeDisplay(this.storage.seq_timer, false, true));
        }
        else {
            if (this.task) clearInterval(this.task);
            this.app.sendTeamMessage('The turret sequence timer has been disabled');
        }
    }
    else
        this.app.sendTeamMessage('Please enter a valid turret sequence timer in seconds (0-3600) (0 to disable)');
}
else if (m == prefix + 'seqtimer' || m == prefix + 'seq-timer') {
    if (this.storage.seq_timer > 0)
        this.app.sendTeamMessage('The turret sequence timer is set to: ' + getTimeDisplay(this.storage.seq_timer, false, true));
    else
        this.app.sendTeamMessage('The turret sequence timer is disabled');
}
else if (m == prefix + 'seqactive' || m == prefix + 'seq-active' ||
    m == prefix + 'seqactive-enable' || m == prefix + 'seq-active-enable' ||
    m == prefix + 'seqactive-disable' || m == prefix + 'seq-active-disable') {
    if (m == prefix + 'seqactive-enable' || m == prefix + 'seq-active-enable')
        this.storage.seq_active = true;
    else if (m == prefix + 'seqactive-disable' || m == prefix + 'seq-active-disable')
        this.storage.seq_active = false;
    else
        this.storage.seq_active = !this.storage.seq_active;
    if (this.storage.seq_active)
        this.func();
    else if (this.task)
        clearInterval(this.task);
    this.app.sendTeamMessage('The turret sequence timer functionality is ' + ((this.storage.seq_active) ? 'enabled' : 'disabled'));
}
```
