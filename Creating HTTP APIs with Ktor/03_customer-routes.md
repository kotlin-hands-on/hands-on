# Customer routes

First, let's tackle the `Customer` side of our application. We need to create a model which defines the data that's associated with a customer. We also need to create a series of endpoints to allow Customers to be added, listed, and deleted.

### The `Customer` model

For our case, a customer should store some basic information in the form of text: A customer should have an `id` by which we can identify them, a first and last name, and an email address. An easy way to model this in Kotlin is by using a data class.

Create a file name `Customer.kt` in a new package named `models` and add the following:

```kotlin
import kotlinx.serialization.Serializable

@Serializable
data class Customer(val id: String, val firstName: String, val lastName: String, val email: String)
```

Note that we are using the `@Serializable` annotation from [`kotlinx.serialization`](https://github.com/Kotlin/kotlinx.serialization). Together with its Ktor integration, this will allow us to generate the JSON representation we need for our API responses automatically – as we will see in just a bit.

First, we need to define a place to put all these potential customers.

### Storing customers

To not complicate the code, for this tutorial we'll be using an in-memory storage (i.e. a mutable list of `Customer`s) – in a real application, we would be storing this information in a database, so that it doesn't get lost after restarting our application. We can simply add this line right after the data class declaration in `Customer.kt` file:

```kotlin
val customerStorage = mutableListOf<Customer>()
```

Now that we have a well-defined `Customer` class and a storage for our customer objects, it's time we create endpoints and expose them via our API!

### Defining the routing for customers

We want to respond to `GET`, `POST`, and `DELETE` requests on the `/customer` endpoint. As such, let's define our routes with the corresponding HTTP methods. Create a file called `CustomerRoutes.kt` in a new package called `routes`, and fill it with the following:

```kotlin
import io.ktor.routing.*

fun Route.customerRouting() {
    route("/customer") {
        get {

        }
        get("{id}") {

        }
        post {

        }
        delete("{id}") {

        }
    }
}
```

In this case, we're using the `route` function to group everything that falls under the `/customer` endpoint. We then create a block for each HTTP method. This is just one approach how we can structure our routes – when we tackle the `Order` routes in the next chapter, we will see another approach.

Notice also how we actually have two entries for `get`: one without a route parameter, and the other with `{id}`. We'll use the first entry to list all customers, and the second to display a specific one.  

#### Listing all customers

To list all customers, we can simply return the `customerStorage` list by using the `call.respond` function in Ktor. which can take a Kotlin object and return it serialized in a specified format. For the `get` handler, it looks like this:

```kotlin
get {
    if (customerStorage.isNotEmpty()) {
        call.respond(customerStorage)
    } else {
        call.respondText("No customers found", status = HttpStatusCode.NotFound)
    }
}
```

In order for this to work, we need to enable [content negotiation](https://ktor.io/servers/features/content-negotiation.html) in Ktor. What does content negotiation do? Let us consider the following request:

```http request
GET http://0.0.0.0:8080/customer
Accept: application/json
```

When a client makes such a request, content negotiation allows the server to examine the `Accept` header, see if it can serve this specific type of content, and if so, return the result.

In our case, we're going to install the `ContentNegotitaion` feature and enable its support for JSON. Let's add the following code to the `Application.module()` function:

```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        json()
    }
}
```

JSON support is powered by `kotlinx-serialization`. We previously used its annotation `@Serializable` to annotate our `Customer` data class, meaning that Ktor now knows how to serialize `Customer`s (and collections of `Customer`s!)

#### Returning a specific customer

Another route we want to support is one that returns a specific customer based on their ID (in this case, `200`):

```http request
GET http://0.0.0.0:8080/customer/200
Accept: application/json
```

In Ktor, paths can also contain [parameters](https://ktor.io/servers/features/routing.html#parameters) that match specific path segments. We can access their value using the indexed access operator (`call.parameters["myParamName"]`). Let's add the following code to the `get("{id}")` entry:


```kotlin
get("{id}") {
    val id = call.parameters["id"] ?: return@get call.respondText(
        "Missing or malformed id",
        status = HttpStatusCode.BadRequest
    )
    val customer =
        customerStorage.find { it.id == id } ?: return@get call.respondText(
            "No customer with id $id",
            status = HttpStatusCode.NotFound
        )
    call.respond(customer)
}
```

First, we check whether the parameter `id` exists in the request. If it does not exist, we respond with a 400 "Bad Request" status code and an error message, and are done. If the parameter exists, we try to `find` the corresponding record in our `customerStorage`. If we find it, we'll respond with the object. Otherwise, we'll return a 404 "Not Found" status code with an error message.

Note that while we return a 400 "Bad request" when the `id` is null, this case should actually never be encountered. Why? Because this would only happen if no parameter `{id}` was passed in – but in this case, the route we defined previously would already handle the request.

#### Creating a customer

Next, we implement the option for a client to `POST` a JSON representation of a client object, which then gets put into our customer storage. Its implementation looks like this:

```kotlin
post {
    val customer = call.receive<Customer>()
    customerStorage.add(customer)
    call.respondText("Customer stored correctly", status = HttpStatusCode.Created)
}
```

`call.receive` integrates with the Content Negotiation feature we configured one of the previous sections. Calling it with the generic parameter `Customer` automatically deserializes the JSON request body into a Kotlin `Customer` object. We can then add the customer to our storage and respond with a status code of 201 "Created".

At this point, it is worth highlighting again that in this tutorial, we are also intentionally glancing over issues that could arise from e.g. multiple requests accessing the storage at the same time. In production, data structures and code that can be accessed from multiple requests / threads at the same time should account for these cases – something that is out of the scope of this hands-on.

#### Deleting a customer

The implementation for deleting a customer follows a similar procedure as we have used for listing a specific customer. We first get the `id` and then modify our `customerStorage` accordingly:

```kotlin
delete("{id}") {
    val id = call.parameters["id"] ?: return@delete call.respond(HttpStatusCode.BadRequest)
    if (customerStorage.removeIf { it.id == id }) {
        call.respondText("Customer removed correctly", status = HttpStatusCode.Accepted)
    } else {
        call.respondText("Not Found", status = HttpStatusCode.NotFound)
    }
}
```

Similar to the definition of our `get` request, we make sure that the `id` is not null. If the `id` is absent, we respond with a 400 "Bad Request" error.

### Registering the routes

Up until now, we have only defined our routes inside an extension function on `Route` – so Ktor doesn't know about our routes yet, and we need to register them. While we could certainly add each route directly in `Application.module` inside a `routing` block, it's more maintainable to group route registration in the corresponding file. We then just call the corresponding function to register all of them. Once we look at our implementation for `Orders`, this will hopefully be even more apparent. 

Let's add the following code to our `CustomerRoutes.kt` file:

```kotlin
fun Application.registerCustomerRoutes() {
    routing {
        customerRouting()
    }
}
```

Now we just need to invoke this function in our `Application.module()` function in `Application.kt`:

```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        json()
    }
    registerCustomerRoutes()
}
```

We've now completed the implementation for the customer-related routes in our API. If you would like to validate that everything works right away, you can skip ahead to the chapter about _Manually testing HTTP endpoints_. If you can still bear the suspense, we can move on to the implementation of order-related routes.
