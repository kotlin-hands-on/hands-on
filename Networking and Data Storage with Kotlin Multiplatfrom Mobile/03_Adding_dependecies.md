# Adding dependencies to the multiplatform library

We will use the following multiplatform libraries in our project:

* `kotlinx.coroutines` – we will use coroutines for asynchronous code.
* `Ktor` – this will be our HTTP client for retrieving data over the internet.
* `kotlinx.serialization` – to deserialize JSON responses into objects of entity classes.
* `SQLDelight` – this will generate Kotlin code from SQL queries to create a type-safe database API.

To add a multiplatform library to the KMM module, we need to add dependency instructions (in our case, implementation) to the dependencies block of the relevant source sets in the 'build.gradle.kts' file in the **KMM module's directory**. You can read more about adding dependencies in the [documentation](https://kotlinlang.org/docs/mobile/add-dependencies.html#multiplatform-libraries).

### Adding kotlinx.coroutines

We need to specify a dependency on `kotlinx.coroutines` in the common source set in order to add them to our project. We will do this by adding the following line to the `build.gradle.kts` file in the KMM module directory:

```kotlin
val coroutinesVersion = "1.5.0-native-mt"

sourceSets {
    commonMain {
        dependencies {
            // ...
            implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:$coroutinesVersion")
        }
    }
}
```

A dependency to the platform-specific (iOS and Android) parts of `kotlinx.coroutines` will be added automatically by the Multiplatform Gradle plugin.


### Adding kotlinx.serialization

Before we can add the `kotlinx.serialization` library, we first need to add the plugin required by the build system. 

In the `build.gradle.kts` file in the **project root directory**, we need to specify the classpath for the plugin in the build system dependencies:

```kotlin
buildscript {
    // ...
    val kotlinVersion = "1.5.21"

    dependencies {
        // ...
        classpath("org.jetbrains.kotlin:kotlin-serialization:$kotlinVersion")
    }
}
```

Add this line to the plugins block at the very beginning of the `build.gradle` file in the **KMM module directory**:

```kotlin 
plugins {
    // ...
   kotlin("plugin.serialization")
}
```

Now we need to add the library itself to the module. We do this in the same way that we added the `kotlinx.serialization`:

```kotlin
val serializationVersion = "1.2.2"

sourceSets {
    val commonMain by getting {
        dependencies {
            // ...
            implementation("org.jetbrains.kotlinx:kotlinx-serialization-core:$serializationVersion")
        }
    }
}
```

### Adding Ktor

We can add Ktor in the same way we added the `kotlinx.coroutines` library. In addition to specifying the core dependency in the common source set, we also need to:
* Add serialization [feature](https://ktor.io/clients/http-client/features.html) to use `kotlinx.serialization` for processing network requests and responses.
* Provide the platform engines by adding dependencies on the corresponding artifacts in the platform source sets.

```kotlin
val ktorVersion = "1.6.1"

sourceSets {
    val commonMain by getting {
        dependencies {
            // ...
            implementation("io.ktor:ktor-client-core:$ktorVersion")
            implementation("io.ktor:ktor-client-serialization:$ktorVersion")
        }
    }
    val androidMain by getting {
        dependencies {
            // ...
            implementation("io.ktor:ktor-client-android:$ktorVersion")
        }
    }
    val iosMain by getting {
        dependencies {
            // ...
            implementation("io.ktor:ktor-client-ios:$ktorVersion")
        }
    }
}
```

### Adding SQLDelight

As with the `kotlinx.serialization` library, we first need to add the compiler plugin. To do this, we need to:

Define the SQLDelight version in the root `gradle.properties` file to ensure that we use the same version for both the plugin and the libraries:

```
sqlDelightVersion=1.4.2
```

In the `build.gradle` file in the **project root directory**, we need to specify the classpath for the plugin in the build system dependencies:

```kotlin
buildscript {
    // ...
    val sqlDelightVersion: String by project

    dependencies {
        // ...
        classpath("com.squareup.sqldelight:gradle-plugin:$sqlDelightVersion")
    }
}

```

Add this line to the plugins block at the very beginning of the `build.gradle` file in the KMM module directory:

```kotlin
plugins {
    // ...
   id("com.squareup.sqldelight")
}
```

Now we need to add the library to the module. We do this in the same way that we added the Ktor library, specifying the core artifact in the common source set and platform drivers in the iOS and Android source sets in the `build.gradle` file of the KMM module:

```kotlin
val sqlDelightVersion: String by project

sourceSets {
    val commonMain by getting {
        dependencies {
            // ...
            implementation("com.squareup.sqldelight:runtime:$sqlDelightVersion")
        }
    }
    val androidMain by getting {
        dependencies {
            // ...
            implementation("com.squareup.sqldelight:android-driver:$sqlDelightVersion")
        }
    }
    val iosMain by getting {
        dependencies {
            // ...
            implementation("com.squareup.sqldelight:native-driver:$sqlDelightVersion")
        }
    }
}
```

Once we've done this, we need to sync the Gradle project.

## Results so far

Now we have added all the dependencies we need to our KMM module. You can find the state of the project after this section on the [final branch](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/tree/final) in the GitHub repository.

