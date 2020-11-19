# Building SDK

Our iOS and Android applications will communicate with the SpaceX API through
our KMM Module, which will provide a public `SpaceXSDK` class. Let's create this class in the `com.jetbrains.handson.kmm.shared` package of the common source set. It will be the facade over `Database` and `SpaceXApi` classes:

```kotlin
package com.jetbrains.handson.kmm.shared

import com.jetbrains.handson.kmm.shared.cache.Database
import com.jetbrains.handson.kmm.shared.cache.DatabaseDriverFactory
import com.jetbrains.handson.kmm.shared.network.SpaceXApi
import com.jetbrains.handson.kmm.shared.entity.RocketLaunch

class SpaceXSDK(databaseDriverFactory: DatabaseDriverFactory) {
    private val database = Database(databaseDriverFactory)
    private val api = SpaceXApi()
}
```

To create a `Database` class instance, we need to provide the `DatabaseDriverFactory` platform instance to it, so we will inject it from the platform code through the `SpaceXSDK` class constructor.

Our class will contain one function for getting all launch information. The logic is simple: depending on the value of `forceReload` it will return cached values or load data from the internet and then update the cache with the results. If there is no cached data, it will load information from the internet independently of `forceReload`flag value:

```kotlin
@Throws(Exception::class) suspend fun getLaunches(forceReload: Boolean): List<RocketLaunch> {
        val cachedLaunches = database.getAllLaunches()
        return if (cachedLaunches.isNotEmpty() && !forceReload) {
            cachedLaunches
        } else {
            api.getAllLaunches().also {
                database.clearDatabase()
                database.createLaunches(it)
            }
        }
    }
```

Clients of our SDK could use a `forceReload` flag to load the latest information about the launches, which would allow the user to use the pull-to-refresh gesture.

To handle exceptions produced by the Ktor client, in Swift, we need to mark our function with `@Throws` annotation. All Kotlin exceptions are unchecked, while Swift has only checked errors. Thus, to make Swift code aware of expected exceptions, Kotlin functions should be marked with an `@Throws` annotation specifying a list of potential exception classes.

You can find the state of the project after this section on the [final branch](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/tree/final) in the GitHub repository.
