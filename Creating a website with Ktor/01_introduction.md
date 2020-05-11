# Introduction

In this hands-on tutorial we're going to create an interactive website using Kotlin and [Ktor](https://ktor.io), a framework for building connected applications.

Using the different features and integrations provided by Ktor, we will see how to host static content like images and HTML pages. We will see how supported HTML templating engines like [Freemarker](http://freemarker.org/) make it easy to control how data from our application is rendered in the browser. By using [kotlinx.html](https://github.com/Kotlin/kotlinx.html), we'll learn about a domain-specific language that allows us to mix Kotlin code and markup directly, allowing us to write our site's display logic in pure Kotlin.

### What we will build

The goal of this hands-on is to write a minimal journal app. We'll start by seeing how Ktor can serve static files and pages, and then move on to dynamically rendering Kotlin objects representing blog entries in a nicely formatted fashion, making use of our template engine. To make things interactive, we will add the ability to submit new entries to our journal directly from the browser – leaving us with a nice way to temporarily store and view our thoughts, for example our opinion on working through this hands-on tutorial:

![](./assets/ktor_journal.gif)

You can find the [template project](https://github.com/kotlin-hands-on/creating-website-ktor) as well as the source code of the [final](https://github.com/kotlin-hands-on/creating-website-ktor/tree/final) application on the corresponding GitHub repository.

Let's dive right in and start setting up our project for development!