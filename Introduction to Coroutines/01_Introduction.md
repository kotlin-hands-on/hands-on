# Introduction

In this hands-on tutorial, you'll get familiar with the concept of coroutines.
Coroutines help you to gain all the benefits of asynchronous
and non-blocking behaviour without lack of readability. 
You'll see how to use coroutines to perform network requests
without blocking the underlying thread and without using callbacks.

You'll learn:

* why and how to use suspend functions to perform networks requests
* how to send requests concurrently using coroutines
* how to share information between different coroutines using channels

We'll also discuss how coroutines are different from other existing solutions.
 
You're expected to be familiar with the basic Kotlin syntax,
but the prior knowledge of coroutines is not required.

For network requests we'll use the [Retrofit](https://square.github.io/retrofit/) library which already
supports coroutines, but the approach shown in this tutorial is universal and works similarly for a different library.

This tutorial is based on the ["Asynchronous Programming with Kotlin"](https://kotlinconf.com/workshops/) workshop
by Roman Elizarov at KotlinConf.

### Set-up

You'll need IntelliJ IDEA starting from the version 2018.1.
You can download the latest free community version [here](https://www.jetbrains.com/idea/download/).

Make sure you have the Kotlin plugin of the major version 1.3.
To update the Kotlin plugin, use `Tools | Kotlin | Configure  Kotlin Plugin Updates`.

### Downloading project

Clone the repository from IntelliJ IDEA by choosing `VSC | Checkout from Version Control | Git` and specifying the project path:
[http://github.com/kotlin-hands-on/coroutines](http://github.com/kotlin-hands-on/coroutines). 

...or clone it from the command line:

```
$ git clone http://github.com/kotlin-hands-on/coroutines
```
 
Note that for your convenience, the project contains the `solutions` branch, so you can use `Compare with branch...`
action to compare your solution with the proposed one.

Alternatively, you can [download the project](http://github.com/kotlin-hands-on/coroutines/archive/master.zip) directly
(but only the `main` branch).


### Generating GitHub developer token

We'll be using GitHub API.
For that, you'll need to specify your Github account name and either a password or a token.
If you have two-factor authentication enabled at GitHub, then only a token will work. 

Generate new GitHub token to use GitHub API from your account here:
[https://github.com/settings/tokens/new](https://github.com/settings/tokens/new).
Specify the name of your token, e.g. `coroutines-hands-on`:

![](./assets/1-intro/GeneratingToken.png)

You don't need to select any scopes, click on "Generated token" below the screen.
Copy the generated token somewhere, we'll use it soon.

### Running the code

Open `src/contributors/main.kt` file and run the `main` function.
You should see a window like the one below (yet without any data loaded).
If the fonts are too small, you can adjust them by specifying a different default size in the `main` function.

![](./assets/1-intro/InitialWindow.png)

Our program loads the contributors for all the repositories under the given organization.
By default, the oganization is "kotlin", but you can choose another one. 
Later we'll add the logic sorting users by the number of their contributions.

Now put in your login and token (or password) to the corresponding fields.
Make sure that the 'BLOCKING' option is chosen in the dropdown menu, then click on "Load contributors".
The UI should freeze for some time and then show the list of the contributors.
Make sure you see the loaded data in order to continue.
 
We're going to compare different ways of implementing this logic, and see how coroutines change the picture.
Before we start using coroutines, let's see what other solutions are available. 
At first, how to implement the logic in a blocking way: easy to read and reason about, but wrong since it freezes UI.
Then we'll use callbacks to fix that, and then compare these solutions with the one using coroutines.
Finally, we'll see how to use Channels to share information between different coroutines.

Let's start!
