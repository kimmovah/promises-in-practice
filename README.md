# Promises in practice
A handful of examples on how to deal with common Promise related
problems in JS.


## Example 1: Making callback code return promises

```javascript
/** Wrap callback functions with a new Promise
 *
 * When dealing with code that uses callbacks for async
 * you need to wrap the callback function with a new 
 * promise instance and resolve/reject
 * it in the callback.
 *
 * Note: This is really the only time when you need to
 * use new Promise(). Once you have Promise instance,
 * it will always return another promise instance when
 * then() is called. You can just chain from that.
 * See examples below.
 */
function myPromiseReturningFn() {
  return new Promise((resolve, reject) => {
    doAsyncStuffWithCb((err, result) => {
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

// then() and catch() blocks pretty much work the same way as try/catch
// but with async.

```

## Example 2: Basic promise chaining

```javascript

/**
 * Let's assume a situation where we get a user
 * and then get the user's friends.
 *
 * Since calling then() on a promise always generates a new
 * promise, we can chain the two async functions nicely.
 */
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
then() block, return it and chain another then(). This avoids the nesting hell.

```javascript
// same functionality as above, but with nesting
getUser()
.then((user) => {
  // do another async action
  return getUsersFriends(user);
  .then((friends) => {
    // do something with friends data.
  });
});

// imagine you have 5 async calls you need to do in sequence. The nesting would be unmanageable.
```

## Example 3: Error handling
Promise.reject() will create a new Promise instance that is already rejected with the given value. We can use that to chain the error
handling.

```javascript
getUser()
.catch((err) => {
  log('getting user failed: ', err);
  return Promise.reject(err);
})
.then((user) => {
  return getUsersFriends(user)
  .catch((err) => {
    log('getting friends failed: ', err);
    return Promise.reject(err);
  })
})
.then((friends) => {
  // do something with friends data.
})
.catch((err) => {
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

```javascript
/**
 * Always return a Promise even if you can return synchronously,
 * so that the dev using your fn doesn't
 * have to worry about sync/async results.
 */
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

Getting multiple things at the same time.
```javascript
function getStuffAndThings() {
  let promises = [
    getStuff(),
    getThings()
  ];
  // Waits for all promises to be resolved.
  // Rejects if any promise in the array rejects.
  return Promise.all(promises);
}

getStuffAndThings()
.then(([stuff, things]) => {
  // do stuff with stuff and things
});
```
