
# The JVM Application

Let's switch closer to the code. We will open a [Ktor](https://ktor.io)
web server application written in Kotlin/JVM and port it to the Multiplatform
project. 

It is time to clone the
[github.com/kotlin-hands-on/intro-mpp](https://github.com/kotlin-hands-on/intro-mpp)
repository and open `step-001` branch. We may also just download the
[archive](https://github.com/kotlin-hands-on/intro-mpp/archive/step-001.zip)
from GitHub directly. 

Let's open the project in IntelliJ IDEA by pointing it to the `build.gradle.kts`
project file. It is a Gradle project. Throughout the tutorial we will be
using Gradle build system with Kotlin Gradle DSL. We should note that,
Kotlin compiler itself and the Gradle build system require a Java 1.8 or 11
runtime on our computes. Check out the 
[https://jdk.java.net/11](https://jdk.java.net/11/) or another resource 
for the best JRE, OpenJDK, or JDK distribution.

## Running the Application

Before we move on into the conversion to the Multiplatform project,
let's run the application. The easiest way to perform is to start the
`run` task of the Gradle project. One may do that via either the _Gradle_
tool window or console.

We should be able to see the process output like that

```
The Manderbrot renderer is started at http://127.0.0.1:8888

Open the following in a browser
  http://127.0.0.1:8888/mandelbrot

For a zoomed image try
  http://127.0.0.1:8888/mandelbrot?top=-0.32&left=-0.72&bottom=-0.28&right=-0.68
  http://127.0.0.1:8888/mandelbrot?top=-0.7&left=0.1&bottom=-0.6&right=0.2

```

We may open several URLs from the output to see different Mandelbrot set
images. Try changing the parameter values for having few more curios
images in addition to the pre-defined parameters. 

 