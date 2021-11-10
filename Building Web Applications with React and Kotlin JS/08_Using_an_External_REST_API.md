# Using an external REST API

So far, we always used hard-coded demo data for the content of our app. Let's change that, and pull some real data from a REST API into the app, instead.

For this tutorial, we've created a small API available at https://my-json-server.typicode.com/kotlin-hands-on/kotlinconf-json/videos/1. It only offers one endpoint, `videos`, which takes a number to access an element from the list. If you visit the API with your browser, you will see that the objects returned from the API follow the same structure as our `Video` objects. That "coincidence" is going to make it pretty easy for us to integrate it with the code we've written so far. Let's get to it.

### Using JS functionality from Kotlin

Browsers come with a large variety of [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API). You can also use them from Kotlin/JS: it includes wrappers for these APIs out of the box. One relevant example is the [fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API), which is used for making HTTP requests - exactly what we need to integrate with our API.

Browser APIs like `fetch` come with a small challenge for developers: they use [callbacks](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function)
to perform non-blocking operations. When multiple callbacks are supposed to run one after the other, they need to be nested. Naturally, the code starts "sloping to the right", with more and more functionality stacked inside each other, and gets harder to read.

With Kotlin, we have a better approach that we can use to get the same thing done: Coroutines.

A second challenge comes again from the dynamically typed nature of JavaScript:
we don't get any guarantees about the type of data we'll get returned from our external API. To tackle this, we'll once again rely on a some outside help: kotlinx.serialization.

### Coroutines instead of callbacks

Coroutines and structured concurrency are one of the strengths of Kotlin, but also a big topic in themselves. In this tutorial, we'll only use a small part of the coroutines API, but if you want to get an in-depth understanding of how coroutines work, try our [coroutines hands-on lab](https://play.kotlinlang.org/hands-on/Introduction%20to%20Coroutines%20and%20Channels/).

Just like with the other dependencies in this tutorial, the `build.gradle.kts` file for the project already has everything set up:

```kotlin
dependencies {
    //. . .
    //Coroutines & serialization (chapter 8)
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.5.2")
}
```

### Making things serializable

When we call our external API, we get back a bunch of JSON-formatted text that still needs to be turned into a Kotlin object for us to work with.

[kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization) is a library that allows us to write exactly these types of conversions:
from JSON strings to Kotlin objects (and the other way around, though we won't use this functionality today).

Just like with the other dependencies in this tutorial, the `build.gradle.kts` file for the project already has everything set up:

```kotlin
plugins {
    // . . .
    kotlin("plugin.serialization") version "1.5.31"
}

dependencies {
    //. . .
    //Serialization (chapter 8)
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.3.0")
}
```

As preparation for fetching our first video, we need to tell the serialization library about the `Video` class.

In `Main.kt`, add the `@Serializable` annotation to its definition:

```kotlin
@Serializable
data class Video(
    val id: Int,
    val title: String,
    val speaker: String,
    val videoUrl: String
)
```

With this small prepartion out of the way, let's move on and fetch our first video!

#### Fetching our first video

Inside `App.kt` (or a new file), add the following function that can `fetch` a video from our API.

```kotlin
suspend fun fetchVideo(id: Int): Video {
    val response = window
        .fetch("https://my-json-server.typicode.com/kotlin-hands-on/kotlinconf-json/videos/$id")
        .await()
        .text()
        .await()
    return Json.decodeFromString(response)
}
```

Let's go through this *suspending function* step by step. We `fetch` a video from the API given an `id`. This reponse takes may take a while, so we `await` its result. Next, we read the body from the response via the `text()`, which once again uses a callback. We wait for it to complete as well.

Before we return the value of the function, we pass it to `Json.decodeFromString`, a function from kotlinx.coroutines. As its name suggests, it converts the JSON text we received from our request into a Kotlin object with the appropriate fields.

Usually, function calls that return a `Promise`, like like `window.fetch`, need to define a callback handler. That handler would then be called once the `Promise` is *resolved* and a result is available. In the above example, we didn't need to do that: with Kotlin's coroutines, we `await` those promises. We benefit from code that looks sequential, but remains non-blocking:
Whenever a function like `await()` is called, the method stops its execution (it *suspends*, hence the keyword). It continues execution once the `Promise` can be resolved.

#### Fanning out

Since we want to give the user of our app a selection of videos, let's get some more results from the API - 25 videos should do.

Let's define a function called `fetchVideos` that requests 25 videos from the API. Because there is no point in requesting the videos sequentially - waiting for the first video to be fetched before requesting the next - we can kick off all the requests concurrently.

We do this by repeatedly calling the [`async`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) function from Kotlin's coroutines. Add the following implementation to your `App.kt`:

```kotlin
suspend fun fetchVideos(): List<Video> = coroutineScope {
    (1..25).map { id ->
        async {
            fetchVideo(id)
        }
    }.awaitAll()
}
```

For reasons of [structured concurrency](https://kotlinlang.org/docs/reference/coroutines/basics.html#structured-concurrency), we have to wrap our implementation in a `coroutineScope`. We can then start 25 asynchronous tasks (one per request) and wait for all of them to complete.

If you want to dive deeper into coroutines, check our [hands-on on coroutines](https://play.kotlinlang.org/hands-on/Introduction%20to%20Coroutines%20and%20Channels/01_Introduction)!

We now have a way to obtain real data in our app.
Time to plug it in!

Add the definition for a `mainScope`, and change your `app` component to start with the following snippet.
Don't forget to replace our demo values with `emptyLists` as well:

```kotlin
val mainScope = MainScope()

val app = fc<Props> {
    var currentVideo: Video? by useState(null)
    var unwatchedVideos: List<Video> by useState(emptyList())
    var watchedVideos: List<Video> by useState(emptyList())

    useEffectOnce {
        mainScope.launch {
            unwatchedVideos = fetchVideos()
        }
    }

    // . . .
```

This code snippet introduces two new concepts: the `MainScope`, and the `useEffectOnce` hook.

The [`MainScope`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-main-scope.html) is once again a part of Kotlin's structured concurrency model,
and creates the scope for our asynchronous tasks to run in.

`useEffectOnce` is another one of React's magic *hooks* (specifically, a simplified version of the [useEffect](https://reactjs.org/docs/hooks-effect.html) hook). It indicates that our component performs a *side effect*. It doesn't just render itself, but also communicates over the network.

This means when you load the page:

- The code of our `app` component will be invoked. This kicks off the code in the `useEffectOnce` block.
- The `app` component is rendered with empty lists for the watched and unwatched videos
- When our API requests finish, the `useEffectOnce` block assigns it to the state of the `app` component. This triggers a re-render.
- The code of the `app` component will be invoked again, but the `useEffectOnce` block *will not* run for a second time.

Having applied these changes, go back to the browser window.
Just like that, you'll see real data in the app!

![image-20190729201914738](./assets/image-20190729201914738.png)

With that, we're done with developing our little demo application.
We've come a long way: we started with a little "Hello, World"-type program, and ended with a full video organizer.

We can still do a little more, though.
If you want to learn how to bundle the application for deployment,
and publishing the app in *the cloud*, check out the next section.

You can find the state of the project after this section on the `07-using-external-rest-api` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js-gradle/tree/07-using-external-rest-api) repository.
