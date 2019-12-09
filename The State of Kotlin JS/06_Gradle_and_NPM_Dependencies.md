# Using Gradle and NPM dependencies

Since `kotlin.js` is just a Gradle plugin, we can define dependencies as we would expect.

Example libraries you can try to set up are `kotlinx.coroutines`, `kotlinx.serialization`, or `ktor clients`. These libraries are all supported by JetBrains and have multiplatform artifacts available on their websites.

Something that would seem a little more challenging is packages from NPM. However, the `kotlin.js` plugin once again makes our life suprisingly easy. Consider the import of an NPM package called `is-sorted`. The corresponding part in the Gradle build file looks as follows:

```kotlin
kotlin {
    target {
        browser {

        }
    }
    sourceSets.main {
        dependencies {
            implementation(npm("is-sorted"))
        }
    }
}
```

Because JavaScript modules are usually dynamically typed and Kotlin is a statically typed language, we need to provide a kind of adapter. In Kotlin, these are called external declarations. For the `is-sorted` package which offers only one function, we can very quickly write it up. Inside the source folder, create a new file called `is-sorted.kt`, and fill it with these contents:

```kotlin
@JsModule("is-sorted")
@JsNonModule
external fun <T> sorted(a: Array<T>): Boolean
```

And just like that, we are ready to use it!

In the `main` method, we can now add some statements to see whether everything works, and check back in the developer tools of our browser to see the hopefully expected results!

```kotlin
console.log("Hello, Kotlin/JS!")
console.log(sorted(arrayOf(1,2,3)))
console.log(sorted(arrayOf(3,1,2)))
```

If everything went well, we see the magical three lines:

```kotlin
Hello, Kotlin/JS!
true
false
```

If you'd like to learn more about how to write declarations, please refer to the ["Calling JavaScript from Kotlin"](https://kotlinlang.org/docs/reference/js-interop.html) section of the Kotlin documentation.