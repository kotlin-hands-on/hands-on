# Automated testing

While manual testing is great and necessary, it also makes sense to have automated testing of endpoints. 

Thanks to `ktor-server-tests`, Ktor allows us to test endpoints without having to start up the entire underlying engine (such as Netty). The framework ships with a few helper methods for running
tests requests, one significant one being `withTestApplication`.

Let's write a unit test to ensure that our order route returns properly formatted JSON content. We create a new file under `test/kotlin` called `OrderTests.kt` and add the following code:

```kotlin
class OrderRouteTests {
    @Test
    fun testGetOrder() {
        withTestApplication({ module(testing = true) }) {
            handleRequest(HttpMethod.Get, "/order/2020-04-06-01").apply {
                assertEquals(
                    """{"number":"2020-04-06-01","contents":[{"item":"Ham Sandwich","amount":2,"price":5.5},{"item":"Water","amount":1,"price":1.5},{"item":"Beer","amount":3,"price":2.3},{"item":"Cheesecake","amount":1,"price":3.75}]}""",
                    response.content
                )
                assertEquals(HttpStatusCode.OK, response.status())
            }
        }
    }
}
```

By using `withTestApplication`, we're indicating to our application that we want to run it as a test. Using the `handleRequest` helper method (also shipped as part of Ktor), we define request to a specific endpoint, in this case `/order/{id}`.

Note that since our string contains a lot of quotation marks around keys and values (like `"number"`), this is a great place to use [raw strings](https://kotlinlang.org/docs/reference/basic-types.html#string-literals) using triple-quotes (`"""`), saving us the hassle of individually escaping every special character inside the string.

If we try and compile this code however, it won't work. This is due to the parameter being passed to our application (`testing = true`). For this to work, we need to add the corresponding parameter to our application:

```kotlin
fun Application.module(testing: Boolean = false) {
    install(ContentNegotiation) {
        json()
    }
    registerCustomerRoutes()
    registerOrderRoutes()
}
```

With this, we can now run our unit test from the IDE and see the results. Much like we've done for this endpoint, we can add all other endpoints as tests and automate the testing of our HTTP API.

And just like that, we have finished building our small JSON-based HTTP API. Of course, there are tons of topics you can still explore around Ktor and building APIs with it, so your learning journey doesn't have to stop here!

