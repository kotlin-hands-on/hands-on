
# The JVM Application

Let's switch to the code. We will open the [Ktor](https://ktor.io)
web server written in Kotlin/JVM, and port it to the Multiplatform
project. 

It is time to clone the
[github.com/kotlin-hands-on/intro-kotlin-mutliplatform](https://github.com/kotlin-hands-on/intro-kotlin-mutliplatform/tree/step-001)
repository and open the `step-001` branch. We could also just download the
[archive](https://github.com/kotlin-hands-on/intro-kotlin-mutliplatform/archive/step-001.zip)
from GitHub directly. 

Let's open the `build.gradle.kts` project file in IntelliJ IDEA. It is a Gradle project. 
Throughout this tutorial, we will be
using the Gradle build system with Kotlin Gradle DSL. We should note that
the Kotlin compiler itself, and the Gradle build system requires a Java 1.8 or 11
runtime. Check out the 
[https://jdk.java.net/11](https://jdk.java.net/11/) or another resource 
for the JRE, OpenJDK, or JDK distribution.

## The JVM Application Internals

We have several files in the project with the implementation of the
Mandelbrot set rendering. We select a 2D rectangular area (e.g. `[-2.0 .. 2.0] x [-2.0 .. 2.0]`)
of investigation as an input parameter. Every pixel of a resulting image is linearly mapped into
a point of that 2D area. A computation is performed for each pixel to find the color of it.
There are the following files under the
`src/main/kotlin/com/jetbrains/handson/introMpp` folder of the project:

* `color.kt` - formulas to compute color for an image pixel
* `complex.kt` - complex numbers implementation, used in Mandelbrot set formulas
* `geometry.kt` - mapping between image coordinates and 2D coordinates of the rendering area
* `graphics.kt` - code that generates a PNG image from an in-memory representation
* `render.kt` - the main loop that runs computations for each image pixel, contains Mandelbrot set formulas
* `main.kt` - Ktor HTTP web server, entry point

It is a good moment now to quickly look through the source code before we proceed with running the application. 

## Running the Application

Before we move on into the conversion to the Multiplatform project,
let's run the application. The easiest way to do this is to start the
`run` task of the Gradle project by running the console command
`./gradlew run` on macOS and Linux or `gradlew.bat run` on Windows.
Alternatively, we may start these tasks directly
[from an IDE](https://www.jetbrains.com/help/idea/work-with-gradle-tasks.html).
We can do this either via the _Gradle_
tool window or the console. Please note, the task will start the web server for
us, which will run endlessly. We can stop it manually when the
server is no longer needed.

We will be able to see a process output like this:

```
The Mandelbrot renderer is started at http://127.0.0.1:8888

Open the following in a browser
  http://127.0.0.1:8888/mandelbrot

For a zoomed image try
  http://127.0.0.1:8888/mandelbrot?top=-0.32&left=-0.72&bottom=-0.28&right=-0.68
  http://127.0.0.1:8888/mandelbrot?top=-0.7&left=0.1&bottom=-0.6&right=0.2

```

The `top`, `left`, `bottom` and `right` parameters define a rectangular area
of the 2D space where the [Mandelbrot Set](https://en.wikipedia.org/wiki/Mandelbrot_set)
is rendered. The parameters define the area as follows `[left .. right ] x [top .. bottom]`.

Let's open several URLs from the output to see the different Mandelbrot set
images. We can try changing the coordinates back and forth to see more images.

Let's implement an HTML page, and a client-side application now to better present
the rendered fractal images now!
