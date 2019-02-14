---
title: Using Async and Await
status: 
layout: post
categories: [Redux, React, React-router]
tags: [Redux, React, React-router]
published: True
---

Promises - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise

Async - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
keyword `async` can be written infront of a function.
it signifies that this is function is asynchronous.
Although doesnt make sense to do, but i could have just a regular function and write async in front of it.
```
async function sandeep(){
	console.log("in sandeep");
};
sandeep();
```

An `async` function always returns a promise
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

Where `aysync` shines even more is when we use `await` in its body.

`Await` is a keyword we can apply before a promise and in the body of an `await` function. It will stop the flow of the function its in and will continue when the promise its awaiting on resolved. It returns the value the promises its awaiting on resolves with.

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

In the above function, normally the return statement exectues immediately. But because we have an await statement in there, it stops the flow, and then the return statement is only running after the awaited promises finish. Now we know that JS is single threaded and it cant block a function. So whats really happening is that an async function be default automatically returns a promise and then actual return is just resolving the promise with the its value.

Without `await` we can not access the implictly returned promise. therefore if we want to do this without an await, then we must declare our own promise and return that explicity.

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
Note again that in the function above I had to explictly return a promise because without `await` i had no way to access and resolve the implicity returned promise.
Also note, if we are going to by explicitly returning a promise and not using an `await`, then we dont have to use `async` keyword as its not doing anything for us.

Recap:
`async` makes the function implicitly return a promise. it also makes the value returned by the function be the value the implicit promise is resolved with.
`await` returns the value the promise its waiting on resolves with. it executes everything below it in the success wrapper called after the promise resolves. And remember that last value of the async function which is the resolved value of the implicit promise, it makes that return run after the promise its awaiting on finishes it in its success callback.