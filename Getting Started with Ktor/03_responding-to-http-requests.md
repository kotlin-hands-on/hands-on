# Responding to HTTP requests

We're now ready to write our actual application which is going to respond to the path `/` with `Hello, World!`. 
For this, let's create a new Kotlin file named `Application.kt` in the folder `kotlin` and enter the following content

```kotlin
import io.ktor.application.call
import io.ktor.http.ContentType
import io.ktor.response.respondText
import io.ktor.routing.get
import io.ktor.routing.routing
import io.ktor.server.engine.embeddedServer
import io.ktor.server.netty.Netty


fun main() {
    val server = embeddedServer(Netty, 8080) {
        routing {
            get("/") {
                call.respondText("Hello, World!", ContentType.Text.Plain)
            }
        }
    }
    server.start(wait = true)
}
```

The first thing we're doing is creating an embedded server and indicating that we're going to self-host using Netty, on port 8080. 
In real-world application we'd usually do this in a [configuration file](https://ktor.io/servers/configuration.html), but for our
purposes this works well.

The next step is to define our actual routing table, which in this case has a single entry `get("/)` responding to the `/` URL. 
When a user makes the request, we'll respond with the corresponding text, which is done using `call.respondText` and passing the content type. In our case
as can be seen, this is `text/plain`. 

Finally we start the server and pass as argument `wait` the value true so that the application doesn't immediately exit, i.e. it stays running until we kill it.

 
