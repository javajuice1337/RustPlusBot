# **RustPlusBot** plugin example: teamvoice

This plugin example demonstrates how to send voice messages to the **RustPlusBot** voice client. You can invite the voice client to your active voice channel using Discord command: `rp!voice_join`.

This plugin example does not create any team chat commands. Instead, it reacts to the team member online and offline messages sent from the **Team Activity Messages** plugin.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onMessageSend Event:

```js
console.log('onMessageSend Event:', obj);
if (obj.message.indexOf('Team member ') == 0 &&
    (obj.message.indexOf(' is now online') > 0 || obj.message.indexOf(' is now offline') > 0)) {
    this.app.sendDiscordVoiceMessage(cmdFormatUndo(obj.message));
}
```
