# Adding persistence

### Why MongoDB?

The main point why we choose MongoDB for a small project like this one is simplicity. MongoDB is fast to set up, has library support for Kotlin, and provides simple, NoSQL document storage, which is more than enough for the basic application we are building. Of course, many monologues and dialogues can be held about the choice of database for any given application, and you are more than welcome to equip your application with a different mechanism for persistence.

### Setting up MongoDB

To set up MongoDB Community Edition on your local development machine, please refer to the tutorials on the official [MongoDB website](https://docs.mongodb.com/manual/installation/#mongodb-community-edition-installation-tutorials) for your respective operating system.

After installation, ensure that you are running the `mongodb-community` service for the rest of the tutorial. We will use it to store and retrieve our list entries.

### Moving to real data

[KMongo](https://litote.org/kmongo/) is a community-created Kotlin framework that makes it easy to work with MongoDB from Kotlin/JVM code. It also plays nicely with `kotlinx.serialization`, which we have previously used in this hands-on to facilitate communication between client and server.

By migrating our code to use an external database, we no longer need to keep a collection of `shoppingListItems`  on the server. Instead, we set up a database client, and obtain a database and a collection from it.

Inside `src/jvmMain/kotlin/Server.kt`, we remove the declaration for `shoppingList`, and add the following three top-level variables:

```kotlin
val client = KMongo.createClient().coroutine
val database = client.getDatabase("shoppingList")
val collection = database.getCollection<ShoppingListItem>()
```

We can now adjust our routes to GET, POST, and DELETE a `ShoppingListItem` to make use of the collection operations that we have available. Still in `src/jvmMain/kotlin/Server.kt`, replace the definitions for the GET, POST, and DELETE routes with the following:

```kotlin
get {
    call.respond(collection.find().toList())
}
post {
    collection.insertOne(call.receive<ShoppingListItem>())
    call.respond(HttpStatusCode.OK)
}
delete("/{id}") {
    val id = call.parameters["id"]?.toInt() ?: error("Invalid delete request")
    collection.deleteOne(ShoppingListItem::id eq id)
    call.respond(HttpStatusCode.OK)
}
```

While GET and POST requests are very simply transformed, the most interesting change is in the DELETE request: we make use of KMongo's [type-safe queries](https://litote.org/kmongo/typed-queries/) to obtain and remove the correct `ShoppingListItem` from our database.

And just like that, we have added database support to our application! We start up the server again using the `run` task, and navigate to `http://localhost:9090/`. At first start, we'll be greeted by an empty shopping list – as is expected when querying an empty database. Any new entries we make will now be persisted to the database by the server; we can see this behavior best by restarting the server and reloading the page.

### Inspecting MongoDB

If we want to see what kind of information is actually persisted in our database – something that might not be entirely obvious, as it is hidden through KMongo's interface – we can inspect the database using external tools.

If you're running IntelliJ IDEA Ultimate Edition or DataGrip version 2019.3 or newer, you have the option of inspecting the database contents with these tools. Alternatively, you can use the `mongo` command line client.

To connect to the local MongoDB instance, we start by creating a datasource in the "Database" tab in IntelliJ IDEA Ultimate or DataGrip:

![](./assets/mongodb_data_source.png)

If it's our first time connecting to a MongoDB database this way, we might be prompted to download missing drivers:

![](./assets/download_missing_drivers.png)

When working with a local MongoDB installation that uses default settings, no adjustments need to be made to the configuration. We can test the connection with the "Test Connection button", which should output the MongoDB version, as well as some additional information. After confirming with the OK button, we can use the Database tool window to navigate to our collection and have a look at everything stored in it.

![image-20200407175857354](/assets/image-20200407175857354.png)

With this, we've reached the end of building our application – but of course, such a wonderful app deserves a bigger stage than `localhost`. It is ready to be shown to the world! Stick around if you'd like to learn how to bring the application onto the web by deploying it to the cloud!

### Relevant Gradle configuration

Kmongo is added with a single dependency to the project – a specific version that includes coroutine and serialization support out of the box:

```kotlin
val jvmMain by getting {
    dependencies {
        // . . .
        implementation("org.litote.kmongo:kmongo-coroutine-serialization:$kmongoVersion")
    }
}
```
