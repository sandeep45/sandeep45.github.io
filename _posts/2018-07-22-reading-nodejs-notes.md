---
title: Reading NodeJS Notes
status: 
layout: post
categories: [NodeJs]
tags: [NodeJs]
published: False
---

node js -> v8 & libuv
v8 lets you run JS outside of the browser
node has modules like fk, http etc. which then call C code in libuv
libuv lets you access fs, networking, concurrency etc.

`conxt {x} = process.binding('crypto')` is being used to import a function from C world to js world

So you can write your code in C++. then use process.binding to import it in JS and use it via JS.
`/lib` -> JS code
`/src` -> C++ code

Threads -> a block of instructions for the CPU to process
OS Scheduler -> decides which thread runs, manages priority and can detect pauses in threads like when waiting for I/O and run different threads to optimize
More CPU cores and multi-threading and hyperthreading are ways to process more threads at the same time.

node starts off 1 thread and then starts the event loop in this thread
tick - every time the event loop runs or circles

setTimeout
setInterval
setImmediate

The event loop which node starts up, keeps ticking as long as there is a pending:
- async function like setTimeout, setInterval or setImmediate
- OS Call like listening on port
- Long Running OS operation

Inside the event loop, once it ran, it will do
1 see if any of the setTimeout or setInterval have been completed and if so it will call its callbacks.
2 looks at completion of any pending OS tasks or pending long running operations and if so calls their callbacks. 
3. pauses for a pendingOS Task or pending Long Running Task to finsh or a timer is about to expire. This is important as we have to wait for something to happen otherwise our thread will just keep running at Full Speed and wont let anything else happen normally.
4.  see if any setImmediate has completed and calls its callbacks
5. handle any close events

The event loop which node starts when the program starts is in a single thread, but the other things we call from our node program are not necessarily single threaded. They may do things using multiple threads or processes and therefore those things may happen parallely. When they are done, callbacks are fired and then node even loop handles it in its single thread. Some of the other processes you are calling from the node event loop are also written and provided in Node e.g. FS, which utitlize multi threads so if you call them to do multiple things like read file1 and file 2, it can do that parallely. This gives way for us to say that Node is multi-threaded and its just the event loop part which is single threaded.

Libuv has 4 threads to do heavy computation work. The number 4 is because i have 2 cores. And its multithreaded/hyperthreaded. This means that each core can process 2 threads at the same time. It will process 2 threads together, but will take double the time. So now we have 4 threads. Now if we run a libuv task 4 times, they will all 4 of them finish in about the same time and each of them will take double the amount of time they would have taken if only 1 was ran. And finally if a 5th task was ran, it will take longer because it has to wait for a thread to free up.

UV_THREADPOOL_SIZE - We can change the number of our threads in the threadPool for libUv to use. If we increase the number of threads the thread scheduler will make sure that all threads get an equal amount of CPU time. note that this will also equally make them all slower.

`http` module is returning data and calling the callback everytime it gets a chunk of data.

libuv delegates work like http network requests to the OS.

pendingOSTasks is what refers to tasks of networking which are given to the OS

Cluster Mode for boosting performance of nodeJS APP
Cluster wont work with nodemon, so you have to restart

cluster.isMaster
cluser.fork()

ab -c 50 -n 500 http://localhost:3000/fast

When we have more processes than the number of cores, and each process is working on a core intensive task, then it will start all tasks together, but because the cores is cycling between various processes, it propotinally takes longer for all the tasks. So if it can now handle 5 times more processes, but then it also takes 5 times longer. 
On the other hand if we kept the number of process to the same as the number of cores, then we would do work in chunks, and at the least the early onces will return first even if the total time it took is the same.
Note this applies only in the case that the work being done in CPU core intensive and is locking it up.

PM2 for managing process
  
webworker-thread
Worker

 
  


 

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


