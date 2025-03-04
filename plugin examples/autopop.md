# **RustPlusBot** plugin example: autopop

This plugin example demonstrates how to automatically run the `!pop` command every 5 minutes.

This plugin example does not create any team chat commands. Instead, it creates a timer that runs every 5 minutes executing the `!pop` command.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onConnected Event:

```js
console.log('onConnected Event');
if (!this.autopopTask) this.autopopTask = null;
if (!this.autopopFunc) {
    this.autopopFunc = function() {
        var self = this;
        if (self.autopopTask) clearInterval(self.autopopTask);
        self.autopopTask = setInterval(async function() {
            var prefix = await self.app.getPrefix('all');
            self.app.runCommand(prefix + 'pop');
        }, 5 * 60 * 1000); // 5 minutes
    };
}
this.autopopFunc();
```

> [!NOTE]
> After placing the *onConnected* code block in the plugin editor, you will need to press the Play button :arrow_forward: to simulate the event.

#### onDisconnected Event:

```js
console.log('onDisconnected Event');
if (this.autopopTask) {
    clearInterval(this.autopopTask);
    this.autopopTask = null;
}
```
