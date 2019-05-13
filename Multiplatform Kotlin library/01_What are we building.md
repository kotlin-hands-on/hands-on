# What are we building?

Our goal is to build a small multiplatform library to demonstrate the ability to share the code between the platforms and its benefits. In order to have a small implementation to focus on the multiplatform machinery, we will write a library which converts raw data (strings and byte arrays) to the Base64 format which can be used on JVM, JS, and any available K/N platform. On JVM implementation will be using java.util.Base64 which is known to be extremely efficient because JVM is aware of this particular class and compiles it in a special way. On JS we will be using the native Buffer API and on Kotlin/Native we will write our own implementation. We will cover this functionality with common tests and then publish the resulting library to Maven.


### Big cat is watching you man

![cat](./assets/cat.jpg)