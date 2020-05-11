# Form submission and kx.html

The `<form>` we defined in the previous chapter sends a `POST` request to `/submit` containing the `headline` and `body` of our new journal entry. Let's make our application correctly consume this information and submission of new journal entries! We define a handler for the `/submit` route inside our `Application.module()`'s `routing` block like this:

```kotlin
post("/submit") {
    val params = call.receiveParameters()
    val headline = params["headline"] ?: return@post call.respond(HttpStatusCode.BadRequest)
    val body = params["body"] ?: return@post call.respond(HttpStatusCode.BadRequest)
    val newEntry = BlogEntry(headline, body)
    blogEntries.add(0, newEntry)
    // TODO: send a status page to the user
}
```

[`receiveParameters`](https://ktor.io/servers/calls/requests.html#post) allows us to parse form data (for both `urlencoded` and `multipart`). We then extract the `headline` and `body` fields from the form, ensuring they are both not null, and create a new `BlogEntry` object from this information, adding it to the beginning of our `blogEntries` list.

For more detailed information on the fancy features that are available in the context of Ktor's request model, check out the [official documentation](https://ktor.io/servers/calls/requests.html).

To show the user that the submission was successful, we still want to send back a bit of HTML. We could re-use our knowledge and create a FreeMarker template for this "Success" page as well – but to cover some more Ktor functionality, we will try an alternative approach instead. When we autocomplete on the `call` object, we can see that Ktor allows us to respond to requests using a variety of functions:

![Respond](./assets/respond.png)

Amongst these is `call.respondHtml`. This function allows us to craft our HTML response using Kotlin syntax, thanks to Ktor's integration with [kotlinx.html](https://github.com/Kotlin/kotlinx.html) (a [DSL](https://kotlinlang.org/docs/reference/type-safe-builders.html) designed for seamlessly combining Kotlin code and HTML-like tags).

At the bottom of the `post("/submit")` handler block, let's add the following code:

```kotlin
call.respondHtml {
    body {
        h1 {
            +"Thanks for submitting your entry!"
        }
        p {
            +"We've submitted your new entry titled "
            b {
                +newEntry.headline
            }
        }
        p {
            +"You have submitted a total of ${blogEntries.count()} articles!"
        }
        a("/") {
            +"Go back"
        }
    }
}
```

Notice how this code combines Kotlin-specific logic (like `count`ing the entries, or using string interpolation) with HTML-like syntax!

To test the route, let's re-run our application, navigate to [`http://localhost:8080/`](http://localhost:8080/), and submit a new entry. If everything has gone according to play, we'll now see the HTML page, courtesy of kotlinx.html!

![Static HTML](./assets/submit.png)

### We're done!

It's time to pat ourselves on the back – we've put together a nice little journal application, and have learned about many topics in the meantime. From static files to templating, from basic routing to kotlinx.html, we've covered a lot of ground.

![](./assets/ktor_journal.gif)

This concludes the guided part of this hands-on. We have included the final state of the journal application in the GitHub repository on the [`final` branch](https://github.com/kotlin-hands-on/creating-website-ktor/tree/final). But of course, your journey doesn't have to stop here. Check out the _What's next_ section to get an idea of how you could expand the application, and where to go if you need help in your endeavors!
