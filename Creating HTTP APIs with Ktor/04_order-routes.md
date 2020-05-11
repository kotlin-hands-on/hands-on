# Order routes

Now that we have API endpoints for `Customer`s done, let's move on to `Orders`. While some of the implementation is rather similar, we will be using a different way of structuring our application routes, and include routes that sum up the price of individual items in an order. 


### Defining the model

The orders we want to store in our system should be identifiable by an order number (which might contain dashes), and should contain a list of order items. These order items should have a textual description, the number how often this item appears in the order, as well as the price for the individual item (so that we can compute the total price of an order on demand).

We create a new file called `Order.kt` and fill it with the definition of the two data classes:

```kotlin
import kotlinx.serialization.Serializable

@Serializable
data class Order(val number: String, val contents: List<OrderItem>)

@Serializable
data class OrderItem(val item: String, val amount: Int, val price: Double)
```

We also once again need a place to store our orders. To skip having to define a `POST` route – something you're more than welcome to attempt on your own using the knowledge from the `Customer` routes – we will prepopulate our `orderStorage` with some sample orders. We can again define it as a top-level declaration inside the `Order.kt` file.

```kotlin
val orderStorage = listOf(Order(
    "2020-04-06-01", listOf(
        OrderItem("Ham Sandwich", 2, 5.50),
        OrderItem("Water", 1, 1.50),
        OrderItem("Beer", 3, 2.30),
        OrderItem("Cheesecake", 1, 3.75)
    )),
    Order("2020-04-03-01", listOf(
        OrderItem("Cheeseburger", 1, 8.50),
        OrderItem("Water", 2, 1.50),
        OrderItem("Coke", 2, 1.76),
        OrderItem("Ice Cream", 1, 2.35)
    ))
)
```

### Defining order routes
We respond to a set of `GET` requests with three different patterns:



```kotlin
GET http://0.0.0.0:8080/order/
Content-Type: application/json

GET http://0.0.0.0:8080/order/{id}
Content-Type: application/json

GET http://0.0.0.0:8080/order/{id}/total
Content-Type: application/json
```

The first will return all orders, the second will return an order given the `id`, and the third will return the total of an order (prices of individual `OrderItems` multiplied by number of each item).

With orders, we're going to follow a different pattern when it comes to defining routes. 
Instead of grouping all routes under a single `route` function with different
HTTP methods, we'll use individual functions.

#### Listing all and individual orders

For listing all orders, we'll follow the same pattern as with customers – the difference 
being that we're defining it in its own function. Let's create a file called `OrderRoutes.kt` inside the `routes` package, and start with the implementation of the route inside a function called `listOrdersRoute()`.

```kotlin
fun Route.listOrdersRoute() {
    get("/order") {
        if (orderStorage.isNotEmpty()) {
            call.respond(orderStorage)
        }
    }
}
```

We apply the same structure to individual orders – with a similar implementation to customers, but encapsulated in its own function:

```kotlin
fun Route.getOrderRoute() {
    get("/order/{id}") {
        val id = call.parameters["id"] ?: return@get call.respondText("Bad Request", status = HttpStatusCode.BadRequest)
        val order = orderStorage.find { it.number == id } ?: return@get call.respondText(
            "Not Found",
            status = HttpStatusCode.NotFound
        )
        call.respond(order)
    }
}
```

#### Totalizing an order

Getting the total amount of an order consists of iterating over the items of an order and 
totalizing this. Implemented as a `totalizeOrderRoute` function, it looks like this, which besides the summing process should already look familiar:

```kotlin
fun Route.totalizeOrderRoute() {
    get("/order/{id}/total") {
        val id = call.parameters["id"] ?: return@get call.respondText("Bad Request", status = HttpStatusCode.BadRequest)
        val order = orderStorage.find { it.number == id } ?: return@get call.respondText(
            "Not Found",
            status = HttpStatusCode.NotFound
        )
        val total = order.contents.map { it.price * it.amount }.sum()
        call.respond(total)
    }
}
```

A small thing to note here is that we are not limited to suffixes of routes for parameters – as we can see, it's absolutely possible to have a section in the middle be a route parameter (`/order/{id}/total`).

### Registering the routes

Finally, much like the case of customers, we need to register the routes. Hopefully, this makes it clear why grouping routes makes more sense as the number of routes grow. Still in `OrderRoutes.kt`, we add an `Application` extension function called `registerOrderRoutes`:

```kotlin
fun Application.registerOrderRoutes() {
    routing {
        listOrdersRoute()
        getOrderRoute()
        totalizeOrderRoute()
    }
}
```

We then add the function call in our `Application.module()` function in `Application.kt`:

```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        json()
    }
    registerCustomerRoutes()
    registerOrderRoutes()
}
```

Now that we have everything wired up, we can finally start testing our application, and see if everything works as we would expect it to!