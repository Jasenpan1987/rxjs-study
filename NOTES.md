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
var getElementDrags = elem => {
  elem.mouseDown = Observerble.fromEvent(elem, "mousedown");

  elem.mouseUp = Observerble.fromEvent(elem, "mouseup");

  elem.mouseMove = Observerble.fromEvent(elem, "mousemove");

  return elem.mouseDown
    .map(mouseDown => document.mouseMove.takeUntil(document.mouseUp))
    .concatAll();
};

var img = document.querySelector("#img-1");

getElementDrags(img).forEach(pos => (img.position = position));
```

## 1.5 MergeAll & switchLatest

mergeAll

```js
{
  ...{1}
  ......{2.........3}
  .........{}
  ............{4}
}.mergeAll();
// {...1...2......4...3}
```

- In concatAll, data is merged in sequence
- In mergeAll, data is merged in first come first serve order
- Both of them output data pieces as soon as the data arrived

switchLatest

```js
{
  ...{1}
  ......{2.........3}
  .........{}
  ............{4}
}.switchLatest();
// {...1...2..{4}}
```

- starts like mergeAll, takes data piece one of a time
- when it waiting for 3, it sees the next data stream, then it will automatically close the previous strem
- and again, it sees 4, the previous stream will be closed again
- it can be used to perform a cancel request

## 1.6 A search box example

Suppose we have a search box for searching movie names with auto complete, then we have the following hard part for auto completion:

1. there might be a race condition: we type in "A" and it send a request for "A" to the server, and we type "B", it send a request for "AB", what if the "AB" request come back earlier than the request of "A"

2. there might be a need for debounce: if a user types "ABCDE" in a very quick manner, there should only be one request sent out for "ABCDE"

```js
var keyPress = Observable.fromEvent(
  document.querySelector("#searchbox", "keypress")
);
var searchResultSets = keyPress
  .throttle(200)
  .map(
    input => getJSON(`/search?q=${input.value}`).retry(3)
    // .takeUntil(keyPress)
  )
  // .concatAll();
  .switchLatest();

searchResultSets.forEach(
  resultSets => updateSearchResults(resultSets),
  error => showErrorMessage("Server appears to be down")
);
```

In theory, everything can be mapped into 4 steps:

1. _What collections do I have?_

   - User type events

   - JSON from the server

2. _What collections do I want?_
3. _How can we convert the collections we have to the collections we want?_
4. _Once we got the collections we want, what do I going to do with them?_
