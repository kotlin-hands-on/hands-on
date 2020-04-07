# Application Initialization 


Let's set up our `Application.module` function to have it ready to
configure our routes and other features needed for our application.

```kotlin
fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

fun Application.module() {

}
```

In later steps we'll add features and routing to the function. 

