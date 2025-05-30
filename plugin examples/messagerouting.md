# **RustPlusBot** plugin example: messagerouting

This plugin example demonstrates how to route team chat messages to custom Discord channels. You can specify wildcard matches for the messages and have them posted to their own Discord channels.

To obtain the channel ID for a specific Discord channel, please follow this guide: https://support.discord.com/hc/en-us/articles/206346498-Where-can-I-find-my-User-Server-Message-ID

Be sure to select the option to not post team chat messages to Discord in **RustPlusBot** settings, otherwise the messages will be duplicated.

> :ballot_box_with_check: Do not post team chat messages to the Main Discord Channel

This plugin example does not create any team chat commands. Instead, it reacts to receiving a team chat message.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onMessageReceive Event:

```js
console.log('onMessageReceive Event:', obj);
const messageRouting = [
    // Add your team chat message routing rules below, before the '*' (catch all) routing rule.
    // Set the wildcard to match the routing rule, and then set the channel to the channel ID.
    // Set ignore to true for the routing rule to throw out (ignore) the team chat message.
    // Set tts to true to have the message spoken using the Discord TTS functionality.
    // Set voice to true to have the message spoken using the RustPlusBot voice client.
    // Routing rules are processed from top to bottom, and stops when a rule is matched.
    // ------------------------------------------------------------------------------------------------------
    // Example 1: Post all Smart Alarm team chat messages to their own Discord channel.
    // { wildcard: "[ALARM]*", channel: "1234567890", ignore: false, tts: false, voice: false },
    // ------------------------------------------------------------------------------------------------------
    // Example 2: Post all Patrol Helicopter team chat messages to their own Discord channel.
    // { wildcard: "The Patrol Helicopter*", channel: "1234567890", ignore: false, tts: false, voice: false },
    // ------------------------------------------------------------------------------------------------------
    // Example 3: Ignore all team member AFK team chat messages (do not post)
    // { wildcard: "Team member '*' is*AFK*", channel: "", ignore: true, tts: false, voice: false },
    // ------------------------------------------------------------------------------------------------------
    { wildcard: "", channel: "", ignore: false, tts: false, voice: false },
    { wildcard: "*", channel: "", ignore: false, tts: false, voice: false },
];
const wildcardToRegExp = (s) => {
    const regExpEscape = (s) => {
        return s.replace(/[|\\{}()[\]^$+*?.]/g, '\\$&');
    };
    return new RegExp('^' + s.split(/\*+/).map(regExpEscape).join('.*') + '$');
};
for (var i = 0; i < messageRouting.length; i++) {
    if (messageRouting[i].wildcard.length > 0 && wildcardToRegExp(messageRouting[i].wildcard).test(obj.message)) {
        if (messageRouting[i].channel.length > 0 && !messageRouting[i].ignore) {
            this.app.postDiscordMessage({
                message: obj.name + ': ' + obj.message,
                channel: messageRouting[i].channel,
                tts: messageRouting[i].tts
            });
            if (messageRouting[i].voice)
                this.app.sendDiscordVoiceMessage(obj.message);
        }
        break;
    }
}
```

> [!IMPORTANT]
> Make sure you set the channel ID for the catch all routing rule `"*"`, otherwise all non-routed messages will be ignored by default.

> [!TIP]
> The `wilcard` field is case-sensitive so make sure the message is entered exactly as it appears in team chat.
