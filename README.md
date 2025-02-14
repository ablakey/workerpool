# workerpool

[![Greenkeeper badge](https://badges.greenkeeper.io/josdejong/workerpool.svg)](https://greenkeeper.io/)

**workerpool** offers an easy way to create a pool of workers for both dynamically offloading computations as well as managing a pool of dedicated workers. **workerpool** basically implements a [thread pool pattern](http://en.wikipedia.org/wiki/Thread_pool_pattern). There is a pool of workers to execute tasks. New tasks are put in a queue. A worker executes one task at a time, and once finished, picks a new task from the queue. Workers can be accessed via a natural, promise based proxy, as if they are available straight in the main application.

**workerpool** runs on node.js, Chrome, Firefox, Opera, Safari, and IE10+.


## Features

- Easy to use
- Runs in the browser and on node.js
- Dynamically offload functions to a worker
- Access workers via a proxy
- Cancel running tasks
- Set a timeout on tasks
- Handles crashed workers
- Small: 5 kB minified and gzipped


## Why

JavaScript is based upon a single event loop which handles one event at a time. Jeremy Epstein [explains this clearly](http://greenash.net.au/thoughts/2012/11/nodejs-itself-is-blocking-only-its-io-is-non-blocking/):

> In Node.js everything runs in parallel, except your code.
> What this means is that all I/O code that you write in Node.js is non-blocking,
> while (conversely) all non-I/O code that you write in Node.js is blocking.

This means that CPU heavy tasks will block other tasks from being executed. In case of a browser environment, the browser will not react to user events like a mouse click while executing a CPU intensive task (the browser "hangs"). In case of a node.js server, the server will not respond to any new request while executing a single, heavy request.

For front-end processes, this is not a desired situation.
Therefore, CPU intensive tasks should be offloaded from the main event loop onto dedicated *workers*. In a browser environment, [Web Workers](http://www.html5rocks.com/en/tutorials/workers/basics/) can be used. In node.js, [child processes](https://nodejs.org/api/child_process.html) and [worker_threads](https://nodejs.org/api/worker_threads.html) are available. An application should be split in separate, decoupled parts, which can run independent of each other in a parallelized way. Effectively, this results in an architecture which achieves concurrency by means of isolated processes and message passing.


## Install

Install via npm:

    npm install workerpool


## Load

To load workerpool in a node.js application (both main application as well as workers):

```js
var workerpool = require('workerpool');
```

To load workerpool in the browser:

```html
<script src="workerpool.js"></script>
```

To load workerpool in a web worker in the browser:

```js
importScripts('workerpool.js');
```


## Use

### Offload functions dynamically

In the following example there is a function `add`, which is offloaded dynamically to a worker to be executed for a given set of arguments.

**myApp.js**
```js
var workerpool = require('workerpool');
var pool = workerpool.pool();

function add(a, b) {
  return a + b;
}

pool.exec(add, [3, 4])
    .then(function (result) {
      console.log('result', result); // outputs 7
    })
    .catch(function (err) {
      console.error(err);
    })
    .then(function () {
      pool.terminate(); // terminate all workers when done
    });
```

Note that both function and arguments must be static and stringifiable, as they need to be send to the worker in a serialized form. In case of large functions or function arguments, the overhead of sending the data to the worker can be significant.


### Dedicated workers

A dedicated worker can be created in a separate script, and then used via a worker pool.

**myWorker.js**
```js
var workerpool = require('workerpool');

// a deliberately inefficient implementation of the fibonacci sequence
function fibonacci(n) {
  if (n < 2) return n;
  return fibonacci(n - 2) + fibonacci(n - 1);
}

// create a worker and register public functions
workerpool.worker({
  fibonacci: fibonacci
});
```

This worker can be used by a worker pool:

**myApp.js**
```js
var workerpool = require('workerpool');

// create a worker pool using an external worker script
var pool = workerpool.pool(__dirname + '/myWorker.js');

// run registered functions on the worker via exec
pool.exec('fibonacci', [10])
    .then(function (result) {
      console.log('Result: ' + result); // outputs 55
    })
    .catch(function (err) {
      console.error(err);
    })
    .then(function () {
      pool.terminate(); // terminate all workers when done
    });

// or run registered functions on the worker via a proxy:
pool.proxy()
    .then(function (worker) {
      return worker.fibonacci(10);
    })
    .then(function (result) {
      console.log('Result: ' + result); // outputs 55
    })
    .catch(function (err) {
      console.error(err);
    })
    .then(function () {
      pool.terminate(); // terminate all workers when done
    });
```

Worker can also initialize asynchronously:

**myAsyncWorker.js**
```js
define(['workerpool/dist/workerpool'], function(workerpool) {

  // a deliberately inefficient implementation of the fibonacci sequence
  function fibonacci(n) {
    if (n < 2) return n;
    return fibonacci(n - 2) + fibonacci(n - 1);
  }

  // create a worker and register public functions
  workerpool.worker({
    fibonacci: fibonacci
  });

});
```

## Examples

Examples are available in the examples directory:

[https://github.com/josdejong/workerpool/tree/master/examples](https://github.com/josdejong/workerpool/tree/master/examples)


## API

The API of workerpool consists of two parts: a function `workerpool.pool` to create a worker pool, and a function `workerpool.worker` to create a worker.

### pool

A workerpool can be created using the function `workerpool.pool`:

`workerpool.pool([script: string] [, options: Object]) : Pool`

When a `script` argument is provided, the provided script will be started as a dedicated worker. When no `script` argument is provided, a default worker is started which can be used to offload functions dynamically via `Pool.exec`. Note that on node.js, `script` must be an absolute file path like `__dirname + '/myWorker.js'`. In a browser environment, `script` can also be a data URL like `'data:application/javascript;base64,...'`. This allows embedding the bundled code of a worker in your main application. See `examples/embeddedWorker` for a demo.

The following options are available:

- `minWorkers: number | 'max'`. The minimum number of workers that must be initialized and kept available. Setting this to `'max'` will create `maxWorkers` default workers (see below).
- `maxWorkers: number`. The default number of maxWorkers is the number of CPU's minus one. When the number of CPU's could not be determined (for example in older browsers), `maxWorkers` is set to 3.
- `maxQueueSize: number`. The maximum number of tasks allowed to be queued. Can be used to prevent running out of memory. If the maximum is exceeded, adding a new task will throw an error. The default value is `Infinity`.
- `workerType: 'auto' | 'web' | 'process' | 'thread'`.
  - In case of `'auto'`, workerpool will automatically pick a suitable type of worker: when in a browser environment, `'web'` will be used. When in a node.js environment, `worker_threads` will be used if available (Node.js >= 11.7.0), else `child_process` will be used.
  - In case of `'web'`, a Web Worker will be used. Only available in a browser environment.
  - In case of `'process'` (default), `child_process` will be used. Only available in a node.js environment.
  - In case of `'thread'`, `worker_threads` will be used. If `worker_threads` are not available, an error is thrown. Only available in a node.js environment.

A worker pool contains the following functions:

- `Pool.exec(method: Function | string, params: Array | null) : Promise.<*, Error>`<br>
  Execute a function on a worker with given arguments.
  - When `method` is a string, a method with this name must exist at the worker and must be registered to make it accessible via the pool. The function will be executed on the worker with given parameters.
  - When `method` is a function, the provided function `fn` will be stringified, send to the worker, and executed there with the provided parameters. The provided function must be static, it must not depend on variables in a surrounding scope.

- `Pool.proxy() : Promise.<Object, Error>`<br>
  Create a proxy for the worker pool. The proxy contains a proxy for all methods available on the worker. All methods return promises resolving the methods result.

- `Pool.stats() : Object`<br>
   Retrieve statistics on workers, and active and pending tasks.

   Returns an object containing the following properties:

   ```js
   {
     totalWorkers: 0,
     busyWorkers: 0,
     idleWorkers: 0,
     pendingTasks: 0,
     activeTasks: 0
   }
   ```

- `Pool.terminate([force: boolean [, timeout: number]])`

  If parameter `force` is false (default), workers will finish the tasks they are working on before terminating themselves. When `force` is true, all workers are terminated immediately without finishing running tasks. If `timeout` is provided, worker will be forced to terminal when the timeout expires and the worker has not finished.

- `Pool.clear([force: boolean])`<br>
  *Deprecated: use `Pool.terminate` instead*<br>.
  Clear all workers from the pool. If parameter `force` is false (default), workers will finish the tasks they are working on before terminating themselves. When `force` is true, all workers are terminated immediately without finishing running tasks.

The function `Pool.exec` and the proxy functions all return a `Promise`. The promise has the following functions available:

- `Promise.then(fn: Function.<result: *>)`<br>
  Get the result of the promise once resolve.
- `Promise.catch(fn: Function.<error: Error>)`<br>
  Get the error of the promise when rejected.
- `Promise.cancel()`<br>
  A running task can be cancelled. The worker executing the task is enforced to terminate immediately.
  The promise will be rejected with a `Promise.CancellationError`.
- `Promise.timeout(delay: number)`<br>
  Cancel a running task when it is not resolved or rejected withing given delay in milliseconds. The timer will start when the task is actually started, not when the task is created and queued.
  The worker executing the task is enforced to terminate immediately.
  The promise will be rejected with a `Promise.TimeoutError`.

Example usage:

```js
var workerpool = require('workerpool');

function add(a, b) {
  return a + b;
}

var pool1 = workerpool.pool();

// offload a function to a worker
pool1.exec(add, [2, 4])
    .then(function (result) {
      console.log(result); // will output 6
    })
    .catch(function (err) {
      console.error(err);
    });

// create a dedicated worker
var pool2 = workerpool.pool(__dirname + '/myWorker.js');

// supposed myWorker.js contains a function 'fibonacci'
pool2.exec('fibonacci', [10])
    .then(function (result) {
      console.log(result); // will output 55
    })
    .catch(function (err) {
      console.error(err);
    });

// create a proxy to myWorker.js
pool2.proxy()
    .then(function (myWorker) {
      return myWorker.fibonacci(10)
    })
    .then(function (result) {
      console.log(result); // will output 55
    })
    .catch(function (err) {
      console.error(err);
    });

// create a pool with a specified maximum number of workers
var pool3 = workerpool.pool({maxWorkers: 7});
```


### worker

A worker is constructed as:

`workerpool.worker([methods: Object.<String, Function>])`

Argument `methods` is optional can can be an object with functions available in the worker. Registered functions will be available via the worker pool.

Example usage:

```js
// file myWorker.js
var workerpool = require('workerpool');

function add(a, b) {
  return a + b;
}

function multiply(a, b) {
  return a * b;
}

// create a worker and register functions
workerpool.worker({
  add: add,
  multiply: multiply
});
```

Asynchronous results can be handled by returning a Promise from a function in the worker:

```js
// file myWorker.js
var workerpool = require('workerpool');

function timeout(delay) {
  return new Promise(function (resolve, reject) {
    setTimeout(resolve, delay)
  });
}

// create a worker and register functions
workerpool.worker({
  timeout: timeout
});
```

### Utilities

Following properties are available for convenience:

- **platform**: The Javascript platform. Either *node* or *browser*
- **isMainThread**: Whether the code is running in main thread or not (Workers)
- **cpus**: The number of CPUs/cores available


## Roadmap

- Implement functions for parallel processing: `map`, `reduce`, `forEach`,
  `filter`, `some`, `every`, ...
- Implement graceful degradation on old browsers not supporting webworkers:
  fallback to processing tasks in the main application.
- Implement session support: be able to handle a series of related tasks by a
  single worker, which can keep a state for the session.


## Related libraries

- https://github.com/learnboost/cluster
- https://github.com/adambom/parallel.js
- https://github.com/padolsey/operative
- https://github.com/calvinmetcalf/catiline
- https://github.com/Unitech/pm2
- https://github.com/godaddy/node-cluster-service
- https://github.com/ramesaliyev/EasyWebWorker
- https://github.com/rvagg/node-worker-farm


## Build

First clone the project from github:

    git clone git://github.com/josdejong/workerpool.git
    cd workerpool

Install the project dependencies:

    npm install

Then, the project can be build by executing the build script via npm:

    npm run build

This will build the library workerpool.js and workerpool.min.js from the source
files and put them in the folder dist.


## Test

To execute tests for the library, install the project dependencies once:

    npm install

Then, the tests can be executed:

    npm test

To test code coverage of the tests:

    npm run coverage

To see the coverage results, open the generated report in your browser:

    ./coverage/lcov-report/index.html


## Publish

- Describe changes in HISTORY.md
- Update version in package.json, run `npm install` to update it in `package-lock.json` too.
- Push to github
- Deploy to npm via `npm publish`
- Add a git tag with the version number like:
  ```
  git tag v1.2.3
  git push --tags
  ```


## License

Copyright (C) 2014-2019 Jos de Jong <wjosdejong@gmail.com>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
