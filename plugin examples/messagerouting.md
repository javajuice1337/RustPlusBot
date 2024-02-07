# **RustPlusBot** plugin example: messagerouting

This plugin example demonstrates how to route team chat messages to custom Discord channels. You can specify wildcard matches for the messages and have them posted in their own Discord channels.

To obtain the channel ID for a specific Discord channel, use this guide: https://turbofuture.com/internet/Discord-Channel-ID

Be sure to select the option to not post team chat messages to Discord in **RustPlusBot** settings, otherwise the messages will be duplicated.

> Do not post team chat messages to the Main Discord Channel

This plugin example does not create any team chat commands.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onMessageReceive Event:

```
console.log('onMessageReceive Event:', obj);
const messageRouting = [
    // Add your team chat message routing rules below, before the '*' (catch all) routing rule.
    // Set the wildcard to match the routing rule, and then set the channel to the channel ID.
    // Routing rules are processed from top to bottom, and stops when a rule is matched.
    // Set ignore to true for the routing rule to throw out the team chat message.
    // ----------------------------------------------------------------------------------------
    // Example: Post all Smart Alarm team chat messages to their own Discord channel.
    // { wildcard: '[ALARM]*', channel: '', ignore: false },
    // ----------------------------------------------------------------------------------------
    // Example: Post all Patrol Helicopter team chat messages to their own Discord channel.
    // { wildcard: 'The Patrol Helicopter*', channel: '', ignore: false },
    // ----------------------------------------------------------------------------------------
    { wildcard: '', channel: '', ignore: false },
    { wildcard: '*', channel: '', ignore: false },
];
const wildcardToRegExp = (s) => {
    const regExpEscape = (s) => {
        return s.replace(/[|\\{}()[\]^$+*?.]/g, '\\$&');
    };
    return new RegExp('^' + s.split(/\*+/).map(regExpEscape).join('.*') + '$');
};
for (var i = 0; i < messageRouting.length; i++) {
    if (messageRouting[i].wildcard.length > 0 && messageRouting[i].channel.length > 0 &&
        wildcardToRegExp(messageRouting[i].wildcard).test(obj.message)) {
        if (!messageRouting[i].wildcard.ignore) {
            this.app.postDiscordMessage({
                message: obj.name + ': ' + obj.message,
                channel: messageRouting[i].channel,
                tts: false
            });
        }
        break;
    }
}
```
