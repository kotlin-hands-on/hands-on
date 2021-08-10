# Blocking request

We will use the developer GitHub API, which performs the requests under your account
and uses your password and authentication token you've provided.

We will use [Retrofit](https://square.github.io/retrofit/) to perform HTTP requests to Github.
It allows us to request the list of repositories under the given organization,
and the list of contributors for each repository:

```kotlin
interface GitHubService {
    @GET("orgs/{org}/repos?per_page=100")
    fun getOrgReposCall(
        @Path("org") org: String
    ): Call<List<Repo>>

    @GET("repos/{owner}/{repo}/contributors?per_page=100")
    fun getRepoContributorsCall(
        @Path("owner") owner: String,
        @Path("repo") repo: String
    ): Call<List<User>>
}
```

The function `loadContributorsBlocking` that is defined in `src/tasks/Request1Blocking.kt`
uses this API to fetch the list of contributors for the given organization.
Let's see how it is implemented:

```kotlin
fun loadContributorsBlocking(service: GitHubService, req: RequestData) : List<User> {
    val repos = service
        .getOrgReposCall(req.org)          // #1
        .execute()                         // #2
        .also { logRepos(req, it) }        // #3
        .body() ?: listOf()                // #4

    return repos.flatMap { repo ->
        service
            .getRepoContributorsCall(req.org, repo.name)      // #1
            .execute()                                        // #2
            .also { logUsers(repo, it) }                      // #3
            .bodyList()                                       // #4
    }.aggregate()
}
```

At first, we get a list of the repositories under the given organization and store it in the `repos` list.
Then for each repository, we request the list of contributors and merge all these lists into one final list
of contributors.

Each `getOrgReposCall` and `getRepoContributorsCall` returns an instance of `Call` class to us (`#1`).
At this point, no request is sent. 
We invoke `Call.execute` to perform the request (`#2`).
`execute` is a synchronous call which blocks the underlying thread.

When we get the response, we log the result by calling the specific `logRepos()` and `logUsers()` functions (`#3`).
If the HTTP response contains an error, this error will be logged here.

Lastly, we need to get the body of the response, which contains the desired data.
For simplicity in this tutorial, we'll use an empty list as a result in case
there is an error, and log the corresponding error (`#4`).
To avoid repeating `.body() ?: listOf()` over and over, 
we declare an extension function `bodyList`:

```kotlin
fun <T> Response<List<T>>.bodyList(): List<T> {
    return body() ?: listOf()
}
```   

`logRepos` and `logUsers` log the received information straight away.
While running the code, look at the system output, it should show something like this:

```
1770 [AWT-EventQueue-0] INFO  Contributors - kotlin: loaded 40 repos
2025 [AWT-EventQueue-0] INFO  Contributors - kotlin-examples: loaded 23 contributors
2229 [AWT-EventQueue-0] INFO  Contributors - kotlin-koans: loaded 45 contributors
...
```

The first item on each line is the amount of milliseconds that have passed since the program started, then the thread name in square brackets.
We can see from which thread the loading request is called on.
The final item on each line is the actual message: how many repositories or contributors were loaded.

This log demonstrates that all the results were logged from the main thread.
When we run the code under a `BLOCKING` request, we will find that the window will freeze and won't react to input
until the loading is finished.
All the requests are executed from the same thread as we've called `loadContributorsBlocking` from,
which is the main UI thread (in Swing it's an AWT event dispatching thread).
This main thread gets blocked, and that explains why the UI is frozen:

![](./assets/2-blocking/Blocking.png)

After the list of contributors has loaded, the result is updated.
If we look at how `loadContributorsBlocking` is called, we find that the `updateResults` goes right
after the `loadContributorsBlocking` call:

```kotlin
val users = loadContributorsBlocking(service, req)
updateResults(users, startTime)
```

This code can be found in `src/contributors/Contributors.kt`;
the function `loadContributors` is responsible for choosing the way
the contributors are loaded.
`updateResults` is a function that updates the UI.
As a result, it must always be called from the UI thread.

To familiarize yourself with the task domain (we'll need this later), please go through the following simple task.
Currently, if we run the code, we can see that each contributor name is repeated several times, once for every
project they have taken part in. This task is to implement the `aggregate` function combining the users so that each 
contributor is added only once. The `User.contributions` property should contain the total number of contributions
of the given user to *all* of the projects.
The resulting list should be sorted in descending order according to the number of contributions.

#### Task

Open `src/tasks/Aggregation.kt` and implement `List<User>.aggregate()` function.
The corresponding test file `test/tasks/AggregationKtTest.kt` shows an example of the expected result.
You can jump between the source code and the test class automatically [using IntelliJ IDEA
shortcut](https://www.jetbrains.com/help/idea/create-tests.html#test-code-navigation).

After implementing this task, the resulting list for `Kotlin` organization should be similar to the following:

![](./assets/2-blocking/Aggregate.png)

Note how the users are sorted by the total number of their contributions.  


#### Solution

A possible solution:

```kotlin
fun List<User>.aggregate(): List<User> =
    groupBy { it.login }
        .map { (login, group) -> User(login, group.sumOf { it.contributions }) }
        .sortedByDescending { it.contributions }
```

First, we group users by their login.
`groupBy` returns a map from login to all occurrences of the user with this login in different repositories.
Then for each map entry, we count the total number of contributions for each user and 
create a new instance of `User` class by the given name and sum of contributions.
At the end, we sort the resulting list in a descending order.

An alternative is to use the function `groupingBy` instead of `groupBy`.
