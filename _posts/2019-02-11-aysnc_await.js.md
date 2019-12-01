---
title: Using Async and Await
status: 
layout: post
categories: [Redux, React, React-router]
tags: [Redux, React, React-router]
published: True
---

[Promises Mozilla Documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

[Async Mozilla Documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)

keyword `async` can be written in front of any function and it signifies that this is function is asynchronous i.e. it returns a promise.

Although it doesnt make sense to do, but i could have just a regular function and write async in front of it.
```
async function sandeep(){
	console.log("in sandeep");
};
sandeep();
```

An `async` function **always** returns a promise
```
async function sandeep(){
	console.log("in sandeep");
};
sandeep().then(
(e) => {
	console.log("done", e);
}
);
```

The value this function returns is the value passed in to the `success` handler of the `then`.
```
async function sandeep(){
	console.log("in sandeep");
  return 5;
};
sandeep().then(
(e) => {
	console.log("done", e);
}
);
```

So what is `async` getting us?
Well it makes the function return a promise. And makes that promise resolved with the actual value the function returns.
So its a magic keyword. In the function above we are normally returning `5`. But by adding `async` in the begining, it changes to return a promise and resolve that promise with `5`.
 
Now there is more to it. `aysync` does another awesome thing and that is, it allowing us to use `await` in its body. `await` can only be used in a function which has been marked `async`

`Await` is a keyword we can write before the call to a function which returns a promise and in the body of an `async` function. It will stop the flow of the function its in and will continue when the promise its awaiting on resolved. It returns the value the promises its awaiting on resolves with.

```
async function sandeep() {
  console.log("sandeep");
  var a = 0, b = 0;
  a = await Promise.resolve(2);
  b = await Promise.resolve(3);
  return a + b;
}
sandeep().then(r => console.log(r)); // 5
```

In the above function, normally the return statement executes immediately. But because we have an `async` function it is going to return a promise instead. Now the `await` statement in there stops the flow, and then the return statement runs after the awaited promises finish. 

We keep saying that `await` stops the flow. That is not true. We know that JS is single threaded and it can't/should-not block a function. `await` merely wraps everything below it in the success callback of the promise its waiting on and then assigns the value its promise resolves with to the variable on the left of the `await` statement.

Without `async` we can not access the implicitly returned `promise`. therefore if we want to do this without an `async`, then we must 1.) declare our own `promise` and return that explicitly. Now since we didnt `async`, we can't even leverage `await`, this means that we also need to 2.) wrap all statements below functions which return promise in callbacks.
```
function sandeep() {
  console.log("sandeep");
  var a = 0, b = 0;
  var p = new Promise(resolve => {
    Promise.resolve(2).then(
    r => {
      a = r;
      return Promise.resolve(3);
    }).then(r => {
      b = r;
    }).then ( () => {
      resolve(a + b);
    })
  });
  return p;
}
sandeep().then(r => console.log(r)); // 5
```
Note again that in the function above I had to explicitly return a promise because without `await` i had no way to access and resolve the implicit returned promise.

Also note, if we are going explicitly return a promise and not using an `async`, then we dont have to use `await` keyword as it cant be used in normal function

Recap:

- `async` makes the function implicitly return a promise. it also makes the value returned by the function be the value the implicit promise is resolved with.

- `await` returns the value the promise its waiting on resolves with. it executes everything below it in the success wrapper called after the promise resolves. And remember that last value of the async function which is the resolved value of the implicit promise, it makes that return run after the promise its awaiting on finishes it in its success callback.