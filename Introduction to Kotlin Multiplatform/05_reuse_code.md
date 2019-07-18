# Avoiding Code Duplication

Right now we have a code duplication in our project. We've just hard-coded
the URL of the JVM backend in the JS frontend. In the `src/jsMain/kotlin/main.kt`
we have the following lines

```kotlin
const val jvmBackend = "http://127.0.0.1:8888"
```

In the `src/jvmMain/kotlin/main.kt` we have:

```kotlin
fun main() {
  val host = "127.0.0.1"
  val port = 8888

  val server = embeddedServer(Netty, host = host, port = port) //...
```

## Using Kotlin/Common

The [kotlin-multiplatform](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html) plugin
supports the Kotlin/Common code and is a way to share code
between different targets. Let's use it now to avoid duplication
of the JVM server address. 

Now we need to create the common file for the project by the
`src/commonMain/kotlin/common.kt` path with the following contents

```kotlin
const val jvmHost = "127.0.0.1"
const val jvmPort = 8888
``` 

The declarations from the Kotlin files under
`src/commonMain/kotlin` are visible to all targets. We can only
use the platform-independent subset of the Kotlin standard library
from the Kotlin/Common source files. The Kotlin Multiplatform libraries are allowed too.
It allows us to easily share code between platforms.

Let's now update the code in both the `jsMain` and `jvmMain` source
sets to use the newly added `jvmHost` and `jvmPort` constants.

## Using Common Code

Let's use the new constants in the JS code from the `src/jsMain/kotlin/main.kt`

```kotlin
const val jvmBackend = "http://$jvmHost:$jvmPort"
```

In the `src/jvmMain/kotlin/main.kt` we should have

```kotlin
fun main() {
  val host = jvmHost
  val port = jvmPort

  val server = embeddedServer(Netty, host = host, port = port) //...
```

We can inline the `host` and `port` variables to simplify the code above.

## Running the App

Now it is time to start both the server-side JVM process,
by running the `run` task in Gradle, and the client-side
part by running the `jsRun` task (please note, it may
now work on Windows due to a known bug).

## Completed Code

We may use the `step-004` branch of the
[github.com/kotlin-hands-on/intro-mpp](https://github.com/kotlin-hands-on/intro-mpp)
repository as the solution of the tasks that we done above. 
We may also just download the
[archive](https://github.com/kotlin-hands-on/intro-mpp/archive/step-004.zip)
from GitHub directly.
  