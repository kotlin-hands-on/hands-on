# Home page with templates

It's time to build the main page of our journal which is in charge of displaying multiple journal entries. We will create this page with the help of a *template engine*. Template engines are quite common in web development, and Ktor supports a [variety of them](https://ktor.io/servers/features/templates.html). In our case we're going to choose [FreeMarker](https://freemarker.apache.org/).

### Adding FreeMarker as a Ktor feature

[Features](https://ktor.io/servers/features.html) are a mechanism that Ktor provides to enable support for certain functionality, such as encoding, compression, logging, authentication, among others. While the implementation details of Ktor features (acting as interceptors / middleware providing extra functionality) aren't relevant for this hands-on tutorial, we will use this mechanism to `install` the `FreeMarker` feature, by adding the following lines to the top of our `Application.module()` definition in the `Application.kt` file:

```kotlin
fun Application.module() {
    install(FreeMarker) {
        templateLoader = ClassTemplateLoader(this::class.java.classLoader, "templates")
        outputFormat = HTMLOutputFormat.INSTANCE
    }
    routing {
        // Our existing code . . .
    }
}
```
In the configuration block for the `FreeMarker` feature, we're passing two parameters:

- The `templateLoader` setting tells our application that FreeMarker templates will be located in the `templates` directory inside our application `resources`: ![Template](./assets/templates_location.png)
- The `outputFormat` setting helps convert control characters provided by the user to their corresponding HTML entities. This ensures that when one of our journal entries contains a String like `<b>Hello</b>`, it is actually printed as `<b>Hello</b>`, not **Hello**. This so-called [escaping](https://freemarker.apache.org/docs/dgui_misc_autoescaping.html) is an essential step in preventing [XSS attacks](https://owasp.org/www-community/attacks/xss/).

Now that Ktor knows where to find our FreeMarker templates, we can start writing the template for the journal main page.

### Writing the journal template

Our journal main page should contain everything we need for a basic overview of our journal: a title, header image, a list of journal entries, and a form for adding new journal entries. To define this page layout, we use the FreeMarker Template Language (`ftl`), which contains our HTML source as well as FreeMarker variable definitions and instructions on how to use the variables in the context of the page.

Let's create a file called `index.ftl` in the `resources/templates` directory and fill it with the following content:

```xml
<#-- @ftlvariable name="entries" type="kotlin.collections.List<com.jetbrains.handson.website.BlogEntry>" -->
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Kotlin Journal</title>
</head>
<body style="text-align: center; font-family: sans-serif">
<img src="/static/ktor.png">
<h1>Kotlin Ktor Journal </h1>
<p><i>Powered by Ktor, kotlinx.html & Freemarker!</i></p>
<hr>
<#list entries as item>
    <div>
        <h3>${item.headline}</h3>
        <p>${item.body}</p>
    </div>
</#list>
<hr>
<div>
    <h3>Add a new journal entry!</h3>
    <form action="/submit" method="post">
        <input type="text" name="headline">
        <br>
        <textarea name="body"></textarea>
        <br>
        <input type="submit">
    </form>
</div>
</body>
</html>
```

As you can see, we are using FTL syntax to define, access and iterate over variables. The variable `entries` which we loop over has the type `kotlin.collections.List<com.jetbrains.handson.website.BlogEntry>`. This is simply the fully-qualified name of a Kotlin `List<BlogEntry>`. However, we haven't created the `BlogEntry` class yet, but now that we are referencing it, it seems like a good time to fix that!

If you'd like to learn more about FreeMarker's syntax, check out [their official documentation](https://freemarker.apache.org/docs/dgui_quickstart.html).

### Defining a model for the journal entries

As defined by our usage in the FreeMarker template, the `BlogEntry` class needs to attributes: a `headline` and a `body`, both of type `String`. Let's create a file named `BlogEntry.kt` next to `Application.kt`, and fill it with the corresponding Kotlin data class, which can then be injected into the template:

```kotlin
package com.jetbrains.handson.website

data class BlogEntry(val headline: String, val body: String)
```

It would also make sense if we defined a temporary storage for our entries. We can do this in the same file as a top-level declaration:

```kotlin
val blogEntries = mutableListOf(BlogEntry(
    "The drive to develop!",
    "...it's what keeps me going."
))
```

At this point, we have defined a template and the model that is will be used for rendering it. Now, Ktor just needs to pass our stored journal entries and serve the resulting page.

### Serving the templated content

The overview page we just templated is the center point of our application. So, it would make sense to make it available under the `/` route. Let's add a route for it to the `routing` block inside our `Application.module()`:


```kotlin
get("/") {
    call.respond(FreeMarkerContent("index.ftl", mapOf("entries" to blogEntries), ""))
}
```

We can now run the application. Opening [`http://localhost:8080/`](http://localhost:8080/) in a browser, we should see our header image, headline and subtitle, and a list of journal entries (well, just one for now) alongside a form for submitting new entries:

![FreeMarker Browser Output](./assets/main_page.png)

Looks like our display logic is working just fine! Now, we only need to make the "Submit" button work, and we'll be able to view and add new entries for our journal!