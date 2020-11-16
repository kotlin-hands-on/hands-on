# Creating the chat client

Because we use Ktor as a WebSocket client library, the code and methods we can use are very similar to those on the server. We can build a first, very simple implementation of sending and receiving messages by changing the file `client/src/main/kotlin/com/jetbrains/handson/chat/client/ChatClient.kt` to look as follows:

```kotlin
import io.ktor.client.*
import io.ktor.client.features.websocket.*
import io.ktor.http.*
import io.ktor.http.cio.websocket.*
import io.ktor.util.*
import kotlinx.coroutines.*

@KtorExperimentalAPI
fun main() {
    val client = HttpClient {
        install(WebSockets)
    }
    runBlocking {
        client.webSocket(method = HttpMethod.Get, host = "127.0.0.1", port = 8080, path = "/chat") {
            while(true) {
                val othersMessage = incoming.receive() as? Frame.Text ?: continue
                println(othersMessage.readText())
                val myMessage = readLine()
                if(myMessage != null) {
                    send(myMessage)
                }
            }
        }
    }
    client.close()
    println("Connection closed. Goodbye!")
}
```

Here, we first create an `HttpClient` and set up Ktor's `WebSocket` feature (the analog of installing the `WebSocket` feature in our server application's module in an earlier chapter). Functions in Ktor responsible for making network calls use the suspension mechanism from Kotlin's coroutines, so we wrap our network-related code in a `runBlocking` block. Inside the WebSocket handler, we once again process incoming messages and send outgoing messages: we ignore frames which do not contain text, read incoming text, and send the user input to the server.

However, this "straightforward" implementation actually contains an issue which prevents it from being used as a proper chat client: when invoking `readLine()`, our program waits until the user enters a message. During this time, we can't see any messages which have been typed out by other users. Likewise, because we invoke `readLine()` after every received message, we would only ever see one new message at a time.

You can also validate this for yourself: with the server process running, start two instances of the chat client by clicking play icon in the gutter in `client/src/main/kotlin/com/jetbrains/handson/chat/client/ChatClient.kt`. Use the tabs in the "Run" tool window to navigate between the two client instances, and send some messages back and forth.

![image-20201111191815343](./assets/image-20201111191815343.png)

Let's address this issue, and build a better solution!

### Improving our solution

A better structure for our chat client would be to separate the message output and input mechanisms, allowing them to run concurrently: when new messages arrive, they are printed immediately, but our users can still start composing a new chat message at any point.

We know that to output messages, we need to be able to receive them from the WebSocket's `incoming` channel, and print them to the command line. Let’s add a function called `outputMessages()` to the `ChatClient.kt` file with the following implementation for this functionality:

```kotlin
suspend fun DefaultClientWebSocketSession.outputMessages() {
    try {
        for (message in incoming) {
            message as? Frame.Text ?: continue
            println(message.readText())
        }
    } catch (e: Exception) {
        println("Error while receiving: " + e.localizedMessage)
    }
}
```

Because the function operates in the context of a `DefaultClientWebSocketSession`, we define `outputMessages()` as an extension function on the type. We also don’t forget to add the `suspend` modifier – because iterating over the `incoming` channel suspends the coroutine while no new message is available.

Next, let’s define a second function which allows the user to input text. Add a function called `inputMessages()` in `ChatClient.kt` with the following implementation

```kotlin
suspend fun DefaultClientWebSocketSession.inputMessages() {
    while (true) {
        val message = readLine() ?: ""
        if (message.equals("exit", true)) return
        try {
            send(message)
        } catch (e: Exception) {
            println("Error while sending: " + e.localizedMessage)
            return
        }
    }
}
```

Once again defined as a suspending extension function on `DefaultClientWebSocketSession`, this function's only job is to read text from the command line and send it to the server, or to return when the user types `exit`.

Where we previously had one loop which had to take care of reading input and printing output, we now have separated these tasks into their own functions, which can operate independently from each other.

#### Wiring it together

Let's make use of our two new functions! We can call them inside the body of our WebSocket handler by changing the code of our `main()` method in `ChatClient.kt` to the following:

```kotlin
@KtorExperimentalAPI
fun main() {
    val client = HttpClient {
        install(WebSockets)
    }
    runBlocking {
        client.webSocket(method = HttpMethod.Get, host = "127.0.0.1", port = 8080, path = "/chat") {
            val messageOutputRoutine = launch { outputMessages() }
            val userInputRoutine = launch { inputMessages() }
            
            userInputRoutine.join() // Wait for completion; either "exit" or error
            messageOutputRoutine.cancelAndJoin()
        }
    }
    client.close()
    println("Connection closed. Goodbye!")
}
```

This new implementation improves the behavior of our application: Once the connection to our chat server is established, we use the `launch` function from Kotlin's Coroutines library to launch the two long-running functions `outputMessages()` and `inputMessages()` on a new coroutine (without blocking the current thread). The launch function also returns a `Job` object for both of them, which we use to keep the program running until the user types `exit` or encounters a network error when trying to send a message. After `inputMessages()` has returned, we cancel the execution of the `outputMessages()` function, and `close` the client.

Until this happens, both input and output can happily happen concurrently, with new messages being received while the client sits idle, and the option to start composing a new message at any point.

#### Let's give it a try!

We have now finished implementing our WebSocket-based chat client with Kotlin and Ktor. To celebrate our success, let’s give it a try! With the chat server running, start some instances of the chat client using the play button, and talk to yourself! Even if you send multiple messages right after each other, they should be correctly displayed on all connected clients.

You might still notice some smaller usability issues caused by the limitations of terminal input, like incoming messages overwriting messages which are currently being composed. Managing more complex terminal user interfaces is outside the scope of this tutorial, though, and as such left as an exercise to the reader 😉.

[You can also find the final version of the project on the final branch on GitHub.](https://github.com/kotlin-hands-on/chat-app-websockets/tree/final)

![app_in_action](./assets/app_in_action.gif)

That's it for this hands-on tutorial on WebSockets with Ktor – time to congratulate yourself for building a whole application! If you're looking for some inspiration of where to take this project next, as well as related materials, continue to the next section.

