# Introduction

In this practical hands-on guide, we will create a Kotlin Multiplatform Mobile (_KMM_) application for Android and iOS that includes a module with shared code for both platforms. The application will retrieve data over the internet from a public API, save it in a local database, and display it in a list in the application. We will implement business logic and data access layers only once in the KMM module, while the UI of both applications will be native.

We will use the **SpaceX** API, which provides public access to information about SpaceX rocket launches. The SpaceX API has detailed [documentation](https://docs.spacexdata.com/?version=latest). Our application will display a list of rocket launches together with the launch date, results, and a detailed description of the launch. This is what our applications will look like:

<img alt="Emulator and Simulator" src="./assets/android-and-ios.png" width="700">

We will use the [Ktor](https://ktor.io/clients/index.html) library to handle HTTP requests and [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization) for request serialization. We will use [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines) to write asynchronous code. The [SQLDelight](https://github.com/cashapp/sqldelight) library will be used for database operations. We will use the [CoroutineWorker](https://github.com/Autodesk/coroutineworker) library to manage coroutines in background threads.

You can find the [template project](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage) as well as the source code of the [final application](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/tree/final) on the corresponding GitHub repository.

## Preparing the local environment

In this tutorial, we will be using 
* [Android Studio](https://developer.android.com/studio/) with the [Kotlin Multiplatform Mobile plugin](https://plugins.jetbrains.com/plugin/13881-mobile-multiplatform) for working with the shared module and Android project.
* [Xcode](https://developer.apple.com/xcode/) for building the iOS application.

Please follow the [setup environment guide](https://kotlinlang.org/docs/mobile/setup.html) for detailed instructions on how to configure your environment.
