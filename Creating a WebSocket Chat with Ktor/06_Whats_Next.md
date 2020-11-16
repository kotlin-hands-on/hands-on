# What's next

Congratulations on finishing this tutorial on creating a chat application using Kotlin, Ktor & WebSockets. We now have a basic command-line application which allows multiple clients to have a conversation over the network in a shared chat.

### Feature requests

At this point, we have implemented the absolute basics for a chat service, both on client and server side. If you want to, you can keep expanding on this project. To get you started, here are a few ideas of how to improve the application, in no particular order:

- **Custom usernames!** Instead of automatically assigning numbers to your users, you can ask users on application startup to enter a user name, and persist this name alongside the Connection information on the server.
- **Private messages!** If your users have something to say, but don't want to share it with the whole group, you could implement a /whisper command, which only relays the message to a certain person or select group of participants. You could even expand this functionality to handle more generic **chat commands!**
- **Nicer UI!** So far, the client's user interface is very rudimentary, with only text input and output. If you're feeling adventurous, you can pick up a framework like [TornadoFX](https://tornadofx.io/), [Compose for Desktop](https://www.jetbrains.com/lp/compose/), or other, and try implementing a fancy user interface for the chat.
- **Mobile app!** The Ktor client libraries are also available for mobile applications. Feel free to try integrating what you have learned in this tutorial in the context of an Android application, and build the next big mobile chat product!

### Learning more about Ktor

You can find more hands-on tutorials on Ktor and its features on this site. For in-depth information about the framework, including further demo projects, check out [ktor.io](https://ktor.io/).

### Community, help and troubleshooting

To find more information about Ktor, check out the official website. If you run into trouble, check out the [Ktor issue tracker](https://github.com/ktorio/ktor/issues) on GitHub – and if you can't find your problem, don't hesitate to file a new issue.

You can also join the official [Kotlin Slack](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up). We have channels for #ktor and more available, and a helpful community that supports each other for Kotlin related problems.