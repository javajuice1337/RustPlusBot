# **RustPlusBot** plugin example: mirrormessages

This plugin example demonstrates how to mirror all team chat messages by posting them to another Discord server via a webhook.

This plugin example does not create any team chat commands. Instead, it reacts to receiving a team chat message.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onMessageReceive Event:

```
console.log('onMessageReceive Event:', obj);
const webhook_url = "https://"; // create a webhook in your Discord server and paste the url here
this.app.postDiscordWebhook(webhook_url, obj.name + ': ' + obj.message);
```
