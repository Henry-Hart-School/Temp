# Documentation

* [Class: `P2PT extends EventEmitter`](#class-p2pt-extends-eventemitter)
  * [Event: `peerconnect`](#event-peerconnect)
  * [Event: `data`](#event-data)
  * [Event: `msg`](#event-msg)
  * [Event: `peerclose`](#event-peerclose)
  * [Event: `trackerconnect`](#event-trackerconnect)
  * [Event: `trackerwarning`](#event-trackerwarning)
  * [`new P2PT(announceURLs = [], identifierString = '')`](#new-p2ptannounceurls---identifierstring--)
  * [`setIdentifier(identifierString)`](#setidentifieridentifierstring)
  * [`start()`](#start)
  * [`requestMorePeers()`](#requestmorepeers)
  * [`send(peer, msg[, msgID = ''])`](#sendpeer-msg-msgid--)
  * [`destroy()`](#destroy)

## Class: `EasyMsg`
To use `EasyMsg`, you must have `EasyMsg.js` imported. It is defined as a global variable.

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

### Event: `peerconnect`
This event is emitted when a new peer connects.

Arguments passed to Event Handler: `peer` Object

### Event: `data`
This event is emitted for every chunk of data received.

Arguments passed to Event Handler: `peer` Object, `data` Object

### Event: `msg`
This event is emitted once all the chunks are received for a message.

Arguments passed to Event Handler: `peer` Object, `msg` Object

### Event: `peerclose`
This event is emitted when a peer disconnects.

Arguments passed to Event Handler: `peer` Object

### Event: `trackerconnect`
This event is emitted when a successful connection to tracker is made.

Arguments passed to Event Handler: `WebSocketTracker` Object, `stats` Object

### Event: `trackerwarning`
This event is emitted when some error happens with connection to tracker.

Arguments passed to Event Handler: `Error` object, `stats` Object

### `new P2PT(announceURLs = [], identifierString = '')`
Instantiates the class
* **Arguments:**
  * **announceURLs:** `Array`
    * **Description:** List of announce tracker URLs
    * **Default:** `[]`
  * **identifierString:** `String`
    * **Description:** Identifier used to discover peers in the network
    * **Default:** `''`

```javascript
// Find public WebTorrent tracker URLs here : https://github.com/ngosang/trackerslist/blob/master/trackers_all_ws.txt
const trackersAnnounceURLs = [
  "wss://tracker.openwebtorrent.com",
  "wss://tracker.sloppyta.co:443/",
  "wss://tracker.novage.com.ua:443/",
  "wss://tracker.btorrent.xyz:443/",
]

// This 'myApp' is called identifier and should be unique to your app
const p2pt = new P2PT(trackersAnnounceURLs, 'myApp')
```

In Typescript, the `P2PT` class accepts an optional type parameter to constrain the type of messages you can pass to the `send` function. It doesn't constrain the type of messages you recieve, since any peer *could* send anything.
```typescript
type Msg = 'hello' | { goodbye: boolean }
const p2pt = new P2PT<Msg>(trackersAnnounceURLs, 'myApp')
// ... find a peer ...
p2pt.send(peer, 'some_message') // TS typecheck error: Argument of type 'string' is not assignable to parameter of type 'Msg'.
p2pt.send(peer, 'hello') // ok!
p2pt.send(peer, { goodbye: true }) // ok!
```

### `setIdentifier(identifierString)`
Sets the identifier string used to discover peers in the network
* **Arguments:**
  * **identifierString:** `String`
    * **Description:** Identifier used to discover peers in the network
* **Returns:** `void`

### `requestMorePeers()`
Request More Peers
* **Arguments:** None
* **Returns:** `Promise`
  * **resolve(peers)**
    * **peers:** Object

### `send(peer, msg[, msgID = ''])`

* **Arguments:**
  * **peer:** `Object`
    * **Description:** Stores information of a Peer
  * **msg:** `Object`
    * **Description:** Message to send
  * **msgID:** `Number`
    * **Description:** ID of message if it's a response to a previous message. You won't need to pass this
    * **Default:** `''`
* **Returns:** `Promise`
  * **resolve([peer, msg])**
    * **peer:** `Object`
    * **msg:** `Object`

### `destroy()`
Destroy the P2PT Object
* **Arguments:** None
* **Returns:** `void`
