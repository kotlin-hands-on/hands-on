# Creating the Project

Ktor projects can be created using either Gradle or Maven. In this tutorial we're going to create it from scratch so that
we understand how things are built. However, we can also use [start.ktor.io](https://start.ktor.io) and have it package
the project file for us to download. Alternatively there is an [IntelliJ IDEA plugin](https://plugins.jetbrains.com/plugin/10823-ktor) for Ktor that performs the same steps as the 
web.


## Getting the right dependencies

Ktor is split up into multiple artifacts allowing for fine-grained control. Depending on what we need we can include the [corresponding artifacts](https://ktor.io/quickstart/artifacts.html).
All Ktor packages are available on [Bintray](https://bintray.com/kotlin/ktor/).

## Setting up the Gradle build file

In this tutorial we're going to use Gradle, albeit the same set-up can be done using [Maven](https://ktor.io/quickstart/quickstart/maven.html).

Let's create a new `build.gradle` file, either using [IntelliJ IDEA](https://jetbrains.com/idea) project wizard or manually
and for it to have the following content


```groovy
plugins {
    id 'org.jetbrains.kotlin.jvm' version '1.3.71'
}

group 'com.jetbrains.handson'
version '1.0-SNAPSHOT'

repositories {
    jcenter()
    mavenCentral()
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8"
    implementation "io.ktor:ktor-server-core:1.3.2"
    implementation "io.ktor:ktor-server-netty:1.3.2"
}

compileKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
compileTestKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
```

Note that in case of using IntelliJ IDEA, the only lines we'd need to add would be `jcenter()` in `repositories` and the corresponding
dependencies for Ktor, which in this case we're including two, `server-core` which is required for all server-side applications, and `server-netty` which
is the module that provides us with the ability to self-host our applications using Netty. Ktor allows applications to run within an Application Server compatible with Servlets, such as Tomcat, or as an embedded application, using Jetty, Netty or CIO. In our
case it will be Netty. If we were to to use something else like Jetty, we'd include the corresponding module `server-jetty`.

Once we have the our build file ready, we can refresh it (if within the IDE) to download the corresponding dependencies and start writing some code. Our 
project structure should look like so

![Gradle file](./assets/gradle.png)


