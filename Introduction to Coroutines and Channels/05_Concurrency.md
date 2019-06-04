# Concurrency

Kotlin coroutines are extremely cheap in comparison with threads.
Each time when you want to start a new computation asynchronously, you can create a new coroutine.

To start a new coroutine you use one of the main "coroutine builders": `launch`, `async`, or `runBlocking`.
Different libraries can define additional coroutine builders.

`async` starts a new coroutine and returns a `Deferred` object.
`Deferred` represents a concept known by other names as `Future` or `Promise`:
it stores a computation, but it *defers* the moment when you get the final result; 
it *promises* the result sometime in the *future*.

The main difference between `async` and `launch` is that `launch` is used for starting
a computation that isn't expected to return a specific result.
`launch` returns `Job`, which represents the coroutine.
You can wait until it completes by calling `Job.join()`. 

`Deferred` is a generic type which extends `Job`.
Your `async` call can return `Deferred<Int>` or `Deferred<CustomType>`
depending on what the lambda returns (the last expression inside the lambda is the result).

To get the result of a coroutine, you call `await()` on the `Deferred` instance.
While waiting for the result, the coroutine you call this `await` from, suspends:

```run-kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val deferred: Deferred<Int> = async {
        loadData()
    }
    println("waiting...")
    println(deferred.await())
}

suspend fun loadData(): Int {
    println("loading...")
    delay(1000L)
    println("loaded!")
    return 42
}
```

`runBlocking` is used as a bridge between regular and `suspend` functions, between blocking and non-blocking worlds.
It works as an adaptor starting the top-level main coroutine and is intended primarily to be used in `main` functions
and in tests.

If you have a list of deferred objects, you can call `awaitAll` to await the results of all of them:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val deferreds: List<Deferred<Int>> = (1..3).map {
        async {
            delay(1000L * it)
            println("Loading $it")
            it
        }
    }
    val sum = deferreds.awaitAll().sum()
    println("$sum")
}
```

When you start each "contributors" request in a new coroutine,
all the requests are started asynchronously.
A new request can be sent before the result for the previous one received:  

![](./assets/5-concurrency/Concurrency.png)

That brings us approximately the same total loading time as `CALLBACKS` version before.
But you don't need any callbacks.
What's more, `async` explicitly emphasizes in the code which parts run concurrently.  

#### Task

Implement `loadContributorsConcurrent` function in the `Request5Concurrent.kt` file.
Use the previous `loadContributorsSuspend` function.
 
#### Tip

As we'll discuss below, you can only start a new coroutine inside a coroutine scope.
Thus, copy the content of `loadContributorsSuspend` inside `coroutineScope` call,
so that you could call `async` functions there:

```kotlin
suspend fun loadContributorsConcurrent(req: RequestData): List<User> = coroutineScope {
     // ...
}
```

Base the solution on the following scheme:

```kotlin
val deferreds: List<Deferred<List<User>>> = repos.map { repo ->
    async {
        // load contributors for each repo
    }
}
deferreds.awaitAll() // List<List<User>> 
```

#### Solution

You wrap each "contributors" request in `async`.
That creates as many coroutines as the number of repositories we have.
But since it's cheap to create a new coroutine, that's no issue in it.
You can create as many as you need.
 
`async` returns `Deferred<List<User>>`. 
You no longer can use `flatMap`: the `map` result is a list of `Deferred` objects now, not a list of lists.
`awaitAll()` returns `List<List<User>>`, so you simply need to call `flatten().aggregate()` to get the result: 

```kotlin
suspend fun loadContributorsConcurrent(req: RequestData): List<User> = coroutineScope {
    val service = createGitHubService(req.username, req.password)
    val repos = service
        .getOrgRepos(req.org)
        .also { logRepos(req, it) }
        .bodyList()

    val deferreds: List<Deferred<List<User>>> = repos.map { repo ->
        async {
            service.getRepoContributors(req.org, repo.name)
                .also { logUsers(repo, it) }
                .bodyList()
        }
    }
    deferreds.awaitAll().flatten().aggregate()
}
```

Run the code and check the log; you should see that all the coroutines still run on the main UI thread.
We haven't yet employed multithreading in any way,
but already got all the benefits of running coroutines concurrently!

It's very easy to change this code to run "contributors" coroutines on different threads from the common thread pool.
Specify `Dispatchers.Default` as the context argument for `async` function:

```kotlin
async(Dispacthers.Default) { ... }
``` 

`CoroutineDispatcher` determines what thread or threads the corresponding coroutine should be run on.
If you don't specify one as an argument, then `async` will use the dispatcher from the outer scope.

`Dispatchers.Default` represents a shared pool of threads on JVM.
This pool provides a means for parallel execution. 
It consists of as many threads as CPU cores available but still has two threads if there's only one core. 

Modify now the code of the `loadContributorsConcurrent` function so that new coroutines were started on different
threads from the common thread pool.
Also, add additional logging before sending the request:

```kotlin
async(Dispatchers.Default) {
    log("starting loading for ${repo.name}")
    service.getRepoContributors(req.org, repo.name)
        .also { logUsers(repo, it) }
        .bodyList()
}
```

After introducing this change, you can observe in the log that each coroutine can be started on one thread
from the thread pool and resumed on another:

```
1946 [DefaultDispatcher-worker-2 @coroutine#4] INFO  Contributors - starting loading for kotlin-koans
1946 [DefaultDispatcher-worker-3 @coroutine#5] INFO  Contributors - starting loading for dokka
1946 [DefaultDispatcher-worker-1 @coroutine#3] INFO  Contributors - starting loading for ts2kt
...
2178 [DefaultDispatcher-worker-1 @coroutine#4] INFO  Contributors - kotlin-koans: loaded 45 contributors
2569 [DefaultDispatcher-worker-1 @coroutine#5] INFO  Contributors - dokka: loaded 36 contributors
2821 [DefaultDispatcher-worker-2 @coroutine#3] INFO  Contributors - ts2kt: loaded 11 contributors
``` 

For instance, in this log excerpt, `coroutine#4` is started on the thread `worker-2` and continued on the thread `worker-1`.

To run the coroutine only on the main UI thread, you specify `Dispatchers.Main` as an argument:  

```kotlin
launch(Dispatchers.Main) {
    updateResults()
}
```

If the main thread is busy when you start a new coroutine on it,
the coroutine becomes suspended and scheduled for execution on this thread.
The coroutine resumes only when the thread gets free.

Note that it's considered a good practice to use the dispatcher from the outer scope rather than to explicitly specify it
on each end-point.
In our case, when you define `loadContributorsConcurrent` without passing `Dispatchers.Default` as an argument,
you can then call this function in any way: in the context with `Default` dispatcher,
in the context with main UI thread, or in the context with your custom dispatcher.
As we'll see later, when calling `loadContributorsConcurrent` from tests, you can call it in the context with 
`TestCoroutineDispatcher` which simplifies testing. Thus this solution is much more flexible. 

Here's how you specify the dispatcher on the caller-side:

```kotlin
launch(Dispatchers.Default) {
    val users = loadContributorsConcurrent(service, req)
    withContext(Dispatchers.Main) {
        updateResults(users, startTime)
    }
}
```

Apply this change to your project, while letting `loadContributorsConcurrent` start coroutines in the inherited context.
Run the code and make sure that coroutines are executed on the threads from the thread pool. 

`updateResults` should be called on the main UI thread, so we call it with the context of `Dispatchers.Main`.
`withContext` calls the given code with the specified coroutine context, suspends until it completes,
and returns the result.
An alternative but more verbose way to express that would be to start a new coroutine and explicitly wait (by
suspending) until it completes: `launch(context) { ... }.join()`. 

How exactly works "using the dispatcher from the outer scope"?
The more correct way to say that would be "using the dispatcher from the outer scope's context".
What is a coroutine scope and how it's different from a coroutine context?
(These two might be confused).
Why did you need to start new `async` "contributors" coroutines inside `coroutineScope` call?
Let's discuss that next.  
