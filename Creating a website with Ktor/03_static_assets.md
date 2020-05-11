# Static files and pages

Before we dive into making a _dynamic_ application, let's start by doing something a bit easier, but probably just as important – let's get Ktor to serve some *static* files.

In the context of our journal, there are a number of things that we probably want to serve as static files – one example being a header image (a logo that identifies our site). Luckily for us, the template repository already has a PNG file included which we can use: `ktor.png` inside the folder `src/main/resources/files`:

![](./assets/ktor_image_location.png)

For serving static content, we can use a specific [`routing`](https://ktor.io/servers/features/routing.html) function already built in to Ktor named `static`. The function takes two parameters: the route under which the static content should be made available, and a lambda where we can define the location from where the content should be served. In the file called `Application.kt`, let's change the implementation for `Application.module()` to look like this:

```kotlin
fun Application.module() {
    routing {
        static("/static") {
            resources("files")
        }
    }
}
```

This instructs Ktor that everything under the URL `/static` should be served using the `files` directory inside `resources`.

### Running our application for the first time

_Seeing is believing_ – so let's *see* if our application is performing as expected! We can run our application by pressing the green "play" button next to `fun main(...)` in our `Application.kt`. IntelliJ IDEA will start the application, and after a few seconds, we should see the confirmation that the app is running:

```
[main] INFO  Application - Responding at http://0.0.0.0:8080
```

Let's open [`http://localhost:8080/static/ktor.png`](http://localhost:8080/static/ktor.png) in a browser. We see that Ktor serves the static file:

![](./assets/ktor_logo_in_browser.png)

Of course, we are not limited to images – HTML files, or CSS and JavaScript would work just as well. We can take advantage of this fact to add a small "About me" page as a first real part of our journal application – a static page that can contain some information about us, this project, or whatever else we might fancy.

To do so, let's create a new file inside `src/main/resources/files/` called `aboutme.html`, and fill it with the following contents:

```xml
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Kotlin Journal</title>
</head>
<body style="text-align: center; font-family: sans-serif">
<img src="/static/ktor.png" alt="ktor logo">
<h1>About me</h1>
<p>Welcome to my static page!</p>
<p>Feel free to take a look around.</p>
<p>Or go to the <a href="/">main page</a>.</p>
</body>
</html>
```

If we re-run the application and navigate to [`http://localhost:8080/static/aboutme.html`](http://localhost:8080/static/aboutme.html), we can see our first page in all its glory. As you can see, we can even reference other static files – like `ktor.png` – inside this HTML.

![](./assets/aboutme.png)

Of course, we could also organize our files in subdirectories inside `files`; Ktor will automatically take care of mapping these paths to the correct URLs.

If you'd like to learn more about serving static files with Ktor, check out the [official documentation](https://ktor.io/servers/features/static-content.html).

However, a static page that contains a few paragraphs can hardly be called a journal yet. Let's move on and learn about how *templates* can help us in writing pages that contain dynamic content, and how to control them from within our application.
