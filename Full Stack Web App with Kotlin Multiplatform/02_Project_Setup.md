# Project Setup

If we were to start a fresh idea from zero, the wizard for multiplatform projects which is available in IntelliJ IDEA would be our go-to solution. In order to reduce the amount of time spent adjusting Gradle imports while going through this tutorial, a starter template is available for this specific tutorial that includes configuration and all required dependencies for Kotlin/JS, Kotlin/JVM, as well as common code.

[**Please clone the project repository from GitHub, and open it in IntelliJ IDEA.**](https://github.com/kotlin-hands-on/jvm-js-fullstack)

Of course, we still don't want to treat our Gradle configuration as a black box. Throughout the hands-on, we can have a look at parts of our Gradle build file and how it relates to what we are trying to achieve.

**You don't need to make any changes to the Gradle configuration throughout this whole hands-on – so if you want to get right to programming, feel free to skip over parts of this page, or move on directly to the next section.**

**The section below and other sections talking about Gradle structures are only supposed to help your understanding, and prepare you for any projects where you might need to write your own configurations.**

We start by having a coarse look at the structure of a Kotlin project that targets multiple platforms.

### Multiplatform Gradle Structure

Like all Kotlin projects targeting more than one platform, our project uses the Kotlin `multiplatform` Gradle plugin. It allows us to manage the targets we need for our application (Kotlin/JVM and Kotlin/JS) from within the same Gradle project, and exposes a number of tasks. For a more detailed look, check out the reference on [Building Multiplatform Projects with Gradle](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html). Besides `multiplatform`, we add two more plugins:

- The [`application`](https://docs.gradle.org/current/userguide/application_plugin.html) plugin, which takes care of running the JVM (server) part of our application.
- The [`serialization`](https://github.com/Kotlin/kotlinx.serialization#gradle) plugin, which ensures that multiplatform conversion between Kotlin objects and their text representation (JSON) is available.

```kotlin
plugins {
    kotlin("multiplatform") version "1.3.71"
    application //to run JVM part
    kotlin("plugin.serialization") version "1.3.70"
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
        // . . .
    }
}
```

For more detailed information on targets, check out the [respective section](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html#setting-up-targets) in the guide.

#### Source Sets

Kotlin source sets are a collection of Kotlin sources, along with their resources, dependencies, and language settings, that belong to one or more targets. They are used to set up our platform-specific and common dependency blocks.

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

Each source set also corresponds to a folder in the `src` directory. In our project, we see the corresponding three folders `commonMain`, `jsMain`, and `jvmMain`, which contain their own `resources` and `kotlin` folders.

For more detailed information on source sets, check out the [respective section](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html#configuring-source-sets) in the guide. There are a few other snippets that are yet to be explained – but we will discuss them at later chapters in this hands-on, when they become relevant to what we are trying to achieve.

For now, we will focus on building our application, starting with a strong backbone – a simple API server.