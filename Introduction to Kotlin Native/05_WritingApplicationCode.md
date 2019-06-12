# Writing the Application Code

Now that we have our library and the correspoing Kotlin stubs, we can consume them from our application. 


Create a new file called `Main.kt` under the corresponding `src/platform` folder. In our case on macOS it would be `src/macosMain/kotlin/Main.kt` 

Write the following as part of the `main` function

```kotlin
import libcurl.*
import kotlinx.cinterop.*

fun main(args: Array<String>) {
    if (args.size == 1) {
        val curl = curl_easy_init()
        if (curl != null) {
            curl_easy_setopt(curl, CURLOPT_URL, args[0])
            curl_easy_setopt(curl, CURLOPT_FOLLOWLOCATION, 1L)
            val res = curl_easy_perform(curl)
            if (res != CURLE_OK) {
                println("curl_easy_perform() failed " + curl_easy_strerror(res)?.toKString())
            }
            curl_easy_cleanup(curl)
        }
    } else {
        println("Please provide a URL")
    }
}
```
