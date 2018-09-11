# Introduction

All of the following api can be adapted to observables

- DOM event
- web socket
- Server-sent event
- Node Stream
- Service worker
- jQuery event
- xhr
- setInterval

Event Subscription

```js
var handler = e => console.log(e);

// subscribe
document.addEventListener("mousemove", handler);

// unsubscribe
document.removeEventListener("mousemove", handler);
```

Observable.forEach

```js
// subscribe
var subscription = mouseMove.forEach(console.log);

// unsubscribe
subscription.dispose();
```
