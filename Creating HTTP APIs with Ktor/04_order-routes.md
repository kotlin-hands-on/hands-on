# Order Routes

Now that we have our `/customer` endpoint done, let's move on to `Orders`, where we'll
be responding to GET requests, but using two different patterns

```
GET http://0.0.0.0:8080/order/
Content-Type: application/json

GET http://0.0.0.0:8080/order/{id}
Content-Type: application/json


GET http://0.0.0.0:8080/order/{id}/total
Content-Type: application/json
```

The first will return all orders, the second will return an order given the `id`, and the third will return the total of
an order.

## Defining the model

Let's first define our model and storage 

```kotlin
import kotlinx.serialization.Serializable

@Serializable
data class Order(val number: String, val contents: List<OrderItem>)

@Serializable
data class OrderItem(val item: String, val amount: Int, val price: Double)

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

In this case, and as we don't have a POST verb, we'll prepopulate our database with some orders.

## Defining Order Routes

With orders, we're going to follow a different pattern when it comes to defining routes. 
Instead of grouping all routes under a single `route` function with different
verbs, we'll use individual functions

### Listing all orders, and a specific order

For listing all orders, we'll follow the same pattern as with customers, with the difference 
that we're defining it in its own function:

```kotlin
fun Route.listOrdersRoute() {
    get("/order") {
        if (orderStorage.isNotEmpty()) {
            call.respond(orderStorage)
        }
    }
}
```

For returning a specific order, once again similar to customers, but again in its own function

```kotlin
fun Route.getOrderRoute() {
    get("/order/{id}") {
        val id = call.parameters["id"]
        if (id != null) {
            val order = orderStorage.find { it.number.compareTo(id) == 0 }
            if (order != null) {
                call.respond(order)
            } else {
                call.respondText("Not Found", status = HttpStatusCode.NotFound)
            }
        }
    }
}
``` 

### Totalizing an order

Getting the total amount of an order consists of iterating over the the items of an order and 
totalizing this. The code itself is straightforward. 

```kotlin
fun Route.totalizeOrderRoute() {
    get("/order/{id}/total") {
        val id = call.parameters["id"]
        if (id != null) {
            val order = orderStorage.find { it.number.compareTo(id) == 0 }
            if (order != null) {
                val total = order.contents.map { it.price * it.amount }.sumByDouble { it }
                call.respondText("Total for order is $total")
            } else {
                call.respondText("Not Found", status = HttpStatusCode.NotFound)
            }
        } else {
            call.respondText("Bad Request", status = HttpStatusCode.BadRequest)
        }
    }
}
```

What we notice here is that we're using a route parameter is part of the URL and not necessarily 
trailing, which is absolutely possible. 

Rest of the code is similar to what we've already seen.

## Registering the route 

Finally, much like the case of customers, we need to register the routes. As mentioned in the previous 
step, in this case we can see why grouping routing makes things more sense.

```kotlin
fun Application.registerOrderRoutes() {
    routing {
        listOrdersRoute()
        getOrderRoute()
        totalizeOrderRoute()
    }
}
```

We then add the function call in our `Application.module` function:

```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        json()
    }
    registerCustomerRoutes()
    registerOrderRoutes()
}
```