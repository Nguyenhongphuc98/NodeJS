# Events and Timers
## Event emitters
Những object dùng để tạo event gọi là event emitters.

```javascript
var events = require("events");
var emitter = new events.EventEmitter();
// should use camelCase for event name
emitter.emit("eventName", args);
```

### Listening for events

```javascript
var events = require("events");
var emitter = new events.EventEmitter();
var username = "colin";
var password = "password";
// an event listener
// only trigger the event emit after it added
emitter.on("userAdded", function(username, password) {
 console.log("Added user " + username);
});
// add the user
// then emit an event
emitter.emit("userAdded", username, password);

// only excute first emit
emitter.once("foo", function() {
    console.log("In foo handler");
});
emitter.emit("foo");
```

Get listener count
```javascript
var events = require("events");
var EventEmitter = events.EventEmitter;
// get the EventEmitter constructor from the events module
var emitter = new EventEmitter();
emitter.on("foo", function() {});
emitter.on("foo", function() {});

// node js define EventEmitter.listenerCount as class method (not static)
console.log(EventEmitter.listenerCount(emitter, "foo")); // output: 2
```
Get listener array, modify not effect origin
```javascript
var events = require("events");
var EventEmitter = events.EventEmitter;
var emitter = new EventEmitter();
emitter.on("foo", function() { 
    console.log("In foo handler"); 
});
emitter.listeners("foo").forEach(function(handler) {
    handler();
});
```

The newListener Event call when have new listener registered.

```javascript
var events = require("events");
var emitter = new events.EventEmitter();
emitter.on("newListener", function(eventName, listener) {
 console.log("Added listener for " + eventName + " events");
});
emitter.on("foo", function() {});
```

Note that this event is exists when we create ee.
You need careful when custome this event because alway have emit with 2 param: ```eventName, listener```.
```javascript
var events = require("events");
var emitter = new events.EventEmitter();
emitter.on("newListener", function(date) {
    console.log(date.getTime());
});
emitter.emit("newListener", new Date());

// this cause trigger 2 listenr with name "newListener"
// but one of theme access first params as date
// infact this just a function
emitter.on("foo", function() {}); // error
```

Removing Event Listeners
```javascript
emitter.removeAllListeners([eventName])
// handle must be a bound to a named reference (not asyanonymous)
emitter.removeListener("foo", handler);
```


Detecting Potential Memory Leaks
```javascript
// cause warning message if num of listeners reached
// set 0 is unlimit
emitter.setMaxListeners(n)
```
