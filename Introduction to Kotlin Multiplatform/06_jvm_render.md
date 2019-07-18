# Common Rendering and JVM

It is always good to question ourselves – do we really need the
JVM backend for the project? What if we could
run the [Mandelbrot Set](https://en.wikipedia.org/wiki/Mandelbrot_set)
rendering on the client? 

Let's implement the client-side rendering and see how what happens.

## Sharing Code

We've just added our first Kotlin file into the `commonMain`
source set. Let's move all the files from the `jvm` source set
into the `common` source set so we share the code with JavaScript.

Wait just a second. It won't work this way, because we are using JVM APIs
to render images in our code. We need to pick another API to use in
the browser Kotlin/JS part.
We will instead use the
[HTML Canvas](https://www.w3schools.com/html/html5_canvas.asp)
to render our images in JavaScript.

It is easy to share code between platforms with Kotlin. 
[Kotlin Multiplatform](https://kotlinlang.org/docs/reference/multiplatform.html)
introduces the `expect` and `actual` keywords especially for this. 
We will use the `expect` keyword in the `commonMain` sources to specify
the declarations that we can only implement for a given target.
The `actual` keyword binds the `expect`-ed declarations with their
platform specific counterparts.

Let's have a look at how it works.

## Gradle Dependency

Before we proceed with moving the code, we should include another Gradle
dependency for the `commonMain` source set. Let's add a few more lines
to the `build.gradle.kts` for this:

```kotlin
kotlin.sourceSets["commonMain"].dependencies {
  implementation(kotlin("stdlib-common"))
  implementation("org.jetbrains.kotlinx:kotlinx-html-common:0.6.12")
}
```

That code adds the dependency from the Kotlin standard library. In addition
to that we include the dependency from the
[kotlinx.html](https://github.com/Kotlin/kotlinx.html)
common library. It adds ways to re-use the HTML, rendering
code between our JVM server-side and JS client-side.

We should refresh the Gradle project model now to make sure
the IDE sees all our project model changes. Let's click on the _refresh_
icon in the Gradle tab to accomplish this.

## Moving Code

Sharing code can be done with the following algorithm

* move all the source files into the `src/commonMain/kotlin` folder
* replace the missing platform-specific symbols with `expect` declarations
* add the `actual` keyword for the `expect` declarations for all platforms

Now it is time to run the algorithm on our project. We move
the following files from the `src/jvmMain/kotlin` into `src/commonMain/kotlin`
preserving the package folders `com/jetbrains/handson/introMpp`:

* `color.kt`
* `complex.kt`
* `geometry.kt`
* `render.kt`

We do not need to move the `main.kt` and `graphics.kt` files because these files are
platform specific. The first one starts the HTTP server, which we do not need
on the client-side. The `graphics.kt` file uses AWT to render the image.
     
There will be several errors in the `color.kt` file. We are missing the
`java.awt.Color` class definition. Let's fix it by adding the `expect`
declarations for colors in `src/commonMain/kotlin/com/jetbrains/handson/introMpp/common.kt`:

```kotlin
object Colors

expect class Color
expect fun Colors.newColor(r: Int, g: Int, b: Int): Color
expect val Colors.BLACK : Color
```

The `expect class`, `expect fun`, and `expect val` declarations in the Kotlin/Common
code, instructs the compiler that we'd like to use these declarations in our Kotlin/Common
code, but we are not able to provide implementations. Implementations are provided via the
`actual` keyword independently for the JVM and JS targets (in general,
[Kotlin/Native](https://kotlinlang.org/docs/reference/native-overview.html) can
also be [used](https://kotlinlang.org/docs/tutorials/native/mpp-ios-android.html)
to target iOS and other platforms).

Now that we've fixed the code in the `color.kt` file to use our expected `Color` class
and the `Colors` object to deal with colors (instead of using the `java.awt.Color` class).
We need to remove the import in the `color.kt` and use the `Colors.newColor`
instead of the `Color` constructor call.

The last fix has to be performed in the `render.kt` file. We need to remove the
import of the `java.awt.Color` class and switch to the `Colors.BLACK`
property in the middle of the file.

## JVM Implementation

We've just fixed the code in the `commonMain` source set by using the `expect` declarations.
It means that every target (e.g. JVM or JS) has to provide the `actual` declarations
for every `expect` declarations in the `commonMain`. Let's fix the `jvmMain`
source set first.

We will need to re-create the `color-actual.kt` file under the
`src/jvmMain/kotlin/com/jetbrains/handson/introMpp` folder
with the following `actual` declarations:

```kotlin
actual typealias Color = java.awt.Color

actual fun Colors.newColor(r: Int, g: Int, b: Int) = Color(r, g, b)
actual val Colors.BLACK: Color get() = Color.BLACK
```
 
By these declarations we say that our expected class `Color`
should be `java.awt.Color` class from the JVM. We also provide implementations for
two more required functions.

We should be able to start the JVM application now. Let's check
it by running the Gradle `run` task by running the console command
`./gradlew run` on macOS and Linux or `gradlew.bat run` on Windows.
Alternatively, we may start these tasks directly
[from an IDE](https://www.jetbrains.com/help/idea/work-with-gradle-tasks.html).
We should see similar output in the console:

```
The Mandelbrot renderer is started at http://127.0.0.1:8888

Open the following in a browser
  http://127.0.0.1:8888/mandelbrot

For a zoomed image try
  http://127.0.0.1:8888/mandelbrot?top=-0.32&left=-0.72&bottom=-0.28&right=-0.68
  http://127.0.0.1:8888/mandelbrot?top=-0.7&left=0.1&bottom=-0.6&right=0.2

```

## Completed Code

We can use the `step-005` branch of the
[github.com/kotlin-hands-on/intro-mpp](https://github.com/kotlin-hands-on/intro-mpp)
repository as the solution for the tasks that we have done above. 
We can also download the
[archive](https://github.com/kotlin-hands-on/intro-mpp/archive/step-005.zip)
from GitHub directly.
  