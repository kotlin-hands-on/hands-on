# A first echo server

### Implementing an echo server

Let’s start our server development journey by building a small “echo” service which accepts WebSocket connections, receives text content, and sends it back to the client. We can implement this service with Kotlin and Ktor by adding the following implementation for `Application.module()` to `server/src/main/kotlin/com/jetbrains/handson/chat/server/Application.kt`:

```kotlin
fun Application.module() {
    install(WebSockets)
    routing {
        webSocket("/chat") {
            send("You are connected!")
            for(frame in incoming) {
                frame as? Frame.Text ?: continue
                val receivedText = frame.readText()
                send("You said: $receivedText")
            }
        }
    }
}
```

We first enable WebSocket-related functionality provided by the Ktor framework by installing the `WebSockets` Ktor feature. This allows us to define endpoints in our routing which respond to the WebSocket protocol (in our case, the route is `/chat`). Within the scope of the `webSocket` route function, we can use various methods for interacting with our clients (via the `DefaultWebSocketServerSession` receiver type). This includes convenience methods to send messages and iterate over received messages.

Because we are only interested in text content, we skip any non-text `Frame`s we receive when iterating over the incoming channel. We can then read any received text, and send it right back to the user with the prefix `"You said:"`.

At this point, we have already built a fully-functioning echo server – a little service that just sends back whatever we send it. Let's try it out!

### Trying out the echo server

For now, we can use a web-based WebSocket client to connect to our echo service, send a message, and receive the echoed reply. Once we have finished implementing the server-side functionality, we will also build our own chat client in Kotlin.

Let's start the server by pressing the play button in the gutter next to the definition of fun main in our server's Application.kt. After our project has finished compiling, we should see a confirmation that the server is running in IntelliJ IDEAs "Run" window: `Application - Responding at http://0.0.0.0:8080`. To try out the service, we can open the WebSocket client provided at https://www.websocket.org/echo.html and use it to connect to `ws://localhost:8080/chat`.

![image-20201022122125926](./assets/image-20201022122125926.png)

Then, we can enter any kind of message in the "Message" window, and send it to our local server. If everything has gone according to plan, we should see the `SENT` and `RECEIVED` messages and in the logs, indicating that our echo-server is functioning just as intended.

With this, we now have a solid foundation for bidirectional communication through WebSockets. Next, let's expand our program more closely resemble a chat server, allowing multiple participants to share messages with others.
