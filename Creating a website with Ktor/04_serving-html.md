# Serving HTML 

When we autocomplete on the `call` object, we can see that Ktor allows us to respond to any request using a variety of functions:

![Respond](./assets/respond.png)

Amongst these we can use `call.respondHtml` to send back HTML. This uses [kotlinx.html](https://github.com/Kotlin/kotlinx.html), a DSL that allows us to create HTML tags directly using Kotlin syntax. 

In our case, let's create a new endpoint, namely `/html`, that returns some HTML. We can do this by adding the following
code to our application:

```kotlin
routing {
    // other code . . .
    get("/html") {
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
    // other code . . .
}
```

Notice how we can seamlessly combine Kotlin code with HTML tags because after all, it's all Kotlin!

In the code above we've added the `get` action to respond to the `/html` endpoint. Inside the `respondHtml` call, we're adding our actual markup. On calling this 
endpoint, the result should be as shown below:

![Static HTML](./assets/static-html.png)



