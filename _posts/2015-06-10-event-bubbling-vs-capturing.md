---
layout: post
title: Event Bubbling vs Capturing
categories: [javascript]
tags: []
published: True
---

Be default the event are setup as *Bubbling*. If you are using jQuery, then it only works with Bubbling. Older versions of IE also only support Bubbling.

Bubbling setup means that if I have:

````
  window.document.addEventListener("click", function(){
    console.log("click handler of doc")
  });

  window.document.body.addEventListener("click", function(){
    console.log("click handler of body")
  });

````

then clicking on the page will first process the handler on the body and then process the handler on the document. So in other words the event is **bubbling up** from the inner DOM element to the outter most DOM element

On the other hand, to setup in capture mode would be like this:

````
  window.document.addEventListener("click", function(){
    console.log("click handler of doc")
  }, true);

  window.document.body.addEventListener("click", function(){
    console.log("click handler of body")
  }, true);

````

Now clicking on the page will first process the handler on the document and then process the handler on the body. So in other words the event is capture right on the outtermost DOM element and then allowed to go down to the inner DOM elements.

Syntax for reference:

````
  eventTarget.addEventListener(type,listener,[,useCapture]);
````

By Default useCapture is false. It means it is in the bubbling phase.



