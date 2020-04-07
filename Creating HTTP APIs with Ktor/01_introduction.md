# Introduction

In this tutorial we're going to create an HTTP API application that can serve as 
a backend for any application, be this mobile, web, desktop, or even a B2B service. 

If you're new to Ktor make sure you check out how to create new projects 
with [Getting Started with Ktor](https://play.kotlinlang.org/hands-on/Getting%20Started%20with%20Ktor/01_introduction). 

This tutorial is going to follow some more real-world practices such as using configuration
files for configuring the application start-up, as well as organizing routes in a way that is 
maintainable. 

We'll be using two ways to define routes and organize these by files. This certainly isn't the only
way to define routes in applications but does offer a more maintainable approach. For other styles and options 
check out the blog post by [Hadi Hariri](https://twitter.com/hhariri) on [Routing in Ktor](https://hadihariri.com/2020/04/02/Routing-in-Ktor/).

[Source code on GitHub](https://github.com/kotlin-hands-on/creating-http-api-ktor)
