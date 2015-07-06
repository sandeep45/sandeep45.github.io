---
layout: post
title: sample requirejs app
categories: []
tags: []
published: True

---

__First__ new need an HTML page which will use the JS files.

````
<!-- Proxy.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Sample App called Proxy </title>
</head>
<body>
<h1>Proxy Page</h1>
  <script data-main="/assets/built/proxy-main.js" src="/assets/require.js" async=true></script>
</body>
</html>
````

Here, I am loading one script file called require.js. It has a data-tag called `data-main` which has path to the `main` js file. Its common practice to name this file `something-main` or `main`. In this file we define main logic of this app or page.

__Second__ lets write out the `proxy-main.js` file.

````
requirejs.config({
    baseUrl: '/assets'
});

requirejs([
    "helper/calculator"
  ],
  function(calculator) {
    console.log("starting proxy main. doing addition");

    var ans = calculator.add(5,6);

    console.log("leaving proxy main after addition and getting: " + ans);
});
````

Here, I am defining the base directory to access all javascript files on the web from the webserver. Since I am using rails all my javascript files on the web are available on `/assets`.

__Note__, that even though my assets are in `/app/assets/javascripts/*` or in `/vendor/assets/javascripts/*` they are still served from the path of `/assets` and therefore my baseUrl is `/assets`

The second thing I have is a call to `requirejs`. Its first parameter is an array of depencies and the second paramater is a callback function. This function first loads all dependencies and then executes the callback function. It passes each loaded module as parameter to the callback function.

__Next__, Let me show you one of the modules:

````
define([], function(){

  var obj = {};

  obj.add = function(a,b){
    console.log("in add with", a, b);
    var result = a+b;
    console.log("result is", result);
    return result;
  };

  return obj;

});

````

Here I am define a module called `exports`. The syntax is similar to that of `require`. It first lists its own depencies and then a callback function. In the callback function the module returns an object which has all public available properties of this module. This object is what gets passed in to the callback function when this module is loaded.

__To Conclude__ we added an html file called `Proxy.html`, which laods the `require.js` and specifies `proxy-main.js` as the main entry point. From that point the main file loads its depenencies and executes the callback. The callback uses the dependency and executes the `add` function.

![Browser Screenshot running Proxy.html](/assets/sample_requirejs_app/browser.png)