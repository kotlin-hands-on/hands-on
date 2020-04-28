# Static Content

The first type of content we want to serve up on our website are existing static files. These could be HTML files, images, or any other type of file. 

For serving static content, we can use a specific routing function already built in to Ktor named `static`. The function takes two parameters: the 
URL which corresponds to static contents, and a lambda where we can define the `resources` bundle that serves this content:

```kotlin
fun Application.module() {
    routing {
        static("/static") {
            resources("files")
        }
    }
}
```

In our case, we're saying that everything under the URL `/static` should be served using the `files` folder under `resources`. Our project structure
would thus have the following layout:

![Project Structure](./assets/project-structure.png)

We fill our `index.html` file with the following contents:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Static Page</title>
</head>
<body>
    <h1>Static Page</h1>
<p>

</p>
<p>
    This is a static page!
</p>
<img src="ktor.png" alt="Ktor Logo">
</body>
</html>
```

Notice that we can now reference other static files within this folder inside the HTML (`ktor.png` image tag).

If we want to organize our files further, we can of course create a folder structure inside the `files` folder, which will be mapped to the correct URLs.

If you'd like to learn more about serving static files with Ktor, check out the [documentation](https://ktor.io/servers/features/static-content.html).

