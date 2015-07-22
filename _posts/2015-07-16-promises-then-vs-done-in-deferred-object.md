---
layout: post
title: Promises: Then vs Done in Deferred Object
categories: [promises]
tags: [javascript, promises, jquery, deferred object]
published: True

---

They (`done` and `then`) look similar in basic cases. That is because they are the same. Here is an example where they look and do the same thing.

Lets say we have these two functions `func1` and `func2`. Both of them print to the console after 1 second and resolve their promise after 1 second.

````
var func1 = function(){
  var deferred = $.Deferred();
  window.setTimeout(function(){
    console.log("func1");
    deferred.resolve();
  }, 1000);
  return deferred.promise();
}

var func2 = function(){
  var deferred = $.Deferred();
  window.setTimeout(function(){
    console.log("func2");
    deferred.resolve();
  }, 1000);
  return deferred.promise();
}
````

Now, lets call them one after the other using then.

````
func1().then(func2);

\\ Output

[5:54:00] => func1
[5:54:01] => func2
````

Now, lets call them one after the other using done.

````
func1().done(func2);

\\ Output

[5:54:00] => func1
[5:54:01] => func2
````

As you can see its the same exact output. But they are different. `done` fires after the promise resolves and returns the *original* promise, other `dones` also get resolved then. with `then` one can return a *new* promise, so the next `then` wont fire till its immediate previous one finishes. lets take an example:

We now have three functions `func1`, `func2` and `func3`. All of them print to the console after 1 second and resolve their promise after 1 second.

````
var func1 = function(){
  var deferred = $.Deferred();
  window.setTimeout(function(){
    console.log("func1");
    deferred.resolve();
  }, 1000);
  return deferred.promise();
}

var func2 = function(){
  var deferred = $.Deferred();
  window.setTimeout(function(){
    console.log("func2");
    deferred.resolve();
  }, 1000);
  return deferred.promise();
}

var func3 = function(){
  var deferred = $.Deferred();
  window.setTimeout(function(){
    console.log("func3");
    deferred.resolve();
  }, 1000);
  return deferred.promise();
}
````

Now, lets call them one after the other using then.

````
func1().then(func2).then(func3);

\\ Output

[5:54:00] => func1
[5:54:01] => func2
[5:54:02] => func3
````

Now, lets call them one after the other using done.

````
func1().done(func2).done(func3);

\\ Output

[5:54:00] => func1
[5:54:01] => func2
[5:54:01] => func3
````

Here you can see the difference. In the case of done, all functions after the the first one fired immediately after the first done completed.


##### deferred.done( doneCallbacks [, doneCallbacks ] )

>The deferred.done() method accepts one or more arguments, all of which can be either a single function or an array of functions. When the Deferred is resolved, the doneCallbacks are called. Callbacks are executed in the order they were added. Since deferred.done() returns the **original** deferred object, other methods of the deferred object can be chained to this one, including additional .done() methods.

````
var d = $.Deferred();
var d1 $.Deferred();
var d2 = $.Deferred();
d.done(function(x){ console.log("in done 1",x); return d1 }).
  done(function(x) { console.log("in done 2",x); return d2}).
  done(function(x) { console.log("in done 3",x); })

d.resolve("yay");
=> VM895:5 in done 1 yay
=> VM895:6 in done 2 yay
=> VM895:7 in done 3 yay
d1.resolve();
d2.resolve();
````

##### deferred.then( doneFilter [, failFilter ] [, progressFilter ] )

>the deferred.then() method returns a **new** promise that can filter the status and values of a deferred through a function. The doneFilter and failFilter functions filter the original deferred's resolved / rejected status and values. Since deferred.then returns a Promise, other methods of the Promise object can be chained to this one, including additional  .then() methods. If the filter function used is null, or not specified, the promise will be resolved or rejected with the same values as the original. These filter functions can return a new value to be passed along to the promise's .done() or .fail() callbacks, or they can return another observable object (Deferred, Promise, etc) which will pass its resolved / rejected status and values to the promise's callbacks. http://jsfiddle.net/cqac2/

````
var d = $.Deferred();
var d1 = $.Deferred();
var d2 = $.Deferred();
d.then(function(x) { console.log("in then 1",x); return d1 }).
  then(function(y) { console.log("in then 2",y); return d2 }).
  then(function(x) { console.log("in then 3",x);})

d.resolve("foo")
=> VM702:2 in then 1 foo

d1.resolve("boo")
=> VM702:2 in then 2 boo

d2.resolve("who") who
=> VM702:2 in then 3
````

##### My Understanding

In *then*, the next *then* wont get called till the first *then*'s returned promise resolves. So the first *then* is controlling when the next *then* gets called. This is different from the *done*. In *done*, the first *done* gets called and then the next *done*. What the first *done*, returns and resolves or rejects has no bearing on the second *done* being called. All *done* has is precedence. The first *done* gets called first and then the next one and then the one after that.

##### UseCase

*then* is perefect for chaining deferred tasks where you dont want the next task to run till the first one has completed. To achieve this effect call each task chained by *then's* and return the first deferred task in the first done and second deferred task in the secone one and so on and forth. The deferred task returned becomes the deferred object on which the second *then* is now anchored and wont run till the first one is complete.

e.g.

````
var d = $.Deferred();
d.then(function(){
  var d1 = $.Deferred();
   console.log("in done 1");
   window.setTimeout(function(){ d1.resolve(); },1000)
   return d1.promise(); }
  ).
  then(function(){
    var d1 = $.Deferred();
    console.log("in done 1");
    window.setTimeout(function(){ d1.resolve(); },1000)
    return d1.promise(); }
  ).

````

When we have one `then` vs one `done`, then they work exactly the same. When you have multiple then's, then things work differently

References:
- [jQuery Then](http://api.jquery.com/deferred.then/)
- [jQuery Done](http://api.jquery.com/deferred.done/)


