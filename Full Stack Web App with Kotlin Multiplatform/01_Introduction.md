# Introduction

In this hands-on, we will build a connected application consisting of two parts:

- A server, using Kotlin/JVM
- A web client, using Kotlin/JS

We will build a simple JSON API, and learn how to use the API from a web app using Kotlin & React.

We write both parts of the project together, as one Kotlin Multiplatform project.
This will allow us to manage both server and client from within the same Gradle project, and keep an overview of our whole system from within our IDE.
It's also a great way to learn more about code and data model sharing between client and server for our app.
This will help us to make sure that our whole system is consistent, and lets us avoid duplicating code unnecessarily.

By using popular Kotlin multiplatform libraries such as [`kotlinx.serialization`](https://github.com/Kotlin/kotlinx.serialization), [`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines), and the [Ktor](https://ktor.io/) framework for API server and client, we will build an app that is concise, performant, and type-safe – *even when communicating across network boundaries*.

For this hands-on, you are expected to have an understanding of Kotlin. Some knowledge about basic concepts in React and Kotlin Coroutines may be helpful in understanding some sample code, but is not strictly required.

### What we will build

When going out to the supermarket, it's always good to know what we want to buy, rather than rely on impulse purchases (That's how you end up with candy and chips, and not enough fruits and vegetables!)

So, for our demo application, we will build a simple shopping list application that allows us to plan out our groceries.

To focus on the essentials, we will keep the user interface simple: a list of planned purchases and a field to enter new shopping items. By clicking on an element in the shopping list, it gets removed.

We'll also allow the user to specify a priority level for list entries.
By adding an exclamation point `!` to an item, it gets a higher priority.
We'll use this information to order the shopping list.

![](/assets/finished.gif)

There's a lot ahead of us! First, we need to build a small backend that serves a JSON API. Then, we need to build our web interface which consumes that API.
As a bonus, we'll also expand the server of the application to include a proper database, making data persistent across multiple reloads and restarts of the application.

We'll also talk about how to deploy the shopping list to a cloud provider in the last chapter (so that while we're standing in the supermarket, the app will still be available). 

Before we dive in and start coding our application, let's talk about why choosing Kotlin Multiplatform for this project can be beneficial.

### Benefits of a multiplatform architecture

When we're done, the whole app will be written in Kotlin: the backend will use Kotlin/JVM, the frontend will use Kotlin/JS. This has a number of benefits: besides syntax, it also allows us to share our libraries and  programming paradigms (such as using coroutines for concurrency), on both frontend and backend.

Using Kotlin throughout the whole stack also makes it possible to write classes and functions that can be used from both the JVM and JS targets of our application. In this tutorial, we will primarily use this functionality to share a typesafe representation of our data between client and server.

By delegating the task of serialization and deserialization from and to typesafe objects to the `kotlinx.serialization` multiplatform library, we can make the communication of data safe and practically trivial.

Motivated enough? Let's dive in!
