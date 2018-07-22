---
title: Reading NodeJS Notes
status: 
layout: post
categories: [NodeJs]
tags: [NodeJs]
published: False
---

## getting started with nodejs
.editor
so cool. i wish ruby had this.
now i can write code, with indentation write in the console.
.save & .load
so cool. lets save the console history and directly load it back.
ruby has this, but gotta use history and its not this clean.

# nodejs

global object gets you all global stuff
in their we have global.process, global.process.cwd()
or you can just do `process` or `process.cwd()`

```
const puts = (x) => global.process.stdout.write("yo yo " + x + " \n")
> puts("hey")
yo yo hey
```

`argv` on the process is an array of argumnets passed in when the process was ran

`nextTick` is the same as `setTimeout( () => {}, 0)`
tick is the next cycle when node runs through the code to be executed.

octet streams which we get from file system or TCPIP. so we use a buffer to process it.

```
buf = new Buffer(5);
buf.write("Xxxxx")
buf.toString('utf8', 0 , 2)
> 'ab'
buf.toString('utf8', 0 , 5)
> 'abcde'
Buffer.byteLength("as")
> 2
Buffer.byteLength(buf)
> 5
```

everything we dont scope to something, gets put on `global`. This is like `window` in the browser

```
var a = () => console.log("say hi")
> a()
say hi
> global.a()
say hi
```

And just like in javascript, when we make a module and put a `const` or `var` in it, then `require` or `import` that module via webpack, the variables are global only to that module and only the things exported are returned back. the same happens in Node as well.


`require` works just like how it works in webpack. you can give relative path to include your own lib, or just name of the module and it will search all required folders - global modules -> local node_modules -> directory traverse and work upwards

to export, things add whatever you build as an attribute to the `exports` object. then when some does `x = require('./file.js')`, they will have your attributes available on the `x` object

```
\\ file.js
exports.a = () => console.log("yo");
```

```
x = require('file.js')
x.a() // "yo"
```

Another option of exporting is to add the entire object you want to return to `module.exports`

```
module.exports = {
  a: () => console.log("yo");
}
```

Note: in both cases the objects `module` & `exports` were __magically__ available in our `file.js`.

Semantic Version'ing System
2.3.5
major.minor.patch
^2.2.5
the carrot makes the minor version be a wild = 2.3.5, 2.4.0, 2.5.9 etc.
~2.2.4
the told makes the patch level be a wild = 2.2.4, 2.2.5, 2.2.6 etc.
*
installs the latest version

`npm adduser` and then `npm publish` and then update version and repeat

when using node REPL
.help gives u all the commands
.editor lets your write a bunch of code in multiline style
and .load lets you load in any file you have written like you had just typed it in the shell. this is important becuase all variables and scopes are the same as the file was typed in.

EventEmitter
make an instance
add listeners using `on`. just specify the event name and handler function
then fire then by using `emit`
add as many handlers for an an event type as you want.

to check for GC start node with `--gc` option
Scavenge Operation -> identifies which objects are no longer being used
Mark-Sweep Operation -> works on old space -> stops the node process
Heap and Generation
New Space -> where objects are created
Old Space -> objects which survive 2 scavenge operations
Heap Dump -> snapshot of memory allocation at a particular point of time
kill -usr2 pid # gives a heapsnapshot file, this can be loaded in chrome, needs heapdump npm module

Starting a node project
do npm init -y
and npm install --save express
create index.js

express - app server
nodemon - autorestart
npm command `start` to run `node index.js` etc.

serve static images from a folder
build index page to show all users
build show page to show specific user data based on url

body-parser npm package to parse body

write the index and show content to a file and read from file
keep flle as jsob
use readFile, unlinkSync and writeFileSync
note that on delete we need to update both index and its show file
same applies to create

for a route handler, we can have multiple functions as handlers
in next we can pass in `route`, to skip other handlers and go to next route handler

reponse methods:
handle 404 page
do redirect on error
do download for .json files
do json to write out json

use .all method to write out logs

use `router` to setup individual routes for each subpath

one way of defining multiple handlers for various mehthods on the same path is:

```
app.route("/:username").all().get().put() // etc.
```

for one subpath, built all actions on a router
then in the main app, give the path and the router you want to handle it.

read from file, streaming, non blocking and write to browser
read from file, streaming, parase the json, if it matches and then write the matched ones to browser


use a seperate json file for routes
file with a bunch of names and we want them all mapped to a handler which does something via names
colon to accept variables and access via params
regex also works
order matters
next allows chaining handlers

resp.send("sds") // it seems to be setting html typw
render jade views

read data from a file in a non-blocking stream
then use jsonStream to parse it, format it
and then write non-blocking to another file or response

checkout middlewheres listed on the express website

put users in mongodb instead of json file
setup mongodb and its client
use mongoose for ORM
do index, update, delete etc. to mongo using mongoose


