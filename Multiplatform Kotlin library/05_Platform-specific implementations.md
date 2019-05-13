# Platform-specific implementations

Now it is time to provide an `actual` implementation of `Base64Factory` for every platform.


## JVM

We are starting with an implementation for the JVM. Let's create a file `Base64.kt` in `jvmMain/kotlin/jetbrains/base64` folder and provide a simple implementation, which delegates to `java.util.Base64`:

<div class="language-kotlin" markdown="1" mode="kotlin" theme="idea" data-highlight-only>

```kotlin
actual object Base64Factory {
    actual fun createEncoder(): Base64Encoder = JvmBase64Encoder
}

object JvmBase64Encoder : Base64Encoder {
    override fun encode(src: ByteArray): ByteArray = Base64.getEncoder().encode(src)
}

```

</div>

Pretty simple, isn't it? We have provided a platform-specific implementation, but used a straightforward delegation to an implementation someone else has written!


## JS

Our JS implementation will be very similar to the JVM one. We create a file `Base64.kt` in `jsMain/kotlin/jetbrains/base64` and provide an implementation 
which delegates to NodeJS `Buffer` API:

<div class="language-kotlin" markdown="1" mode="kotlin" theme="idea" data-highlight-only>

```kotlin
actual object Base64Factory {
    actual fun createEncoder(): Base64Encoder = JsBase64Encoder
}

object JsBase64Encoder : Base64Encoder {
    override fun encode(src: ByteArray): ByteArray {
        val buffer = js("Buffer").from(src)
        val string = buffer.toString("base64") as String
        return ByteArray(string.length) { string[it].toByte() }
    }
}
```

</div>

## Native

On the generic Native platform we don't have the luxury to use someone else's implementation, so we will have to write one ourselves. I won't explain the implementation details here,
but it's pretty straightforward and follows Base64 format description without any optimizations:

<div class="language-kotlin" markdown="1" mode="kotlin" theme="idea" data-highlight-only>

```kotlin
private val BASE64_ALPHABET: String = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
private val BASE64_MASK: Byte = 0x3f
private val BASE64_PAD: Char = '='
private val BASE64_INVERSE_ALPHABET = IntArray(256) {
    BASE64_ALPHABET.indexOf(it.toChar())
}

private fun Int.toBase64(): Char = BASE64_ALPHABET[this]

actual object Base64Factory {
    actual fun createEncoder(): Base64Encoder = NativeBase64Encoder
}

object NativeBase64Encoder : Base64Encoder {
    override fun encode(src: ByteArray): ByteArray {
            fun ByteArray.getOrZero(index: Int): Int = if (index >= size) 0 else get(index).toInt()
            // 4n / 3 is expected Base64 payload
            val result = ArrayList<Byte>(4 * src.size / 3) 
            var index = 0
            while (index < src.size) {
                val symbolsLeft = src.size - index
                val padSize = if (symbolsLeft >= 3) 0 else (3 - symbolsLeft) * 8 / 6
                val chunk = (src.getOrZero(index) shl 16) or (src.getOrZero(index + 1) shl 8) or src.getOrZero(index + 2)
                index += 3
        
                for (i in 3 downTo padSize) {
                val char = (chunk shr (6 * i)) and BASE64_MASK.toInt()
                    result.add(char.toBase64().toByte())
                }
                // Fill the pad with '='
                repeat(padSize) { result.add(BASE64_PAD.toByte()) }
            }
    
            return result.toByteArray()
        }
    }
```

</div>

Now we have implementations on all the platforms and it is time to move to testing of our library.
