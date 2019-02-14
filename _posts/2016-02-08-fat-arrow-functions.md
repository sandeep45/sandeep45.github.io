---
layout: post
title: Fat Arrow Functions
categories: [javascript, es6, functions]
tags: [javascript, es6, functions]
published: True

---

## Background

The fat arrow notation to defined functions in JavaScript is **not** just syntactical sugar. It adds both convenience in terms of writing and reading code, and also changes how the function is logically executed.

## Syntax

Lets start with a basic and nostalgic example of "hello world".

In the existing function syntax:

````
var say = function(){
  console.log("hello world");
}
say();
````

With Fat arrows this becomes:

````
var say2 = () => {
 console.log("hello world");
}
say2();
````

Now lets add a parameter, to the function:

````
var say = function(word){
  console.log(`hello ${word}`);
}
say("NY");
````

With Fat arrows this becomes:

````
var say2 = word => {
 console.log(`hello ${word}`);
}
say2("NY");
````

Since it was only 1 parameters, we didn't have to put parenthesis around the parameter. When we have more than 1 parameter, then parenthesis are a requirement.

````
var say2 = (word1, word2) => {
 console.log(`hello ${word1} and ${word2}`);
}
say2("NY, NJ");
````

Now lets return a single value:

````
var say = function(word){
  return `hello ${word}`;
}
var result = say("NY");
console.log("result is: ", result);
````

And with fat arrow we get an automatic return:

````
var say = (word) => `hello ${word}`;
var result = say("NY");
console.log("result is: ", result);
````

Since we used the concise format where the body has no braces and is only one line, then the value of expression is automatically returned. This can be useful when writing a callback function for a function like `map`, `filter` etc.

````
var names = ["yo yo honey singh", "pankaj udaas"];
var lengths = names.map(n => n.length); // [17 , 12]
console.log(lengths);
````

### Syntax

````
(param1, param2, …, paramN) => { statements }
(param1, param2, …, paramN) => expression
         // equivalent to:  => { return expression; }

// Parentheses are optional when there's only one parameter:
(singleParam) => { statements }
singleParam => { statements }

// A function with no parameters requires parentheses:
() => { statements }
````

So far we have seen how the fat-arrow has syntactical changes to the language, making it easier to read/write. Now lets look at how it has changed logic of the function.

## lexical `this`

JavaScript has lexical scoping. I have earlier covered [Lexical Scoping](http://sandeep45.github.io/javascript/this/es6/fat-arrow/2016/02/03/this-keyword-in-javascript-part2.html). Here is a quick example:

````
var f2;
var f1 = function(){
  var x = "abc";
  f2 = function(){
    var y = "123"
    console.log(y);
    console.log(x);
  }
}

f1();
f2();
````

Outputs:

````
// 123
// abc
````

As you can see in the example above, `x` is resolved in function `f2` even though it doesn't exist in `f2`. This is because of lexical scoping. It dictates that a scope of a function includes the scope of all its outer functions as well, even after they have exited and finished running.

Now `this` has been and still is a special keyword. Its value can change depending upon how its called and is not lexically scoped. This concept has changed in fat arrow function. The value of `this` in fat arrow functions is lexically scoped just like any other variables. Here is an example:

````
var MyMath = function(num){
  this.number = num;
  this.doubler = () => this.number*2;
}

var m1 = new MyMath(5);
console.log(m1.doubler()); // 10

var f = m1.doubler;
console.log(f()); // 10

var f2 = m1.doubler.bind({number: 0});
console.log(f2()); // 10
````

The last two functions: `f` and `f2` are example where fat arrow functions shines and changed the logic. If this function was not defined via fat arrows, then the value of `this` would have been not been computed lexically and it would held a different value.

````
var MyMath = function(num){
  this.number = num;
  this.doubler = function() { return this.number*2; }
}

var m1 = new MyMath(5);
console.log(m1.doubler()); // 10

var f = m1.doubler;
console.log(f()); // NaN

var f2 = m1.doubler.bind({number: 0});
console.log(f2()); // 0
````

## Conclusion

Fat-Arrows give us syntactical sugar for writing functions and they compute `this` lexically.

References:
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions
- https://hacks.mozilla.org/2015/06/es6-in-depth-arrow-functions/