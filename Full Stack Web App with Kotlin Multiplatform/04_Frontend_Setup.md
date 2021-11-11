# Frontend setup

A basic version of our server is already done - now, it is time to make it human-usable. For this, we want to build a small Kotlin/JS web app that can query the API of our server, display them in the form of a list, and allow the user to add and remove elements.

### Prerequisites

#### Serving the frontend

Unless explicitly configured otherwise, a Kotlin Multiplatform project only means that we can build our application for each platform: in our case, the JVM, and JavaScript. However, for our application to function properly, we need to have both the backend and the frontend compiled. In fact, we want our backend to also serve all the assets belonging to the frontend – an HTML page and the corresponding `.js` file.

**In our template project, the adjustments to the Gradle file are already made:** Whenever we run our server with the `run` Gradle task, our frontend also gets built and is also included in the resulting artifacts. To learn more about how this works, refer to the "Relevant Gradle configuration" section. 

The template also already comes with a boilerplate `index.html` file to the `src/commonMain/resources` folder. It doesn't need to contain a lot: a `root` node for rendering our components (as discussed in the following chapter), and a `script` tag that includes our application. It looks like this:

```xml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Full Stack App!</title>
</head>
<body>
<div id="root"></div>
<script src="shoppinglist.js"></script>
</body>
</html>
```

It might not be immediately obvious why this file is placed in the `common` resources as opposed to a `jvm` source set. This is a little trick: If we ever need to run only the browser application without the backend, the tasks for running the JS application in the browser (`jsBrowserDevelopmentRun` and `jsBrowserProductionRun`) will have access to the file as well.

While we don't need to do anything to make sure the file is properly available on the server, we still need to instruct Ktor to provide the `.html` and `.js` files to a browser when requested.

#### Serving HTML and JavaScript files from ktor

For simplicity's sake, we will serve the `index.html` file on the root route `/`, and expose our JavaScript artifact in the root directory, as well.

In `src/jvmMain/kotlin/Server.kt`, add the corresponding routes to the `routing` block:

```kotlin
get("/") {
    call.respondText(
        this::class.java.classLoader.getResource("index.html")!!.readText(),
        ContentType.Text.Html
    )
}
static("/") {
    resources("")
}
```

To confirm that everything went as planned, run the application again via the Gradle `run` task, and navigate to `http://localhost:9090/`. You should see a page saying "Hello, Kotlin/JS", signaling that our script code is working properly – and that it is time for us to build some proper UI!

![image-20200407171403846](./assets/image-20200407171403846.png)

#### A word on optimization: `development` and `production` modes for Kotlin/JS

While we are developing, our build system generates *development* artifacts. This means that no optimizations are applied when our Kotlin code gets turned into JavaScript. That makes compile times faster, but also means we get larger JS files. When we deploy our application to the web, this is something we want to avoid.

To instruct Gradle to generate optimized production assets, we set the environment variable `ORG_GRADLE_PROJECT_isProduction=true`. If we are running our application on some kind of deployment system, we can configure it to set this environment variable during the build. If we want to try out production mode locally, we can do so via the terminal, or by adding the variable to the run configuration in IntelliJ IDEA, via the "Edit Configurations..." action:

![](./assets/edit_run_configuration.png)

In the "Run/Debug Configurations" menu, we can now set the environment variable:

![image-20200408153933129](./assets/image-20200408153933129.png)

Subsequent builds with this run configuration will perform all optimizations available for the frontend part of our application, including dead code elimination, but will be slower than development builds – so while we are developing, it would be a good idea to remove this flag again.

### Relevant Gradle configuration

The Gradle configuration for our application contains a snippet that binds the execution and packaging of our server-side JVM application to depend on the build of our frontend application, respecting the settings regarding `development` and `production` from the environment variable. It makes sure that whenever a `jar` file is built from our application, it includes our Kotlin/JS code:

```kotlin
// include JS artifacts in any JAR we generate
tasks.getByName<Jar>("jvmJar") {
    val taskName = if (project.hasProperty("isProduction")
        || project.gradle.startParameter.taskNames.contains("installDist")
    ) {
        "jsBrowserProductionWebpack"
    } else {
        "jsBrowserDevelopmentWebpack"
    }
    val webpackTask = tasks.getByName<KotlinWebpack>(taskName)
    dependsOn(webpackTask) // make sure JS gets compiled first
    from(File(webpackTask.destinationDirectory, webpackTask.outputFileName)) // bring output file along into the JAR
}
```

The `jvmJar` task, which we're modifying here, is called by the `application` plugin (responsible for the `run` task), but also the `distributions` plugin, which is responsible for the `installDist` task (amongst others). This means that the combined build will work when we `run` our application, and also when we prepare it for deployment to another target system or cloud platform.

To ensure that the `run` task properly recognizes the JS artifacts, its classpath is also adjusted:

```kotlin
tasks.getByName<JavaExec>("run") {
    classpath(tasks.getByName<Jar>("jvmJar")) // so that the JS artifacts generated by `jvmJar` can be found and served
}
```