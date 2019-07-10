# Concurrency

Kotlin coroutines are extremely inexpensive in comparison to threads.
Each time when we want to start a new computation asynchronously, we can create a new coroutine.

To start a new coroutine we can just use one of the main "coroutine builders": `launch`, `async`, or `runBlocking`.
Different libraries can define additional coroutine builders.

`async` starts a new coroutine and returns a `Deferred` object.
`Deferred` represents a concept known by other names such as `Future` or `Promise`:
it stores a computation, but it *defers* the moment you get the final result; 
it *promises* the result sometime in the *future*.

The main difference between `async` and `launch` is that `launch` is used for starting
a computation that isn't expected to return a specific result.
`launch` returns `Job`, which represents the coroutine.
It is possible to wait until it completes by calling `Job.join()`. 

`Deferred` is a generic type which extends `Job`.
An `async` call can return a `Deferred<Int>` or `Deferred<CustomType>`
depending on what the lambda returns (the last expression inside the lambda is the result).

To get the result of a coroutine, we call `await()` on the `Deferred` instance.
While waiting for the result, the coroutine that this `await` is called from, suspends:

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
It works as an adaptor for starting the top-level main coroutine and is intended primarily to be used in `main` functions
and in tests.

To get a better understanding of what's going on in this example, watch the following video:

[Video explaining the coroutines sample](https://youtu.be/zEZc5AmHQhk) 

If there is a list of deferred objects, it is possible to call `awaitAll` to await the results of all of them:

```run-kotlin
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

When starting each "contributors" request in a new coroutine,
all the requests are started asynchronously.
A new request can be sent before the result for the previous one is received:  

![](./assets/5-concurrency/Concurrency.png)

That brings us approximately the same total loading time as the `CALLBACKS` version before.
But without needing any callbacks.
What's more, `async` explicitly emphasizes in the code which parts run concurrently.  

#### Task

Implement a `loadContributorsConcurrent` function in the `Request5Concurrent.kt` file.
Use the previous `loadContributorsSuspend` function.
 
#### Tip

As we'll discuss below, we can only start a new coroutine inside a coroutine scope.
So, copy the content from `loadContributorsSuspend` to the `coroutineScope` call,
so that we can call `async` functions there:

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

We wrap each "contributors" request with `async`.
That will create as many coroutines as number of repositories we have.
But since it's really inexpensive to create a new coroutine, it's not an issue.
We can create as many as we need.
 
`async` returns `Deferred<List<User>>`. 
We can no longer use `flatMap`: the `map` result is a list of `Deferred` objects now, not a list of lists.
`awaitAll()` returns `List<List<User>>`, so we simply need to call `flatten().aggregate()` to get the result: 

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

Run the code and check the log; we can see that all the coroutines still run on the main UI thread.
We haven't yet employed multithreading in any way,
but already we have the benefits of running coroutines concurrently!

It's very easy for us to change this code to run "contributors" coroutines on different threads from the common thread pool.
Specify `Dispatchers.Default` as the context argument for the `async` function:

```kotlin
async(Dispatchers.Default) { ... }
``` 

`CoroutineDispatcher` determines what thread or threads the corresponding coroutine should be run on.
If we don't specify one as an argument, then `async` will use the dispatcher from the outer scope.

`Dispatchers.Default` represents a shared pool of threads on JVM.
This pool provides a means for parallel execution. 
It consists of as many threads as there are CPU cores available, but still it has two threads if there's only one core. 

Modify the code in the `loadContributorsConcurrent` function so that new coroutines are started on different
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

After introducing this change, we can observe in the log that each coroutine can be started on one thread
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

To run the coroutine only on the main UI thread, we should specify `Dispatchers.Main` as an argument:  

```kotlin
launch(Dispatchers.Main) {
    updateResults()
}
```

If the main thread is busy when we start a new coroutine on it,
the coroutine becomes suspended and scheduled for execution on this thread.
The coroutine will only resume when the thread becomes free.

Note that it's considered good practice to use the dispatcher from the outer scope rather than to explicitly specify it
on each end-point.
In our case, when we define `loadContributorsConcurrent` without passing `Dispatchers.Default` as an argument,
we can then call this function in any way: in the context with a `Default` dispatcher,
in the context with the main UI thread, or in the context with a custom dispatcher.
As we'll see later, when calling `loadContributorsConcurrent` from tests, we can call it in the context with 
`TestCoroutineDispatcher` which simplifies testing. Thus this solution is much more flexible. 

Here's how we specify the dispatcher on the caller-side:

```kotlin
launch(Dispatchers.Default) {
    val users = loadContributorsConcurrent(service, req)
    withContext(Dispatchers.Main) {
        updateResults(users, startTime)
    }
}
```

Apply this change to our project, while letting `loadContributorsConcurrent` start coroutines in the inherited context.
Run the code and make sure that the coroutines are executed on the threads from the thread pool. 

`updateResults` should be called on the main UI thread, so we call it with the context of `Dispatchers.Main`.
`withContext` calls the given code with the specified coroutine context, suspends until it completes,
and returns the result.
An alternative but more verbose way to express this would be to start a new coroutine and explicitly wait (by
suspending) until it completes: `launch(context) { ... }.join()`. 

How exactly does "using the dispatcher from the outer scope" work?
A more correct way to say this would be "using the dispatcher from the outer scope's context".
What is a coroutine scope and how is it different from a coroutine context?
(These two might be confused).
Why did we need to start new `async` "contributors" coroutines inside `coroutineScope` call?
Let's discuss this next.  
