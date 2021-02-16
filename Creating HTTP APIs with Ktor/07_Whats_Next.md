# What's next

With this step, we've finalized our HTTP API application. From here on we can add other 
features such as Authentication, etc.

### Feature requests

- **Authentication**: currently, the API is open to whomever would like to access it. If you want to restrict access, have a look at Ktor's support for [JWT](https://ktor.io/docs/jwt.html) and other authentication methods.

- **Learn more about route organization!** If you'd like to learn about different ways to organize your routes with Ktor, check out [this article](https://hadihariri.com/2020/04/02/Routing-in-Ktor/) by Hadi Hariri.

- **Persistence!** Currently, all our journal entries vanish when we stop our application, as we are only storing them in a variable. You could try integrating your application with a database like PostgreSQL or MongoDB, using one of the plenty projects that allow database access from Kotlin, like [Exposed](https://github.com/JetBrains/Exposed) or [KMongo](https://litote.org/kmongo/).

- **Integrate with a client!** Now that we are exposing data, it would make sense to explore how this data can be consumed again! Try writing an API client using the [Ktor HTTP client](https://ktor.io/clients/), for example, or try accessing it from a website using JavaScript or Kotlin/JS!
  
  To make sure your API works nicely with **browser clients**, you should also set up a policy for [Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS). The simplest and most permissive way to do this with Ktor would be by adding the following snippet to the top of `Application.module()`:

  ```kotlin
  install(CORS) {
      anyHost()
  }
  ```

### Learning more about Ktor

On this page, you will find a set of hands-on tutorials that also focus more on specific parts of Ktor. For in-depth information about the framework, including further demo projects, check out [ktor.io](https://ktor.io/).

### Community, help and troubleshooting

To find more information about Ktor, check out the official website. If you run into trouble, check out the [Ktor issue tracker](https://github.com/ktorio/ktor/issues) on GitHub – and if you can't find your problem, don't hesitate to file a new issue.

You can also join the official [Kotlin Slack](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up). We have channels for `#ktor` and more available, and a helpful community that supports each other for Kotlin related problems.
