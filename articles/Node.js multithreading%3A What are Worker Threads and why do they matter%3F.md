# Node.js multithreading: What are Worker Threads and why do they matter?

Since the release of [Node.js v10.5.0](https://nodejs.org/en/blog/release/v10.5.0/) there’s a new _worker\_threads_ module available.

What exactly is this new worker thread module and why do we need it? In this post, we will talk about the historical reasons concurrency is implemented in JavaScript and Node.js, the problems we might find, current solutions and the future of parallel processing with worker threads.

### Living in a single-threaded world

JavaScript was conceived as a single-threaded programming language that ran in a browser. Being _single-threaded_ means that only one set of instructions is executed at a time in the same process (the browser in this case or just the current tab in modern browsers).

This made things easier for implementation and for developers using the language. JavaScript was initially a language only useful for adding some interaction to web pages, form validations, etc. Nothing that required the complexity of multithreading.

 [Ryan Dahl](https://github.com/ry) , the creator of Node.js, saw this limitation as an opportunity. He wanted to implement a server-side platform based on asynchronous I/O, which means you don’t need threads (which makes things a lot easier). Concurrency can be a very hard problem to solve. Having many threads accessing the same memory can produce race conditions that are very hard to reproduce and fix.

### Is Node.js single-threaded?

So, our Node.js applications are single-threaded, right? Well, kind of.

Actually, we can run things in parallel, but we don’t create threads and we don’t sync them. The virtual machine and the operating system run the I/O in parallel for us and when it’s time to send data back to our JavaScript code, the JavaScript part is the one that runs in a single thread.

In other words, everything runs in parallel, except for our JavaScript code. Synchronous blocks of JavaScript code are always run one at a time:

```javascript
let flag = false
function doSomething() {
  flag = true
  // More code (that doesn't change `flag`)...

  // We can be sure that `flag` here is true.
  // There's no way other code block could have changed
  // `flag` since this block is synchronous
}
```

This is great if all we do is asynchronous I/O. Our code consists of small portions of synchronous blocks that run fast and pass data to files and streams. So our JavaScript code is so fast that it doesn’t block the execution of other pieces of JavaScript. A lot more time is spent waiting for I/O events to happen than JavaScript code being executed. Let’s see this with a quick example:

```javascript
db.findOne('SELECT ... LIMIT 1', function(err, result) {
  if (err) return console.error(err)
  console.log(result)
})
console.log('Running query')
setTimeout(function() {
  console.log('Hey there')
}, 1000)
```

Maybe this query to the database takes a minute but the “Running query” message will be shown immediately after invoking the query. And we will see the “Hey there” message a second after invoking the query if the query is still running or not. Our Node.js application just invokes the function and does not block the execution of other pieces of code. It will get notified through the callback when the query is done and we will receive the result.

### CPU intensive tasks

What happens if we need to do synchronous intense stuff? Such as doing complex calculations in memory in a large dataset? Then we might have a synchronous block of code that takes a lot of time and will block the rest of the code. Imagine that a calculation takes 10s. If we are running a web server that means that all of the other requests get blocked for at least 10s because of that calculation. That’s a disaster. Anything more than 100ms could be too much.

JavaScript and Node.js were not meant to be used for CPU-bound tasks. Since JavaScript is single threaded this will freeze the UI in the browser and queue any I/O event in Node.js.

Going back to our previous example. Imagine we now have a query that returns a few thousand results and we need to decrypt the values in our JavaScript code:

```javascript
db.findAll('SELECT ...', function(err, results) {
  if (err) return console.error(err)
  
  // Heavy computation and many results
  for (const encrypted of results) {
    const plainText = decrypt(encrypted)
    console.log(plainText)
  }
})
```

We will get the results in the callback once they are available. Then, no other JavaScript code is executed until our callback finishes its execution. Usually, as we said before, the code is minimal and fast enough, but in this case, we have many results and we need to do heavy computations on them. This might take a few seconds, and during that time any other JavaScript execution is queued, which means, we might be blocking all our users during that time if we are running a server in the same application.

### Why we will never have threads in JavaScript

So, at this point, many people will think that somebody needs to add a new module in the Node.js core and allow us to create and sync threads. That should be it, right? It’s a shame we don’t have a nice way of solving this use case in a mature server-side platform as Node.js.

Well, if we add threads, then we are changing the nature of the language. We cannot just add threads as a new set of classes or functions available. We need to change the language. Languages that support multithreading have keywords such as “synchronized” in order to enable threads to cooperate. For example [in Java even some numeric types are not atomic](https://dzone.com/articles/longdouble-are-not-atomic-in-java) , meaning that if you don’t synchronize their access you could end up having two threads changing the value of a variable and resulting that after both threads have accessed it, the variable has a few bytes changed by one thread and a few bytes changed by the other thread and thus, not resulting in any valid value.

### The naïve solution: tick, tick, tick

Node.js won’t evaluate the next code block in the event queue until the previous one has finished executing. So one simple thing we can do is split our code into smaller synchronous code blocks and call _setImmediate(callback)_ to tell Node.js we are done and that it can continue executing pending things that are in the queue.

It can continue on the next iteration or ‘tick’ of the event loop. Let’s see how we can refactor some code to take advantage of this. Let’s imagine we have a large array that we want to process and every item on the array requires CPU-intensive processing:

Like we said before if we do this the processing of the whole array will take too much time and will block the rest of the JavaScript execution. So let’s split this into smaller chunks and use _setImmediate(callback)_ :

```javascript
const crypto = require('crypto')

const arr = new Array(200).fill('something')
function processChunk() {
  if (arr.length === 0) {
    // code that runs after the whole array is executed
  } else {
    console.log('processing chunk');
    // pick 10 items and remove them from the array
    const subarr = arr.splice(0, 10)
    for (const item of subarr) {
      // do heavy stuff for each item on the array
      doHeavyStuff(item)
    }
    // Put the function back in the queue
    setImmediate(processChunk)
  }
}

processChunk()

function doHeavyStuff(item) {
  crypto.createHmac('sha256', 'secret').update(new Array(10000).fill(item).join('.')).digest('hex')
}

// This is just for confirming that we can continue
// doing things
let interval = setInterval(() => {
  console.log('tick!')
  if (arr.length === 0) clearInterval(interval)
}, 0)

```

Now we process ten items each time and call _setImmediate(callback)_ so if there’s something else the program needs to do, it will do it between those chunks of ten items. I’ve added a setInterval() for demonstrating exactly that.

As you can see the code gets more complicated. And many times the algorithm is a lot more complex than this so it’s hard to know where to put the _setImmediate()_ to find a good balance. Besides, the code now is asynchronous and if we depend on third-party libraries we might not be able to split the execution into smaller chunks.

### Background processes

So _setImmediate()_ is maybe okay for some simple use cases, but it’s far from being an ideal solution. Also, we didn’t have threads (for good reasons) and we don’t want to modify the language. Can we do parallel processing without threads? Yes, what we need is just some kind of background processing: a way of running a task with input, that could use whatever amount of CPU and time it needs, and return a result back to the main application. Something like this:

```javascript
// Runs `script.js` in a new environment without sharing memory.
const service = createService('script.js')
// We send an input and receive an output
service.compute(data, function(err, result) {
  // result available here
})
```

The reality is that we can already do background processing in Node.js. We can [fork the process](https://nodejs.org/api/child_process.html#child_process_child_process_fork_modulepath_args_options) and do exactly that using message passing. The main process can communicate with the child process by sending and receiving events. No memory is shared. All the data exchanged is “cloned” meaning that changing it in one side doesn’t change it on the other side. Like an HTTP response, once you have sent it, the other side has just a copy of it. If we don’t share memory, we don’t have race conditions and we don’t need threads. Problem solved!

Well, hold on. This is a solution, but it’s not the ideal solution. Forking a process is an expensive process in terms of resources. And it is slow. It means running a new virtual machine from scratch using a lot of memory since processes don’t share memory. Can we reuse the same forked process? Sure, but sending different heavy workloads that are going to be executed synchronously inside the forked process, has two problems:

*   Yes, you are not blocking the main app, but the forked process will only be able to process one task at a time. If you have two tasks, one that will take 10s and one that will take 1s (in that order), it’s not ideal to have to wait 10s to execute the second task. Since we are forking processes we want to take advantage of the scheduling of the operating system and all the cores of our machine. The same way you can listen to music and browse the internet at the same time you can fork two processes and execute all the tasks in parallel.
*   Besides, if one task crashes the process, it will leave all tasks sent to the same process unfinished.

In order to fix these problems we need multiple forks, not only one, but we need to limit the number of forked processes because each one will have all the virtual machine code duplicated in memory, meaning a few Mbs per process and a non-trivial boot time. So, like database connections, we need a pool of processes ready to be used, run a task at a time in each one and reuse the process once the task has finished. This looks complex to implement, and it is! Let’s use [worker-farm](https://www.npmjs.com/package/worker-farm) to help us out:

```javascript
// main app
const workerFarm = require('worker-farm')
const service = workerFarm(require.resolve('./script'))
 
service('hello', function (err, output) {
  console.log(output)
})

// script.js
// This will run in forked processes
module.exports = (input, callback) => {
  callback(null, input + ' ' + world)
}
```

### Problem solved?

So, problem solved? Yes, we have solved the problem, but we are still using a lot more memory than a multithreaded solution. Threads are still very lightweight in terms of resources compared to forked processes. And this is the reason why worker threads were born!

Worker threads have isolated contexts. They exchange information with the main process using message passing, so we avoid the race conditions problem threads have! But they do live in the same process, so they use a lot less memory.

Well, you can share memory with worker threads. You can pass SharedArrayBuffer objects that are specifically meant for that. Only use them if you need to do CPU-intensive tasks with large amounts of data. They allow you to avoid the serialization step of the data.

### Let’s start using worker threads!

You can start using worker threads today if you run Node.js v10.5.0 or higher, but keep in mind that this is an _experimental API_ that is subject to change. In fact, it is not available by default: you need to enable it by using _ — experimental-worker_ when invoking Node.js.

Also, keep in mind that creating a Worker (like threads in any language) even though it’s a lot cheaper than forking a process, can also use too many resources depending on your needs. In that case, the docs recommend you create a pool of workers. You can probably look for a generic pool implementation or a specific one in NPM instead of creating your own pool implementation.

But let’s see a simple example. First, we are going to implement the main file where we are going to create a Worker Thread and give it some data. The API is event-driven but I’m going to wrap it into a promise that resolves in the first message received from the Worker:

```javascript
// index.js
// run with node --experimental-worker index.js on Node.js 10.x
const { Worker } = require('worker_threads')

function runService(workerData) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./service.js', { workerData });
    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0)
        reject(new Error(`Worker stopped with exit code ${code}`));
    })
  })
}

async function run() {
  const result = await runService('world')
  console.log(result);
}

run().catch(err => console.error(err))

```

As you can see this is as easy as passing the file name as an argument and the data we want the Worker to process. Remember that this data is _cloned_ and it is not in any shared memory. Then, we wait for the Worker Thread to send us a message by listening to the “message” event.

Now, we need to implement the service.

```
const { workerData, parentPort } = require('worker_threads')

// You can do any heavy stuff here, in a synchronous way
// without blocking the "main thread"
parentPort.postMessage({ hello: workerData })

```

Here we need two things: the _workerData_ that the main app sent to us, and a way to return information to the main app. This is done with the _parentPort_ that has a _postMessage_ method where we will pass the result of our processing.

That’s it! This is the simplest example, but we can build more complex things, for example, we could send multiple messages from the Worker Thread indicating the execution status if we need to provide feedback. Or if we can send partial results. For example, imagine that you are processing thousands of images, maybe you want to send a message per image processed but you don’t want to wait until all of them are processed.

In order to run the example, remember to use the _experimental-worker_ flag if you are in Node.js 10.x:

node --experimental-worker index.js

For additional information check out the [worker\_threads](https://nodejs.org/docs/latest-v10.x/api/worker_threads.html) documentation.

### What about web workers?

Maybe you have heard of Web workers. They are a more mature API **for the web** and [well supported](https://caniuse.com/#search=web%20workers) by modern browsers. The API is different because the needs and technical conditions are different, but they can solve similar problems in the browser runtime. It can be useful if you are doing crypto, compressing/decompressing, image manipulation, computer vision (e.g. face recognition), etc. in your web application.

### Conclusion

Worker threads is a promising experimental module if you need to do CPU-intensive tasks in your Node.js application. It’s like threads without shared memory and thus, without the potential race conditions they introduce. Since it’s still experimental I would wait before using it and I would just use [worker-farm](https://www.npmjs.com/package/worker-farm) (or similar modules) to do background processing. In the future, your program should be easy to migrate to worker threads once they are mature enough!

**********
[javascript](/tags/javascript.md)
[НЕ ПЕРЕВЕДЕНО](/tags/%D0%9D%D0%95%20%D0%9F%D0%95%D0%A0%D0%95%D0%92%D0%95%D0%94%D0%95%D0%9D%D0%9E.md)
[node.js](/tags/node.js.md)
