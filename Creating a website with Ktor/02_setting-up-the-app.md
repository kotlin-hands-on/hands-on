# Setting up the application

While the steps for this application are similar to the [Getting Started with Ktor](https://play.kotlinlang.org/hands-on/Getting%20Started%20with%20Ktor/01_introduction) tutorial, there is one small change. We'll be using an external configuration file to configure 
the port and some other parameters of our application. This means that instead of creating an embedded server and defining the port 
in code

```kotlin
fun main() {
    val server = embeddedServer(Netty, 8080) {
        // Routing code
    }
    server.start(wait = true)
}
```

we'll be using the follow structure

```kotlin
fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

fun Application.module() {
    routing {
        // Routing code
    }
}
```

and an `application.conf` file which uses [HOCON - Human-Optimized Config Object Notation](https://github.com/lightbend/config/blob/master/HOCON.md) to configure the application. This file
resides in the `resources` folder of the project.

```kotlin
ktor {
    deployment {
        port = 8080
        port = ${?PORT}
    }
    application {
        modules = [ com.jetbrains.handson.website.ApplicationKt.module ]
    }
}
```

The main reason for this is that in real-world applications, we want this configuration to be external, and amongst other things, allow us to have development and production
configurations.

### Adding logging

In addition we're adding logging to the application using the [logback](http://logback.qos.ch/) library. For this we simply need to add
the corresponding artifact dependency in our `build.gradle` file, and place the `logback.xml` configuration file 
in the `resources` folder

**application.conf**

```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{YYYY-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="trace">
        <appender-ref ref="STDOUT"/>
    </root>
    <logger name="org.eclipse.jetty" level="INFO"/>
    <logger name="io.netty" level="INFO"/>
</configuration>
```

**build.gradle** 

```groovy
plugins {
    id 'org.jetbrains.kotlin.jvm' version "$kotlin_version"
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
    implementation "io.ktor:ktor-html-builder:$ktor_version"
    implementation "ch.qos.logback:logback-classic:$logback_version"
}

compileKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
compileTestKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
```

The versions are defined in the corresponding `gradle.properties` file

```
ktor_version=1.3.2
kotlin.code.style=official
kotlin_version=1.3.70
logback_version=1.2.1
```
