---
layout: post
title: Promises in Javascript
comments: true
---

> Handling asynchronous tasks in web development is very common. One way to handle this is through promises.

Let’s say that we have the following code in that we are running multiple print commands
```javascript
console.log('step 1');
console.log('step 2');
console.log('step 3');
console.log('step 4');
console.log('step 5');
console.log('step 6');
console.log('step 7');
console.log('step 8');
```

The way the javascript interpreter works is that it runs the code line by line. And in the above example if any of the steps is taking an abnormal amount of time then javascript will wait for that step to finish before moving on to the next step. Now let's say step 5 is taking 10 minutes to finish then our application will be stuck for 10 minutes before moving on to the next step.

Now here come the promises. Promises allow you to write asynchronous code so the promises can take the long-running task out of our main execution thread and put them and run them in parallel and they will resolve and give us the value whenever they are finished without blocking the main thread.

## How to create promise?
To create a promise we use the built-in javascript promise constructor. So let's create a promise.

```javascript
new Promise((resolve, reject) => {
    // ...
    // ...
    // ...
});
```
To define a promise we use `new Promise` and pass it a callback that will be the executor where all our logic will be. And this callback accepts two parameters first is `resolve` and the second is `reject`. So you call the `resolve` method when everything goes well, and the promise was successful, and you want to return the value. Where `reject` is called when a failure happens or the operation was not successful due to some errors. 

So let's simulate an API call where we will wait for two seconds, and then we will return some dummy users.

```javascript
const getUsers = new Promise((resolve, reject) => {
    window.setTimeout(() => {
        resolve([
            'john',
            'doe',
        ]);  
    }, 2000);
});
```  

Now when you will run the above code the `getUsers` will contain the promise. And this promise will have three methods. 
- then
- catch
- finally

```javascript
getUsers
    .then((users) => {
        console.log(users); // ['john', 'doe']
    })
    .catch((error) => {
        console.log(error);   // error...
    })
    .finally(() => {
        console.log('promise settled');    // promise settled
    });
``` 

`then` is called when the promise is successful. So whenever you will call the `resolve` method or whatever you will pass to it will be passed to `then` callback. As in the above example, the `then` method after two seconds received the list of users that was passed to the `resolve` method.     

Similar to `then` there's `catch` method. It will receive errors. When you will create and execute the above promise you will not get any error. So if we update the above example and replace `resolve` with `reject` like this.

```javascript
const getUsers = new Promise((resolve, reject) => {
    window.setTimeout(() => {
        reject('failed to get users');  
    }, 2000);
});
```    

Now on executing the above promise you will see an error after two seconds.
 
The third method in the promised object is `finally`. Which is called irrespective of the success or failure of the promise. Now try running the above promises with `resolve` and `reject`. You will get `promise settled` in both cases after the execution of the `then` or `catch`.

## States of promise
A promise can in one of the four states at any point in time.

**Pending State** When a promise is yet to be executed.

**Resolved State** Promise was successful and `resolve` is called. In this case the `then` callback is called.

**Rejected State** Promise action is failed and `reject` is called. In this case, the `catch` callback is called.

**Settled State** Promise was either successful or failed. 

## Multiple promises
You can also run multiple promises in parallel. There are different ways to achieve that where each method behaves differently. 

### `Promise.all`
Wait until all promises are resolved or any of them is rejected.

```javascript
Promise.all([
  // wait two seconds and resolve
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("Promise 1");
    }, 2000);
  }),
  // wait three seconds and resolve
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("Promise 2");
    }, 3000);
  }),
])
  .then((data) => {
    console.log(data); // ['Promise 1', 'Promise 2']
  })
  .catch((error) => {
    console.log(error);
  });
```
 
### `Promise.allSettled`
Unlike `Promise.all` it doesn't fail on the failure of any of the promises and returns all the results with promise status. 

```javascript
Promise.allSettled([
  // wait two seconds and resolve
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("Promise 1");
    }, 2000);
  }),
  // wait one second and reject
  new Promise((resolve, reject) => {
    setTimeout(() => {
      reject("Promise 2");
    }, 1000);
  }),
])
  .then((data) => {
    console.log(data); // [{status: "fulfilled", value: "Promise 1"}, {status: "rejected", reason: "Promise 2"}]
  })
  .catch((error) => {
    console.log(error);
  });
```

### `Promise.race`
Wait until any of the promises is resolved or rejected. In the example, it will print `Promise 2` as it's the first one to reject.

```javascript
Promise.race([
  // wait two seconds and resolve
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("Promise 1");
    }, 2000);
  }),
  // wait one second and reject
  new Promise((resolve, reject) => {
    setTimeout(() => {
      reject("Promise 2");
    }, 1000);
  }),
])
  .then((data) => {
    console.log(data);
  })
  .catch((error) => {
    console.log(error); // Promise 2
  });
```

### `Promise.any`
Returns a single promise as soon as any of the promise from the list is fulfilled.

```javascript
Promise.any([
  // wait two seconds and resolve
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("Promise 1");
    }, 2000);
  }),
  // wait one second and resolve
  new Promise((resolve, reject) => {
    setTimeout(() => {
      reject("Promise 2");
    }, 1000);
  }),
])
  .then((data) => {
    console.log(data); // Promise 1
  })
  .catch((error) => {
    console.log(error);
  });
```

And that wraps it up for this article. Feel free to leave your feedback or questions in the comments section.
