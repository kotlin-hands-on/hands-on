# Project setup

For your own projects, you would usually start out using the Kotlin wizard included with IntelliJ IDEA. To keep things simple, we've made a starter template available for this specific tutorial. It already includes all the configuration and required dependencies for all parts of the project (which includes JVM, JS, and common code).

[**Please clone the project repository from GitHub, and open it in IntelliJ IDEA.**](https://github.com/kotlin-hands-on/jvm-js-fullstack)

It still makes sense to get an understanding of the configuration and project setup. Throughout the hands-on, we will have a look at parts of the Gradle build file, and how it relates to the current tasks at hand.

**You don't need to make any changes to the Gradle configuration throughout this whole hands-on – so if you want to get right to programming, feel free to skip over parts of this page, or move on directly to the next section.**

**The section below and other sections talking about Gradle structures are here to help your understanding, and prepare you for any projects where you might need to write your own configurations.**

Let's start with an overview of the structure for our multiplatform Kotlin project.

### Multiplatform Gradle Structure

Like all Kotlin projects targeting more than one platform, our project uses the Kotlin `multiplatform` Gradle plugin. It provides a single point to configure the targets we need for our application (in our case Kotlin/JVM and Kotlin/JS), and exposes a number of lifecycle tasks for them. For a more detailed look, check out the reference on [Building Multiplatform Projects with Gradle](https://kotlinlang.org/docs/mpp-intro.html). Additionally, we add two more plugins:

- The [`application`](https://docs.gradle.org/current/userguide/application_plugin.html) plugin, which takes care of running the server part of our application, which lives on the JVM.
- The [`serialization`](https://github.com/Kotlin/kotlinx.serialization#gradle) plugin, which ensures provides multiplatform conversions between Kotlin objects and their JSON text representation JSON is available.

```kotlin
plugins {
    kotlin("multiplatform") version "1.6.10"
    application //to run JVM part
    kotlin("plugin.serialization") version "1.6.10"
}
```

#### Targets

The target configuration inside the `kotlin` block is responsible for setting up the platforms we want to support with our project. We configure the two targets `jvm` (server) and `js` (client). If we want to make further adjustments to any of the target configurations, we do so here.

```kotlin
jvm {
    withJava()
}
js {
    browser {
        binaries.executable()
    }
}
```

For more detailed information on targets, check out the [respective section](https://kotlinlang.org/docs/reference/mpp-discover-project.html#targets) in the guide.

#### Source Sets

Kotlin source sets are a collection of Kotlin sources, along with their resources, dependencies, and language settings, that belong to one or more targets. You use them to set up our platform-specific and common dependency blocks.

```kotlin
sourceSets {
    val commonMain by getting {
        dependencies {
            //...
        }
    }

    val jvmMain by getting {
        dependencies {
            //...
        }
    }

    val jsMain by getting {
        dependencies {
            //...
        }
    }
}
```

Each source set also corresponds to a folder in the `src` directory. In our project, we see the three folders `commonMain`, `jsMain`, and `jvmMain`, which contain their own `resources` and `kotlin` folders.

For more detailed information on source sets, check out the [respective section](https://kotlinlang.org/docs/reference/mpp-discover-project.html#source-sets) in the guide. There are a few other snippets that are yet to be explained – but we will discuss them at later chapters in this hands-on, when they become relevant to what we are trying to achieve.

For now, we will focus on building our application, starting with a strong backbone – a simple API server.