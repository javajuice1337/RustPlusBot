# **RustPlusBot** plugin example: hotelfloors

This plugin example demonstrates how to automatically activate multiple similarly named Smart Switches with a single command. This is the same plugin seen in the **MikeTheVike** YouTube video here: https://www.youtube.com/watch?v=ijdQ31TP0hk

By default, the plugin controls paired Smart Switch devices beginning with `floor`. You can change this name by modifying the `device_name` constant.

In this plugin example, the following team chat commands are implemented:

- `!fopen` will turn on all Smart Switches defined in the command argument (space separated or `all`)
- `!fclose` will turn off all Smart Switches defined in the command argument (space separated or `all`)

Plugin command examples:

- `!fopen 1 3 5` will turn on only Smart Switches matching the device name and ending with **1**, **3**, and **5**.
- `!fclose all` will turn off all Smart Switches matching the device name.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onMessageReceive Event:

```
console.log('onMessageReceive Event:', obj);
const device_name = 'floor';
var msg = obj.message,
    m = msg.toLowerCase(),
    prefix = await this.app.getPrefix('all');
if (m.indexOf(prefix + 'fopen ') == 0 || m.indexOf(prefix + 'fclose ') == 0) {
    var q = msg.substr(msg.indexOf(' ') + 1);
    if (q.length > 0) {
        var app = this.app,
            ids = (q != 'all') ? q.split(' ') : [],
            val = (m.indexOf(prefix + 'fopen ') == 0),
            callback = function(cnt) {
            if (cnt > 0) {
                app.sendTeamMessage(((val) ? 'Opened' : 'Closed') + ' floors: ' + q);
            }
            else app.sendTeamMessage('No paired devices named \'' + cmdFormat(device_name) + '\' found');
        },
            cnt = 0;
        this.app.devices.forEach((value, key) => {
            var items = value,
                z = 0;
            for (var x = 0; x < items.length; x++) {
                if (items[x].type == 'Smart Switch' && items[x].name.toLowerCase().indexOf(device_name) == 0 && (ids.length == 0 || ids.indexOf(parseInt(items[x].name.toLowerCase().replace(device_name, '')) + '') >= 0)) {
                    setTimeout((obj) => {
                        app.setEntityValue(obj.id, ((!obj.flag) ? val : !val), () => {
                            cnt--;
                            if (cnt <= 0) callback(z);
                        });
                    }, z * 500, items[x]);
                    cnt++;
                    z++;
                }
            }
        });
        if (cnt == 0) callback(0);
    }
    else this.app.sendTeamMessage('Please enter the floor(s) to open or close');
}
else if (m == prefix + 'fopen') {
    this.app.sendTeamMessage('Example usage: ' + cmdFormat(prefix + 'fopen all'));
}
else if (m == prefix + 'fclose') {
    this.app.sendTeamMessage('Example usage: ' + cmdFormat(prefix + 'fclose all'));
}
```
