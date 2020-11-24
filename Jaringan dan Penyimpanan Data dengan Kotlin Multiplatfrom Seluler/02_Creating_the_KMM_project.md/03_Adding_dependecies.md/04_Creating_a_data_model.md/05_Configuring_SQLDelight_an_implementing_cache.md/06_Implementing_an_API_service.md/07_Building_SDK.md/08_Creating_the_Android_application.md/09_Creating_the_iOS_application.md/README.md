# Creating the KMM project

The KMM plugin for Android Studio includes a "New Kotlin Multiplatform Mobile Project" wizard. With this wizard, we will create the structure for our new project in just a few clicks.

<img alt="KMM Plugin wizard" src="./assets/kmm-wizard.png" width="500">

In Android Studio, we first need to select **File** | **New** | **New Project** and select KMM Application in the list of project templates. Next, we need to specify a name for our new project. On the last step, we can keep the default names for the applications and shared module. Click finish and you’re all done!

![KMM Plugin wizard finish](./assets/kmm-wizard-finish.png)

To view the complete structure of our mobile multiplatform project, let's switch the view from Android to Project. 

<img alt="Project view" src="./assets/project-view.png" width="200">

You can explore the features [included in your project](https://kotlinlang.org/docs/mobile/discover-kmm-project.html) and how to use them, or you can go directly to the next step and start coding!

You can find configured KMM project on the [master branch](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage) with the KMM project created and configured. Now we are ready to start developing our multiplatform application.

  # Adding dependencies to the multiplatform library

  We will use the following multiplatform libraries in our project:

  * `kotlinx.coroutines` - we will use coroutines for asynchronous code.
  * `Ktor` - this will be our HTTP client for retrieving data over the internet.
  * `kotlinx.serialization` - to deserialize JSON responses into objects of entity classes.
  * `SQLDelight` - this will generate Kotlin code from SQL queries to create a type-safe database API.

  To add a multiplatform library to the KMM module, we need to add dependency instructions (in our case, implementation) to the dependencies block of the relevant source sets in the 'build.gradle.kts' file in the ** KMM module's directory *  *.  You can read more about adding dependencies in the [documentation] (https://kotlinlang.org/docs/mobile/add-dependencies.html#multiplatform-libraries).

  ### Adding kotlinx.coroutines

  We need to specify a dependency on `kotlinx.coroutines` in the common source set in order to add them to our project.  …

# Creating an application data model

  Our KMM library will contain the public `SpaceXSDK` class, which will be the facade over networking and cache services.  First, let's create an application data model with 3 entity classes: a class with general information about a launch, a class with a URL link to external information, and a class with information about the rocket.  To do this, we need to add the `com.jetbrains.handson.kmm.shared.entity` package to the` shared / src / commonMain / kotlin` directory and create `Entity.kt` inside it.  Here we'll declare all the data classes for our basic entities:

  `` kotlin
  package com.jetbrains.handson.kmm.shared.entity

  import kotlinx.serialization.SerialName
  import kotlinx.serialization.Serializable

  @Serializable
  data class RocketLaunch (
      @SerialName ("flight_number")
      val flightNumber: Int,
      @SerialName ("mission_name")
      val missionName: String,
      @SerialName ("launch_year")
      val launchYear: Int,
      @SerialName ("launch_date_utc")
      val ...
 # Configuring SQLDelight and implementing cache logic

  ## Configuring SQLDelight

  The SQLDelight library allows us to generate a type-safe Kotlin database API from SQL queries.  During compilation, the generator validates the SQL queries and turns them into Kotlin code that can be used in the KMM module.

  We already added the library to the project in the previous stage.  Now we need to configure it.  To do this, we need to add the sqldelight block, which will contain a list of databases and their parameters, to the end of the KMM module `build.gradle` file:

  `` kotlin
  sqldelight {
      database ("AppDatabase") {
          packageName = "com.jetbrains.handson.kmm.shared.cache"
      }
  }
  ``

  The packageName parameter specifies the package name for the generated Kotlin sources.

  There is an official [plugin] (https://cashapp.github.io/sqldelight/multiplatform_sqlite/intellij_plugin/) for working with **. Sq ** files.

  ## Generating the database API

  First, we need to create the **. Sq ** file, which…

# Implementing an API service

  We will use the [SpaceX public API] (https://docs.spacexdata.com/?version=latest) to retrieve data via the internet.  We need a single method to retrieve the list of all launches from the `v3 / launches` endpoint.

  We need to create a class that will connect the application to the API.  To do this, we'll create the `com.jetbrains.handson.kmm.shared.network` package, and in it we'll create the` SpaceXApi` class.  This class will execute network requests and deserialize JSON responses into entities from the `entity` package.  To do this, we will initialize and store the `httpClient` property with the Ktor` HttpClient` instance:

  `` kotlin
  package com.jetbrains.handson.kmm.shared.network

  import com.jetbrains.handson.kmm.shared.entity.RocketLaunch
  import io.ktor.client.HttpClient
  import io.ktor.client.features.json.JsonFeature
  import io.ktor.client.features.json.serializer.KotlinxSerializer
  import io.ktor.client.request. *
  import kotlinx.serialization.json.Json

  cla ...

# Building SDK

  Our iOS and Android applications will communicate with the SpaceX API through
  our KMM Module, which will provide a public `SpaceXSDK` class.  Let's create this class in the `com.jetbrains.handson.kmm.shared` package of the common source set.  It will be the facade over `Database` and` SpaceXApi` classes:

  `` kotlin
  package com.jetbrains.handson.kmm.shared

  import com.jetbrains.handson.kmm.shared.cache.Database
  import com.jetbrains.handson.kmm.shared.cache.DatabaseDriverFactory
  import com.jetbrains.handson.kmm.shared.network.SpaceXApi
  import com.jetbrains.handson.kmm.shared.entity.RocketLaunch

  class SpaceXSDK (databaseDriverFactory: DatabaseDriverFactory) {
      private val database = Database (databaseDriverFactory)
      private val api = SpaceXApi ()
  }
  ``

  To create a `Database` class instance, we need to provide the` DatabaseDriverFactory` platform instance to it, so we will inject it from the platform code through the `SpaceXSDK` class constructor.

  Our class will contain…

# Creating the Android application

  The configuration we need was configured by the KMM Android Studio plugin, so we have our KMM module already connected to our Android application.  Before implementing the UI and presentation logic, let's add all the required dependencies to the `androidApp / build.gradle.kts`:

  `` kotlin
  // ...

  dependencies {
      implementation (project (": shared"))
      implementation ("org.jetbrains.kotlinx: kotlinx-coroutines-android: 1.3.9")
      implementation ("androidx.core: core-ktx: 1.3.1")
      implementation ("com.google.android.material: material: 1.2.0")
      implementation ("androidx.appcompat: appcompat: 1.2.0")
      implementation ("androidx.constraintlayout: constraintlayout: 2.0.0")
      implementation ("androidx.recyclerview: recyclerview: 1.1.0")
      implementation ("androidx.swiperefreshlayout: swiperefreshlayout: 1.1.0")
      implementation ("androidx.cardview: cardview: 1.0.0")
  }

  // ...
  ``

  ## Implementing the UI: display the list of rocket launches

  Let's modify `activity ...
# Creating the iOS application

  For the iOS part of our project, we will use SwiftUI to build the user interface and MVVM pattern to connect the UI to our KMM module, which contains all the business logic we need.  Our KMM module is already connected to the iOS project - all the configuration we need was done by the Android Studio plugin wizard.  We can use it by simply importing it, just as we import regular iOS dependencies:

  `` swift
  import shared
  ``

   So all we need to do now is implement the SwiftUI views we need and fill them with the data.

  ## Implementing the UI: display the list of rocket launches

  First, let's create a SwiftUI view for displaying an item from the list.  To do this, we create a `RocketLaunchRow` SwiftUI view.  It will be based on `HStack` and` VStack` views.  We will provide extensions on the `RocketLaunchRow` structure with some useful helpers for displaying our data.  To do this, in our Xcode project we create the new file with the type ** SwiftUI View ** named `Rocke ...
