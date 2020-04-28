# Introduction

In this tutorial we'll see how to create a chat application using [Ktor](https://ktor.io). We'll be developing
both the client and server using this framework. 

## WebSockets

WebSockets is a sub-protocol of HTTP. It starts as a normal HTTP request with an upgrade request header, and the connection switches to be a 
bidirectional communication instead of a request response one.

The smallest unit of transmission that can be sent as part of the WebSocket protocol, is a Frame. 
A WebSocket Frame defines a type, a length, and a payload that might be binary or text. Internally those frames might be transparently sent in several TCP packets. We can think of Frames as WebSocket messages. 
Frames could be the following types: 

* Text
* Binary
* Close
* Ping/Pong.

As consumers of Ktor, we'd normally be handling `Binary` and `Text` frames. The others are usually
handled by Ktor. 

## Creating the project

This tutorial assumes that you're familiar with Ktor on the server and will skip the steps of setting up a new Ktor project. 
For convenience, the `build.gradle` with the necessary dependencies is below:

```groovy
plugins {
    id 'org.jetbrains.kotlin.jvm' version "$kotlin_version"
    id 'org.jetbrains.kotlin.plugin.serialization' version "$kotlin_version"
}


group 'com.jetbrains.handson'
version '1.0-SNAPSHOT'

repositories {
    jcenter()
    mavenCentral()
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8"
    implementation "io.ktor:ktor-server-core:$ktor_version"
    implementation "io.ktor:ktor-server-netty:$ktor_version"
    implementation 'io.ktor:ktor-websockets:1.3.1'
}


compileKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
compileTestKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
```


If you're new to Ktor, check out some of the previous tutorials on getting up and running:

* [Getting Started with Ktor](https://play.kotlinlang.org/hands-on/Getting%20Started%20with%20Ktor/01_introduction)
* [Creating HTTP APIs with Ktor](https://play.kotlinlang.org/hands-on/Creating%20HTTP%20APIs%20with%20Ktor/01_introduction)


[Source code on GitHub](https://github.com/kotlin-hands-on/chat-app-websockets)
