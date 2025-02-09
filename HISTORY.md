# workerpool history
https://github.com/josdejong/workerpool


## 2019-08-25, version 5.0.0

- Deprecated option `nodeWorker` and created a new, more extensive option
  `workerType` giving full control over the selected type of worker.
  Added new option `'web'` to enforce use a Web Worker. See #85, #74.
- In a node.js environment, the default `workerType` is changed from
  `'process'` to `'thread'`. See #85, #50.
- Improved detection of environment (`browser` or `node`), fixing wrong
  detection in a Jest test environment. See #85.


## 2019-08-21, version 4.0.0

- Pass argument `--max-old-space-size` to child processes. Thanks @patte.
- Removed redundant dependencies, upgraded all devDependencies.
- Fixed Webpack issues of missing modules `child_process` and `worker_threads`.
  See #43.
- Bundled library changed due to the upgrade to Webpack 4. This could possibly
  lead to breaking changes.
- Implemented new option `maxQueueSize`. Thanks @colomboe.
- Fixed exiting workers when the parent process is killed. Thanks @RogerKang.
- Fixed #81: Option `minWorkers: 'max'` not using the configured `maxWorkers`.
- Fixed not passing `nodeWorker` to workers initialized when creating a pool.
  Thanks @spacelan.
- Internal restructure of the code: moved from `lib` to `src`.


## 2019-03-12, version 3.1.2

- Improved error message when a node.js worker unexpectedly exits (see #58).
  Thanks @stefanpenner.

- Allocate debug ports safely, this fixes an issue cause workers to exit
  unexpectedly if more then one worker pool is active, and the process is
  started with a debugger (`node debug` or `node --inspect`).
  Thanks @stefanpenner.


## 2019-02-25, version 3.1.1

- Fix option `nodeWorker: 'auto'` not using worker threads when available.
  Thanks @stefanpenner.


## 2019-02-17, version 3.1.0

- Implemented support for using `worker_threads` in Node.js, via the new option
  `nodeWorker: 'thread'`. Thanks @stefanpenner.


## 2018-12-11, version 3.0.0

- Enable usage in ES6 Webpack projects.
- Dropped support for AMD module system.


## 2018-09-12, version 2.3.3

- Fixed space in license field in `package.json`. Thanks @sagotsky.


## 2018-09-08, version 2.3.2

- Add licence field to `package.json`. Thanks @greyd.


## 2018-07-24, version 2.3.1

- Fixed bug where tasks that are cancelled in a Pool's queue
  causes following tasks to not run. Thanks @greemo.


## 2017-09-30, version 2.3.0

- New method `Pool.terminate(force, timeout)` which will replace
  `Pool.clear(force)`. Thanks @jimsugg.
- Fixed issue with never terminating zombie child processes.
  Thanks @jimsugg.


## 2017-08-20, version 2.2.4

- Fixed a debug issue: look for `--inspect` within argument strings,
  instead of exact match. Thanks @jimsugg.


## 2017-08-19, version 2.2.3

- Updated all examples to neatly include `.catch(...)` callbacks.


## 2017-07-08, version 2.2.2

- Fixed #25: timer of a timeout starting when the task is created
  instead of when the task is started. Thanks @eclipsesk for input.


## 2017-05-07, version 2.2.1

- Fixed #2 and #19: support for debugging child processes. Thanks @tptee.


## 2016-11-26, version 2.2.0

- Implemented #18: method `pool.stats()`.


## 2016-10-11, version 2.1.0

- Implemented support for registering the workers methods asynchronously.
  This enables asynchronous initialization of workers, for example when
  using AMD modules. Thanks @natlibfi-arlehiko.
- Implemented environment variables `platform`, `isMainThread`, and `cpus`.
  Thanks @natlibfi-arlehiko.
- Implemented option `minWorkers`. Thanks @sergei202.


## 2016-09-18, version 2.0.0

- Replaced conversion of Error-objecting using serializerr to custom
  implementation to prevent issues with serializing/deserializing functions.
  This conversion implementation loses the prototype object which means that
  e.g. 'TypeError' will become just 'Error' in the main code. See #8.
  Thanks @natlibfi-arlehiko.


## 2016-09-12, version 1.3.1

- Fix for a bug in PhantomJS (see #7). Thanks @natlibfi-arlehiko.


## 2016-08-21, version 1.3.0

- Determine `maxWorkers` as the number of CPU's minus one in browsers too. See #6.


## 2016-06-25, version 1.2.1

- Fixed #5 error when loading via AMD or bundling using Webpack.


## 2016-05-22, version 1.2.0

- Implemented serializing errors with stacktrace. Thanks @mujx.


## 2016-01-25, version 1.1.0

- Added an error message when wrongly calling `pool.proxy`.
- Fixed function `worker.pool` not accepting both a script and options. See #1.
  Thanks @freund17.


## 2014-05-29, version 1.0.0

- Merged function `Pool.run` into `Pool.exec`, simplifying the API.


## 2014-05-14, version 0.2.0

- Implemented support for cancelling running tasks.
- Implemented support for cancelling running tasks after a timeout.


## 2014-05-07, version 0.1.0

- Implemented support for both node.js and the browser.
- Implemented offloading functions.
- Implemented worker proxy.
- Added docs and examples.


## 2014-05-02, version 0.0.1

- Module name registered at npm.
