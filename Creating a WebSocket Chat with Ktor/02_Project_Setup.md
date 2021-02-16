# Project setup

Because our application will be two independent parts (chat server and chat client) we structure our application as two separate Gradle projects. Since these two projects are completely independent, they could be created manually, via the online [Ktor Project Generator](https://start.ktor.io/#), or the [plugin for IntelliJ IDEA](https://plugins.jetbrains.com/plugin/16008-ktor).

To skip over these configuration steps, a starter template is available for this specific tutorial, which includes all configuration and required dependencies for our two projects already.

[**Please clone the repository from GitHub, and open it in IntelliJ IDEA.**](https://github.com/kotlin-hands-on/chat-app-websockets)

The template repository contains two barebones Gradle projects for us to build our project: the `client` and `server` projects. Both of them are already preconfigured with the dependencies that we will need throughout the hands-on, so you **don't need to make any changes to the Gradle configuration.**

It might still be beneficial to understand what artifacts are being used for the application, so let's have a closer look at the two projects and the dependencies they rely on.

### Understanding the project configuration

Our two projects both come with their individual sets of configuration files. Let's examine each one of them a bit closer.

#### Dependencies for the `server` project

The server application specifies three dependencies in its `server/build.gradle.kts` file:

```kotlin
dependencies {
    implementation("io.ktor:ktor-server-netty:$ktor_version")
    implementation("io.ktor:ktor-websockets:$ktor_version")
    implementation("ch.qos.logback:logback-classic:$logback_version")
}
```

- `ktor-server-netty` adds Ktor together with the Netty engine, allowing us to use server functionality without having to rely on an external application container.
- `ktor-websockets` allows us to use the [WebSocket Ktor feature](https://ktor.io/docs/servers-features-websockets.html), the main communication mechanism for our chat.
- `logback-classic` provides an implementation of [SLF4J](http://www.slf4j.org/), allowing us to see nicely formatted logs in our console.

#### Configuration for the `server` project

Ktor uses a [HOCON](https://github.com/lightbend/config/blob/master/HOCON.md) configuration file to set up its basic behavior, like its entry point and deployment port. It can be found at `server/src/main/resources/application.conf`:

```kotlin
ktor {
    deployment {
        port = 8080
    }
    application {
        modules = [ com.jetbrains.handson.chat.ApplicationKt.module ]
    }
}
```

Alongside this file is also a barebones `logback.xml` file, which sets up the `logback-classic` implementation.

#### Dependencies for the `client` project

The client application specifies two dependencies in its `client/build.gradle.kts` file:

```kotlin
dependencies {
    implementation("io.ktor:ktor-client-websockets:$ktor_version")
    implementation("io.ktor:ktor-client-cio:$ktor_version")
}
```

- `ktor-client-cio` provides a [client implementation of Ktor](https://ktor.io/clients/http-client/engines.html#cio) on top of coroutines ("Coroutine-based I/O").
- `ktor-client-websockets` is the counterpart to the `ktor-websockets` dependency on the server, and allows us to consume [WebSockets from the client](https://ktor.io/docs/clients-websockets.html) with the same API as the server.

Now that we have some understanding of the parts that will make our project run, it's time to start building our project! Let's start by implementing a simple WebSocket echo server!