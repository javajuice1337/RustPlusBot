# **RustPlusBot** plugin example: autopop

This example demonstrates how to automatically run the `!pop` command every 5 minutes.

This plugin example does not create any team chat commands. Instead, it creates a timer that runs every 5 minutes executing the `!pop` command.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onConnected Event:

```
console.log('onConnected Event');
if (!this.autopopTask) this.autopopTask = null;
if (!this.autopopFunc) {
    this.autopopFunc = function() {
        var self = this;
        self.autopopTask = setInterval(async function() {
            var prefix = await self.app.getPrefix('all');
            self.app.runCommand(prefix + 'pop');
        }, 5 * 60 * 1000); // 5 minutes
    };
}
this.autopopFunc();
```

#### onDisconnected Event:

```
console.log('onDisconnected Event');
if (this.autopopTask) {
    clearInterval(this.autopopTask);
    this.autopopTask = null;
}
```
