# Serving HTML 

Ktor allows us to respond to any request using a variety of functions

![Respond](./assets/respond.png)

Amongst these we can use `call.respondHtml` to send back HTML using [Kotlinx.HTML](https://github.com/Kotlin/kotlinx.html) which 
is a DSL that allows us to create HTML tags statically using Kotlin. 

In our case, let's create a new endpoint, namely `/html` that returns some HTML. We can do this by adding the following
code to our application

```kotlin
routing {
    // Other code
    call.respondHtml {
        head {
            title {
                +"Returning HTML using Kotlinx.HTML"
            }
        }
        body {
            h1 {
                +"Kotlinx.Html"
            }
            p {
                +"We're using a static HTML DSL"
            }
            val numbers = 1..10 
            ul {
                numbers.forEach { 
                    li {
                        value = it.toString()
                        +"Item $it"
                    }
                }
            }
        }
    }
}
```

Notice how we can seamlessly combine Kotlin code with HTML tags because after all, it's all Kotlin!

in particular we've added the `get` action to respond to the `/html` endpoint and inside we're adding our actual HTML. On calling this 
endpoint, the result should be as shown below

![Static HTML](./assets/static-html.png)



