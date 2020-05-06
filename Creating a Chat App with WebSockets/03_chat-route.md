# A simple echo server

Before we dive into any chat functionality, let's build a minimal example so that we can get an understanding of how WebSockets work in Ktor. We'll define a simple WebSocket handler that just takes whatever you send to it and sends it back to you – an "echo" server.

### Setting up the route
To get every message that arrives on a WebSocket connection to `/chat` echoed back to the user, we create a handler function for the endpoint and register it with our application module.

#### Creating a `Route` extension

Let's create a file named `Chat.kt` and enter the following code:
```kotlin
fun Route.chatRoute() {
    webSocket("/chat") {
        for (frame in incoming) {
            frame as? Frame.Text ?: continue
            val receivedText = frame.readText()
            outgoing.send(Frame.Text(receivedText))
        }
    }
}
```

`webSocket` routes are intended to be long-lived. Since it is a suspension block and uses 
lightweight Kotlin coroutines, it is fine and we can handle (depending on the machine and the complexity) hundreds of 
thousands of connections at once, while keeping our code easy to read and to write.

At this point, the `chatRoute` function is still unused (and our IDE will even warn us about this), so our next step is to register this route with our application.

#### Registering the route

In `Application.kt`, we change the code of `Application.module()` to look as follows:
```kotlin
fun Application.module() {
    install(WebSockets)
    routing {
        chatRoute()
    }
}
```

Note that there are many ways how you can structure your routes with Ktor. If you'd like to learn more about different styles of routing in Ktor, check out Hadi Hariri's blogpost on [Routing in Ktor](https://hadihariri.com/2020/04/02/Routing-in-Ktor/).

This is the first time where we start running our application for the first time!

#### Seeing it in action



