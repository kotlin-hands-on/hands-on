# Automated Testing

While manual testing is great and necessary, it also makes sense to
have automated testing of endpoints. 

Ktor allows us to test endpoints without having to start up the entire underlying 
engine (such as Netty). The framework ships with a few helper methods for running
tests requests, namely, `withTestApplication`

Create a new file under the `test` folder of the project named `OrderTests.kt` and enter the following 
code:
 
```kotlin
class OrderRouteTests {
    @Test
    fun testGetOrder() {
        withTestApplication({ module(testing = true) }) {
            handleRequest(HttpMethod.Get, "/order/2020-04-06-01").apply {
                assertEquals("{\"number\":\"2020-04-06-01\",\"contents\":[{\"item\":\"Ham Sandwich\",\"amount\":2,\"price\":5.5},{\"item\":\"Water\",\"amount\":1,\"price\":1.5},{\"item\":\"Beer\",\"amount\":3,\"price\":2.3},{\"item\":\"Cheesecake\",\"amount\":1,\"price\":3.75}]}", response.content)
                assertEquals(HttpStatusCode.OK, response.status())
            }
        }
    }
}
```

We're indicating to our application that we want to run it as a test, and then using the
`handleRequest` helper method (also shipped as part of Ktor), we'd like to make a request
to a specific endpoint, in this case `/order/{id}`. 

If we try and compile this code however, it won't work. This is due to the parameter being passed in 
to our application (testing = true). For this to work, we need to add this to our application

```kotlin
fun Application.module(testing: Boolean = false) {
    install(ContentNegotiation) {
        json()
    }
    registerCustomerRoutes()
    registerOrderRoutes()
}
```

With this, we can now run our unit test and see the results.

Much like we've done for this endpoint, we can add all other endpoints as tests and automate the testing of our HTTP endpoint.

With this step, we've finalized our HTTP API application. From here on we can add other 
features such as Authentication, etc.


