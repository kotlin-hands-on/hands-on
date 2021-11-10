# Deploying to production and the cloud

We've built a whole app from start to finish - not a small feat!
It's time to get it *out there* - published for the world to see.

### Packaging a production build

The whole project in production mode, run the `build` Gradle task.
You can do that either in the tool window in IntelliJ IDEA, or by running `./gradlew build`.
This generates an optimized build of the project, applying various improvements such as DCE (dead code elimination).

Once the build has finished, you can find all the files needed for deployment in `/build/distributions`. They include the JS files, HTML files and other resources required to run our application.

You can put them on a static HTTP server of your choice, serve them using GitHub Pages, or host them on a cloud provider of your choice.

### Deploying to Heroku

Heroku makes it quite simple to spin up an application that is reachable under its own domain. Their free tier should be enough for development purposes and for showing off to a few of your friends and coworkers!

After successfully [creating an account](https://signup.heroku.com/), [installing and authenticating the CLI client](https://devcenter.heroku.com/articles/heroku-cli), we can create a git repository and attach a Heroku app by running the following commands in the Terminal while in our project root:

```bash
git init
heroku create
git add .
git commit -m "initial commit"
```

Unlike a regular JVM application that would run on Heroku (for example written with Ktor or Spring Boot), our app generates static HTML pages and JS files that need to be served accordingly. We can adjust the required buildpacks to properly serve our program.

```bash
heroku buildpacks:set heroku/gradle
heroku buildpacks:add https://github.com/heroku/heroku-buildpack-static.git
```

To allow the `heroku/gradle` buildpack to run properly, a `stage` task needs to be present in our Gradle build file. Luckily, it is equivalent to our `build` task – and, as luck would have it, the corresponding alias is already included at the bottom of our Gradle build file:

```kotlin
// Heroku Deployment (chapter 9)
tasks.register("stage") {
    dependsOn("build")
}
```

We configure the `buildpack-static` by adding a file called `static.json` to the root of our project. We add the `root` property inside the file as follows:

```xml
{
    "root": "build/distributions"
}
```

We can now trigger a deployment, for example by running the following command:

```bash
git add -A
git commit -m "add stage task and static content root configuration"
git push heroku master
```

```note
If you're pushing from a non-main branch (such as the `step` branches from the example repository), you need to adjust the command to push to the `main` remote. (such as `git push heroku 08-deploying-to-production:main`)
```

If everything has gone according to plan, we will see the URL under which we can reach our application.

![image-20190730200111014](./assets/deployingToProduction.png)

You can find the state of the project after this section on the `finished` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js-gradle/tree/finished) repository.
