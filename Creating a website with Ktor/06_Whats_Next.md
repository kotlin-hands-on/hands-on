# What's next

At this point, you should have gotten a basic idea of how to serve static files with Ktor and how the integration of features such as FreeMarker or kotlinx.html can enable you to write basic applications that can even react to user input.

### Feature requests for the journal

At this point, our journal application is still rather barebones, so of course it might be a fun challenge to add more features to the project, and learn even more about building interactive sites with Kotlin and Ktor. To get you started, here's a few ideas of how the application could still be improved, in no particular order:

- **Make it consistent!** You would usually not mix FreeMarker and kotlinx.html – we've taken some liberty here to explore more than one way of structuring your application. Consider powering your whole journal with kotlinx.html or FreeMarker, and make it consistent!
- **Authentication!** Our current version of the journal allows all visitors to post content to our journal. We could use [Ktor's authentication features](https://ktor.io/docs/authentication.html) to ensure that only select users can post to the journal, while keeping read access open to everyone.
- **Persistence!** Currently, all our journal entries vanish when we stop our application, as we are only storing them in a variable. You could try integrating your application with a database like PostgreSQL or MongoDB, using one of the plenty projects that allow database access from Kotlin, like [Exposed](https://github.com/JetBrains/Exposed) or [KMongo](https://litote.org/kmongo/).
- **Make it look nicer!** The stylesheets for the journal are currently rudimentary at best. Consider creating your own style sheet, and serving it as a static `.css` file from Ktor!
- **Organize your routes!** As the complexity of our application increases, so does the number of routes we try to support. For bigger applications, we usually want to add structure to our routing – like separating routes out into separate files. If you'd like to learn about different ways to organize your routes with Ktor, check out [this article](https://hadihariri.com/2020/04/02/Routing-in-Ktor/) by Hadi Hariri.

### Learning more about Ktor

On this page, you will find a set of hands-on tutorials that also focus more on specific parts of Ktor. For in-depth information about the framework, including further demo projects, check out [ktor.io](https://ktor.io/).

### Community, help and troubleshooting

To find more information about Ktor, check out the official website. If you run into trouble, check out the [ktor issue tracker](https://github.com/ktorio/ktor/issues) on GitHub – and if you can't find your problem, don't hesitate to file a new issue.

You can also join the official [Kotlin Slack](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up). We have channels for `#ktor` and more available, and a helpful community that supports each other for Kotlin related problems.

