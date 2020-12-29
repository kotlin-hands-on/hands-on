# Structured concurrency
 
_Coroutine scope_ is responsible for the structure and parent-child relationships between different coroutines.
We always start new coroutines inside a scope.
_Coroutine context_ stores additional technical information used to run a given coroutine,
like the coroutine custom name, or the dispatcher specifying the threads the coroutine should be scheduled on.

When `launch`, `async`, or `runBlocking` are used to start a new coroutine, they automatically create the corresponding scope.
All these functions take a lambda with receiver as an argument, and the implicit receiver type is the `CoroutineScope`:

```kotlin
launch { /* this: CoroutineScope */
}
```

New coroutines can only be started inside a scope.
`launch` and `async` are declared as extensions to `CoroutineScope`, so an implicit or explicit receiver must always
be passed when we call them.
The coroutine started by `runBlocking` is the only exception: `runBlocking` is defined as a top-level function.
But because it blocks the current thread, it is intended primarily to be used in `main` functions and tests as
a bridge function.

When starting a new coroutine inside `runBlocking`, `launch`, or `async` it is started automatically inside the scope: 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { /* this: CoroutineScope */
    launch { /* ... */ }
    // the same as:    
    this.launch { /* ... */ }
}
```

When we call `launch` inside `runBlocking`, we call it as an extension to the implicit receiver
of the `CoroutineScope` type.
Alternatively, we could explicitly write `this.launch`.

We can say that the nested coroutine (started by `launch` in this example) is a child of the outer coroutine
(started by `runBlocking`). This "parent-child" relationship works through scopes: the child coroutine is started from
the scope corresponding to the parent coroutine.

It is possible to create a new scope without starting a new coroutine.
The `coroutineScope` function does this. 
When we need to start new coroutines in a structured way inside a `suspend` function without access to the outer scope,
for example inside `loadContributorsConcurrent`,
we can create a new coroutine scope which automatically becomes a child of the outer scope that this `suspend` function
is called from.

It's also possible to start a new coroutine from the global scope using `GlobalScope.async` or `GlobalScope.launch`.
This will create a top-level "independent" coroutine.

The mechanism providing the structure of the coroutines is called "structured concurrency".
Let's see what benefits structured concurrency has over global scopes:

* The scope is generally responsible for child coroutines,
and their lifetime is attached to the lifetime of the scope. 
* The scope can automatically cancel child coroutines if something goes wrong or
if a user simply changes their mind and decides to revoke the operation.
* The scope automatically waits for completion of all the child coroutines.
Therefore, if the scope corresponds to a coroutine, then the parent coroutine does not complete until all the coroutines
launched in its scope are complete.

When using `GlobalScope.async` there is no structure that binds several coroutines to a smaller scope.
The coroutines started from the global scope are all independent; 
their lifetime is limited only by the lifetime of the whole application.
It is possible to store a reference to the coroutine started from the global scope and wait for its completion or cancel it
explicitly, but it won't happen automatically as it would with a structured one.

### Cancellation of contributors loading

Let's compare two versions of the `loadContributorsConcurrent` function:
one using `coroutineScope` to start all the child coroutines and the other using `GlobalScope`.
We'll compare how both versions behave when we try to cancel the parent coroutine.

We can add a 3-second delay to all the coroutines sending requests, so that we have enough time to cancel the loading after
the coroutines are started, but before the requests are sent:  

```kotlin
suspend fun loadContributorsConcurrent(service: GitHubService, req: RequestData): List<User> = coroutineScope {
    // ... 
    async {
        log("starting loading for ${repo.name}")
        delay(3000)
        // load repo contributors
    }
    // ...
    result
}
```

Copy the implementation of `loadContributorsConcurrent` to `loadContributorsNotCancellable` (in `Request5NotCancellable.kt`)
and remove the creation of a new `coroutineScope`.
Our `async` calls now fail to resolve, so we need to start them via `GlobalScope.async`:

```kotlin
suspend fun loadContributorsNotCancellable(service: GitHubService, req: RequestData): List<User> {   // #1
    // ... 
    GlobalScope.async {   // #2
        log("starting loading for ${repo.name}")
        delay(3000)
        // load repo contributors
    }
    // ...
    return result  // #3
}
```

The function now returns the result directly, not as the last expression inside the lambda (lines `#1` and `#3`),
and all of the "contributors" coroutines are started inside the `GlobalScope`, not as children of the coroutine scope (line `#2`).  

We can start by running the program and choosing to load the contributors via `CONCURRENT` version, we need to wait until all the "contributors" coroutines are started, and then click on "cancel". Looking at the log, we can see that all the requests were indeed canceled because no results were logged:

```
2896 [AWT-EventQueue-0 @coroutine#1] INFO  Contributors - kotlin: loaded 40 repos
2901 [DefaultDispatcher-worker-2 @coroutine#4] INFO  Contributors - starting loading for kotlin-koans
...
2909 [DefaultDispatcher-worker-5 @coroutine#36] INFO  Contributors - starting loading for mpp-example
/* click on 'cancel' */
/* no requests are sent */
```

Now let's repeat the same procedure but this time choose the `NOT_CANCELLABLE` option:

```
2570 [AWT-EventQueue-0 @coroutine#1] INFO  Contributors - kotlin: loaded 30 repos
2579 [DefaultDispatcher-worker-1 @coroutine#4] INFO  Contributors - starting loading for kotlin-koans
...
2586 [DefaultDispatcher-worker-6 @coroutine#36] INFO  Contributors - starting loading for mpp-example
/* click on 'cancel' */
/* but all the requests are still sent: */
6402 [DefaultDispatcher-worker-5 @coroutine#4] INFO  Contributors - kotlin-koans: loaded 45 contributors
...
9555 [DefaultDispatcher-worker-8 @coroutine#36] INFO  Contributors - mpp-example: loaded 8 contributors
```

Nothing happens! No coroutines are canceled, and all the requests are still sent.

Let's look at how the cancellation is implemented in our "contributors" program.
When the `cancel` button is clicked, we need to explicitly cancel the main "loading" coroutine.
Then it automatically cancels all the child coroutines.

That's how we can cancel the "loading" coroutine on the button click: 

```kotlin
interface Contributors {

    fun loadContributors() {
        // ...
        when (getSelectedVariant()) {
            CONCURRENT -> {
                launch {
                    val users = loadContributorsConcurrent(req)
                    updateResults(users, startTime)
                }.setUpCancellation()      // #1
            }
        }
    }

    private fun Job.setUpCancellation() {
        val loadingJob = this              // #2

        // cancel the loading job if the 'cancel' button was clicked:
        val listener = ActionListener {
            loadingJob.cancel()            // #3
            updateLoadingStatus(CANCELED)
        }
        addCancelListener(listener)

        // update the status and remove the listener after the loading job is completed
    }
}    
```

The `launch` function returns an instance of `Job`.
`Job` stores a reference to the "loading coroutine", which loads all the data and updates the results.
We can call the `setUpCancellation()` extension function on it (line `#1`), passing an instance of `Job` as a receiver.
Another way we could express this would be to explicitly write:

```kotlin
val job = launch { ... }
job.setUpCancellation()
```

For readability, inside the `setUpCancellation` function we could refer to its receiver via the new `loadingJob` variable (line `#2`).
Then we can add a listener to the `cancel` button,
so then when it's clicked the `loadingJob` is canceled (line `#3`).

With structured concurrency, we only need to cancel the parent coroutine
and this automatically propagates cancellation to all the child coroutines.

### Using the outer scope's context

When we start new coroutines inside the given scope, it's much easier to ensure that all of them are run
with the same context. And it's much easier to replace the context if needed.

Let's now return to the question from the end of the previous section:
how exactly does "using the dispatcher from the outer scope" work?
(or, more specifically, "using the dispatcher from the outer scope's context")?

The new scope created by the `coroutineScope` or by the coroutine builders, always inherits the context from the outer scope.
In this case, the outer scope is the scope the `suspend loadContributorsConcurrent` was called from:

```kotlin
launch(Dispatchers.Default) {  // outer scope
    val users = loadContributorsConcurrent(service, req)
    // ...
}
```

All the nested coroutines are automatically started with the inherited context;
and the dispatcher is a part of this context.
That's why all the coroutines started by `async` are started with the context of the default dispatcher:

```kotlin
suspend fun loadContributorsConcurrent(
    service: GitHubService, req: RequestData
): List<User> = coroutineScope {
    // this scope inherits the context from the outer scope 
    // ... 
    async {   // nested coroutine started with the inherited context
        // ...
    }
    // ...
}
```

With structured concurrency we can specify the major context elements (like dispatcher) once, when we create a top-level coroutine.
All the nested coroutines then inherit the context and modify it only if needed. 

Note that when we write code with coroutines for UI applications, e.g., for Android, it is common practice to use
`CoroutineDispatchers.Main` by default for the top coroutine and then to explicitly put a different dispatcher when we
need to run the code on a different thread.
