# Channels

Writing code with a shared mutable state is known to be quite difficult and error-prone
(even in this tutorial we had a chance encounter while implementing the solution using callbacks).
Sharing information by communication instead of sharing information using the common mutable state tries to simplify this.
Coroutines can communicate with each other via _channels_.

Channels are communication primitives that allow us to pass data between different coroutines.
One coroutine can _send_ some information to a channel, while the other one can _receive_ this information from it:

![](./assets/8-channels/UsingChannel.png)

A coroutine that sends (produces) information is often called a producer, and a coroutine that receives (consumes)
information is called a consumer. 
When needed, many coroutines can send information to the same channel, and many coroutines can receive information from it:

![](./assets/8-channels/UsingChannelManyCoroutines.png)

Note that when many coroutines receive information from the same channel, each element is handled only once by one
of the consumers; handling it automatically means removing this element from the channel.

We can think of a channel as similar to a collection of elements
(the direct analog would be a queue: elements are added to the one end and received from the another).
However, there's an important difference:
unlike collections, even in their synchronized versions a channel can _suspend_ `send` and `receive` operations.
This happens when the channel is empty or full (the channel's size might be constrained, and then it can be full).

`Channel` is represented via three different interfaces: `SendChannel`, `ReceiveChannel`,
and `Channel` which extends the first two.
You usually create a channel and give it to producers as a `SendChannel` instance, so that only they can send to it,
and to consumers as `ReceiveChannel` instance, so that only they can receive from it.
Note that both `send` and `receive` methods are declared as `suspend`:

```kotlin
interface SendChannel<in E> {
    suspend fun send(element: E)
    fun close(): Boolean
}

interface ReceiveChannel<out E> {
    suspend fun receive(): E
}    

interface Channel<E> : SendChannel<E>, ReceiveChannel<E>
```

The producer can close a channel to indicate that no more elements are coming.

Several types of channels are defined in the library.
They differ in how many elements they can internally store, and whether the `send` call can suspend or not.
For all channel types, the `receive` call behaves in the same manner: it
receives an element if the channel is not empty, and otherwise suspends.

- *Unlimited channel*

![](./assets/8-channels/UnlimitedChannel.png)

An unlimited channel is the closest analog to queue: producers can send elements to this channel,
and it will grow infinitely.
The `send` call will never be suspended.
If there's no more memory, you'll get an `OutOfMemoryError`. 
The difference with a queue appears when a consumer tries to receive from an empty channel
and gets suspended until some new elements are sent to this channel.

- *Buffered channel*

![](./assets/8-channels/BufferedChannel.png)

A buffered channel's size is constrained by the specified number.
Producers can send elements to this channel until the size limit is reached.
All the elements are internally stored.
When the channel is full, the next `send` call on it suspends until more free space appears.

- *“Rendezvous” channel*

![](./assets/8-channels/RendezvousChannel.png)

The “Rendezvous” channel is a channel without a buffer; it's the same as creating a buffered channel with zero size.
One of the functions (`send` or `receive`) always gets suspended until the other is called.
If the `send` function is called and there's no suspended `receive` call ready to process the element,
then `send` suspends.
Similarly, if the `receive` function is called and the channel is empty - or in other words,
there's no suspended `send` call ready to send the element - the `receive` call suspends.
The "rendezvous" name ("a meeting at an agreed time and place") refers to the fact that `send` and `receive`
should "meet on time".

- *Conflated channel* 

![](./assets/8-channels/ConflatedChannel.gif)

A new element sent to the conflated channel will overwrite the previously sent element, so the receiver will always
get only the latest element.
The `send` call will never suspend.

When you create a channel, you specify its type or the buffer size (if you need a buffered one):

```kotlin
val rendezvousChannel = Channel<String>()
val bufferedChannel = Channel<String>(10)
val conflatedChannel = Channel<String>(CONFLATED)
val unlimitedChannel = Channel<String>(UNLIMITED)
``` 

By default, a "Rendezvous" channel is created.

In the following example, we will create a “Rendezvous” channel, two producer coroutines, and a consumer coroutine:  

```run-kotlin
import kotlinx.coroutines.channels.Channel
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    val channel = Channel<String>()
    launch {
        channel.send("A1")
        channel.send("A2")
        log("A done")
    }
    launch {
        channel.send("B1")
        log("B done")
    }
    launch {
        repeat(3) {
            val x = channel.receive()
            log(x)
        }
    }
}

fun log(message: Any?) {
    println("[${Thread.currentThread().name}] $message")
}
```

If you want to better understand what's going on in this example, watch the following video:

![Video explaining the channels sample](https://youtu.be/HpWQUoVURWQ)

#### Task

Implement the function `loadContributorsChannels` that requests all the GitHub contributors concurrently,
but shows intermediate progress at the same time.
Use these two previous functions:
`loadContributorsConcurrent` from `Request5Concurrent.kt` and `loadContributorsProgress` from `Request6Progress.kt`.

#### Tip

Different coroutines that concurrently receive contributor lists for different repositories can send all the received
results to the same channel:

```kotlin
val channel = Channel<List<User>>()
for (repo in repos) {
    launch {
        val users = ...
        // ...
        channel.send(users)
    }
} 
```

Then the elements from this channel can be received one by one and processed:

```kotlin
repeat(repos.size) {
    val users = channel.receive()
    ...
}
```

Since we call the `receive` calls sequentially, no additional synchronization is needed.

#### Solution

As with the `loadContributorsProgress` function, we can create an `allUsers` variable to store
the intermediate states of the "all contributors" list.
When we receive each new list from the channel,
we add it to the list of all users, aggregate the result, and update the state using the `updateResults` callback:

```kotlin
suspend fun loadContributorsChannels(
    service: GitHubService,
    req: RequestData,
    updateResults: suspend (List<User>, completed: Boolean) -> Unit
) = coroutineScope {

    val repos = service
        .getOrgRepos(req.org)
        .also { logRepos(req, it) }
        .bodyList()

    val channel = Channel<List<User>>()
    for (repo in repos) {
        launch {
            val users = service.getRepoContributors(req.org, repo.name)
                .also { logUsers(repo, it) }
                .bodyList()
            channel.send(users)
        }
    }
    var allUsers = emptyList<User>()
    repeat(repos.size) {
        val users = channel.receive()
        allUsers = (allUsers + users).aggregate()
        updateResults(allUsers, it == repos.lastIndex)
    }
}
```

Note that the results for different repositories are added to the channel as soon as they are ready.
At first, when all the requests are sent and no data is received, the `receive` call suspends.
In this case, the whole "load contributors" coroutine suspends.
Then, when the list of users is sent to the channel, the "load contributors" coroutine resumes,
the `receive` call returns this list, and the results are immediately updated. 
 
We've learned how to use suspend functions, execute coroutines concurrently, and share 
information between coroutines using channels.

Note that neither coroutines nor channels will completely eradicate the complexity that comes from concurrency,
but they will definitely make your life easier when you need to reason it and understand what's going on.

Next, we'll discuss how to test the code that uses coroutines.
