# Creating the client

For developing the client, we're going to also use Ktor, as it's not only server-side, but
client-side technology too. 

We have one of two options for creating the client:

* Add it to our existing project as a new module
* Create a completely new project

In our case, we're going with the first option.

## Configuring the project file 

Let's add a new module to our project with the following `build.gradle` file

```groovy
plugins {
    id 'org.jetbrains.kotlin.jvm' version "$kotlin_version"
}


group 'com.jetbrains.handson'
version '1.0-SNAPSHOT'

repositories {
    jcenter()
    mavenCentral()
    maven {
        url "https://kotlin.bintray.com/kotlinx"
    }
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8"
    implementation "io.ktor:ktor-client-websockets:$ktor_version"
    implementation "io.ktor:ktor-client-okhttp:$ktor_version"
}

compileKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
compileTestKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
```

and our `gradle.properties` file contain the following versions

```
ktor_version=1.3.2
kotlin.code.style=official
kotlin_version=1.3.70
logback_version=1.2.1
kotlin_serialization=0.20.0
```

## Creating the client code

As we mentioned, we're going to use Ktor client library to talk to our server. For this, let's create a `main` function in Kotlin
that contains the following code

```kotlin
fun main() {
    val client = HttpClient {
        install(WebSockets)
    }
}
```

This creates a new instance of Ktor's `HttpClient` and initializes it to use `WebSocket` feature.

The next step is to call our endpoint and send it a message. We'll create a loop where we can constantly
send messages until we type the word `exit`

```kotlin
fun main() {
    val client = HttpClient {
        install(WebSockets)
    }
    client.webSocket(
            method = HttpMethod.Get,
            host = "0.0.0.0",
            port = 8080, path = "/chat"
    ) {
        var message = readLine() ?: ""
        while (!message.equals("exit", true)) {
            send(message)
            when (val frame = incoming.receive()) {
                is Frame.Text -> println(frame.readText())
                is Frame.Binary -> println(frame.readBytes())
            }
            message = readLine() ?: ""
        }
    }
    client.close()
    println("Connection closed")
}
```

If we try and write this code, we'll notice an error. This is because we're trying to call
a suspendable function from outside a coroutine. In order for it to work, and for the main application
loop to not terminate, we need to wrap this in a `runBlocking` call, which we can do at the main
block

```kotlin
fun main() = runBlocking {
    val client = HttpClient {
        install(WebSockets)
    }
    client.webSocket(
            method = HttpMethod.Get,
            host = "0.0.0.0",
            port = 8080, path = "/chat"
    ) {
        var message = readLine() ?: ""
        while (!message.equals("exit", true)) {
            send(message)
            when (val frame = incoming.receive()) {
                is Frame.Text -> println(frame.readText())
                is Frame.Binary -> println(frame.readBytes())
            }
            message = readLine() ?: ""
        }
    }
    client.close()
    println("Connection closed")
}
```

Notice that the only thing we've done is change `fun main() { }` to `fun main() = runBlocking { }`.

The rest of the code is self-explanatory as it is similar to what we did on the server: examine the type
of frame and act accordingly.
