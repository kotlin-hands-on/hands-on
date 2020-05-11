# Introduction

In this hands-on, we're going to create an HTTP API using Kotlin and Ktor that can serve as a backend for any application, be it mobile, web, desktop, or even a B2B service. We will see how routes are defined and structured, how serialization features help with simplifying tedious tasks, and how we can test parts of our application both manually and automated.

### What we will build

Throughout the hands-on, we'll build a simple JSON API that allows us to query information about the customers of our fictitious business, as well as the orders we currently want to fulfill.

We will build a convenient way of listing all customers & orders in our system, get information for individual customers & orders, and provide functionality to add new entries and remove old entries.

We will be using two ways to define routes and organize these by files. They certainly aren't the only ways to define routes in applications, but they showcase differently maintainable approaches. For other styles and options check out the blog post by [Hadi Hariri](https://twitter.com/hhariri) on [Routing in Ktor](https://hadihariri.com/2020/04/02/Routing-in-Ktor/).

You can find the [template project](https://github.com/kotlin-hands-on/creating-http-api-ktor/) as well as the source code of the [final](https://github.com/kotlin-hands-on/creating-http-api-ktor/tree/final) application on the corresponding GitHub repository.

