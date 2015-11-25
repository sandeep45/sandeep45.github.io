---
layout: post
title: Queue System Using Jquery
categories: [javascript, jquery]
tags: [queue, javascript, jquery]
published: True

---

jQuery has two handy functions: `queue` and `dequeue`, which can be used to build a system where tasks are entered in to a queue. This can be use to build a queue to do anything, not necessarly DOM related things.

It seems that jQuery has this for the purposed of building animations, but below I am showing how to use this to build a queue of any tasks.

Lets say we have 3 functions:

````
var uploadABC = function(){
  console.log("in uploadABC");
}
````

````
var upload123 = function(){
  console.log("in upload123");
}
````

````
var uploadXYZ = function(){
  console.log("in uploadXYZ");
}
````

Now we would like them to run one after the other. So we can queue all these for executinon. And then the next one, wont run till the first one dequeue's itself.

This is how you will queue them for execution:

````
  $(document).queue(uploadABC);
  $(document).queue(upload123);
  $(document).queue(uploadXYZ);
````

Upon queueing them, since there was nothing else on this queue, `uploadABC` will run immediately. You should see an outout like:

````
in uploadABC
````

After that nothing else will be executed. Both `upload123` and `uploadXYZ` wont run. To run them, the earlier task needs to `dequeue`. *Note here*, just because the task has ran and completed, that doesn't mean it will be automatically dequeued. The deque method needs to be called indicating that the task has completed. If this was an `ajax` call, I would do this in the `complete` method.

Let's manually `dequeue` a task like this:

````
$(document).dequeue();
````

Now we will see the second task which was added to the queue will automatically run and we should see an output like:

````
console.log("in upload123");
````

Now lets `dequeue` again and we will see the next task automaticaly run and produce the following output.

````
$(document).dequeue();
console.log("in uploadXYZ");
````

### Closing Note

A common use case is if you are making multiple ajax requests in response to events emitted on the DOM and want to limit it to one ajax request at any time. To acheive this rather than calling the method which makes the `ajax`, put it on the `queue` and in their `complete` method `dequeue` them. Now you will see the browser only make one request at a time.




