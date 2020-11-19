# Creating an application data model

Our KMM library will contain the public `SpaceXSDK` class, which will be the facade over networking and cache services. First, let's create an application data model with 3 entity classes: a class with general information about a launch, a class with a URL link to external information, and a class with information about the rocket. To do this, we need to add the `com.jetbrains.handson.kmm.shared.entity` package to the `shared/src/commonMain/kotlin` directory and create `Entity.kt` inside it. Here we'll declare all the data classes for our basic entities:

```kotlin
package com.jetbrains.handson.kmm.shared.entity

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class RocketLaunch(
    @SerialName("flight_number")
    val flightNumber: Int,
    @SerialName("mission_name")
    val missionName: String,
    @SerialName("launch_year")
    val launchYear: Int,
    @SerialName("launch_date_utc")
    val launchDateUTC: String,
    @SerialName("rocket")
    val rocket: Rocket,
    @SerialName("details")
    val details: String?,
    @SerialName("launch_success")
    val launchSuccess: Boolean?,
    @SerialName("links")
    val links: Links
)

@Serializable
data class Rocket(
    @SerialName("rocket_id")
    val id: String,
    @SerialName("rocket_name")
    val name: String,
    @SerialName("rocket_type")
    val type: String
)

@Serializable
data class Links(
    @SerialName("mission_patch")
    val missionPatchUrl: String?,
    @SerialName("article_link")
    val articleUrl: String?
)
```

Each serializable class must be marked with the `@Serializable` annotation. The `kotlinx.serialization` plugin automatically generates a default serializer for `@Serializable` classes unless we explicitly pass a link to a serializer through the annotation argument, but in our case we don't need to do that. The `@SerialName` annotation allows us to redefine field names, which lets us declare properties in data classes with names that are more easily readable.

You can find the state of the project after this section on the [final branch](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/tree/final) in the GitHub repository.
