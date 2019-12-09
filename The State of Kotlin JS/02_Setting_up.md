# Setting up for JavaScript (Gradle)

Let's create a project in which we can do our work. If you're using IntelliJ IDEA, setting up for using Kotlin/JS with Gradle is a straightforward task:

Through the _New Project_ wizard, we can select the platform we want to target. For this example, we're selecting _Kotlin/JS for browser_, so that we can see some of the browser specifics lateron. Rest assured however that the _Kotlin/JS for Node.js_ works almost equivalently.

To properly embrace Kotlin even in the build script, let's make sure we tick _Kotlin DSL build script_.

![image-20191203203529370](/assets/image-20191203203529370.png)

On the following page, pick an apt project name. I'll choose `handson` in this case, but you can of course pick whatever you'd like. Confirming our selection and waiting for the Gradle project to finish syncing brings us to our starting point – a barebones project that's ready to be compiled to JavaScript. Opening the `build.gradle.kts` gives us some insights to what is happening under the hood when we build and execute our application:

```kotlin
plugins {
    id("org.jetbrains.kotlin.js") version "1.3.61"
}

group = "org.example"
version = "1.0-SNAPSHOT"

repositories {
    mavenCentral()
}

dependencies {
    implementation(kotlin("stdlib-js"))
}

kotlin.target.browser { }
```

As we can see, the `kotlin.js` Gradle plugin is used to provide JavaScript support for our project.

This state-of-the-art plugin takes care of managing a development environment for us that uses all the latest and greatest things from the JavaScript ecosystem – under the hood, it equips us with a `yarn` and `webpack` installation, and exposes their functinality through a convenient Gradle DSL, as we will see later.

The `kotlin.target.browser` part at the bottom of the file is the entry point for target-specific configurations. Here, we'll later see how to configure tests, or add JavaScript-specific dependencies here.

For now, let's dive right in and start by writing our first program that runs in the browser!