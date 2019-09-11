# Using an external REST API

Until now, we've only used rather boring mock data to display the videos and to render our application. Let's substitute this data by obtaining some actual information from a REST API.

For this use case, we have made a small API available at https://my-json-server.typicode.com/kotlin-hands-on/kotlinconf-json/videos/1. This API offers only a single endpoint, videos, and takes a numeric parameter to access an element from the list. Feel free to try out this rather limited API in the browser for a bit. You will see that the objects returned from the API follow the same structure as our `Video` objects (what a coincidence 😏). In the next section, we'll discuss how to efficiently obtain this data and shape it into the form of our Kotlin object.

### Using JS functionality from Kotlin

Even without pulling in external libraries, browsers already come with a lot of functionality. Included with Kotlin/JS are wrappers for exactly those APIs. They make it easy to access the available functionality in a comfortable and type-safe way straight from your Kotlin code. One example of these wrappers is the functionality for consuming making HTTP requests (arguably the most important part in consuming a RESTful service), in particular, the [fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).

The typical way to do asynchronous programming in JavaScript is through the use of callbacks. This means resolving promises step by step using functions stacked inside of functions... stacked inside of functions… the more complex our code gets, the more heavily indented will it be. Our code would slope to the right, making it harder to parse and to understand the control flow, for us and fellow developers. While our example would be rather tame...

```kotlin
window.fetch("https://url...").then {
    it.json().then {
        it.unsafeCast<Video>()
                //...
    }
}
```

**…we are not going to use this approach**. Instead, we're going to make use of Kotlin's coroutines, a much nicer and more structured way for us to achieve the same thing.

### Coroutines instead of callbacks

Coroutines and structured concurrency are a huge topic in Kotlin. If you want to get an in-depth understanding of how coroutines work, try our [coroutines hands-on lab](https://play.kotlinlang.org/hands-on/Introduction%20to%20Coroutines%20and%20Channels/). Let's get started by adding coroutines as a dependency to our project.

Like the other dependencies we've gotten to know over the course of this hands-on, we can also install coroutines via `yarn`:

```shell
yarn add kotlinx-coroutines-core
```

Restart the development server, and let's fetch our first video using coroutines!

#### Fetching our first video

Inside `App.kt` (or a new file), let's write a method that can `fetch` a video from the API.

```kotlin
suspend fun fetchVideo(id: Int): Video {
    val responsePromise = window.fetch("https://my-json-server.typicode.com/kotlin-hands-on/kotlinconf-json/videos/$id")
    val response = responsePromise.await()
    val jsonPromise = response.json()
    val json = jsonPromise.await()
    return json.unsafeCast<Video>()
}
```

Let's look at what's happening in this *suspending function*. We `fetch` a video from the API given an `id`, wait for it to actually be available, turn it into a JSON, wait again for the completion of that operation, and return it, cast as a `Video` Kotlin object.

A function call like `window.fetch` returns a `Promise` object. We would have to define a callback handler which gets invoked once the `Promise` is *resolved* and a result is available. However, since we are using coroutines in our project, we can `await` those promises. We're writing code that looks sequential but remains non-blocking. Whenever a function like `await()` is called, the method stops its execution (it *suspends*, hence the keyword). It continues execution once the `Promise` can be resolved.

The individual variables were only used for illustration purposes, though. In reality, we can, of course, chain all calls together. We'll end up with a single expression, so we can even use the expression body syntax to express the same processing steps as above:

```kotlin
suspend fun fetchVideo(id: Int): Video =
        window.fetch("https://my-json-server.typicode.com/kotlin-hands-on/kotlinconf-json/videos/$id")
                .await()
                .json()
                .await()
                .unsafeCast<Video>()
```

#### Fanning out

Since we're working with (multiple) lists of videos, we have a good reason to pull in, for example, 25 videos! So, let's define a function `fetchVideos` (note the s) which will fetch 25 videos from the same API as above. Since we want to run all the requests concurrently, we can use the `async` functionality provided by coroutines, leaving us with an implementation that looks like this:

```kotlin
suspend fun fetchVideos(): List<Video> = coroutineScope {
    (1..25).map { id ->
        async {
            fetchVideo(id)
        }
    }.awaitAll()
}
```

For reasons of [structured concurrency](https://kotlinlang.org/docs/reference/coroutines/basics.html#structured-concurrency), we have to wrap our implementation in a coroutineScope. We can then start 25 asynchronous tasks (one per request) and wait for them to complete.

If you want to dive deeper into coroutines, check our [hands-on on coroutines](https://play.kotlinlang.org/hands-on/Introduction%20to%20Coroutines%20and%20Channels/01_Introduction)!

Now that we have a way to obtain real data, it's time for us to plug it into our application. To do this, all we need to do is to expand the `init` function of our `App`:

```kotlin
override fun AppState.init() {
    unwatchedVideos = listOf()
    watchedVideos = listOf()

    val mainScope = MainScope()
    mainScope.launch {
        val videos = fetchVideos()
        setState {
            unwatchedVideos = videos
        }
    }
}
```

Note that even though we are in the `init` function, we are using `setState` to set the unwatchedVideos to the result of our coroutine. Precisely because we are non-blocking, the app has most likely already finished rendering an empty list of `unwatchedVideos`. We can give the React renderer a little nudge in the form of a `setState` invocation to refresh the rendered result. When we go back and check our browser window, we can see that just like that, we have actual data in our application!

![image-20190729201914738](/assets/image-20190729201914738.png)

This concludes the development part of this hands-on. We've come a long way, from an initial "Hello, World"-derivative to a full video organizer.

The full source code for the final application can be found here.

Stick around if you'd like to find out how we can bundle our application for production use, and how to get your app into the hands of real people by publishing it to the cloud!

You can find the state of the project after this section on the `step-07-using-external-rest-api` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js/tree/step-07-using-external-rest-api) repository.