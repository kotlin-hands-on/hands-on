# Using suspend functions

Retrofit recently added the native support for coroutines, and we're going to use it.
Instead of returning `Call<List<Repo>>`, you now define your API call as a `suspend` function:

```kotlin
interface GitHubService {
    @GET("orgs/{org}/repos?per_page=100")
    suspend fun getOrgRepos(
        @Path("org") org: String
    ): List<Repo>
}
```

The main idea is that when you use a `suspend` function to perform a request, the underlying thread isn't blocked.
We'll discuss how exactly it works a bit later. 

To make use of this functionality, you'll need the latest version of Retrofit library (`2.6.0` at the moment).
For this tutorial, you don't need to change anything; the right version is already specified in the project dependencies. 

Note that now `getOrgRepos` returns the result directly instead of returning `Call`.
If the result is unsuccessful, an exception is thrown.

Alternatively, Retrofit allows returning the result wrapped in `Response`.
In this case, you manually get the result body and check for errors.
We'll use the `Response` versions in this tutorial. 

Please add the following declarations to `GitHubService`:

```kotlin
interface GitHubService {
    // getOrgReposCall & getRepoContributorsCall declarations
    
    @GET("orgs/{org}/repos?per_page=100")
    suspend fun getOrgRepos(
        @Path("org") org: String
    ): Response<List<Repo>>

    @GET("repos/{owner}/{repo}/contributors?per_page=100")
    suspend fun getRepoContributors(
        @Path("owner") owner: String,
        @Path("repo") repo: String
    ): Response<List<User>>
}
```

Your task will be to change the code of the function loading contributors to make use of these new `suspend` functions.
 
However, a `suspend` function can't be called everywhere,
if you try to call it from `loadContributorsBlocking`, you'll get an error
"Suspend function 'getOrgRepos' should only be called from a coroutine or another suspend function".
Thus, we need to mark our next version of `loadContributors` function as `suspend` in order to use this new API.

Please now do the following task, and right after that we'll discuss how `suspend` functions work and how
they are different from regular ones.
We'll also see by which means you can call a `suspend` function from a non-suspending one.

#### Task

Copy the implementation of `loadContributorsBlocking` (defined in `src/tasks/Request1Blocking.kt`)
into `loadContributorsSuspend` (defined in `src/tasks/Request4Suspend.kt`).
Then modify it in a way so that the new suspend functions were used instead of ones returning Calls.
Run the program choosing `SUSPEND` option and make sure that UI is responsive while the GitHub requests are performed. 

#### Solution

Modifying the code is straightforward.
You need only to replace `.getOrgReposCall(req.org).execute()` with `.getOrgRepos(req.org)`
and repeat the same replacement for the second "contributors" request.
That's it.
All the rest stays the same: 

```kotlin
suspend fun loadContributorsSuspend(req: RequestData): List<User> {
    val service = createGitHubService(req.username, req.password)
    val repos = service
        .getOrgRepos(req.org)
        .also { logRepos(req, it) }
        .bodyList()

    return repos.flatMap { repo ->
        service.getRepoContributors(req.org, repo.name)
            .also { logUsers(repo, it) }
            .bodyList()
    }.aggregate()
}
```

Note that `loadContributorsSuspend` should be defined as `suspend` function.

We no longer need to call `execute` which returned `Response` before, because now the API functions return the `Response`
directly.
But that's the implementation detail specific to Retrofit library.
With other libraries, API will be different, but the concept stays the same:

_Your code with `suspend` functions looks surprisingly similar to the "blocking" version.
It's readable and expresses exactly what you're trying to achieve._

The major difference with blocking version is that instead of blocking the thread, we suspend the coroutine:

```
thread -> coroutine
block -> suspend
```

Coroutines are often called light-weight threads.
That means you can run code on coroutines similar to how you run code on threads.
However, the operations that were blocking before (and had to be avoided because of that),
now can suspend the coroutine instead.

How to start a new coroutine?
If you look at how we call `loadContributorsSuspend`, you'll find out that we call it inside `launch`.
`launch` is a library function that takes a lambda as an argument: 

```kotlin
launch {
    val users = loadContributorsSuspend(req)
    updateResults(users, startTime)
}
```

`launch` starts a new computation.
This computation is responsible for loading the data and showing the results.
This computation is suspendable: while performing the network requests, it gots "suspended"
and releases the underlying thread.
When the network request returns the result, the computation is resumed.

Such suspendable computation is called a coroutine,
so, we'll simply say that in this case, `launch` _starts a new coroutine_ that is responsible
for loading data and showing the results.

Coroutines are computations that run on top of threads and can be suspended.
By saying "suspended", we mean that the corresponding computation can be paused,
removed from the thread, and stored in memory.
Meanwhile, the thread is free to be occupied with other activities:

![](./assets/4-suspend/SuspensionProcess.gif)

When the computation is ready to be continued, it's got returned to a thread (but not necessarily to the same one). 

Let's return to our `loadContributorsSuspend` example.
Each "contributors" request now waits for the result via the suspension mechanism.
At first, the new request is sent.
Then, while waiting for the response, the whole "load contributors" coroutine becomes suspended
(the one we've discussed above started by the `launch` function). 
The coroutine resumes only after the corresponding response is received:

![](./assets/4-suspend/SuspendRequests.png)

While the response isn't yet received, the thread is free to be occupied with other tasks.
That's why when you load users via the `COROUTINE` option, the UI stays responsive, despite all the requests
take place on the main UI thread.
The log confirms that all the requests are sent on the main UI thread:

```
2538 [AWT-EventQueue-0 @coroutine#1] INFO  Contributors - kotlin: loaded 30 repos
2729 [AWT-EventQueue-0 @coroutine#1] INFO  Contributors - ts2kt: loaded 11 contributors
3029 [AWT-EventQueue-0 @coroutine#1] INFO  Contributors - kotlin-koans: loaded 45 contributors
...
11252 [AWT-EventQueue-0 @coroutine#1] INFO  Contributors - kotlin-coroutines-workshop: loaded 1 contributors
```

The log can also show you the coroutine the corresponding code runs on.
To enable that, open `Run | Edit configurations...` and add the `-Dkotlinx.coroutines.debug` VM option:

![](./assets/4-suspend/RunConfiguration.png)

Then while running `main` with this option, the coroutine name will be attached to the thread name.
You can also modify the template for running all the Kotlin files to enable this option by default.

In our case, all the code runs on one coroutine,
the mentioned above "load contributors" coroutine, denoted as `@coroutine#1`.

However, in this version, while waiting for the result, we don't reuse the thread for sending the other requests,
because we wrote our code in a sequential way. The new request is sent only when the previous result is received.
`suspend` functions treat the thread fairly and don't block it for pure "waiting",
but don't yet bring any concurrency to the picture. Let's see how that can be improved.
