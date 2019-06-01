# Using callbacks

The previous solution works, but blocks the thread and therefore freezes the UI.
The classical approach to avoid the problem of blocking the main thread is to use callbacks.
Instead of calling the code that should be invoked after the operation is completed straightaway,
we extract it into a separate callback, often a lambda, and pass this lambda to the caller in order to be called later.

To make UI responsive, we can either move the whole computation to a separate thread or switch to
Retrofit API using callbacks instead of blocking calls.

### Calling `loadContributors` in the background thread

Let's first move the whole computation to a different thread.
We use `thread` function to start a new thread:

```kotlin
thread {
    loadContributorsBlocking(req)
}
```

Now all the loading was moved to a separate thread, so the main thread is free and can be occupied with different tasks:  

![](./assets/3-callbacks/Background.png)

The signature of `loadContributors` function changes, it takes `updateResults` callback as a second argument
to call it after all the loading completes:

```kotlin
fun loadContributorsBackground(req: RequestData, updateResults: (List<User>) -> Unit)
```

Now when the `loadContributorsBackground` is called, `updateResults` call goes in the callback, not right afterwards as before:

```kotlin
loadContributorsBackground(req) { users ->
    SwingUtilities.invokeLater {
        updateResults(users, startTime)
    }
}
```

By calling `SwingUtilities.invokeLater` we ensure that `updateResults` call which updates the results happens on the main UI thread
(AWT event dispatching thread).

However, if you try to load the contributors via the `BACKGROUND` option, you'll see that the list is updated, but nothing changes.

#### Task

Fix the `loadContributorsBackground()` in `src/tasks/Request2Background.kt` so that the resulting list was shown in the UI.

#### Solution

We forgot to call the callback! The contributors were loaded, as you can see in the log, but the result wasn't displayed.
To fix that, we need to call the `updateResults` on the resulting list of users:

```kotlin
thread {
    updateResults(loadContributorsBlocking(req))
}
```

You should explicitly call the logic passed in the callback.
Otherwise, nothing happens.  

### Using Retrofit callback API

We've moved the whole loading logic to the background thread, but it's still not the best use of resources. 
All the loading requests go sequentially one after another, and while waiting for the loading result,
the thread is blocked, but it could be occupied with some other tasks. Especially, it could start the next loading,
so that the whole result was received earlier!

Handling the data for each repository should be then divided into two parts:
first the loading, then processing the resulting response.
The second "processing" part should be extracted into a callback.
The loading for the second repository can then be started before the result for the first repository
is received (and the corresponding callback is called): 

![](./assets/3-callbacks/Callbacks.png)

The Retrofit callback API can help to achieve that.
We'll use the `Call.enqueue` function that starts an HTTP request and takes a callback as an argument.
In this callback, you specify what needs to be done after each request.

`loadContributorsCallbacks()` in `src/tasks/Request3Callbacks.kt` uses this API.
For convenience, we use `onResponse` extension function declared in the same file.
It takes a lambda as an argument rather than an object expression.

```kotlin
fun loadContributorsCallbacks(req: RequestData, updateResults: (List<User>) -> Unit) {
    val service = createGitHubService(req.username, req.password)
    service.getOrgReposCall(req.org).onResponse { responseRepos ->  // #1
        logRepos(req, responseRepos)
        val repos = responseRepos.bodyList()
        
        val allUsers = mutableListOf<User>()
        for (repo in repos) {
            service.getRepoContributorsCall(req.org, repo.name).onResponse { responseUsers ->   // #2
                logUsers(repo, responseUsers)
                val users = responseUsers.bodyList()
                allUsers += users
            }
        }
        // TODO: Why this code doesn't work? How to fix that?
        updateResults(allUsers.aggregate())
    }
}
```

We extract the logic handling the responses into callbacks: the corresponding lambdas start at lines `#1` and `#2`. 

However, the provided solution doesn't work.
If you run the program and load contributors choosing `CALLBACKS` option, you'll see that nothing is shown.
The tests that immediately return the result, however, pass. Why?

Spend some time thinking of why the given code doesn't work as expected, and after that, continue reading.

#### Task (optional)

Rewrite the code so that the loaded list of contributors was shown.

#### Solution (first attempt)

We're starting many requests concurrently which lets us decrease the total loading time.
However, we don't wait for the loaded result.
We call the `updateResults` callback right after we started all the loading requests,
at that moment, the `allUsers` list is not yet filled with the data.

We can try to fix this code with the following change:

```kotlin
val allUsers = mutableListOf<User>()
for ((index, repo) in repos.withIndex()) {   // #1
    service.getRepoContributorsCall(req.org, repo.name).onResponse { responseUsers ->
        logUsers(repo, responseUsers)
        val users = responseUsers.bodyList()
        allUsers += users
        if (index == repos.lastIndex) {      // #2
            updateResults(allUsers.aggregate())
        }
    }
}  
```

In line `#1` we iterate over a list of repos with index.
Then from each callback, we check whether we're on the last iteration (`#2`).
And if that's the case, we update the result.

However, this code is also incorrect. Why? What's the source of the problem?
Spend some time trying to find an answer to this question, and after that, continue reading.

#### Solution (second attempt)

Since the loading requests are started concurrently, no one guarantees that the result for the last one comes last.
The results can come in any order.
Thus, if we use a comparison of the current index with `lastIndex` as a condition for completion
we risk loosing the results for some of the repos.
If the request processing the last repo returns faster then prior requests (which is likely to happen),
all the results for requests that take more time to process will be lost.

One of the ways to fix that is to introduce an index and check whether we've processed all the repositories already:

```kotlin
val allUsers = Collections.synchronizedList(mutableListOf<User>())
val numberOfProcessed = AtomicInteger()
for (repo in repos) {
    service.getRepoContributorsCall(req.org, repo.name).onResponse { responseUsers ->
        logUsers(repo, responseUsers)
        val users = responseUsers.bodyList()
        allUsers += users
        if (numberOfProcessed.incrementAndGet() == repos.size) {
            updateResults(allUsers.aggregate())
        }
    }
} 
```

Note that we're using the synchronized version of list and `AtomicInteger`, since in a general case there's no guarantee
that different callbacks processing `getRepoContributors` requests will always be called from the same thread.

You can see that writing the right code with callbacks might be non-trivial and error-prone, especially when
there're several underlying threads and synchronization takes place.
Next, we'll discuss how to implement the same logic using `suspend` functions. 

### Note about using RxJava

As we don't expect all the readers of this tutorial to be familiar with RxJava library, we don't describe in detail the version
that uses RxJava.
However, if you're interested, you can use it as an additional exercise and implement the same logic using a reactive approach.
The necessary dependencies and solutions using RxJava can be found in a separate 'rx' branch in the project.
You can also decide to read this tutorial till the end and implement or check the proposed Rx versions after that to get
the proper comparison. 
