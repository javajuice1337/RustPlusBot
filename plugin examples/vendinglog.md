# **RustPlusBot** plugin example: vendinglog

This plugin example demonstrates how to log all vending machine item changes on the server to the Notifications Discord Channel (or the Plugin Notification channel if set in the bot's config).

> Note: Any vending machine item changes at safe zones will be ignored.

This plugin example does not create any team chat commands. Instead, it creates a timer that runs every 30 seconds and analyzes all vending machine map markers for changes.

### Place the following code blocks in their respective events in the Plugin Studio to test this plugin:

#### onConnected Event:

```js
console.log('onConnected Event');
if (!this.mapMarkers) this.mapMarkers = [];
if (!this.vendingTask) this.vendingTask = null;
if (!this.vendingFunc) {
    const getMapMarker = function(id, markers) {
        for (var i = 0; i < markers.length; i++) {
            if (markers[i].id == id) return markers[i];
        }
        return null;
    };
    this.vendingFunc = function() {
        var self = this,
            processVending = function(items, _items) {
            if (!items) return [];
            var list = [];
            for (var x = 0; x < items.length; x++) {
                var id = items[x].itemId + '';
                if (self.app.itemIds.has(id) && self.app.itemIds.has(items[x].currencyId + '')) {
                    if (!_items) {
                        list.push({ status: '', item: self.app.itemIds.get(id) + ((items[x].itemIsBlueprint) ? ' (BP)' : ''), quantity: items[x].quantity, currency: self.app.itemIds.get(items[x].currencyId + '') + ((items[x].currencyIsBlueprint) ? ' (BP)' : ''), cost: Math.round(items[x].costPerItem * items[x].priceMultiplier), stock: items[x].amountInStock, durability: Math.round(items[x].itemCondition / items[x].itemConditionMax * 100) });
                    }
                    else {
                        var status = '';
                        if (x < _items.length && items[x].itemId == _items[x].itemId && items[x].currencyId == _items[x].currencyId) {
                            if (items[x].amountInStock != _items[x].amountInStock) {
                                var delta = items[x].amountInStock - _items[x].amountInStock;
                                status = 'Changed: ' + ((delta > 0) ? '+' : '') + delta;
                            }
                        }
                        else if (x < _items.length) {
                            status = 'Changed';
                        }
                        else {
                            status = 'Added';
                        }
                        if (status.length > 0) {
                            list.push({ status: status, item: self.app.itemIds.get(id) + ((items[x].itemIsBlueprint) ? ' (BP)' : ''), quantity: items[x].quantity, currency: self.app.itemIds.get(items[x].currencyId + '') + ((items[x].currencyIsBlueprint) ? ' (BP)' : ''), cost: Math.round(items[x].costPerItem * items[x].priceMultiplier), stock: items[x].amountInStock, durability: Math.round(items[x].itemCondition / items[x].itemConditionMax * 100) });
                        }
                    }
                }
            }
            if (_items) {
                for (var x = items.length; x < _items.length; x++) {
                    var id = _items[x].itemId + '';
                    if (self.app.itemIds.has(id) && self.app.itemIds.has(_items[x].currencyId + '')) {
                        list.push({ status: 'Removed', item: self.app.itemIds.get(id) + ((_items[x].itemIsBlueprint) ? ' (BP)' : ''), quantity: _items[x].quantity, currency: self.app.itemIds.get(_items[x].currencyId + '') + ((_items[x].currencyIsBlueprint) ? ' (BP)' : ''), cost: Math.round(_items[x].costPerItem * _items[x].priceMultiplier), stock: _items[x].amountInStock, durability: Math.round(_items[x].itemCondition / _items[x].itemConditionMax * 100) });
                    }
                }
            }
            return list;
        },
            processItems = async function(title, name, list, loc_x, loc_y) {
            if (list.length > 0 && !(await self.app.getMonumentName(loc_x, loc_y))) {
                list.sort(function (a, b) {
                    return b.stock - a.stock;
                });
                var _list = [];
                for (var x = 0; x < list.length; x++) {
                    _list.push(((list[x].stock > 0) ? '+' : '-') + ' ' + list[x].item + ' x ' + list[x].quantity + ' | ' + list[x].currency + ' x ' + list[x].cost + ' (' + list[x].stock + ')' + ((list[x].durability > 0) ? ' [' + list[x].durability + '%]' : '') + ((list[x].status.length > 0) ? ' {' + list[x].status + '}' : ''));
                }
                if (_list.length > 0) {
                    var desc = name + '```diff\r\n' + _list.join('\r\n') + '```';
                    self.app.postDiscordNotification(title, desc);
                }
            }
        };
        if (self.vendingTask) clearInterval(self.vendingTask);
        self.vendingTask = setInterval(async function() {
            self.app.getMapMarkers(async (message) => {
                if (message.response && message.response.mapMarkers && message.response.mapMarkers.markers) {
                    if (self.mapMarkers.length > 0) {
                        var observed = [];
                        for (var i = 0; i < message.response.mapMarkers.markers.length; i++) {
                            if (message.response.mapMarkers.markers[i].type == 3) { // 'VendingMachine'
                                var prevMarker = getMapMarker(message.response.mapMarkers.markers[i].id, self.mapMarkers);
                                if (!prevMarker) {
                                    var items = processVending(message.response.mapMarkers.markers[i].sellOrders);
                                    await processItems('Added Vending Machine @ ' + message.response.mapMarkers.markers[i].location, message.response.mapMarkers.markers[i].name, items, message.response.mapMarkers.markers[i].x, message.response.mapMarkers.markers[i].y);
                                }
                                else if (prevMarker.sellOrders) {
                                    var items = processVending(message.response.mapMarkers.markers[i].sellOrders, prevMarker.sellOrders);
                                    await processItems('Changed Vending Machine @ ' + message.response.mapMarkers.markers[i].location, message.response.mapMarkers.markers[i].name, items, message.response.mapMarkers.markers[i].x, message.response.mapMarkers.markers[i].y);
                                }
                                observed.push(message.response.mapMarkers.markers[i].id);
                            }
                        }
                        for (var i = 0; i < self.mapMarkers.length; i++) {
                            if (observed.indexOf(self.mapMarkers[i].id) == -1) {
                                var items = processVending(self.mapMarkers[i].sellOrders);
                                await processItems('Removed Vending Machine @ ' + self.mapMarkers[i].location, self.mapMarkers[i].name, items, self.mapMarkers[i].x, self.mapMarkers[i].y);
                            }
                        }
                    }
                    self.mapMarkers = message.response.mapMarkers.markers;
                }
            });
        }, 30 * 1000); // 30 seconds
    };
}
this.vendingFunc();
```

> **Note:** After placing the *onConnected* code block in the plugin editor, you will need to press the Play button :arrow_forward: to simulate the event.

#### onDisconnected Event:

```js
console.log('onDisconnected Event');
if (this.vendingTask) {
    clearInterval(this.vendingTask);
    this.vendingTask = null;
}
```
