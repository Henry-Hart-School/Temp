# Documentation

## Class: `EasyMsg`
To use `EasyMsg`, you must have `EasyMsg.js` imported. It is defined as a global variable. This is the core library.

### `constructor(configuration, autoConfig, debug)`
*Client:* Sends out offer and waits for server to respond and connect.

*Server:* Starts listening for client offers and connects to any identified clients.
* **Arguments:**
  * **configuration:** `ConnectConfig`
    * **Description:** Specifies library configuration.
    * **Default:** `undefined`
  * **autoConfig:** `Bool`
    * **Description:** Automatically configures the library if configuration is undefined.*
    * **Default:** `true`
  * **debug:** `Bool`
    * **Description:** Enables debug mode, which makes the library much more verbose with console.log().
    * **Default:** `false`
* **Returns:** `void`
* **Example:**
```js
// without default values specified
const normal_lib = new EasyMsg()

// with default values specified
const same_as_normal_lib = new EasyMsg(null, true, false)

// configuration: null, autoConfig: true, debug: true
const debug_lib = new EasyMsg(null, true, true)

const config = new ConnectConfig()
config.signallingCustomFunction = some_custom_signalling_function
config.iceConfig = something_quirky
const custom_lib = new EasyMsg(config)
```
* **Notes**: *Or any equivalent `false` value.

### `async start(connectCallback)`
*Client:* Sends out offer and waits for server to respond and connect.

*Server:* Starts listening for client offers and connects to any identified clients.
* **Arguments:**
  * **connectCallback:** `Function`
    * **Description:** Callback for new connections.
    * **Default:** `undefined`
* **Returns:** `Promise`
* **Example:**
```js
const config = new ConnectConfig()
config.signallingCustomFunction = your_signalling_function
const lib = new EasyMsg(config) // scope of current browser

function new_peer(peer) {
  console.log("new peer with ID " + peer.ID)
}

lib.type = "server" // comment this line out and run in another tab
await lib.start(new_peer)
console.log("lib has started; waiting for new peers")
```
* **Notes**: `connectCallback` is a function that takes one argument, `peer`, which represents the newly connected peer. The `start` function is asynchronous and thus awaitable.

### `connectById(id)`
*Server:* Sets an id to connect to. Only clients with this id will have connections made to them.
* **Arguments:**
  * **id:** `String`
    * **Description:** Id to connect to.
    * **Default:** `undefined`
* **Returns:** `void`
* **Example:**
```js
// only connect to unhappy clients
lib.connectById(":(")
```

### `connectByPattern(regex)`
*Server:* Sets a mask to connect to. Only clients that match will have connections made to them.
* **Arguments:**
  * **regex:** `RegExp`
    * **Description:** New mask.
    * **Default:** `undefined`
* **Returns:** `void`
* **Example:**
```js
// only connect to clients with 4-letter names
lib.connectByPattern(/hello my name is ....!/)
```

### `connectToAll()`
*Server:* Makes EasyMsg attempt to connect to all clients, regardless of id.
* **Returns:** `void`
* **Example:**
```js
// connect without filter
lib.connectToAll()
```

### `disconnectFromId(id)`
*Client:* Disconnects from server.

*Server:*  Disconnects from specified id.
* **Arguments:**
  * **id:** `String`
    * **Description:** Id to disconnect from.
    * **Default:** `undefined`
* **Returns:** `void`
* **Example:**
```js
// disconnect from the King of Babylon
lib.connectById("Nebuchadnezzar")
```

### `disconnectFromAll()`
*Client:* Disconnects from server.

*Server:*  Disconnects from all clients.
* **Returns:** `void`
* **Example:**
```js
// shutdown
lib.disconnectFromAll()
// shutdown the signaller to avoid trying to call a library callback
signaller.shutdown()
// alternatively, clear the signal callback
const signal = lib.configuration.signallingCustomFunction
signal(()=>{}, signallingAction.OnReceive)
// safely remove lib from memory
delete lib
```

### `sendMessage(msg, peer)`
*Client:* Sends message to specified peer (server). Clients should not use this function; use `broadcastMessage` instead.

*Server:*  Sends message to specified peer (client).
* **Arguments:**
  * **msg:** `String`
    * **Description:** Message to send.
    * **Default:** `undefined`
  * **peer:** `Peer`
    * **Description:** Peer to send message to.
    * **Default:** `undefined`
* **Returns:** `Bool` indicating successful message send.
* **Example:**
```js
const config = new ConnectConfig()
config.signallingCustomFunction = your_signalling_function
const lib = new EasyMsg(config) // scope of current browser

function new_peer(peer) {
  lib.sendMessage("you have ID: " + peer.ID, peer)
}

lib.type = "server" // comment this line out and run in another tab
if(lib.type == "client") {
  lib.addReceiveMessageCallback(m=>console.log(m.data))
}
await lib.start(new_peer)
console.log("lib has started; waiting for new peers")
```

### `broadcastMessage(msg)`
*Client:* Sends message to server.

*Server:*  Sends message to all clients.
* **Arguments:**
  * **msg:** `String`
    * **Description:** Message to send.
    * **Default:** `undefined`
* **Returns:** `Bool` if client, indicating successful message send; `void` if server.
* **Example:**
```js
// tell everyone to drink water
lib.broadcastMessage("drink water please")
```

### `addReceiveMessageCallback(msgCallback)`
Sets the receive message callback; enables library to receive other peer's messages.
* **Arguments:**
  * **msgCallback:** `Function`
    * **Description:** Message callback that will be set.
    * **Default:** `undefined`
* **Returns:** `void`
* **Example:**
```js
const config = new ConnectConfig()
config.signallingCustomFunction = your_signalling_function
const lib = new EasyMsg(config) // scope of current browser

function new_peer(peer) {
  lib.sendMessage("you have ID: " + peer.ID, peer)
}

lib.type = "server" // comment this line out and run in another tab
if(lib.type == "client") {
  lib.addReceiveMessageCallback(m=>console.log(m.data))
}
await lib.start(new_peer)
console.log("lib has started; waiting for new peers")
```

### `autoConfig(setConfig)`
Turns autoConfig on/off.
* **Arguments:**
  * **setConfig:** `Bool`
    * **Description:** `true`: autoConfig turned on; `false`: autoConfig turned off.
    * **Default:** `undefined`
* **Returns:** `void`
* **Example:**
```js
// initially create library with autoConfig
const lib = new EasyMsg()
// decide to manually set configuration
lib.autoConfig = false
lib.configuration = new ConnectConfig()
lib.iceConfig = something_custom
```

### `peers`
*Client:* Unused.

*Server:* An array of all clients (that have been identified by the signalling channel, so not necessarily connected ones).
* **Type:** `Array`
* **Example:**
```js
// set up lib, start server
// ...

// log all connected clients after 10 minutes
setTimeout(()=>console.log(lib.peers.filter(peer=>peer.connected)), 60 * 10 * 1000)
```


### `peer_id_map`
*Client:* A map containing either zero entries or one entry: the server, identified by *"SERVER"*.

*Server:* A map containing all peers that have previously connected. Often more efficient for finding a peer from an ID than iterating through `peers`.
* **Type:** `Object`
* **Example:**
```js
// set up lib
// ...

// disconnect the peer with ID 12345678, if it exists
const disconnect_ID = 12345678
if (lib.peer_id_map[disconnect_ID]) {
  lib.disconnectFromID(disconnect_ID)
}
```

## Class: `Peer`
This class is imported with `EasyMsg`. This represents an individual peer (client/server).

### `con`
The Peer's RTCPeerConnection object.
* **Type:** `RTCPeerConnection`
* **Example:**
```js
const connectionCallback = async peer => {
  // get some info about newly connected peers
  console.log(await peer.con.getStats())
}
```

### `channel`
The Peer's RTCDataChannel object.
* **Type:** `RTCDataChannel`
* **Example:**
```js
const connectionCallback = async peer => {
  // log the channel for further inspection
  console.log(peer.channel)
}
```

### `debug`
Determines whether the peer is in debug mode, being more verbose with console.log().
* **Type:** `Bool`
* **Example:**
```js
// debug mode library
const lib = new EasyMsg(null, true, true)

// ...

const connectionCallback = peer => {
  // only need to debug lib, not peers
  peer.debug = false
}
```

### `connected`
Specifies whether the peer is connected or not.
* **Type:** `Bool`
* **Example:**
```js
const connectionCallback = peer => {
  // check whether we are connected every second
  peer.__tmp_interval = setInterval(()=>{
    if(!peer.connected) {
      console.log("LOST CONNECTION")
      clearInterval(peer.__tmp_interval)
    }
  }, 1000);
}
```

## Class: `ConnectConfig`
This class is imported with `EasyMsg`. This is used for the configuration of `EasyMsg`.

### `iceConfig`
Configuration of RTCPeerConnection object.
* **Type:** `Object`
* **Example:**
```js
const config = new ConnectConfig()
config.iceConfig = {
    iceServers: [
        { // google STUN server
          urls: "stun.l.google.com:19302"
        }
    ],
};
const lib = new EasyMsg(config)
```

### `signallingCustomFunction(callback, action, data, repeat)`
Function used to control signalling. This is dynamically set by the developer (you). Alternatively, use `InbuiltSignalling`.
* **Arguments:**
  * **callback:** `Function`
    * **Description:** Callback for reception of offers/answers along signalling channel. This callback is set when *signallingAction.OnReceive* is specified.
    * **Default:** `undefined`
  * **action:** `signallingAction int`
    * **Description:** The action to perform. Can be either *signallingAction.Send* or *signallingAction.OnReceive*.
    * **Default:** `undefined`
  * **data:** `Object`
    * **Description:** Data to transmit over signalling channel. This is used when *signallingAction.OnReceive* is specified.
    * **Default:** `undefined`
  * **repeat:** `Bool`
    * **Description:** Specifies whether to repeatedly transmit data over signalling channel, at an interval you choose. This is most commonly used for clients to repeatedly send out their offers for newly added servers to detect them.
    * **Default:** `undefined`
* **Returns:** `void`.
* **Example:**
```js
// this uses a made-up 'channel' object
// create an interface for 'channel'
function signal_func(callback, action, data, repeat) {
    if (action === signallingAction.Send) {
        // when we are told to send data...
        const send_func = () => {
            // ...send data over channel
            channel.send(data)
        }
        if(repeat) setInterval(send_func, 60 * 1000);
        send_func()
    } else if (action === signallingAction.OnReceive) {
        // when we receive data...
        channel.onreceive = data => {
            // ...call the callback
            callback(data)
        }
    } else {
        // do nothing, invalid signallingAction specified
    }
}
const config = new ConnectConfig()
config.signallingCustomFunction = signal_func
const lib = new EasyMsg(config)
```

## Enum: `signallingType`
This specifies different signalling methods. This is imported with `InbuiltSignalling`.
* Unset: -1
  * Signalling type not specified.
* WebTorrent: 0
  * Not implemented.
* OpenHttp: 1
  * Not implemented.
* OpenTCP_UDP: 2
  * Not implemented.
* Firebase: 3
  * Uses Firebase Realtime Database as signalling channel.
* ScratchSW: 4
  * Not implemented.
* Relay: 5
  * Not implemented.
* Custom: 6
  * Not implemented with `InbuiltSignalling`. Custom signalling implementations should directly interface with *ConnectConfig.signallingCustomFunction*.

## Enum: `signallingAction`
This specifies different signalling actions. This is imported with `InbuiltSignalling`.
* Send: 0
  * Tells *ConnectConfig.signallingCustomFunction* to send data.
* OnReceive: 1
  * Tells *ConnectConfig.signallingCustomFunction* to set a data reception callback.
* Find: 2
  * Not implemented.
* Ask: 3
  * Not implemented.
* Answer: 4
  * Not implemented.

## Class: `InbuiltSignalling`
To use `InbuiltSignalling`, you must have `signalling.js` imported. It is defined as a global variable. This class is a signalling wrapper.

### `constructor(signallingType)`
Initialises `InbuiltSignalling`.
* **Arguments:**
  * **signallingType:** `signallingType int`
    * **Description:** Specifies signalling type (only *signallingType.Firebase* currently supported).
    * **Default:** `signallingType.Unset`
* **Returns:** `void`
* **Example:**
```js
const firebaseConfig = {
    databaseURL: "https://somename.somewhere.firebasedatabase.app"
}

// set up signalling
const signaller = new InbuiltSignalling(signallingType.Firebase)
await signaller.start(firebaseConfig)
config.signallingCustomFunction = signaller.signallingCustomFunction

// start lib
const lib = new EasyMsg(config, true, true)
```

### `async start(options)`
Starts the signaller.
* **Arguments:**
  * **options:** `Object`
    * **Description:** Further options for the signaller. For *signallingType.Firebase*, these specify the *firebaseConfig* passed into *firebase.initialiseApp* (API [here](https://firebase.google.com/docs/database/web/start#web-namespaced-api)).
    * **Default:** `undefined`
* **Returns:** `Promise`
* **Example:**
```js
const firebaseConfig = {
    databaseURL: "https://somename.somewhere.firebasedatabase.app"
}

// set up signalling
const signaller = new InbuiltSignalling(signallingType.Firebase)
await signaller.start(firebaseConfig)
config.signallingCustomFunction = signaller.signallingCustomFunction

// start lib
const lib = new EasyMsg(config, true, true)
```


### `fakeSigChannel`
Captures all signalling traffic (that goes through `InbuiltSignalling`). This is useful for debugging.
* **Type:** `SignallingChannel_Fake`
* **Example:**
```js
// set up lib
// ...

// log all sent messages after 10 minutes
setTimeout(()=>console.log(signaller.fakeSigChannel.sent), 60 * 10 * 1000)
```

## Class: `SignallingChannel_Fake`
This class is imported with `InbuiltSignalling`. This is used for the configuration of `EasyMsg`.

### `sent`
A live capture of all sent traffic (by the current tab) over the signalling channel.
* **Type:** `Array`
* **Example:**
```js
// set up lib
// ...

// log all sent messages after 10 minutes
setTimeout(()=>console.log(signaller.fakeSigChannel.sent), 60 * 10 * 1000)
```


### `received`
A live capture of all received traffic (by the current tab) over the signalling channel.
* **Type:** `Array`
* **Example:**
```js
// set up lib
// ...

// log all received messages after 10 minutes
setTimeout(()=>console.log(signaller.fakeSigChannel.received), 60 * 10 * 1000)
```
