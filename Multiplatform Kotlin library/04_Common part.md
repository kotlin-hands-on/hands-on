# Common part

Now we need to define the classes and interfaces we want to implement. Create the file `Base64.kt` in the `commonMain/kotlin/jetbrains/base64` folder.
Core primitive will be the `Base64Encoder` interface which knows how to convert bytes to bytes in `Base64` format:

<div class="language-kotlin" theme="idea" data-highlight-only>

```kotlin
interface Base64Encoder {
    fun encode(src: ByteArray): ByteArray
}
```

</div>

But the common code should somehow get an instance of this interface, for that purpose we define the factory object `Base64Factory`:

<div class="language-kotlin" theme="idea" data-highlight-only>

```kotlin
expect object Base64Factory {
    fun createEncoder(): Base64Encoder
}
```

</div>

Our factory is marked with the `expect` keyword. `expect` is a mechanism to define a requirement, which every platform should provide in order for the common part to work properly.
So on each platform we should provide the `actual` `Base64Factory` which knows how to create the platform-specific encoder.
You can read more about platform specific declarations