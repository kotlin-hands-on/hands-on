# What's next

### Feature requests

Obviously, our demo application is far from perfect or useful. After going through this hands-on, you can use it as a jumping-off point to explore more advanced topics in the realm of React, Kotlin/JS, and friends.

##### Search

Wouldn't it be great if you could filter the list of talks by title or author? For that, a search field might be a great addition to the app – and an even greater way to learn about how [HTML form elements work in React](https://reactjs.org/docs/forms.html).

##### Persistence

Our application loses track of the viewer's watch list every time the page gets reloaded. Maybe it's time to build a proper backend! Familiarize yourself with one of the web frameworks available for Kotlin (like [Ktor](https://ktor.io/)), and try writing a backend for your application that can save the list of watched and unwatched videos! Or maybe there is a way to [store information on the client](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)...

##### Complex APIs

There's a wonderful world out there, full of datasets and APIs to be explored. Why not try building a visualizer for [cat photos](https://thecatapi.com/)? Or enrich your life with a [royalty-free stock photo API](https://unsplash.com/developers)? There's no shortage of data you'll be able to pull into your application.

### Better style: Responsiveness and grids

Our final product still doesn't fare well under extreme layout circumstances, for example in narrow windows or on phone screens. CSS Grids might be a fun way to explore how to make the app responsive. (Bonus challenge: no media queries!)

### Try more libraries

In the [kotlin-wrappers](https://github.com/JetBrains/kotlin-wrappers) repository, you can find some more wrappers and basic information about how to get started with additional libraries that have official Kotlin bindings, for example (but not limited to):

- [React-Redux](https://github.com/JetBrains/kotlin-wrappers/tree/master/kotlin-react-redux)
- [React-Router-DOM](https://github.com/JetBrains/kotlin-wrappers/tree/master/kotlin-react-router-dom)

### Community, help and troubleshooting

The best way to get help about Create-React-Kotlin-App is to visit the official [YouTrack issue tracker](https://youtrack.jetbrains.com/issues/CRKA). If you can't find your problem, do not hesitate to file a new issue. You can also join the official [Kotlin Slack](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up). We have channels for `#javascript`, `react` and more available.

### Learning more about coroutines

We've only touched the surface of the powerful concepts that make coroutines worth using across your applications. If you're interested in finding out more about how you can write concurrent code, check out the hands-on lab on [coroutines](https://play.kotlinlang.org/hands-on/Introduction%20to%20Coroutines%20and%20Channels/01_Introduction)!

### Learning more about React

The [official guides on React](https://reactjs.org/docs/) are quite plentiful and well-written. Now that you've been exposed to the basic concepts and how they translate to Kotlin, you'll hopefully have an easy time converting some of the other concepts outlined in the official guides into Kotlin, and may even become a React-Kotlin-Master!