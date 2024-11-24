# **RustPlusBot** plugin example: vendingitems

This plugin example demonstrates how to output all vending machine items in CSV or JSON format.

In this plugin example, the following team chat commands are implemented:

- `!vending-csv` output all vending machine item data to the plugin console in CSV format
- `!vending-json` output all vending machine item data to the plugin console in JSON format

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onMessageReceive Event:

```js
console.log('onMessageReceive Event:', obj);
var msg = obj.message,
    m = msg.toLowerCase(),
    app = this.app,
    prefix = await app.getPrefix('all'),
    processMarkers = function(markers) {
    var items = [];
    for (var i = 0; i < markers.length; i++) {
        if (markers[i].type == 3) { // 'VendingMachine'
            if (markers[i].sellOrders && markers[i].sellOrders.length) {
                for (var x = 0; x < markers[i].sellOrders.length; x++) {
                    var id = markers[i].sellOrders[x].itemId + '';
                    if (app.itemIds.has(id) && app.itemIds.has(markers[i].sellOrders[x].currencyId + '')) {
                        items.push({
                            item: app.itemIds.get(id) + ((markers[i].sellOrders[x].itemIsBlueprint) ? ' (BP)' : ''),
                            quantity: markers[i].sellOrders[x].quantity, currency: app.itemIds.get(markers[i].sellOrders[x].currencyId + '') + ((markers[i].sellOrders[x].currencyIsBlueprint) ? ' (BP)' : ''),
                            cost: Math.round(markers[i].sellOrders[x].costPerItem * markers[i].sellOrders[x].priceMultiplier),
                            stock: markers[i].sellOrders[x].amountInStock,
                            durability: Math.round(markers[i].sellOrders[x].itemCondition / markers[i].sellOrders[x].itemConditionMax * 100),
                            location: markers[i].location
                        });
                    }
                }
            }
        }
    }
    return items;
};
if (m == prefix + 'vending-csv') {
    app.getMapMarkers((message) => {
        if (message.response && message.response.mapMarkers && message.response.mapMarkers.markers) {
            var items = processMarkers(message.response.mapMarkers.markers);
            if (items.length > 0) {
                app.sendTeamMessage('Logging ' + items.length + ' vending item(s) to the plugin console (csv format)');
                items = 'item,quantity,cost,stock,durability,location\r\n' + items.map(function(item) {
                    return `"${item.item}","${item.quantity}","${item.cost}","${item.stock}","${item.durability||''}","${item.location}"`;
                }).join('\r\n');
                console.log(items);
            }
            else
                app.sendTeamMessage('No vending items were found on the server');
        }
    });
}
else if (m == prefix + 'vending-json') {
    app.getMapMarkers((message) => {
        if (message.response && message.response.mapMarkers && message.response.mapMarkers.markers) {
            var items = processMarkers(message.response.mapMarkers.markers);
            if (items.length > 0) {
                app.sendTeamMessage('Logging ' + items.length + ' vending item(s) to the plugin console (json format)');
                items = JSON.stringify(items);
                console.log(items);
            }
            else
                app.sendTeamMessage('No vending items were found on the server');
        }
    });
}
```
