---
layout: post
title: this keyword in javascript
categories: [javscript]
tags: [this keyword]
published: True
---

## 'this' keyword

````
printName = function(){ console.log(this.name) }
````

````
window.name = "foo"
printName()
````
RESULT: foo

````
obj = { name: "ooooobj" }
printName.call(obj)
````
RESULT: ooooobj

````
obj.pn = printName
obj.pn()
````
RESULT: ooooobj

````
Object.prototype.haveFun = printName
obj.haveFun()
````
RESULT: ooooobj

````
boundedPrintName = printName.bind({name: "alwaysNameIsMe"})
boundedPrintName()
````
RESULT: alwaysNameIsMe

````
obj.bpn = boundedPrintName
obj.bpn()
````
RESULT: alwaysNameIsMe

````
boundedPrintName.call({name: "nopewontprint"})
````
RESULT: alwaysNameIsMe

Calling f.bind(someObject) creates a new function with the same body and scope as f, but where **this** occurs in the original function, in the new function it is **permanently** bound to the first argument of bind, **regardless of how the function is being used**.

````
C = function(){ this.name = "from constructor"; }
x = new C()
console.log(x.name)
````
RESULT: from constructor

````
$(document).on("click", function(e){ console.log(this == e.currentTarget) })
````
Click anywhere
RESULT: true

````
$(document).on("click", function(e){ console.log(this == e.target) })
````
Click child node in document
RESULT: false

## Part 2

I have written a [part 2](http://sandeep45.github.io/javascript/this/es6/fat-arrow/2016/02/03/this-keyword-in-javascript-part2.html) to this post. It has more examples, concepts and covers how `this` is resolved in with the fat arrow syntax functions defined in ES6.

