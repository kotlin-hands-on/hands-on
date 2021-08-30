# Testing coroutines

Let's discuss how to test the code that uses coroutines.
It'd be nice to test our solutions, make sure that the solution with concurrent coroutines is faster than the solution
with `suspend` functions, and check that the solution with channels is faster than the simple "progress" one.
Let's have a look at how to compare the total running time of these solutions.

Let's mock a GitHub service and make this service return results after the given timeouts: 

```
repos request - returns an answer within 1000 ms delay
repo-1 - 1000 ms delay
repo-2 - 1200 ms delay
repo-3 - 800 ms delay
```

Then, we can test that the sequential solution with the `suspend` functions should take around 4000 ms:

```
4000 = 1000 + (1000 + 1200 + 800)
```

And the concurrent solution should take around 2200 ms:

```
2200 = 1000 + max(1000, 1200, 800) 
```

For the solutions showing progress, we can also check the intermediate results with timestamps.

The corresponding test data is defined in `test/contributors/testData.kt`,
and the files `Request4SuspendKtTest`, ... `Request7ChannelsKtTest` contain the straightforward tests that use mock service calls.

However, we have two problems here:

* These tests take too long to run.
Each test takes around 2 to 4 seconds, so we need to wait for the results each time.
Such an approach is not very efficient.
   
* We can't rely on the exact time the solution runs, because it still takes additional time to warm up and run the code etc.
We could add an additional constant, but then it will differ from a machine to a machine.
Note that the mock service delays should be higher than this constant so that we can see a difference.
If the constant is 0.5 sec, making the delays 0.1 sec won't be enough. 

A better way would be to use special frameworks to test the timing while running the same code several times
(which increases the total time even more), but it's complicated to learn and set up.  

In this case, however, we want to test a very simple thing: that our solutions with provided test delays behave as we expect;
one faster than the other.
We're not yet interested in the real-life performance tests.

To fix these problems, we can use *virtual* time.
For this, we need to use a special test dispatcher.
It keeps track of the virtual time passed from the start, and runs everything immediately in real time.
When we run coroutines on this dispatcher the `delay` will return straight away and advance the virtual time.

The tests using this mechanism run fast, but you can still check what happens at different moments in virtual time.
The total running time dramatically decreases:

![](./assets/9-testing/timeComparison.png)

How to make use of virtual time?
Replace the `runBlocking` invocation with a `runBlockingTest`.
`runBlockingTest` takes an extension lambda to `TestCoroutineScope` as an argument.
When you call `delay` in a `suspend` function inside this special scope,
`delay` will increase the virtual time instead of delaying in real time:  

```kotlin
@Test
fun testDelayInSuspend() = runBlockingTest {
    val realStartTime = System.currentTimeMillis()
    val virtualStartTime = currentTime

    foo()

    println("${System.currentTimeMillis() - realStartTime} ms")  // ~ 6 ms
    println("${currentTime - virtualStartTime} ms")              // 1000 ms
}

suspend fun foo() {
    delay(1000)     // auto-advances without delay
    println("foo")  // executes eagerly when foo() is called
}
```

We can check the current virtual time using the `currentTime` property of `TestCoroutineScope`.
Note that the actual running time in this example is several milliseconds,
while virtual time equals the delay argument exactly, which is 1000 milliseconds.

To enjoy the full effect of "virtual" `delay` in child coroutines,
we should start all the child coroutines with `TestCoroutineDispatcher`. 
Otherwise, it won't work.
This dispatcher is automatically inherited from the other `TestCoroutineScope`, unless we provide a different dispatcher:

```kotlin
@Test
fun testDelayInLaunch() = runBlockingTest {
    val realStartTime = System.currentTimeMillis()
    val virtualStartTime = currentTime

    bar()

    println("${System.currentTimeMillis() - realStartTime} ms")  // ~ 11 ms
    println("${currentTime - virtualStartTime} ms")              // 1000 ms
}

suspend fun bar() = coroutineScope {
    launch {
        delay(1000)     // auto-advances without delay
        println("bar")  // executes eagerly when bar() is called
    }
}
```

We can try calling `launch` with the context of `Dispatchers.Default` in the example above
and make sure the test fails: we will get an exception saying that the job has not completed yet.

We can test the `loadContributorsConcurrent` function in this way only if it starts the child coroutines 
with the inherited context, without modifying it using the `Dispatchers.Default` dispatcher.
We can specify the context elements like the dispatcher when we *call* a function rather than when we *define* it:
which is more flexible and easier to test.

Note that the testing API that supports virtual time is experimental and may change in the future.
By default, we'll see compiler warnings if we use it. 
To suppress these warnings, we need to annotate the test function or the whole class containing the tests.
We can add `@OptIn(ExperimentalCoroutinesApi::class)` to our test class or function.
By adding such annotation, we emphasize that we understand that the API can change and is ready
to update our usages if needed (most probably, automatically).
 
We also need to add the compiler argument telling it that we're using the experimental API:

```kotlin
compileTestKotlin {
    kotlinOptions {
        freeCompilerArgs += "-Xuse-experimental=kotlin.Experimental"
    }
}
```

In the project corresponding to this tutorial, it's already added to the gradle script.

#### Task

Refactor all the following tests in `tests/tasks/` to use virtual time instead of real time:

```
Request4SuspendKtTest.kt
Request5ConcurrentKtTest.kt
Request6ProgressKtTest.kt
Request7ChannelsKtTest.kt
```

Compare the total running time with the time before this refactoring.

#### Tip

Replace `runBlocking` invocation with `runBlockingTest`, and
`System.currentTimeMillis()` with `currentTime`:

```kotlin
@Test
fun test() = runBlockingTest {
    val startTime = currentTime
    // action
    val totalTime = currentTime - startTime
    // testing result 
}
```

Uncomment the commented assertions checking the exact virtual time.
And let's not forget to add `@UseExperimental(ExperimentalCoroutinesApi::class)`.

#### Solution

Here's the solution for the concurrent case:

```kotlin
fun testConcurrent() = runBlockingTest {
    val startTime = currentTime
    val result = loadContributorsConcurrent(MockGithubService, testRequestData)
    Assert.assertEquals("Wrong result for 'loadContributorsConcurrent'", expectedConcurrentResults.users, result)
    val totalTime = currentTime - startTime

    Assert.assertEquals(
        "The calls run concurrently, so the total virtual time should be 2200 ms: " +
                "1000 for repos request plus max(1000, 1200, 800) = 1200 for concurrent contributors requests)",
        expectedConcurrentResults.timeFromStart, totalTime
    )
}
```

When we check the progress, we first check that the results are available exactly at the expected virtual time,
and then after that we check the results: 

```kotlin
fun testChannels() = runBlockingTest {
    val startTime = currentTime
    var index = 0
    loadContributorsChannels(MockGithubService, testRequestData) {
        users, _ ->
        val expected = concurrentProgressResults[index++]
        val time = currentTime - startTime
        Assert.assertEquals("Expected intermediate results after ${expected.timeFromStart} ms:",
            expected.timeFromStart, time)
        Assert.assertEquals("Wrong intermediate results after $time:", expected.users, users)
    }
}
```

The first intermediate result for the last version with 'channels' is available sooner in comparison to the 'progress' version,
and we can see that difference in tests using virtual time.

The tests for the remaining "suspend" and "progress" tasks are very similar;
you can find them in the project 'solutions' branch.

You can find more information about using virtual time and
experimental testing package [here](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/).
Share your feedback with us of how well it works for you!
