---
layout: post
title: chaining asynchronouse requests
categories: [javascript]
tags: [javascript]
published: True

---

Lots of times I find myself making a bunch of async things like ajax requests, one after the other. Where the result of the first one is required in the second one. For e.g.

It usually looks like this:

<iframe width="100%" height="300" src="//jsfiddle.net/sandeep45/ztp2yn6k/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

This right moving code can very quickly become:
* annoying
* not pleasent to write
* hard to maintain
* and hard to debug in
* plus it's killing my 80 char word wrap column
* more stacks than one may like

### A better solution: jQuery's Deferred Concept

Let's start by using the deferred object to re-write the code we had above which had functions wrapped inside the success callbacks.

<iframe width="100%" height="300" src="//jsfiddle.net/sandeep45/grauqgvb/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Ajax methods by default in jQuery are returning a `promise` of a `deferred` object which is created withe every ajax request. jQuery is also internally calling `resolve` on the `deferred` object when the ajax is successfully and calling `reject` when the ajax method fails. This updates the state on the promise and fires our callbacks attached to it via the `then` method.

The thing to note here is that `then` calls the next function only after the first functions promise has been resolved.

So jQuery did a lot of thing's here for us when we used the ajax method. But promises in jquery are not limited to ajax and can be used anywhere you want. Below you will see how.

### Exploring jQuery's Deferred Object

jQuery provides a deferred object which can be created by using its factory method `$.Deferred([beforeStart])`. The deferred object has `promise` which can be returned when you the `value` is unknown and you want to return something so the receiver can proceed and call the next based upon its value. This is done by using the `then` method available on the `promise`. The first method which returned the deferred object can call `resolve` on the deferred object when the value is known and at that time the methods waiting on its promise will execute. Lets see an example.

We want to increase the height of a div from 100 to 500 and then let the next function know to put content in the div. The content should only be placed after the the width has increased.

<iframe width="100%" height="300" src="//jsfiddle.net/sandeep45/r1uw4rbv/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Lets go over the API's available in the jQuery Deferred Object.


* __$.Deferred([beginMethod])__: Factory method to create the deferred object in its initial state of _pending_

* __deferredObj.resolve(args)__: This moves the state of the promise to _resolved_ and thus causes the `done` listeners on the promise to fire. It passes args to the ha__ndler. The args could represent the `value` which was pending and is now available.

* __deferredObj.reject(args)__: This moves the state of the promise to rejected and thus causes the `fail` listeners on the promise to fire. It passes args to the handler.

* __deferredObj.notify(args)__: This does not change the state but causes the `progress` listeners on the promise to fire. It passes args to the handler.

* __deferredObj.state()__: This returns the state of the promise/deferred object. It could be _pending, resolved or rejected_

* __deferredObj.promise()__: gives an object which can be returned and observed for when the `value` is available

* __promiseObject.then(df,[ff],[pf])__: takes methods to be called when the promise is resolved, rejected or has progress

* __promiseObject.done(df)__: takes method to be called when the promise is resolved

* __promiseObject.always(af)__: takes method to be called when the promise is either resolved, rejected or has progress

* __promiseObject.fail(ff)__: takes method to be called when the promise has failed

* __promiseObject.progress(pf)__: takes method to be called when the promise has progress

* __$.when(promise, [promise+])__: returns a `promise` which changes state only when all the promises passed in resolve.

The `deferredObj` and `promiseObject` object are the same thing besides the subtle difference that on the `promise` object one can not call `resolve`, `reject`, `notify` etc. Therefore the `promise` object is only good for listening and thus an excellent candidate to be returned by the original method. Now all methods can be called on the `deferred` object but you should return the `promise` object and add callbacks on it. So in short: **Change state of deferred and listen for state change on promise.**

The `when` factory method is excellent choice when you want to create a new promise which is a combination of other `promises`. For e.g.

<iframe width="100%" height="300" src="//jsfiddle.net/sandeep45/jwhzws4q/2/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Now sometimes, things might finish up and faster than you expect. Meaning the state on a deferred will move from _pending_ to _resolved_ even before you get an opportunity to add an event handler on the promise of that deferred object. This is non-issue here because any handler added after the state has changed would fire immediately if its state had already been reached. For e.g.

<iframe width="100%" height="300" src="//jsfiddle.net/surjo53w/2/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

References:
- [Futures_and_promises Wikipedia](https://en.wikipedia.org/wiki/Futures_and_promises)
- [MSDN Article](https://msdn.microsoft.com/en-us/magazine/gg723713)
- [jQuery Deferred Object](http://api.jquery.com/category/deferred-object/)
- [CommonJS Promises/A Interface](http://wiki.commonjs.org/wiki/Promises/A).