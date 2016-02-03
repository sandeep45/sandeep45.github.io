---
layout: post
title: this-keyword-in-javascript-part2
categories: [javascript, this, es6, fat-arrow]
tags: [javascript, this, es6, fat-arrow]
published: True

---

# `This` keyword in Fat arrow functions

This is a continuaiton of my [previous post](http://sandeep45.github.io/javscript/2015/06/09/this-keyword-in-javascript.html) where I attempted to explain how `this` behaves in javascript. In this post I will continue to explore how `this` keyword works with fat arrow style functions (`=>`) which are available in ES6.

Lets start with a primer on `this`:

1\. `this` refers to `window` if accessed globally. But in `strict` mode, in global mode it would be undefined.

````
(function(){
  this.name = "yo yo honey singh";
})();
console.log(window.name == this.name);
````

````
(function(){
  "use strict";
  this.name = "yo yo honey singh"; // ERROR as this is undefined
})();
````



2\. `this` refers to an instance if in constructor mode

````
function Food(n){
  this.name = n;
}
var f = new Food("chicken makhni");
console.log(f.name); //chicken makhni
````

3\. `this` refers to an object if in a method inside an object.

````
var player1 = {
  name: "sandeep arneja",
  printName: function(){
    console.log("player's name is: " + this.name);
  }
}
player1.printName(); // player's name is: sandeep arneja
````

4\. `this` refers to DOMElement when used as an event listener

````
$(document.body).on("click", function(){
  console.log(this); // Body DOMElement
})
````

5\. `this` refers to an object when it is bounded to it via `bind`, `call` or `apply`

````
  window.name = "yo yo honey singh"
  var sayHello= function(){
    console.log("hello " + this.name);
  }
  var sayHello2 = sayHello.bind({name: "Pankaj Udas"})
  sayHello(); // yo yo honey singh
  sayHello2(); // Pankaj Udas

````

## Scope-By-Flow

In Javascript the `this` keyword is resolved as `scope-by-flow`. I borrowed this term of `scope-by-flow` from a blog post at [jsrocks](http://jsrocks.org/2014/10/arrow-functions-and-their-scope/). What i mean to say is that the value of `this` can change depending upon how the function which has `this` is being called.

````
window.name = "bananas";

var player1 = {
  name: "sandeep arneja",
  printName: function(){
    console.log("player's name is: " + this.name);
  }
}
player1.printName(); // player's name is: sandeep arneja

window.setTimeout(player1.printName ,1000); // player's name is: bananas
````

In the examplea above the flow has changed. Now when `player1.printName` is called, `this` refers to the global window.

````
window.name = "bananas";

var callback = function(cb){
  cb();
}

var player1 = {
  name: "sandeep arneja",
  printName: function(){
    console.log("player's name is: " + this.name);
  }
}
player1.printName(); // player's name is: sandeep arneja

callback(player1.printName); // player's name is: bananas

````

In the example above again the flow has changed, the `printName` function is no longer called on the object, meaining its not like `object`, followed by a `dot`, followed by function.

Similar effects come with the bind, call & apply functions. They change the value of `this` even when the function is a method on an object and `this` is supposed to be pointing to the object.

````
window.name = "global window name";
var a = {
  name: "yo yo honey singh",
  print: function(){
    console.log(this.name);
  }
}
a.print(); // yo yo honey singh
x = a.print;
x(); // global window name
````

Above is another example of how the value of this changed depending upon how it was called. Here `this` is resolved based upon on how it is called. When calling a method, like `object` `.` `methodName`, then the value of `this` in the function is equal to the `object` which appers before the `dot`.


## Function Scope & Lexical Scoping

Lets cover two more concepts and then we will be ready to dive in to how `this` changes with Fat Arrows (`=>`) in ES6.

1\. Function Scope: Any variable which is defined inside a function, is visible inside the entire function. This is different from block scope.

````
var a = function(){
  if(true){
    var b = 5;
  }
  b = b + 9;
  console.log(b);
}
a(); // 14
````

````
(function(){
  b = 5
  console.log(b); // 5
  var b;
  c = 5;
  console.log(c); // 5
})();
console.log(c); // 5
console.log(b); // Uncaught ReferenceError: b is not defined
````

2\. Lexical Scoping: The scope of the inner function includes the scope of the outter function.

````
var outter = function(){
  var name = "outter";
  var age = 100;

  var inner = function(){
    var name = "inner";

    var print = function(){
      console.log("name is : " + name + " and age is: " + age);
    }

    print();
  }

  inner();
}

outter(); // name is inner and age is 100
````

The inner function includes the scope of the outter function, even when after the outter function has returned.

````
var setIt = function(){
   var x = "yo yo honey singh";

   var printer = function(){
    console.log(x);
   }

   return printer;
}

var p = setIt();
p(); // yo yo honey singh
p(); //yo yo honey singh
````

In the example above thanks to Lexical Scoping the `printer` function's scope includes the scope of `setIt` and is therefore able to resolve variable `x`. Note its able to do so even after the outter function, `setIt` has been called and ended execution.

Lexical function plays a key role in resolving variables when in nested functions.

Reference:

- http://pierrespring.com/2010/05/11/function-scope-and-lexical-scoping/


## Fat Arrow and `This`

A function defined with fat arrow syntax or style resolved `this` lexically only. The `this` keyword no longer has the special properties mentioned above. Lets take an example:

````
window.name = "global name";

var x = {
   name: "yo",
   print: function(){
     console.log(this.name);
   },
   print2: () => {
     console.log(this.name);
   }
}

x.print(); // yo
x.print2(); // global name

x.print.call({name: ":)"}); // yo
x.print2.call({name: ":)"}); // global name

````

In the example above when the function `print2` is called, it resolves `this` lexically, just like any other variable. `this` is never the object on which the method was called or bound to. It is what is in scope, which includes the outter functions scope. In this case since there was no outter function, this goes to default value of `window` or it would be `undefined` if we were in strict mode.

Lets take another example:

````
var students = {
  records: [],
  add: function(name){
    this.records.push(name);
  },
  addAll: function(names){
    names.map( (n) => this.add(n));
  }
}

students.addAll(["yo", "jo", "mo", "po"]);
console.log(students.records);
````

In the example above, we used a fat arrow function as a callback to the map function `inside addAll`. In the callback function, `this` keyword was lexically scoped and therefore took the value of this from its parent function. Its parent function is `addAll` where `this` resolved to `students`.

Here is how we would go about doing it without a fat arrow:

````
var students = {
  records: [],
  add: function(name){
    this.records.push(name);
  },
  addAll: function(names){
    names.map(function(n){
      this.add(n);
    });
  }
}

students.addAll(["yo", "jo", "mo", "po"]); // this.add is not a function
console.log(students.records);
````

So `this` does not resolve to students in the callaback of `map`. This makes sense as we know `this` is resolved with a `scopy-by-flow` principle. So we need to pass in the this to this function. Lets use lexical scoping to help us do so.

````
var students = {
  records: [],
  add: function(name){
    this.records.push(name);
  },
  addAll: function(names){
    var s = this;
    names.map(function(n){
      s.add(n);
    });
  }
}
students.addAll(["yo", "jo", "mo", "po"]); // this.add is not a function
console.log(students.records);
````

Now in the example above, the callback function us using variable `s` over `this`. The `AddAll` function which is the parent function of the callback function has stored the value of `this` in a regular variable `s`. The callback function of map has a scope which includes its parent functions scope which is `AddAll` and that function has the variable `s` which holds the value of `this.  Now when the callback function runs it is able to resolve s and get the value of `this`.

Note what i have done above could also be achieved by passing `this` to the map function as it supports passing in a value for `this` and also by binding the callback function to `this`.

````
window.name = "global window name";
var a = {
  name: "yo yo honey singh",
  print: function(){
    console.log(this.name);
  },
  print2: () => {
    console.log(this.name);
  }
}
a.print(); // yo yo honey singh
x = a.print;
x(); // global window name
a.print2(); // global window name
x = a.print2;
x(); // global window name
````

Reference:

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions
- http://derickbailey.com/2015/09/28/do-es6-arrow-functions-really-solve-this-in-javascript/


### Conclusion:

In Fat-Arrow Syntax `this` is lexically scoped, meaning the value of `this` is what its defined around it. In other words the value of `this` is inherited from the enclosing/parent scope.



