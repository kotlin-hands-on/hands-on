# Summary

In the tutorial we:
 - Created an Android application in Android Studio
 - Created an iOS application in Xcode
 - Added Kotlin multiplatform sub-project  
   - with shared Kotlin code
   - compiled it to Android Jar
   - compiled it to iOS Framework
 - Put it all together and re-used Kotlin code
 
We can find all the sources from that tutorial at [GitHub](https://github.com/JetBrains/kotlin-examples/tree/master/tutorials/mpp-iOS-Android).

# Next Steps

This small example of Kotlin code sharing between iOS and Android (and other platforms) 
with Kotlin, Kotlin/Native, and Kotlin multiplatform projects is only the beginning. 
The same approach can work for real applications, independent of their size or complexity.

The Kotlin/Native interop with Swift and Objective-C is covered in the
[documentation](https://kotlinlang.org/docs/reference/native/objc_interop.html).
This topic is also covered in the [Kotlin/Native as an Apple Framework](https://kotlinlang.org/docs/tutorials/native/apple-framework.html)
tutorial.

Multiplatform projects and multiplatform libraries are discussed in the [documentation](https://kotlinlang.org/docs/reference/multiplatform.html) too.

Sharing code between platforms is a powerful technique, but it may be hard to
accomplish without the rich APIs that we have in Android, JVM, and iOS platforms.
Multiplatform libraries can be used to fix that. They bring rich APIs
directly in the common Kotlin code. There are several examples of such libraries:  

- [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines/blob/master/native/README.md)
- [kotlinx.io](https://github.com/Kotlin/kotlinx-io)
- [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization)
- [ktor](https://ktor.io/)
- [ktor-http-client](https://ktor.io/clients/http-client.html)

Are you looking for more APIs? It is easy to create a [multiplatform library](https://kotlinlang.org/docs/tutorials/mpp/multiplatform-library.html) and share it!
