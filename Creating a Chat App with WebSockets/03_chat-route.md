# Chat Route

This first step is to create a route for the WebSocket. In this case we are going to define the `/chat` route, but 
initially, we are going to make that route to act as an “echo” WebSocket route, that will send you back the same 
text messages that we send to it.

Let's create a file named Chat.kt and enter the following code:

```kotlin
fun Route.chatRoute() {
    webSocket("/chat") {
        while (true) {
            when (val frame = incoming.receive()) {
                is Frame.Text -> {
                    val text = frame.readText()
                    outgoing.send(Frame.Text(text))
                }
            }
        }
    }
}
```

`webSocket` routes are intended to be long-lived. Since it is a suspension block and uses 
lightweight Kotlin coroutines, it is fine and we can handle (depending on the machine and the complexity) hundreds of 
thousands of connections at once, while keeping our code easy to read and to write.

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

Finally, let's create the function that registers our route, which will later be called
by the application initialization code

```kotlin
fun Application.registerChatRoutes() {
    routing {
        chatRoute()
    }
}
```

