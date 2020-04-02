# Static Content

The first type of content we want to serve up on our website are existing static files, be these
images or HTML. 

For serving static content we can use a specific routing function already built in to Ktor, named `static`. The function takes two parameters, the 
URL which corresponds to static contents, and a lambda where we can define the `resources` bundle that serves this content.
 
```kotlin
fun Application.module() {
    routing {
        static("/static") {
            resources("files")
        }
    }
}
```

In our case we're saying that everything under the URL `/static` should be served using the `files` folder under `resources`. Our project structure
would thus have the following layout

![Project Structure](./assets/project-structure.png)

Our `index.html` file should have the following contents

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

Notice how we can now reference other static files within this folder inside the HTML (`ktor.png` image tag). 

We can of course create as many sub-folders as we like and organize our files in the manner we most see appropriate. 

