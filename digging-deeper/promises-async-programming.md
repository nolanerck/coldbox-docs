---
description: 'ColdBox Promises, async programming and parallel computations'
---

# Async Programming

## Introduction

ColdBox 6 introduces the concept of asynchronous and parallel programming using Futures and Executors for ColdFusion \(CFML\). We leverage the entire arsenal in the JDK to bring you a wide array of features for your applications. From the ability to create asynchronous pipelines, to parallel work loads, work queues, and scheduled tasks.

Our async package `coldbox.system.async` is also available for all the standalone libraries: WireBox, CacheBox, and LogBox. This means that you can use the async capabilities in ANY ColdFusion \(CFML\) application, not only ColdBox HMVC applications.

> We leverage Java Executors, CompletableFutures and much more classes from the concurrent packages in the JDK.

## AsyncManager

We have created a manager for leveraging all the async/parallel capabilities in ColdBox. We lovingly call it the ColdBox `AsyncManager`. From this manager you will be able to create async pipelines, simple futures, executors and much more.

### What are ColdBox Futures?

A ColdBox future is used for async programming where you can register a task that will execute in a non-blocking approach and trigger dependant computations which could also be asynchronous. This Future object can then be used to monitor the execution of the task and create rich completion/combining pipelines upon the results of such tasks. You can still use a `get()` blocking operation, but that is an oversimplistic approach to async programming because you are ultimately blocking to get the result.

ColdBox futures are backed by Java's `CompletableFuture` API, so the majority of things will apply as well; even Java developers will feel at home. It will allow you to create rich pipelines for creating multiple Futures, chaining, composing and combining results.

#### Why Use Them?

You might be asking yourself, why should I leverage ColdBox futures instead of traditional cfthreads or even the CFML engine's `runAsync()`. Let's start with the first issue, using ColdBox futures instead of `cfthread`.

#### `cfthread` vs ColdBox Futures

`cfthreads` are an oversimplictic approach to async computations. It allows you to spawn a thread backed by a Java Runnable and then either wait or not for it to complete. You then must use the `thread` scope or other scopes to move data around, share data, and well it can get out of hand very quickly. Here are some issues:

* Too oversimplistic
* No concept of a completion stage pipeline
* No control of what executor runs the task
* No way to trap the exceptions and recover
* No way to do parallel executions
* No way to get a result from the computation, except by using a shared scope
* You must track, name and pull information from the threads
* etc.

You get the picture. They exist, but they are not easy to deal with and the api for it is too simplistic.

#### `runAsync()` vs ColdBox Futures

ColdFusion 2018 and Lucee 5 both have introduced the concept of async programming via their `runAsync()` function. Lucee also has the concept of executing collections in parallel via the `each(), map(), filter()` operations as well. However, it is plagued with issues and an over-simplistic approach to async/parallel programming, yet again. Here are a list of deficiencies of their current implementations:

* Backed by a custom wrapper to `java.util.concurrent.Future` and not Completable Futures
* Simplistic error handler with no way to recover or continue executing pipelines after an exception
* No way to choose or reuse the executor to run the initial task in 
* No way to choose or reuse the executor to run the sub-sequent `then()` operations.  Lucee actually creates a new `singleThreadExecutor()` for EVERY `then()` operation.
* No way to operate on multiple futures at once
* No way to have one future win against multiple future operations
* No way to combine futures
* No way to compose futures
* No ability to schedule tasks
* No ability to run period tasks
* No ability to delay the execution of tasks
* Only works with closures, does not work on actually calling component methods
* And so much more

### What are Executors?

All of our futures execute in the server's common `ForkJoin` pool the JDK provides. However, the JDK since version 8 provides you a framework for simplifying the execution of asynchronous tasks. It can automatically provide you with a pool of threads and a simple API for assigning tasks or work loads to them. We have bridged the gap between Java and ColdFusion and now allow you to leverage all the functionality of the framework in your applications. You can create many types of executors and customized thread pools, so your work loads can use them.

Some resources:

* [https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html](https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html)
* [https://www.baeldung.com/java-executor-service-tutorial](https://www.baeldung.com/java-executor-service-tutorial)
* [https://winterbe.com/posts/2015/04/07/java8-concurrency-tutorial-thread-executor-examples/](https://winterbe.com/posts/2015/04/07/java8-concurrency-tutorial-thread-executor-examples/)

### Injection/Retrieval

The manager will be registered in WireBox as `AsyncManager@ColdBox` or can be retrieved from the ColdBox main controller: `controller.getAsyncManager()`.

```javascript
property name="async" inject="asyncManager@coldbox";

controller.getAsyncManager();
```

The super type has a new `async()` method that returns to you the instance of the `AsyncManager` so you can execute async/parallel operations as well.

```javascript
function index( event, rc, prc ){
    async().newFuture();
}
```

## Async Pipelines & Futures

### Creation Methods

You will be able to create async pipelines and futures by using the following `AsyncManager` creation methods:

* `newFuture( [task], [executor] ):Future` : Returns a ColdBox Future. You can pass an optional task \(closure/udf\) and even an optional executor.
* `newCompletedFuture( value ):Future` : Returns a new future that is already completed with the given value.

> Please note that some of the methods above will return a ColdBox `Future` object that is backed by Java's CompletableFuture \([https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)\)

Here are the method signatures for the methods above:

```javascript
/**
 * Create a new ColdBox future backed by a Java completable future
 *
 * @value The actual closure/lambda/udf to run with or a completed value to seed the future with
 * @executor A custom executor to use with the future, else use the default
 * @debug Add debugging to system out or not, defaults is false
 * @loadAppContext Load the CFML engine context into the async threads or not, default is yes.
 *
 * @return ColdBox Future completed or new
 */
Future function newFuture(
    any value,
    any executor,
    boolean debug          = false,
    boolean loadAppContext = true
)

/**
 * Create a completed ColdBox future backed by a Java Completable Future
 *
 * @value The value to complete the future with
 * @debug Add debugging to system out or not, defaults is false
 * @loadAppContext Load the CFML engine context into the async threads or not, default is yes.
 *
 * @return ColdBox Future completed
 */
Future function newCompletedFuture(
    required any value,
    boolean debug          = false,
    boolean loadAppContext = true
)
```

### Usage

There are two ways to start async computations with futures:

1. Via the `newFuture()` constructor
2. Via the `run()` method

The constructor is the shortcut approach and only allows for closures to be defined as the task. The `run()` methods allows you to pass a CFC instance and a `method` name, which will then call that method as the initial computation.

```javascript
// Constructor
newFuture( () => newUser() )

// Run Operations
newFuture().run( () => newUser() )
newFuture().run( supplier : userService, method : "newUser" )
```

Here are the `run()` method signatures:

```javascript
/**
 * Executes a runnable closure or component method via Java's CompletableFuture and gives you back a ColdBox Future:
 *
 * - This method calls `supplyAsync()` in the Java API
 * - This future is asynchronously completed by a task running in the ForkJoinPool.commonPool() with the value obtained by calling the given Supplier.
 *
 * @supplier A CFC instance or closure or lambda or udf to execute and return the value to be used in the future
 * @method If the supplier is a CFC, then it executes a method on the CFC for you. Defaults to the `run()` method
 * @executor An optional executor to use for asynchronous execution of the task
 *
 * @return The new completion stage (Future)
 */
Future function run(
    required supplier,
    method       = "run",
    any executor = variables.executor
)
```

## Parallel Computations

Here are some of the methods that will allow you to do parallel computations. Please note that the `asyncManager()` has shortcuts to these methods, but we always recommend using them via a new future, because then you can have further constructor options like: custom executor, debugging, loading CFML context and much more.

* `all( a1, a2, ... ):Future` : This method accepts an infinite amount of future objects,  closures or an array of closures/futures in order to execute them in parallel.  It will return a future that when you call `get()` on it, it will retrieve an array of the results of all the operations.
* `allApply( items, fn, executor ):array` : This function can accept an array of items or a struct of items of any type and apply a function to each of the item's in parallel.  The `fn` argument receives the appropriate item and must return a result.  Consider this a parallel `map()` operation.
* `anyOf( a1, a2, ... ):Future` : This method accepts an infinite amount of future objects, closures or an array of closures/futures and will execute them in parallel. However, instead of returning all of the results in an array like `all()`, this method will return the future that executes the fastest! Race Baby!
* `withTimeout( timeout, timeUnit )` : Apply a timeout to `all()` or `allApply()` operations.  The `timeUnit` can be: days, hours, microseconds, milliseconds, minutes, nanoseconds, and seconds. The default is milliseconds. 

> Please note that some of the methods above will return a ColdBox `Future` object that is backed by Java's CompletableFuture \([https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)\)

Here are the method signatures for the methods above, which you can call from the `asyncManager` or a newly created future.

```javascript
/**
 * This method accepts an infinite amount of future objects, closures or an array of future objects/closures
 * in order to execute them in parallel.  It will return back to you a future that will return back an array
 * of results from every future that was executed. This way you can further attach processing and pipelining
 * on the constructed array of values.
 *
 * <pre>
 * results = all( f1, f2, f3 ).get()
 * all( f1, f2, f3 ).then( (values) => logResults( values ) );
 * </pre>
 *
 * @result A future that will return the results in an array
 */
Future function all(){

/**
 * This function can accept an array of items or a struct and apply a function
 * to each of the item's in parallel.  The `fn` argument receives the appropriate item
 * and must return a result.  Consider this a parallel map() operation
 *
 * <pre>
 * // Array
 * allApply( items, ( item ) => item.getMemento() )
 * // Struct: The result object is a struct of `key` and `value`
 * allApply( data, ( item ) => item.key & item.value.toString() )
 * </pre>
 *
 * @items An array to process
 * @fn The function that will be applied to each of the array's items
 * @executor The custom executor to use if passed, else the forkJoin Pool
 *
 * @return An array with the items processed
 */
any function allApply( any items, required fn, executor ){

/**
 * This method accepts an infinite amount of future objects or closures and will execute them in parallel.
 * However, instead of returning all of the results in an array like all(), this method will return
 * the future that executes the fastest!
 *
 * <pre>
 * // Let's say f2 executes the fastest!
 * f2 = anyOf( f1, f2, f3 )
 * </pre>
 *
 * @return The fastest executed future
 */
Future function anyOf()

/**
 * This method seeds a timeout into this future that can be used by the following operations:
 *
 * - all()
 * - allApply()
 *
 * @timeout The timeout value to use, defaults to forever
 * @timeUnit The time unit to use, available units are: days, hours, microseconds, milliseconds, minutes, nanoseconds, and seconds. The default is milliseconds
 *
 * @returns This future
 */
Future function withTimeout( numeric timeout = 0, string timeUnit = "milliseconds" )
```

Here are some examples:

```javascript
// Let's find the fastest dns server
var f = asyncManager().anyOf( ()=>dns1.resolve(), ()=>dns2.resolve() );

// Let's process some data
var data = [1,2, ... 100 ];
var results = asyncManager().all( data );

// Process multiple futures
var f1 = asyncManager.newFuture( function(){
    return "hello";
} );
var f2 = asyncManager.newFuture( function(){
    return "world!";
} );
var aResults = asyncManager.newFuture()
    .withTimeout( 5 )
    .all( f1, f2 );

// Process mementos for an array of objects
function index( event, rc, prc ){
    return async().allApply(
        orderService.findAll(),
        ( order ) => order.getMemento()
    );
}
```

### Custom Executors

Please also note that you can choose your own Executor for the parallel computations by passing the `executor` via the `newFuture()` method.

```javascript
var data = [ 1 ... 5000 ];
var results = newFuture( executor : asyncManager.$executors.newCachedThreadPool() )
    .all( data );

// Process mementos for an array of objects on a custom pool
function index( event, rc, prc ){
    return async().allApply(
        orderService.findAll(),
        ( order ) => order.getMemento(),
        async().$executors.newFixedThreadPool( 50 )
    );
}
```

## Executors

The ColdBox AsyncManager will allow you to register and manage different types of executors that can execute your very own tasks! Each executor acts as a singleton and can be configured uniquely. \(See: [https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html)\)

You can also create executors on the fly so your async futures can use them as well for ONLY that execution stage.

> Nice video explaining the Java Executor Service: [https://www.youtube.com/watch?v=6Oo-9Can3H8&t=2s](https://www.youtube.com/watch?v=6Oo-9Can3H8&t=2s)

### Executor Types

The types that we currently support are:

* `fixed` : By default it will build one with **20** threads on it. Great for multiple task execution and worker processing.
* `single` : A great way to control that submitted tasks will execute in the order of submission much like a FIFO queue \(First In First Out\).
* `cached` : An unbounded pool where the number of threads will grow according to the tasks it needs to service. The threads are killed by a default 60 second timeout if not used and the pool shrinks back to 0.
* `scheduled` : A pool to use for scheduled tasks that can run one time or periodically. The default scheduled task queue has **20** threads for processing.

Please note that each executor is unique in its way of operation, so make sure you read about each type in the JavaDocs or watch this amazing video: [https://www.youtube.com/watch?v=sIkG0X4fqs4](https://www.youtube.com/watch?v=sIkG0X4fqs4)

### Executor Methods

Here are the methods you can use for registering and managing singleton executors in your application:

* `newExecutor()` : Create and register an executor according to passed arguments.  If the executor exists, just return it.
* `newScheduledExecutor()` : Create a scheduled executor according to passed arguments. Shortcut to `newExecutor( type: "scheduled" )`
* `newSingleExecutor()` : Create a single thread executor. Shortcut to `newExecutor( type: "single", threads: 1 )`
* `newCachedExecutor()` : Create a cached thread executor. Shortcut to `newExecutor( type: "cached" )`
* `getExecutor()` : Get a registered executor registerd in this async manager
* `getExecutorNames()` : Get the array of registered executors in the system
* `hasExecutor()` : Verify if an executor exists
* `deleteExecutor()` : Delete an executor from the registry, if the executor has not shutdown, it will shutdown the executor for you using the shutdownNow\(\) event
* `shutdownExecutor()` : Shutdown an executor or force it to shutdown, you can also do this from the Executor themselves. If an un-registered executor name is passed, it will ignore it
* `shutdownAllExecutors()` : Shutdown all registered executors in the system
* `getExecutorStatusMap()` : Returns a structure of status maps for every registered executor in the manager. This is composed of tons of stats about the executor.

And here are the full signatures:

```javascript
/**
 * Creates and registers an Executor according to the passed name, type and options.
 * The allowed types are: fixed, cached, single, scheduled with fixed being the default.
 *
 * You can then use this executor object to submit tasks for execution and if it's a
 * scheduled executor then actually execute scheduled tasks.
 *
 * Types of Executors:
 * - fixed : By default it will build one with 20 threads on it. Great for multiple task execution and worker processing
 * - single : A great way to control that submitted tasks will execute in the order of submission: FIFO
 * - cached : An unbounded pool where the number of threads will grow according to the tasks it needs to service. The threads are killed by a default 60 second timeout if not used and the pool shrinks back
 * - scheduled : A pool to use for scheduled tasks that can run one time or periodically
 *
 * @see https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html
 *
 * @name The name of the executor used for registration
 * @type The type of executor to build fixed, cached, single, scheduled
 * @threads How many threads to assign to the thread scheduler, default is 20
 * @debug Add output debugging
 * @loadAppContext Load the CFML App contexts or not, disable if not used
 *
 * @return The ColdBox Schedule class to work with the schedule: coldbox.system.async.tasks.Executor
 */
Executor function newExecutor(
    required name,
    type                   = "fixed",
    numeric threads        = this.$executors.DEFAULT_THREADS,
    boolean debug          = false,
    boolean loadAppContext = true
)

/**
 * Shortcut to newExecutor( type: "scheduled" )
 */
Executor function newScheduledExecutor(
    required name,
    numeric threads        = this.$executors.DEFAULT_THREADS,
    boolean debug          = false,
    boolean loadAppContext = true
)

/**
 * Shortcut to newExecutor( type: "single", threads: 1 )
 */
Executor function newSingleExecutor(
    required name,
    boolean debug          = false,
    boolean loadAppContext = true
)

/**
 * Shortcut to newExecutor( type: "cached" )
 */
Executor function newCachedExecutor(
    required name,
    numeric threads        = this.$executors.DEFAULT_THREADS,
    boolean debug          = false,
    boolean loadAppContext = true
)

/**
 * Get a registered executor registerd in this async manager
 *
 * @name The executor name
 *
 * @throws ExecutorNotFoundException
 * @return The executor object: coldbox.system.async.tasks.Executor
 */
Executor function getExecutor( required name )

/**
 * Get the array of registered executors in the system
 *
 * @return Array of names
 */
array function getExecutorNames()

/**
 * Verify if an executor exists
 *
 * @name The executor name
 */
boolean function hasExecutor( required name )

/**
 * Delete an executor from the registry, if the executor has not shutdown, it will shutdown the executor for you
 * using the shutdownNow() event
 *
 * @name The scheduler name
 */
AsyncManager function deleteExecutor( required name )

/**
 * Shutdown an executor or force it to shutdown, you can also do this from the Executor themselves.
 * If an un-registered executor name is passed, it will ignore it
 *
 * @force Use the shutdownNow() instead of the shutdown() method
 */
AsyncManager function shutdownExecutor( required name, boolean force = false )

/**
 * Shutdown all registered executors in the system
 *
 * @force By default (false) it gracefullly shuts them down, else uses the shutdownNow() methods
 *
 * @return AsyncManager
 */
AsyncManager function shutdownAllExecutors( boolean force = false )

/**
 * Returns a structure of status maps for every registered executor in the
 * manager. This is composed of tons of stats about the executor
 *
 * @name The name of the executor to retrieve th status map ONLY!
 *
 * @return A struct of metadata about the executor or all executors
 */
struct function getExecutorStatusMap( name )
```

### Single Use Executors

If you just want to use executors for different completion stages and then discard them, you can easily do so by using the `$executors` public component in the `AsyncManager`. This object can be found at: `coldbox.system.async.util.Executors`:

```javascript
asyncManager.$executors.{creationMethod}()
```

The available creation methods are:

```javascript
this.DEFAULT_THREADS = 20;
/**
 * Creates a thread pool that reuses a fixed number of threads operating off a shared unbounded queue.
 *
 * @see https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html
 *
 * @threads The number of threads in the pool, defaults to 20
 *
 * @return ExecutorService: The newly created thread pool
 */
function newFixedThreadPool( numeric threads = this.DEFAULT_THREADS )

/**
 * Creates a thread pool that creates new threads as needed, but will
 * reuse previously constructed threads when they are available.
 *
 * @see https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html
 *
 * @return ExecutorService: The newly created thread pool
 */
function newCachedThreadPool()

/**
 * Creates a thread pool that can schedule commands to run after a given delay,
 * or to execute periodically.
 *
 * @see https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html
 *
 * @corePoolSize The number of threads to keep in the pool, even if they are idle, default is 20
 *
 * @return ScheduledExecutorService: The newly created thread pool
 */
function newScheduledThreadPool( corePoolSize = this.DEFAULT_THREADS )
```

This way you can use them and discard them upon further processing.

```javascript
// Create an IO bound pool with 100 threads to process my data
var dataQueue = [ data, data ... data1000 ];
var results = asyncManager
    .newFuture( executor : asyncManager.$executors.newFixedThreadPool( 100 ) )
    .allApply( dataQueue, ( item ) => dataService.process( item )  );

// Create two types of thread pools: 1) CPU intesive, 2) IO Intensive
var cpuBound = asyncManager.$executors.newFixedThreadPool( 4 );
var ioBound = asyncManager.$executors.newFixedThreadPool( 50 );

// Process an order with different executor pools
newFuture( () => orderService.getOrder(), ioBound )
    .thenAsync( (order) => enrichOrder( order ), cpuBound )
    .thenAsync( (order) => performPayment( order ), ioBound )
    .thenAsync( (order) => dispatchOrder( order ), cpuBound )
    .thenAsync( (order) => sendConfirmation( order ), ioBound );
```

### ColdBox Task Scheduler

Every ColdBox application will register a task scheduler called `coldbox-tasks` which ColdBox internal services can leverage it for tasks and schedules. It is configured with 20 threads and using the `scheduled` type. You can leverage if you wanted to for your own tasks as well.

### ColdBox Application Executor Registration

We have also added a new configuration struct called `executors` which you can use to register global executors or per-module executors.

```javascript
// In your config/ColdBox.cfc
function configure(){

    executors = {
        "name" : {
            "type" : "fixed",
            "threads" : 100
        }
    };

}
```

Each executor registration is done as a struct with the name of the executor and at most two settings:

* `type` : The executor type you want to register
* `threads` : The number of threads to assign to the executor

#### ModuleConfig.cfc

You can also do the same at a per-module level in your module's `ModuleConfig.cfc`.

```javascript
// In your ModuleConfig.cfc
function configure(){

    executors = {
        "name" : {
            "type" : "fixed",
            "threads" : 100
        }
    };

}
```

ColdBox will register your executors upon startup and un-register and shut them down when the module is unloaded.

### WireBox Injection DSL

We have extended WireBox so you can inject registered executors using the following DSL: `executors:{name}`

```javascript
// Inject an executor called `taskScheduler` the same name as the property
property name="taskScheduler" inject="executor";
// Inject the `coldbox-tasks` as `taskScheduler`
property name="taskScheduler" inject="executor:coldbox-tasks";
```

