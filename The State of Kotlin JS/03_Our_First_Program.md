# Writing and running our first program

Since in this example, we chose to target the browser, we need a page that can be loaded which has the JavaScript embedded. Create and fill an HTML file at this location: `/src/main/resources/index.html`

```xml
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello, Kotlin/JS!</title>
</head>
<body>

</body>
<script src="handson.js"></script>
</html>
```

Depending on how we named our project, the embedded `js` file has a different name. For example, if you named your project `followingAlong`, make sure to embed `followingAlong.js`. Through some magic happening in the background, your project will be bundled up into this single JavaScript artifact that bears the same name as your project.

Now, all we need to do is actually write some executable code. As so many other programming stories, we will start out by writing a – somewhat JavaScript specific – _Hello, Kotlin/JS!_ application. The code for this is deceptively simple. Create the file `src/main/kotlin/App.kt` and fill it with these contents:

```kotlin
fun main() {
    console.log("Hello, Kotlin/JS!")
}
```

To run this first web app, we execute the `run` Gradle task. From the command line, it can be invoked via the Gradle wrapper:

```./gradlew run```

From IntelliJ IDEA, we can find the `run` action in the Gradle tool window:

![](/assets/run_gradle_task_from_ide.png)

On first start, the `kotlin.js` Gradle plugin will download all required dependencies to get us up and running. After a few seconds, the embedded `webpack-dev-server` will spring to life, and we should be greeted with a very empty browser window!

![image-20191203204605636](/assets/image-20191203204605636.png)

But this is expected. As they say, still waters run deep. Let's right click the page and choose the _Inspect_ action. Inside the DevTools, we can navigate to the console, where will be met with the results of our executed JavaScript code.

![image-20191203204730722](/assets/image-20191203204730722.png)

Before we continue adding some content to this page, let's enable automatic reloads whenever we make modifications to our source code.

### Enabling Live Reload (Continuous Compilation)

Instead of manually compiling and executing our project every time we want to see the changes we made, we can make use of the _continuous compilation_ mode that is supported by Kotlin/JS. Instead of using the regular `run` command, we instead invoke the Gradle wrapper in _continuous_ mode:

```./gradlew run --continuous```

If you are working from inside IntelliJ IDEA, we can pass the same flag via the _run configuration_. After running the Gradle `run` task for the first time from the IDE, IntelliJ IDEA automatically generates a run configuration for it, which we can edit:

![](/assets/edit_configurations.png)

In the "Run/Debug Configurations" dialog, we can add the `--continuous` flag to the arguments for the run configuration:

![](/assets/run_debug_configurations.png)

But of course, we can do better than a blank page. Let's actually set up a small page, and start manipulating it with the help of Kotlin/JS.