# Project setup

If we were to start a fresh idea from zero, Ktor would have a few ways of setting up a preconfigured Gradle project: [start.ktor.io](https://start.ktor.io/) and the [Ktor IntelliJ IDEA plugin](https://plugins.jetbrains.com/plugin/10823-ktor) make it easy to create a starting-off point for projects using a variety of features from the framework.

For this tutorial, however, we have made a starter template available that includes all configuration and required dependencies for the project.

[**Please clone the project repository from GitHub, and open it in IntelliJ IDEA.**](https://github.com/kotlin-hands-on/creating-http-api-ktor/)

The template repository contains a basic Gradle projects for us to build our project. Because it already contains all dependencies that we will need throughout the hands-on, **you don't need to make any changes to the Gradle configuration.**

It is still beneficial to understand what artifacts are being used for the application, so let's have a closer look at our project template and the dependencies and configuration it relies on.

### Dependencies

For this hands-on, the `dependencies` block in our `build.gradle` file is probably the most interesting part:

```groovy
dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8"
    implementation "io.ktor:ktor-server-core:$ktor_version"
    implementation "io.ktor:ktor-server-netty:$ktor_version"
    implementation "ch.qos.logback:logback-classic:$logback_version"
    implementation "io.ktor:ktor-serialization:$ktor_version"
    implementation "org.jetbrains.kotlinx:kotlinx-serialization-runtime-common:$kotlin_serialization"

    testImplementation "io.ktor:ktor-server-tests:$ktor_version"
}
```
Let's briefly go through these dependencies one-by-one:

- `ktor-server-core` adds Ktor's core components to our project.
- `ktor-server-netty`  adds the [Netty](https://netty.io/) engine to our project, allowing us to use server functionality without having to rely on an external application container.
- `logback-classic` provides an implementation of [SLF4J](http://www.slf4j.org/), allowing us to see nicely formatted logs in our console.
- `ktor-serialization` provides a convenient mechanism for converting Kotlin object into a serialized form like JSON, and vice versa. We will use it to format our APIs output, and to consume user input that is structured in JSON. In order to use `ktor-serialization`, we also need the `kotlinx-serialization-runtime-common` dependency, and have to apply the `org.jetbrains.kotlin.plugin.serialization` plugin.
- `ktor-server-tests` allows us to test parts of our Ktor application without having to use the whole HTTP stack in the process. We will use this to define unit tests for our project.

### Configurations: `application.conf` and `logback.xml`

The repository also includes a basic `application.conf` in HOCON format, located in the `resources` folder. Ktor uses this file to determine the port on which it should run, and it also defines the entry point of our application. If you'd like to learn more about how a Ktor server is configured, check out the [official documentation](https://ktor.io/servers/configuration.html).

Also included in the same folder is a `logback.xml` file, which sets up the basic logging structure for our server. If you'd like to learn more about logging in Ktor, check out the [official documentation](https://ktor.io/servers/logging.html). 

### Entry point

Our `application.conf` configures the entry point of our application to be `com.jetbrains.handson.website.ApplicationKt.module`. This corresponds to the `Application.module()` function in `Application.kt`, which currently doesn't do anything:

```kotlin
fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

fun Application.module() {

}
```

The entry point of our application is important because we install Ktor's features and define routing for our API here – something we can start with right now! 

