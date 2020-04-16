# Introduction

In this hands-on, we will learn how to build a connected application consisting of a Kotlin/JVM server part, and a Kotlin/JS client part. This includes providing simple JSON API, as well as consuming this API from a Kotlin + React application running in the browser.

We structure our application as a Kotlin multiplatform project. Throughout the hands-on, we will see how this allows us to manage both server and client from within the same Gradle project, and keep an overview of our whole system from within our IDE. We will also explore how multiplatform projects enable us to share code and data models between client and server in our application. This will help us to make sure that our whole system is consistent, and lets us avoid duplicating code unnecessarily.

By using popular Kotlin multiplatform libraries such as `kotlinx.serialization`, `kotlinx.coroutines`, and the `ktor` framework for API server and client, we will see how we can keep our application concise, performant, and type-safe – *even when communicating across network boundaries*.

For this hands-on, you are expected to have an understanding of Kotlin. Some knowledge about basic concepts in React and Kotlin Coroutines may be helpful in understanding some of the sample code, but is not strictly required.

### What we will build

When going out to the supermarket, it's always good to know what we want to buy, rather than rely on impulse purchases. That's how you end up with candy and chips, and not enough fruits and vegetables! So, for our demo application, we will build a simple shopping list application that allows us to plan out our groceries.

In the interest of concise and easy to understand code, we will keep the user interface of our application quite simple: a list of planned purchases and a field to enter any new shopping plans. By clicking on an element in our shopping list, we automatically remove it.

Because not all purchases might be equally important, we also allow our users to specify a priority level. Simply by entering a number of exclamation points `!` to their shopping list entry, the user can assign a priority to the item. We'll also make sure that the list is always ordered, so that we can see the most important entries first.

![](/assets/finished.gif)

To achieve all this functionality, we will first build a small backend that serves a JSON API. Afterwards, we will use the data served by this API to populate our simple web interface. Last but not least, we'll expand the server of the application to include a proper database, to make sure that our data persists even across multiple reloads and restarts of the application.

To make the app available even while we're standing in the supermarket, we'll also cover how to deploy the shopping list to a cloud provider in the last chapter.

Before we dive in and start coding our application, let's talk for a brief moment about why choosing a Kotlin multiplatform architecture for such a type of project can be benefitial.

### Benefits of a multiplatform architecture

We will write this whole application in Kotlin: The backend will use Kotlin/JVM, the frontend will use Kotlin/JS. This has a number of benefits: Besides syntax, it also allows us to share our libraries and  programming paradigms (such as using coroutines for concurrency), on both frontend and backend.

Using Kotlin throughout the whole stack also makes it possible to write classes and functions that can be used from both the JVM and JS targets of our application. For this application, we will primarily use this functionality to share a typesafe representation of our data between client and server.

By delegating the task of serizaliation and deserialization from and to typesafe objects to multiplatform Kotlin libraries such as `kotlinx.serialization`, we can make the communication of data safe and practically trivial.

Motivated enough? Let's dive in!
