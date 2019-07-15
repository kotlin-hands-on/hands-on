# Turning Multiplatform

The first step for transitioning to the Multiplatform project
is to switch from the Gradle plugins to the
[kotlin-multiplatform](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html) plugin.

We will make the following changes to the project:

* change the Gradle plugin to `kotlin("multiplatform")`
* fix the `run` task 
* move source files to the JVM source set

Let's implement these steps one-by-one.

## Change Kotlin Plugin

We fix the `build.gradle.kts` project file and replace the following
lines

```kotlin
plugins {
  kotlin("jvm") version "1.3.40"
}
```

with 

```kotlin
plugins {
  kotlin("multiplatform") version "1.3.40"
}

kotlin {
  jvm()
}

```

Now we use the [kotlin-multiplatform](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html)
Gradle plugin.
We need to click the  _Apply Dependencies_ action on the yellow stripe in
the `build.gradle.kts` file to get the IDE to apply our changes to the project.

The `kotlin { jvm() }` block instructs the Kotlin Multiplatform Gradle
plugin that we would like to have a Kotlin/JVM target

## Fixing the Dependencies Block 
Now it is time to fix the `dependencies{}` block. The easiest way to do that
is to add the following prefix to it in the `build.gradle.kts` file:

```kotlin
kotlin.sourceSets["jvmMain"].dependencies { /* ... */ }
```

A Multiplatform Kotlin project has many source sets that are compiled 
to different targets, including, for example JVM, JavaScript, iOS. 
By the `kotlin.sourceSets["jvmMain"]` expression we specify the
dependencies of the `jvm` source set's `main` target. For example, 
saying `kotlin.sourceSets["jvmTest"]` would configure the `test` source
set of the same target. Refer to the
[kotlin-multiplatform](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html) plugin
documentation for more details

## Fixing the `run` task

The source sets that are introduced by the 
[kotlin-multiplatform](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html) plugin
are not the same as we have in [Gradle Java](https://docs.gradle.org/current/userguide/java_plugin.html)
plugin or in `kotlin("jvm")` plugin. The easiest way to fix the `run` 
task is to replace it with the following code:   

```kotlin
val run by tasks.creating(JavaExec::class) {
  group = "application"
  main = "org.jonnyzzz.kotlin.fractals2.MainKt"
  kotlin {
    val main = targets["jvm"].compilations["main"]
    dependsOn(main.compileAllTaskName)
    classpath(
            { main.output.allOutputs.files },
            { configurations["jvmRuntimeClasspath"] }
    )
  }
  ///disable app icon on macOS
  systemProperty("java.awt.headless", "true")
}
```

We may need to refresh the Gradle project model now to make sure
the IDE sees all our project model changes. Let's click on the _refresh_
icon in the Gradle tab to accomplish this.

## Fixing the Sources Location

Source file locations are different in the 
[kotlin-multiplatform](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html) plugin,
it depends on the name of the target, that we pass in the `kotlin{ jvm() {..} }` block.
In our case, the sources should be in the folder `src/jvmMain/kotlin`.
One may use custom names, for example, `kotlin { jvm("customName") }` and
thus the source path would be `src/customNameMain/kotlin`.

Let's move the source files from the `src/main/kotlin` into the `src/jvmMain/kotlin`
folder now. 

## Testing the Application

By that moment we should have the JVM only Kotlin Multiplatform project. 
Before we proceed with Kotlin/JS and Kotlin/Common, let's relax
and make sure our application works. For that, we may call the
same Gradle `run` task as we did in the previous part of the Hands-On.

We should be still able to see the process output like that

```
The Mandelbrot renderer is started at http://127.0.0.1:8888

Open the following in a browser
  http://127.0.0.1:8888/mandelbrot

For a zoomed image try
  http://127.0.0.1:8888/mandelbrot?top=-0.32&left=-0.72&bottom=-0.28&right=-0.68
  http://127.0.0.1:8888/mandelbrot?top=-0.7&left=0.1&bottom=-0.6&right=0.2

```

Let's open several URLs from the output to see different Mandelbrot set
images. Try changing the parameter values to see few more curios and self-repeating
images. 

## Completed Code

We may use the `step-002` branch of the
[github.com/kotlin-hands-on/intro-mpp](https://github.com/kotlin-hands-on/intro-mpp)
repository as the solution of the tasks that we've done above. 
Alternatively, use the
[archive](https://github.com/kotlin-hands-on/intro-mpp/archive/step-002.zip)
from GitHub directly