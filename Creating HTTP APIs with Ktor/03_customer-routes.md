# Customer Routes

Our first step is to create a series of endpoints to allow Customers to be 
added, listed, and deleted. 


## Storage 
To not complicate the code, for this tutorial
we'll be using an in memory storage (i.e. a mutable list of customers). Usually we'd 
be storing this information in a database. 

Create a file name `Customer.kt` in a new package named `models` and inside it copy the following
contents

```kotlin
import kotlinx.serialization.Serializable


@Serializable
data class Customer(val id: String, val firstName: String, val lastName: String, val email: String)


val customerStorage = mutableListOf<Customer>()
```

This defines our data class for representing a customer, as well as creating our storage.


## Defining the routing for Customer

We want to respond to GET, POST, DELETE requests on `/customer` endpoint. As such, let's define our routes with the 
corresponding verbs. For now let's leave the code blocks empty.

```kotlin
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

In this case, we're using the `route` function to group everything that falls under the `/customer` endpoint. We then create
a block for each verb (for the `Orders` example we'll see a different approach). Notice how we actually have two entries
for `get`, one without a route parameter, and the other with (`{id`). We'll use the first entry to list all customers, and the second
to display a specific one.  

#### Listing all customers

To list all customers, we can simply return the `customerStorage` list by using `call.respond` function in Ktor, which can
take any object and return it serialized in a specific format:

```kotlin
get {
    if (customerStorage.isNotEmpty()) {
        call.respond(customerStorage)
    } else {
        call.respondText("No customers found", status = HttpStatusCode.NotFound)
    }
}
```

In order for this to work however, we need to tell Ktor how we need to enable Content Negotiation in Ktor. This means that
when a client makes the following request:

```http request
GET http://0.0.0.0:8080/customer
Accept: application/json
```

the server can examine the `Accept` header, see if it can serve this specific type of content, and if so, return the result.

In our case we're going to support JSON, which we can indicate when configuring Content Negotiation. Let's add the following
code to the `Application.module` function we defined in the previous step:

```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        json()
    }
}
```

### Returning a specific customer

The following request should return a specific customer

```http request
GET http://0.0.0.0:8080/customer/200
Accept: application/json
```

where the router parameter is `200`. In Ktor, we can access this value use `call.parameters[{param_name}]`. Let's add
the following code to the `get({id})` entry:


```kotlin
get("{id}") {
    val id = call.parameters["id"]
    if (id != null) {
        val customer = customerStorage.find { it.id.compareTo(id) == 0 }
        if (customer != null) {
            call.respond(customer)
        } else {
            call.respondText("Not Found", status = HttpStatusCode.NotFound)
        }
    }
}
```

The first line is checking to see if the parameter exists. If it does exist (i.e. does not return null), we'll then 
try to locate the corresponding record. If we find it, we'll respond with the object. Otherwise, we'll return a 
`Not Found` message with the 404 status code.

You may be wondering whether we should be handling the scenario where `id` is null. In principle this isn't required as
this would only happen if a parameter `{id}` were not passed in, in which case the previous route would handle the request.

### Creating a customer

Creating a customer is very straightforward. 

```kotlin
post {
    val customer = call.receive<Customer>()
    customerStorage.add(customer)
    call.respondText("Customer stored correctly", status = HttpStatusCode.Accepted)
}
``` 

We first receive the customer using `call.receive` which automatically deserializes the JSON request into the `Customer` object.
Next, we add the customer to storage and respond with status code 201.

It's important to highlight again that we're using a mutable list as storage, and for demo purposes we're not properly writing to 
it with lock access, something that ideally should be done as multiple requests could come in at the same time. We've omitted this code
as in production we wouldn't really be using in-memory storage like this.

### Deleting a customer

Deleting a customer follows a similar procedure to listing a specific customer. We first get the `id` and then process the request.

```kotlin
delete("{id}") {
    val id = call.parameters["id"]
    if (id != null) {
        if (customerStorage.removeIf { it.id == id }) {
            call.respondText("Customer removed correctly", status = HttpStatusCode.Accepted)
        } else {
            call.respondText("Not Found", status = HttpStatusCode.NotFound)
        }
    } else {
        call.respondText("Bad Request", status = HttpStatusCode.BadRequest)
    }
}
```

One difference with the `get` is that we're explicitly checking to make sure the `id` is not null. If it is, we can respond
with a bad request 400 code. This is because we don't have a pattern `delete` with no route parameters, as we did in the case
of `get`. 

## Registering the routes

The only thing left now is for us to register our routes. While we could certainly add each route directly in `Application.module`
under `routing`, it's more maintainable to group route registration in the corresponding file and merely call the corresponding
function to register all of them (this will be even more apparent when we look at `Orders` in the next step). 

Let's add the following code to our `CustomerRoutes.kt` file

```kotlin
fun Application.registerCustomerRoutes() {
    routing {
        customerRouting()
    }
}
```

Now we just need to invoke this function in our `Application.module` function

```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        json()
    }
    registerCustomerRoutes()
}
```







 