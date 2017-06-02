# Promises in practice
A handful of examples on how to deal with common Promise related
situations in JS. If you are not familiar with Promises at all or async in JS
this is a very good resource to start: [You don't know JS: Async & performance](https://github.com/getify/You-Dont-Know-JS/tree/master/async%20%26%20performance)

## Example 1: Making callback code return Promises

Wrap callback functions with a `new Promise()` to make old
callback code work with Promises.

When dealing with code that uses callbacks for async
you need to wrap the callback function with a new 
Promise instance and resolve/reject
it in the callback.

Note: This is really the only time when you need to
use `new Promise()`. Once you have a Promise instance,
it will always return another Promise instance when
`then()` is called. You can just chain from that.

```javascript
function myPromiseReturningFn() {
  return new Promise((resolve, reject) => {
    myCallbackFn((err, result) => {
      if (err) {
        reject(err);
      } else {
        resolve(result);
      }
    });
  });
}

myPromiseReturningFn()
.then((result) => {
  // do stuff with async results
})
.catch((err) => {
  //handle async error
});
```
`then()` and `catch()` blocks pretty much work the same way as try/catch
but with async.

## Example 2: Basic Promise chaining
Let's assume a situation where we get a user and then get the user's friends.
So it's two async calls in sequence.

Since calling `then()` on a Promise always generates a new
Promise, we can chain the two async functions nicely.

```javascript
getUser()
.then((user) => {
  // do another async action
  return getUsersFriends(user);
})
.then((friends) => {
  // do something with friends data.
});

```
Basic rule here is that when ever you have some async action within
`then()` block, return it and chain another `then()`. You should never see `then()` within another `then()`. Don't go into the pyramid of doom.

```javascript
// same functionality as above, but with nesting
getUser()
.then((user) => {
  // do another async action
  return getUsersFriends(user)
  .then((friends) => {
    // do something with friends data.
  });
});
```
Imagine you have 5 async calls you need to do in sequence. The nesting would be unmanageable. We would be back in the old callback pyramids.

## Example 3: Error handling
Promise.reject() will create a new Promise instance that is already rejected with the given value. We can use that to chain the error
handling.

```javascript
getUser()
.catch((userError) => {
  log('getting user failed: ', userError);
  // returning a rejection will be caught by the next catch() block.
  // then() blocks are skipped.
  return Promise.reject(userError);
})
.then((user) => {
  return getUsersFriends(user)
  .catch((friendsError) => {
    log('getting friends failed: ', friendsError);
    return Promise.reject(friendsError);
  })
})
.then((friends) => {
  // do something with friends data.
})
.catch((error) => {
  doCleanup();
});
```

## Example 4: Promise returning function with existing Promise

```javascript
function myAsyncFn() {
  return doSmthAsync()
  .catch((err) => {
    log(err);
    return Promise.reject(err);
  });
}

myAsyncFn()
.then((results) => {
  // do stuff with results
})
.catch((err) => {
  // handle error
});
```
Abstracting a larger chain

```javascript
function myAsyncFn() {
  return doSmthAsync()
  .then((results) => {
    return doSomeMoreAsync(results);
  })
  .then((moreResults) => {
    return evenMore();
  });
}

myAsyncFn()
.then((result) => {
  // do stuff
})
.catch((err) => {
  // handle error
});
```

## Example 5: Function returning both sync and async results
Always return a Promise even if you can return synchronously,
so that the dev using your API doesn't have to worry about sync/async results.
`Promise.resolve()` creates a Promise and resolves it immediately with any value passed to it.

```javascript
getStuff() {
  cachedResults = checkCache();
  if (cachedResults) {
    return Promise.resolve(cachedResults);
  }
  return getStuffFromServer()
  .then((stuff) => {
    cacheMyStuff(stuff);
    return stuff;
  });
}

getStuff()
.then((stuff) =>{
  // do stuff with stuff
});
```

## Example 6: Parallel async actions

Get multiple things at the same time.
```javascript
function getStuffAndThings() {
  let promises = [
    getStuff(),
    getThings()
  ];
  // Waits for all Promises to be resolved.
  // Rejects if any Promise in the array rejects.
  return Promise.all(promises);
}

getStuffAndThings()
.then(([stuff, things]) => {
  // do stuff with stuff and things
});
```

## Stuff to read

More great Promise examples:
[We have a problem with promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)

Asynchronous JavaScript in detail: [You don't know JS: Async & performance](https://github.com/getify/You-Dont-Know-JS/tree/master/async%20%26%20performance)

I highly recommend the whole "You don't know JS" series to anyone wanting
to know more about other weird JS mechanics and what you can encounter when you
work with JS code bases.

Also when you feel confident with Promises, check out async/await syntax (ES7 only though).
