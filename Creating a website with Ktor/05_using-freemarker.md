# Using Freemarker

In this last step we're going to use a templating engine to render out pages. While static files and HTML are quite useful, template engines
are not only quite common in web development but also have their own advantages. Ktor supports a variety of them but in our case we're going to
use [FreeMarker](https://freemarker.apache.org/)

### Adding FreeMarker as a Features

Features allow us to add certain type of functionality to Ktor. They work by being `inserted` into different segments of the request/response pipeline
and act accordingly. There are many types of features such as `Authentication`, `CORS`, `Compression`, etc. If wondering how Features are related
to FreeMarker, it is because lock much of the Ktor functionality, templates are nothing but Features.

In order to use a feature we need to *install* it. This is done during the application initialization. For this, let's add the following code
to our `Application.module` function

```kotlin
fun Application.module() {
    install(FreeMarker) {
        templateLoader = ClassTemplateLoader(this::class.java.classLoader, "templates")
    }
    routing {
        // Routes defined below...
    }
}
```

This tells our application that the templates will be located inside the folder `templates` located under the application `resources`. 

![Template](./assets/templates.png)

Our `index.ftl`
file should contain the following contents

```injectedfreemarker

<#-- @ftlvariable name="data" type="com.jetbrains.handson.website.IndexData" -->
<html>
<title>
        FreeMarker
</title>
<body>
<h1>FreeMarker</h1>
<p>
        Using FreeMarker Template Engine
</p>
<ul>
    <#list data.items as item>
        <li>${item}</li>
    </#list>
</ul>
</body>
</html>
```

The next is to create the corresponding Kotlin data class that will be used to inject the data into the template. Let's create
a class named `IndexData` with the following properties

```kotlin
package com.jetbrains.handson.website

data class IndexData(val items: List<Int>)
```

Finally let's add an endpoint that will serve this contents, by defining yet another route


```kotlin
        get("/freemarker") {
            call.respond(FreeMarkerContent("index.ftl", mapOf("data" to IndexData(listOf(1, 2, 3))), ""))
        }
```

Remember that this needs to be placed in the `routing` function in `Application.module`. Note that in real-world applications we usually
separate routes out into multiple files for better maintainability. 

We can now run the application and get the corresponding response to the url `/freemarker`

![FreeMarker Browser Output](./assets/freemarker.png)






In order to do this we need to indicate to Ktor where we want these files to be served from. This is done
in the `routing` entry using a [Feature](https://ktor.io/servers/features.html).  

 