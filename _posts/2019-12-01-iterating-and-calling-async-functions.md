---
title: Iterating and calling an async function
status: True
layout: post
categories: [javascript, async, await]
tags: [javascript, async, await]
published: True
---

A Recap on `async` and `await`:

- `async` next to the function definition makes the function return an implicit `promise` and make the explicitly returned value, be the value the promise is resolved with. 
- Separately if you want to use the function's resolved value, then it can be called with an `await`. 
- Now to  call something in an `await`, you yourself, must be in an `async` function
- when you call an `async` function without `await`, the inside function will just return a promise. as that is what `async` does.
- without an `await` on the outside function, you are getting a promise and now you need to manually wrap all following lines in the success callback of the promise. 
- This is something you didnt need to do had you used `await` since it automatically assigns the resolved value to the variable on its left.

A simple asynchronous function:

```
const giveNumber = () => {
  return new Promise(((resolve, reject) => {
    setTimeout(() => {
      resolve(Math.round(Math.random() * 10));
    }, 1000);
  }));
};
```

It works as expected. It gives a promise and we can get the resolved value in the success callback.

```
giveNumber().then(d => console.log(d));
```

We can `await` on the function and get the resolved value automatically assigned to the variable on the left to the `await` keyword.

```
(async () => {
  const answer = await giveNumber();
  console.log(answer);
})();
```

We can call the asynchronous function in a for loop. Since we call the function with an `await` we get the resolved value and our array has all N resolved values.

```
const result = [];
(async () => {
  let answer = undefined;
  for(var i=0;i<5;i++){
    answer = await giveNumber();
    console.log(answer);
    result.push(answer);
  }
})();
console.log(result);
```

Things go very different when we use `map` to iterate call the asynchronous function.
```
var answers = [];
(async () => {
  answers = [1,2,3,4,5].map(async (i) => {
    const answer = await giveNumber();
    console.log(answer);
    return  answer;
  });
})();
console.log(answers);
``` 

Here is what happened:
1. The result is now an Array of Promises, not an Array of strings.
2. map is going to call the callback function normally instead of calling them with an await. this means that map will finish immediately and results will come afterwards asynchronously when they finish.
3. the function runs 5 times immediately. the 2nd call no longer waits for the first one to finish.

To Solve this problem we can just wait for these 5 promises to finish and then get the values we need in the success callback. This can be made further simpler, by wrapping these 5 small promises in to 1 grand promise which comprises of these 5 promises:

```
async function foo() {
  const result = [1,2,3,4,5].map(async (i) => {
    const answer = await giveNumber();
    console.log(answer);
    return  answer;
  });
  console.log('i print before the answers are printed')
  return await Promise.all(result);
}

foo().then(x => console.log(x));
```

Another issue, is the line after the map function call. Since the callback function is not called with an await by map, it doesnt block the remainder of the function till the asynchronous function resolves. 

This means we end up see'ing the print statement after the map function immediately after the map function. it does not wait for the promises to resolve.

```
i print before promises resolve
Promise {<pending>}
2VM7221:4 6
VM7221:4 1
VM7221:4 8
VM7221:4 10
```
 