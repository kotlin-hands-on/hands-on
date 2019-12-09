# Introduction

```warning
These materials use create-react-app-kotlin-app. Going forward, the whole build experience for Kotlin/JS will be unified under the new and already available kotlin.js Gradle plugin. While topics unrelated to the build system remain the same, there will be changes regarding dependencies, running and deploying your application.

**You can expect this tutorial to be updated to reflect these changes in early 2020.**
```

In this hands-on tutorial, we will take a look at how we can use Kotlin/JS in conjunction with the popular framework [React](https://reactjs.org/) to build appealing and maintainable applications for the browser. React lets us build web applications in a modern and well-structured way, focusing on the reusability of components and well-structured management of the application state, and is supported by a large ecosystem of community-created materials and components.

Using Kotlin to write React applications allows us to reuse our knowledge about language paradigms, syntax, and tooling to build front-end applications for modern browsers and make use of Kotlin libraries while leveraging the capabilities of the JavaScript platform and its ecosystem.

During this hands-on, we will learn how to build our first application with Kotlin/JS and React using the `create-react-kotlin-app` tool. We will look at the usual kinds of tasks associated with building a typical, simple React application. 

We will explore how domain-specific languages can be used to help express concepts in a concise and uniform fashion without sacrificing readability, allowing us to write a fully-fledged application completely in Kotlin. We will also learn how to use ready-made components created by the community, make use of external libraries, and how to publish our final application.

You're expected to have a basic knowledge of Kotlin, and *very basic* knowledge of HTML and CSS. Prior knowledge of the basic concepts behind React may be helpful in understanding some of the sample code, but is not strictly required.

### What we are going to build

The annual [*KotlinConf*](https://kotlinconf.com/) is the event to visit if you want to learn more about Kotlin and exchange with the community. With 1300 attendees and a ton of workshops and sessions, KotlinConf 2018 offered a wealth of information. Its talks are publicly available on YouTube, so it may be useful for Kotlin aficionados to have an overview of the talks, quickly mark them as *seen* or *unseen*, and watch them all from one page: perfect for a Kotlin-binge-session! That is exactly the goal of *KotlinConf Explorer* – the project that we will create through this hands-on.

![image-20190729201914738](/assets/image-20190729201914738.png)

You can find the source code of the final application as well as the intermediate steps on the corresponding [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js) repository. Each step is available from its own branch, and is linked at the bottom of each corresponding section.

Let's start by setting up our development environment and installing the tools we’ll need to get us up and running.

