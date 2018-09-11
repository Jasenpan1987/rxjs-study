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

## 1.1 Observable.forEach

```js
// subscribe
var mouseMove = Observable.fromEvent(element, "mousemove");
var subscription = mouseMove.forEach(
  // on data
  event => console.log(event),
  // on error
  error => console.error(error),
  // on complete
  () => console.log("done")
);

// unsubscribe
subscription.dispose();
```

Observerable pattern is like the generator pattern, but this time, the data sender has no control of how and when the data is sent, it simply throw data as much as it can, but the receiver has control of data and how much to receive.

## 1.2 Concert DOM event to observable

```js
Observable.fromEvent = function(dom, eventName) {
  // returning Observable object
  return {
    forEach: function(observer) {
      var handler = e => observer.onNext(e);
      dom.addEventListener(eventName, handler);

      // returning subscription object
      return {
        dispose: function() {
          dom.removeEventListener(eventName, handler);
        }
      };
    }
  };
};
```

## 1.3 Observable in action

Let's say `{1...2.....3}` is a valid javascript syntax, means observable over the time, and each dot represent to 10 second. So we can do the following

forEach

```js
{1...2.....3}.forEach(console.log) // 1, 2, 3
```

map

```js
{1...2.....3}
  .map(x => x + 1)
  .forEach(console.log) // 1, 2, 3
```

filter

```js
{1...2.....3}
  .filter(x => x > 1)
  .forEach(console.log) // 2, 3
```

concatAll

```js
{
  {1},
  ...{2.........3},
  ......{},
  .........{4},
}.concatAll()
// {1...2.........34}
```

If nothing subscribe an observable, nothing will happen.

## 1.4 Take until

```js
{...1...2........3}.takeUntil(
{..........4})
// {...1...2}
```

this stops the observer to taking data, just like `removeEventListner` on DOM element

Implement drag and drop

```js
var getElementDrags = elem =>
  elem.mouseDown
    .map(mouseDown => document.mouseMove.takeUntil(document.mouseUp))
    .concatAll();

var img = document.querySelector("#img-1");

getElementDrags(img).forEach(pos => (img.position = position));
```
