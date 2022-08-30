# Introduction

In this hands-on tutorial, we're going to get familiar with the concept of coroutines.
Coroutines give us all the benefit of asynchronous
and non-blocking behavior but without the lack of readability. 
We'll see how we can use coroutines to perform network requests
without blocking the underlying thread and without using callbacks.

We'll learn:

* Why and how to use suspend functions to perform network requests.
* How to send requests concurrently using coroutines.
* How to share information between different coroutines using channels.

We'll also discuss how coroutines are different from other existing solutions.
 
You're expected to be familiar with basic Kotlin syntax,
but prior knowledge of coroutines is not required.

For network requests we'll use the [Retrofit](https://square.github.io/retrofit/) library which already
supports coroutines, but the approach shown in this tutorial is universal and works similarly for different libraries.

This tutorial is based on the ["Asynchronous Programming with Kotlin"](https://kotlinconf.com/workshops/) workshop
by Roman Elizarov at KotlinConf.

### Set-up

We'll need IntelliJ IDEA; you can download the latest free community version [here](https://www.jetbrains.com/idea/download/).

Make sure version 1.4 of the Kotlin plugin is installed.
To update the Kotlin plugin, use `Tools | Kotlin | Configure Kotlin Plugin Updates`.

### Downloading project

Clone the repository from IntelliJ IDEA by choosing `Get from VCS` on the splash screen, or selecting `File | New | Project from Version Control...`,
and specifying the project path:
[http://github.com/kotlin-hands-on/intro-coroutines](http://github.com/kotlin-hands-on/intro-coroutines). 

...or clone it from the command line:

```
$ git clone http://github.com/kotlin-hands-on/intro-coroutines
```
 
The project includes the `solutions` branch, so it is possible to use the `Compare with branch...`
action to compare your solution with the proposed one.

### Generating GitHub developer token

We'll be using GitHub API.
For that, you need to specify your Github account name and either a password or a token.
If you have two-factor authentication enabled on GitHub, then only a token will work. 

You can generate a new GitHub token to use the GitHub API from your account here:
[https://github.com/settings/tokens/new](https://github.com/settings/tokens/new).
Specify the name of your token, for example, `coroutines-hands-on`:

![](./assets/1-intro/GeneratingToken.png)

There is no need to select any scopes, click on "Generate token" at the bottom of the screen.
Copy the generated token somewhere; we'll use it soon.

### Running the code

Open the `src/contributors/main.kt` file and run the `main` function.
There should then be a window like the one below (but without any data loaded).
If the fonts are too small, adjust them by specifying a different default size in the `main` function.

![](./assets/1-intro/InitialWindow.png)

Our program loads the contributors for all the repositories under the given organization.
By default, the organization is "kotlin" but it could be any other one. 
Later we'll add logic to sort the users by the number of their contributions.

Now fill in the GitHub username and token (or password) in the corresponding fields.
Make sure that in the variant dropdown menu the 'BLOCKING' option is chosen, then click on "Load contributors".
The UI should freeze for some time and then show the list of the contributors.
You can open the program output to make sure the data is loaded:
The information is logged after each successful request.
Make sure the loaded data can be seen before continuing.
 
We're going to compare different ways of implementing this logic, and see how coroutines change things.
Before we start using coroutines, let's take a look at what other solutions are available. 
First, we'll look at how to implement the logic in a blocking way: easy to read and reason for it, but wrong since it freezes the UI.
Then we'll use callbacks to fix it, and compare these solutions with one that uses coroutines.
Finally, we'll see how to use Channels to share information between different coroutines.

Let's begin!
