# **RustPlusBot** plugin example: macro

This example demonstrates how to use a single team chat command to execute multiple chat commands.

In this plugin example, the following team chat commands are implemented:

- `!poptime` runs commands `!pop` and `!time`
- `!daynight` runs commands `!day` and `!night`

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onMessageReceive Event:

```
console.log('onMessageReceive Event:', obj);
var msg = obj.message,
    m = msg.toLowerCase(),
    prefix = await this.app.getPrefix('all');
if (m.indexOf(prefix + 'poptime') == 0) {
    var app = this.app;
    app.runCommand(prefix + 'pop');
    app.runCommand(prefix + 'time');
}
else if (m.indexOf(prefix + 'daynight') == 0) {
    var app = this.app;
    app.runCommand(prefix + 'day');
    app.runCommand(prefix + 'night');
}
```
