# Implementing an API service

We will use the [SpaceX public API](https://docs.spacexdata.com/?version=latest) to retrieve data via the internet. We need a single method to retrieve the list of all launches from the `v3/launches` endpoint.

We need to create a class that will connect the application to the API. To do this, we’ll create the `com.jetbrains.handson.kmm.shared.network` package, and in it we'll create the `SpaceXApi` class. This class will execute network requests and deserialize JSON responses into entities from the `entity` package. To do this, we will initialize and store the `httpClient` property with the Ktor `HttpClient` instance:

```kotlin
package com.jetbrains.handson.kmm.shared.network

import com.jetbrains.handson.kmm.shared.entity.RocketLaunch
import io.ktor.client.HttpClient
import io.ktor.client.features.json.JsonFeature
import io.ktor.client.features.json.serializer.KotlinxSerializer
import io.ktor.client.request.*
import kotlinx.serialization.json.Json

class SpaceXApi {
    private val httpClient = HttpClient {
        install(JsonFeature) {
            val json = Json { 
                ignoreUnknownKeys = true
                useAlternativeNames = false
            }
            serializer = KotlinxSerializer(json)
        }
    }
}
```

To deserialize the result of the `GET` request, we install the JSON [Ktor client feature](https://ktor.io/clients/http-client/features.html). It processes the request and the response payload as JSON, serializing and de-serializing them using a specific serializer. To use `kotlinx.serialization` we provide a `KotlinxSerializer` instance to the feature builder.

To send requests we need to define the URL as a constant within the companion object class:

```kotlin
companion object {
    private const val LAUNCHES_ENDPOINT = "https://api.spacexdata.com/v3/launches"
}
```

Next we declare our data retrieval method that will return the list of `RocketLaunch`:

```kotlin
suspend fun getAllLaunches(): List<RocketLaunch> {
    return httpClient.get(LAUNCHES_ENDPOINT)
}
```

The `getAllLaunches` function has the `suspend` modifier because it contains a call of the suspend function `get()`, which contains an asynchronous operation to retrieve data over the internet and can only be called from within a coroutine or another suspend function. The network request will be executed in the HTTP client's own thread pool. 

## Adding Android internet access permission

In order for our Android application to access the internet, we need to declare the appropriate permission in the application manifest. Since all network requests are made from the KMM module, it makes sense to add internet access permission to this module's manifest. Our `shared/src/androidMain/AndroidManifest.xml` file should contain the following:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.jetbrains.handson.kmm.app.kmmlib" >
    <uses-permission android:name="android.permission.INTERNET"/>
</manifest>
```

You can find the state of the project after this section on the [final branch](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/tree/final) in the GitHub repository.
