# Sockets Low Level
### Overview

Sails exposes several low-level methods for realtime communication with the client via `sails.sockets`.  These methods are implemented using a [Socket.io](http://socket.io) connection which is available as `sails.io`; however, using the `sails.sockets` methods instead will future-proof your app against possible changes in underlying implementation.  If your app is mainly sending messages to the client regarding changes in your models, you should try and use the [model PubSub methods](https://github.com/balderdashy/sails-docs/blob/0.10/reference/ModelMethods.md#publishcreate-datasocket-) instead.

# sails.sockets.join( `socket`, `roomName` )
### Purpose
Subscribe a socket to a generic room.

### Overview
#### Parameters
|   |          Description        | Accepted Data Types | Required ? |
|---|-----------------------------|---------------------|------------|
| 1 | Socket object |      `object`       | Yes        |
| 2 |           Room Name        | `string`            | Yes         |

### Example Usage
```javascript
// Controller action

subscribeToFunRoom: function(req, res) {
    var roomName = req.param('roomName');
    sails.sockets.join(req.socket, roomName);
    res.json({
      message: 'Subscribed to a fun room called '+roomName+'!'
    });
}
```

*Note: `req.socket` is only valid if the action is triggered via a socket request, e.g. `socket.get('/subscribeToFunRoom/someRoomName')`*

# sails.sockets.leave( `socket`, `roomName` )
### Purpose
Unsubscribe a socket from a generic room.

### Overview
#### Parameters
|   |          Description        | Accepted Data Types | Required ? |
|---|-----------------------------|---------------------|------------|
| 1 | Socket object |      `object`       | Yes        |
| 2 |           Room Name        | `string`            | Yes         |

### Example Usage
```javascript
// Controller action

leaveFunRoom: function(req, res) {
    var roomName = req.param('roomName');
    sails.sockets.leave(req.socket, roomName);
    res.json({
      message: 'Left a fun room called '+roomName+'!'
    });
}
```

*Note: `req.socket` is only valid if the action is triggered via a socket request, e.g. `socket.get('/leaveFunRoom/someRoomName')`*

# sails.sockets.broadcast( `roomName`, [`event`], `data`, [`socketToOmit`] )
### Purpose
Broadcast a message to a room.

### Overview
#### Parameters
|   |          Description        | Accepted Data Types | Required ? |
|---|-----------------------------|---------------------|------------|
| 1 |           Room Name        | `string`            | Yes         |
| 2 |           Event Name        | `string`            | No         |
| 3 |           Message Data        | `object`            | Yes         |
| 4 |           Socket to skip        | `object`            | No         |

 If the event name is omitted, `"message"` will be used by default.  Thus, `sails.sockets.broadcast(roomName, data)` is also a valid usage.
 
 If `socketToOmit` is provided, that socket will *not* receive the message.  This is useful if you trigger the broadcast from a client, but don't want that client to receive the message itself (for example, sending a message to everybody else in a chat room).

### Example Usage
```javascript
// Controller action

sayHiToFunRoom: function(req, res) {
    var room = req.param('roomName');
    sails.sockets.broadcast(room, 'chat', {msg: 'Hi there!', from: req.session.userId, room: room}, req.socket);
    res.json({
      message: 'Message sent!'
    });
}
```

```javascript
// Client-side -- subscribe to all "chat" events
socket.on('chat', function(data) {console.log("Chat from user #", data.from, " in room ", data.room, ": ", data.msg)});
```

# sails.sockets.blast( [`event`], `data`, [`socketToOmit`] )
### Purpose
Broadcast a message to all connected sockets.

### Overview
#### Parameters
|   |          Description        | Accepted Data Types | Required ? |
|---|-----------------------------|---------------------|------------|
| 1 |           Event Name        | `string`            | No         |
| 2 |           Message Data        | `object`            | Yes         |
| 3 |           Socket to skip        | `object`            | No         |

 If the event name is omitted, `"message"` will be used by default.  Thus, `sails.sockets.broadcast(roomName, data)` is also a valid usage.
 
 If `socketToOmit` is provided, that socket will *not* receive the message.

### Example Usage
```javascript
// Controller action

sayHiToEverybody: function(req, res) {
    sails.sockets.blast({msg: "User #"+req.session.userId+" logged on."}, req.socket);
    res.json({
      message: 'Message sent!'
    });
}
```

```javascript
// Client-side -- subscribe to all "message" events
socket.on('message', function(data) {console.log("Global message: ", data.msg});
```

# sails.sockets.id( `socket` )
### Purpose
Get the ID of a socket object.

### Overview
#### Parameters
|   |          Description        | Accepted Data Types | Required ? |
|---|-----------------------------|---------------------|------------|
| 1 |           Socket        | `object`            | Yes         |

A socket object's ID can be used to send direct messages to that socket (see `sails.sockets.emit`) or get information about the rooms that the socket is subscribed to (see `sails.sockets.socketRooms`).


### Example Usage
```javascript
// Controller action

socketId: function(req, res) {
    res.json({
      message: 'My socket ID is: '+sails.sockets.id(req.socket)
    });
}
```

# sails.sockets.emit( `socketIds`, [`event`], `data`)
### Purpose
Send a message to one or more sockets by ID.

### Overview
#### Parameters
|   |          Description        | Accepted Data Types | Required ? |
|---|-----------------------------|---------------------|------------|
| 1 |           IDs of sockets to receive message        | `string`, `array`            | Yes         |
| 2 |           Event name        | `string`            | No         |
| 3 |           Message data        | `object`            | Yes         |

### Example Usage
```javascript
// Controller action

sayHiToFriend: function(req, res) {
    var friendId = req.param('friendId');
    sails.sockets.emit(friendId, 'privateMessage', {from: req.session.userId, msg: 'Hi!'});
    res.json({
      message: 'Message sent!'
    });
}
```

# sails.sockets.subscribers( `roomName` )
### Purpose
Get the IDs of all sockets subscribed to a room.

### Overview
#### Parameters
|   |          Description        | Accepted Data Types | Required ? |
|---|-----------------------------|---------------------|------------|
| 1 |           Room name        | `string` | Yes         |

### Example Usage
```javascript
// Controller action

getRoomSubscribers: function(req, res) {
    var roomName = req.param('roomName');
    var subscribers = JSON.stringify(sails.sockets.subscribers());
    res.json({
      message: 'The subscribers are: '+subscribers
    });
}
```


# sails.sockets.socketRooms( `socket` )
### Purpose
Get the list of rooms a socket is subscribed to

### Overview
#### Parameters
|   |          Description        | Accepted Data Types | Required ? |
|---|-----------------------------|---------------------|------------|
| 1 |           Socket        | `object` | Yes         |

### Example Usage
```javascript
// Controller action

getMyRooms: function(req, res) {
    var roomNames = JSON.stringify(sails.sockets.socketRooms(req.socket));
    res.json({
      message: 'I am subscribed to: '+roomNames
    });
}
```



# sails.sockets.rooms()
### Purpose
Get the list of all current socket rooms

### Overview
#### Parameters
None.
### Example Usage
```javascript
// Controller action

getRoomsList: function(req, res) {
    var roomNames = JSON.stringify(sails.sockets.rooms());
    res.json({
      message: 'A list of all the rooms: '+roomNames
    });
}
```

*Note: In Socket.io, all sockets are automatically subscribed to a global room with an empty name ('').  This room is not returned as part of the array in `sails.sockets.rooms`* 
