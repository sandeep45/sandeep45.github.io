---
layout: post
title: getting started with requirejs
categories: []
tags: []
published: True

---

when loading modules dont write `.js` extension. it changes the path to be non-intutive

Modules in requirejs are well scoped objects. They are different from commonJS pattern of modules. They explictly define dependecies, and get handle to those dependencies. They are an extension of the Module pattern. They dont rely on globals to refer to other modules.



````
// awesome.js

// this module depends on two files dep1 and dep2.
// they are first loaded and their returned values are passed in to dep1 and dep2
// the function is not called till the dependent modules are loaded.
// This is the standard AMD define

define(["dep1", "dep2"], function(dep1, dep2){
  // init code here
  obj = {};

  // this is what is returned by the this awesome module
  return {
    awesome: obj
  }
});
````

for debnugging you can load a module like below. This does a __synchornous__ load
It only works if the module was defined as dependy earler in an aysnc load.

````
xyz = require("helplers/xyz")
xyz.bla()
````

### Intersting modules:


1. exports

In standard AMD style module definition ask for the exports module as a dependecy, then receive it in the params.
Now whatever you want to make available to the world, just add it on to the exports module.
No need to return it.
the added stuff on exports will be available on the variable which handles the require of the module
so you are doing the following in the define function:

````
exports.bla = function() { console.log("bla")}
````

instead of

````
return {
  bla: function() { console.log("bla") }
}
````

__Note:__ Adding methods to the exports module, only adds it on the `exports` object for that module. On the next module when `exports` module is mentioned as dependency it will not have the methods added on the `exports` module previously.

2. module

Just ask for module in the dependency list and receive it. Now you can do:

````
console.log(module.id)
console.log(module.url)
````

3. require

Just ask for module in the dependency list and receive it. Now you can do:

````
require.toUrl("helper/calculator") \\ "/assets/helper/calculator"
````

### Circular depencies.

A needs B. B needs A.
When A starts, it will pause to Load B. B will ask for A. B will get empty A. B will finish. Then A will gets its B and then A will finish.

Now in B, you have an empty A. To solve this do `require(A)` this will reget the loaded A. Just make sure to specify require as a dependency and use that to reget A.



### require vs requirejs vs define

require and requirejs are the same.

````
requrie === requirejs // true
````

require is a way to load a module which has been defined. For example to load the `logger` module I could do:

````
require(["logger"], function(logger){
  logger.bla("S");
})
````

Here i am calling `require`, specifying an already defined module called `logger` and using it by calling its `bla` method.

define is a way to define a module. For example to define a `logger` module I could do:

````
\\ logger.js
define(function(){
  return {
    bla: function(x){
      alert(x);
    }
  }
})
````

Here i called `define` and defined the `logger` module. in this module I returned the `bla` function i want to expose.

Sometimes define looks very similar to exports because define can also depend and use other modules just like require can use other modules. Let me show you the same `logger` module, this time using a module

````
\\logger.js
define(["popup"], function(popup){
  return {
    bla: function(x){
      popup.show(x);
    }
  }
});
````

Here the logger module I defined, also has a dependancy called `popup` and thus it looks like logger.

### Loading a regular js file via require

It will work. the js file being loaded will run and in the end the callback function in the require statement will fire. The callback function will get undefined for the variable which represents the plain js

````
require(["plainjsfile"], function(plainjsfile){
  console.log("yo");
  console.log(plainjsfile); // undefined
});

//plianjsfile.js
console.log("in plainjsfile");
````

Outputs

````
yo
undefined
in plainjsfile
````

### Accessing the DOM

If you need the DOM, then use the domReady plugin. It has a domReady function which fires its callback after the dom is ready. I prefer to use its bang version which loads the module's callback after the dom is ready. I like this because it makes the whole module's defination not execute till the dom is ready. many modules can just say domReady! as a dependency and they wont run till its done.

for example, i have a logger module which needs the dom. so here is how i would define it:

````
//logger.js
define(["domReady!"], function(){
  exports.info = function(x){
    window.document.getElementById("messageBox").innerText = x;
  }
});
````

### Depending on DOM and using jQuery

I have a file called `domReady.js` and `jquery.js`. So i can setup my module like:

````
//logger.js
define(["domReady!", "jquery"], function($){
  exports.info = function(x){
    $("#messageBox").innerText = x;
  }
});
````

### Use in __PROD__

To use in production, we need to minify all resoures in to 1 file and load that file instead. First I defined a `proxy-build.js`

````
({
    baseUrl: "app/assets/javascripts/",
    optimize: "uglify2",
    generateSourceMaps: true,
    name: "proxy-main",
    out: "app/assets/javascripts/built/proxy-main.js",
    preserveLicenseComments: false
})
````

1. Here i am defining the base directory where all my stuff will be looked upo from.
2. Then I am using `uglify2` as it supports source maps
3. asking to generate soucemaps
4. giving name of the __one__ input file
5. giving path to the __one__ output file
6. giving permission to remove license as its required for sourcemaps

Now in the `built` directory I have one minified file for production.