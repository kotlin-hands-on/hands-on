# Exchanging messages

Let’s turn our echo server into a real chat server! To do this, we need to make sure messages from the same user are all tagged with the same username. Also, we want to make sure that messages are actually broadcast – sent to all other connected users.

### Modeling connections

Both of these features need us to be able to keep track of the connections our server is holding – to know which user is sending the messages, and to know who to broadcast them to.

Ktor manages a WebSocket connection with an object of the type `DefaultWebSocketSession`, which contains everything required for communicating via WebSockets, including the `incoming` and `outgoing` channels, convenience methods for communication, and more. For now, we can simplify the problem of assigning user names, and just give each participant an auto-generated user name based on a counter. Add the following implementation to a new file in `server/src/main/kotlin/com/jetbrains/handson/chat/server/` called `Connection.kt`: 

```kotlin
import io.ktor.http.cio.websocket.*
import java.util.concurrent.atomic.*

class Connection(val session: DefaultWebSocketSession) {
    companion object {
        var lastId = AtomicInteger(0)
    }
    val name = "user${lastId.getAndIncrement()}"
}
```

Note that we are using `AtomicInteger` as a thread-safe data structure for the counter. This ensures that two users will never receive the same ID for their username – even when their two Connection objects are created simultaneously on separate threads.

### Implementing connection handling and message propagation

We can now adjust our server's program to keep track of our Connection objects, and send messages to all connected clients, prefixed with the correct user name. Adjust the implementation of the `routing` block in `server/src/main/kotlin/com/jetbrains/handson/chat/server/Application.kt` to the following code:

```kotlin
routing {
    val connections = Collections.synchronizedSet<Connection?>(LinkedHashSet())
    webSocket("/chat") {
        println("Adding user!")
        val thisConnection = Connection(this)
        connections += thisConnection
        try {
            send("You are connected! There are ${connections.count()} users here.")
            for (frame in incoming) {
                frame as? Frame.Text ?: continue
                val receivedText = frame.readText()
                val textWithUsername = "[${thisConnection.name}]: $receivedText"
                connections.forEach {
                    it.session.send(textWithUsername)
                }
            }
        } catch (e: Exception) {
            println(e.localizedMessage)
        } finally {
            println("Removing $thisConnection!")
            connections -= thisConnection
        }
    }
}
```

Our server now stores a (thread-safe) collection of `Connection`s. When a user connects, we create their `Connection` object (which also assigns itself a unique username), and add it to the collection. We then greet our user and let them know how many users are currently connecting. When we receive a message from the user, we prefix it with the unique name associated with their `Connection` object, and send it to all currently active connections. Finally, we remove the client's `Connection` object from our collection when the connection is terminated – either gracefully, when the incoming channel gets closed, or with an `Exception` when the network connection between client and server gets interrupted unexpectedly.

To see that our server is now behaving correctly – assigning user names and broadcasting them to everybody connected – we can once again run our application using the play button in the gutter and use the browser-based WebSocket client on https://www.websocket.org/echo.html to connect to `ws://localhost:8080/chat`. This time, we can use two separate browser tabs to validate that messages are exchanged properly.

![image-20201111155400473](./assets/image-20201111155400473.png)

As we can see, our finished chat server can now receive and send messages with multiple participants. Feel free to open a few more browser windows and play around with what we have built here!

In the next chapter, we will write a Kotlin chat client for our server, which will allow us to send and receive messages directly from the command line. Because our clients will also be implemented using Ktor, we will get to reuse much of what we learned about managing WebSockets in Kotlin.
