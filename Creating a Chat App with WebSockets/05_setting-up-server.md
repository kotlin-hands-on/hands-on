# Setting up the server

The only thing left to complete the server is to initialize it with support for web sockets and 
register the corresponding routes.

Let's create a file named `Application.kt` with the following code

```kotlin
fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

fun Application.module() {
    install(WebSockets)
    registerChatRoutes()
}
``` 

And our `application.conf` (remember to place this under resources) will be

```groovy
ktor {
    deployment {
        port = 8080
    }
    application {
        modules = [ com.jetbrains.handson.chat.ApplicationKt.module ]
    }
}
```

We should now be able to run the application, although we can't really test it yet. For that, let's 
write our client! 
