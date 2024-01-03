# **RustPlusBot** plugin example: timedfireworks

This plugin example demonstrates how to activate multiple Smart Switches with different delays, allowing you to create a timed firework show. The Smart Switches should be directly wired to Igniters near the fireworks.

In this plugin example, the following team chat command is implemented:

- `!fireworks` controls the following paired Smart Switch device names: fireworks1, fireworks2, fireworks3

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onMessageReceive Event:

```
console.log('onMessageReceive Event:', obj);
var msg = obj.message,
    m = msg.toLowerCase(),
    prefix = await this.app.getPrefix('all'),
    dprefix = await this.app.getPrefix('device');
if (m == prefix + 'fireworks') {
    var app = this.app;
    setTimeout(function() {
        app.runCommand(dprefix + 'on fireworks1');
        app.runCommand(dprefix + 'off fireworks1');
    }, 5 * 1000); // 5 seconds delay
    setTimeout(function() {
        app.runCommand(dprefix + 'on fireworks2');
        app.runCommand(dprefix + 'off fireworks2');
    }, 10 * 1000); // 10 seconds delay
    setTimeout(function() {
        app.runCommand(dprefix + 'on fireworks3');
        app.runCommand(dprefix + 'off fireworks3');
    }, 15 * 1000); // 15 seconds delay
}
```
