# Propagating messages

### Maintaining a list of open connections

Now that we have a simple echo route implemented, before being able to do a multi-chat, we need to keep
a list of open connections. For this we can use a synchronized set.

Add the following code to the `chatRoute` function:

```kotlin
fun Route.chatRoute() {
    val connections = Collections.synchronizedSet(LinkedHashSet<DefaultWebSocketServerSession>())
    webSocket("/chat") {
        connections += this
        try {
            while (true) {
                when (val frame = incoming.receive()) {
                    is Frame.Text -> {
                        val text = frame.readText()
                        outgoing.send(Frame.Text(text))
                    }
                }
            }
        } finally {
            connections -= this
        }
    }
}
```

We're creating a `connections` set, and then for every request made to `/chat`, we'll be adding
the connection to this set. Once the chat has finalized, we'll remove it.

### Fanning out

Now that we have a list of connections, instead of just echoing back the message to a single
connection, we want to send it to all the connections we maintain. 

Let's update the code in the `chatRoute` function to perform this action. Essentially, we replace the `outgoing.send` with a `forEach` on `connections` and sending it to each element in the list.

```kotlin

                        connections.forEach {
                            it.outgoing.send(Frame.Text(text))
                        }

```



## Giving connections some identity

It would be nicer to actually give each connection some metadata, i.e. for instance
a username. Let's first create a data class to represent a connection. 

Create a new file named `Connection` and add the following code to it:

```kotlin
class Connection(val session: DefaultWebSocketSession) {
    companion object {
        var lastId = AtomicInteger(0)
    }
    val name = "user${lastId.getAndIncrement()}"
}
```

Notice how we're now referencing the actual session as a property of our `Connection` class. We'll use this
when we want to send data. Lastly, for the sake of simplicity, we're using a simple 
counter to name users. 

Now let's update our code in `chatRoute` to use this new class:

```kotlin

fun Route.chatRoute() {
    val connections = Collections.synchronizedSet(LinkedHashSet<Connection>())

    webSocket("/chat") {
        connections += Connection(this)
        try {
            while (true) {
                when (val frame = incoming.receive()) {
                    is Frame.Text -> {
                        val receivedText = frame.readText()
                        connections.forEach {
                            val textToSend = "[${it.name}] $receivedText:"
                            it.session.outgoing.send(Frame.Text(textToSend))
                        }
                    }
                }
            }
        } finally {
            connections -= Connection(this)
        }
    }
}
```

In addition to changing the `connections` to instantiate a list of `Connection` class,
we also have updated the message we're sending to indicate who is sending. 



