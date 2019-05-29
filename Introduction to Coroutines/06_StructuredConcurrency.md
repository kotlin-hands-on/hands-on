# Structured concurrency
 
_Coroutine scope_ is responsible for structure and parent-child relation between different coroutines.
You always start new coroutines inside a scope.
_Coroutine context_ stores additional technical information used to run a given coroutine,
like the dispatcher specifying the thread or threads the coroutine should be scheduled on.

When `launch`, `async` or `runBlocking` start a new coroutine, they automatically create the corresponding scope.
All these functions take lambda with receiver as an argument, and the type of the implicit receiver is `CoroutineScope`:

```kotlin
launch { /* this: CoroutineScope */
}
```

New coroutines can only be started inside a scope.
`launch` and `async` are declared as extensions to `CoroutineScope`, so an implicit or explicit receiver must always
be passed when you call them.
The coroutine started by `runBlocking` is the only exception: `runBlocking` is defined as a top-level function.
But because it blocks the current thread, it is intended primarily to be used in `main` functions and in tests as
a bridge function.

When you start a new coroutine inside `runBlocking`, `launch` or `async`, you automatically start it inside the scope: 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { /* this: CoroutineScope */
    launch { /* ... */ }
    // the same as:    
    this.launch { /* ... */ }
}
```

When you call `launch` inside `runBlocking`, you call it as an extension to the implicit receiver
of the `CoroutineScope` type.
Alternatively, you can explicitly write `this.launch`.

We can say that the nested coroutine (started by `launch` in this example) is a child of the outer coroutine
(started by `runBlocking`). This "parent-child" relation works through scopes: the child coroutine is started from
the scope corresponding to the parent coroutine.

You can create a new scope without starting a new coroutine.
The `coroutineScope` function does that. 
When you need to start new coroutines in a structured way inside a `suspend` function without access to the outer scope,
like inside `loadContributorsConcurrent`,
you can create a new coroutine scope which automatically becomes a child of the outer scope this `suspend` function
is called from.

That's also possible to start a new coroutine from the global scope by saying `GlobalScope.async` or `GlobalScope.launch`.
That will create a top-level "independent" coroutine.

The mechanism providing structure of coroutines is called "structured concurrency".
What benefits does the structured concurrency bring in comparison with global scopes?
The scope is generally responsible for children coroutines, and their lifetime is attached to the lifetime of the scope. 
The scope can automatically cancel children coroutines if something goes wrong or,
if the user simply changes her mind and decides to revoke the operation.
The scope automatically waits for completion of all the children coroutines.
Therefore, if the scope corresponds to a coroutine, then the parent coroutine does not complete until all the coroutines
launched in its scope complete.

When using `GlobalScope.async` we don't have the structure that binds several coroutines to a smaller scope.
The coroutines started from global scope are all independent; 
their lifetime is only limited by the lifetime of the whole application.
You can store the reference to the coroutine started from the global scope and wait for its completion or cancel it
explicitly, but it won't happen automatically as with the structured ones.

### Cancellation of contributors loading

Let's compare two versions of `loadConstributorsConcurrent` function:
the one using `coroutineScope` to start all the children coroutines and the one using `GlobalScope`.
We'll compare how both versions behave when you try to cancel the parent coroutine.

Add 3 seconds delay to all coroutines sending the requests, so that you had enough time to cancel loading after
the moment the coroutines were started but before the moment the requests were sent:  

```kotlin
suspend fun loadContributorsConcurrent(req: RequestData): List<User> = coroutineScope {
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

Copy the implementation of `loadContributorsConcurrent` to `loadContributorsNotCancellable` (in `Request6NotCancellable.kt`)
and remove creation of a new `coroutineScope`.
Your `async` calls now fail to resolve, so start them via `GlobalScope.async`:

```kotlin
suspend fun loadContributorsNotCancellable(req: RequestData): List<User> {   // #1
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
and all the "contributors" coroutines are started inside `GlobalScope`, not as children of coroutine scope (line `#2`).  

Run the program, choose to load the contributors via `CONCURRENT` version, wait until all the "contributors" coroutines
are started and then click on "cancel". Looking at the log, you'll find out that all the requests were indeed cancelled
because no results were logged:

```
2896 [AWT-EventQueue-0 @coroutine#1] INFO  Contributors - kotlin: loaded 30 repos
2901 [DefaultDispatcher-worker-2 @coroutine#4] INFO  Contributors - starting loading for kotlin-koans
...
2909 [DefaultDispatcher-worker-5 @coroutine#21] INFO  Contributors - starting loading for kotlinx.coroutines
/* click on 'cancel' */
/* no requests are sent */
```

Now repeat the same procedure but choose the `NOT_CANCELLABLE` option:

```
2570 [AWT-EventQueue-0 @coroutine#1] INFO  Contributors - kotlin: loaded 30 repos
2579 [DefaultDispatcher-worker-1 @coroutine#4] INFO  Contributors - starting loading for kotlin-koans
...
2586 [DefaultDispatcher-worker-6 @coroutine#21] INFO  Contributors - starting loading for kotlinx.coroutines
/* click on 'cancel' */
/* but all the requests are still sent: */
6402 [DefaultDispatcher-worker-6 @coroutine#4] INFO  Contributors - kotlin-koans: loaded 45 contributors
...
7428 [DefaultDispatcher-worker-8 @coroutine#21] INFO  Contributors - kotlinx.coroutines: loaded 100 contributors
```

Nothing happens! No coroutines are cancelled, all the requests are still sent.

Let's see how the cancellation is implemented in our "contributors" program.
When the `cancel` button is clicked you need to explicitly cancel the main "loading" coroutine.
Then it automatically cancels all the children coroutines.

That's how we cancel the "loading" coroutine on the button click: 

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

The `launch` function returns us an instance of `Job`.
It's the reference to the "loading coroutine" which loads all the data and updates the results.
We call `setUpCancellation()` extension function on it (line `#1`), passing an instance of `Job` as a receiver.
Another way to express that would be to write explicitly:

```kotlin
val job = launch { ... }
job.setUpCancellation()
```

For readability, inside the `setUpCancellation` function we refer to its receiver via the new `loadingJob` variable (line `#2`).
Then we add the listener to the `cancel` button,
so that when it's clicked, the `loadingJob` is cancelled (line `#3`).

With structured concurrency you only need to cancel the parent coroutine,
and then it automatically propagates cancellation to all the children coroutines.

### Using the outer scope's context

When you start new coroutines inside the given scope, it's much easier to ensure that all of them are run
with the same context.
And it's much easier to replace the context if needed.

Let's now explicitly return to the question at the end of the previous section:
how exactly works "using the dispatcher from the outer scope"
(or, more correct, "using the dispatcher from the outer scope's context")?

The new scope created by `coroutineScope` or by coroutine builders always inherits the context from the outer scope.
In our case, the outer scope is the scope the `suspend loadContributorsConcurrent` was called from:

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
suspend fun loadContributorsConcurrent(req: RequestData): List<User> = coroutineScope {
    // this scope inherits the context from the outer scope 
    // ... 
    async {   // nested coroutine started with the inherited context
        // ...
    }
    // ...
}
```

Structured concurrency allows to specify major context elements (like dispatcher) once when you create a top-level coroutine.
All the nested coroutines then inherit the context and modify it only if needed. 

Note that when you write the code with coroutiens for UI applications, e.g. for Android, the common practice is to use
`CoroutineDispatchers.Main` by default for the top coroutine, and to explicitly put a different dispatcher when it's
needed to run the code on a different thread.
