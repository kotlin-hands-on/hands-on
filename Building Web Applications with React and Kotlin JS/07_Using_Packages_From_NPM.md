# Using packages from NPM

We have come a long way since writing our first little snippet, and our app is looking and working better than ever.
It's still missing some essentials to make it usable, though.
Let's add two more features:

- A video player that actually plays videos.
- Some social share buttons to help people share content they enjoy.

Instead of building this functionality ourselves, let's use React's ecosystem,
and import pre-made components for these use cases.

### Adding the video player component

Let's replace the placeholder video player with an actual YouTube player.
To do so, we'll use the [react-youtube-lite](https://www.npmjs.com/package/react-youtube-lite) package from npm.
It has a simple enough API and small documentation as well, so it seems perfect for our purposes.

The template project for this tutorial already includes the `react-youtube-lite` package in the `build.gradle.kts` file.
This is the responsible snippet:

```kotlin
dependencies {
    //...
    //Video Player (chapter 7)
    implementation(npm("react-youtube-lite", "1.5.0"))
    //...
}
```

As you can see,
npm dependencies can be added to a Kotlin/JS project
by using the `npm` function in the `dependencies` block of the build file.
The Gradle plugin then takes care of downloading and installing these dependencies for you.
(To do so, it uses its own bundled installation of the [`yarn`](https://yarnpkg.com/) package manager.)

To use the JavaScript package from inside our React application, we need to tell the Kotlin compiler what to expect.
We do this by providing it with [external declarations](https://kotlinlang.org/docs/js-interop.html).

Create a file called `ReactYouTube.kt`, and add the following content:

```kotlin
@file:JsModule("react-youtube-lite")
@file:JsNonModule

import react.*

@JsName("ReactYouTubeLite")
external val ReactPlayer: ComponentClass<dynamic>
```

This is a bit of adapter-work that we need to do.
When the compiler sees an external declaration like `ReactPlayer`,
it assumes that the implementation for the corresponding class is provided by our dependency,
and doesn't generate code for it.

Unfortunately, JavaScript imports and exports aren't the simplest topic.
It can sometimes be tricky to find the correct combination between annotations
to get the Kotlin compiler on the same page as us.
These last two lines are equivalent to a JavaScript import like `require("react-youtube-lite").default;`.
It tells the compiler that we're certain we'll get a component conforming to `RClass<dynamic>` at runtime.

#### Typed wrappers for the video player component

The snippet above is cheating a little.
The generic type for the props accepted by `ReactPlayer` is set to `dynamic`.
That means the compiler will just accept any code, at the risk of breaking things at runtime.

A better alternative would be to create an `external interface`,
which specifies what kind of properties belong to the props for this external component.
We can infer the props interface
based on the [README](https://www.npmjs.com/package/react-youtube-lite) for the component:
`react-youtube-lite` takes a prop called `url` of type `String`.

Adjust the content of `ReactPlayer.kt` accordingly:

```kotlin
@file:JsModule("react-youtube-lite")
@file:JsNonModule

import react.*

@JsName("ReactYouTubeLite")
external val ReactPlayer: ComponentClass<ReactYouTubeProps>

external interface ReactYouTubeProps : Props {
    var url: String
}
```

With this plumbing out of the way,
we can now use our new `ReactPlayer` to replace the gray placeholder rectangle in our `VideoPlayer` component.

Back in `VideoPlayer.kt`, replace the `img` tag with the following snippet:

```kotlin
ReactPlayer {
    url = props.video.videoUrl
}
```



### Adding social share buttons

Like most things, KotlinConf talks are best enjoyed together.
An easy way to enjoy the conference talks together with friends would be to have some "share" buttons.
For our demo, a messenger and email would probably be enough, though these could be expanded to include more platforms.
Once more, we can use an off-the-shelf React component for this purpose.
We'll choose
[react-share](https://github.com/nygardk/react-share/blob/master/README.md).

Once again, this npm library is already included in the `build.gradle.kts` file of our project:

```kotlin
dependencies {
    //...
    //Share Buttons (chapter 7)
    implementation(npm("react-share", "4.4.0"))
    //...
}
```

Just like the video player,
we will need to write some basic external declarations in order to use `react-share` from Kotlin.
Looking at the [examples on GitHub](https://github.com/nygardk/react-share/blob/master/demo/Demo.tsx#L61),
it becomes clear that each share button consists of two React components.
For example, there's an`EmailShareButton` and an `EmailIcon`.
The different types of share buttons and icons all share the same kind of interface, which makes our job easier.
Creating the external declarations for each component happens the same way as we already did for the video player.

Add the following code to a new file called `ReactShare.kt`:

```kotlin
@file:JsModule("react-share")
@file:JsNonModule

import react.ComponentClass
import react.Props

@JsName("EmailIcon")
external val EmailIcon: ComponentClass<IconProps>

@JsName("EmailShareButton")
external val EmailShareButton: ComponentClass<ShareButtonProps>

@JsName("TelegramIcon")
external val TelegramIcon: ComponentClass<IconProps>

@JsName("TelegramShareButton")
external val TelegramShareButton: ComponentClass<ShareButtonProps>

external interface ShareButtonProps : Props {
    var url: String
}

external interface IconProps : Props {
    var size: Int
    var round: Boolean
}
```

Let's plug our new components into the user interface of our application.

In `VideoPlayer.kt`, add two share buttons in a `div` right above our usage of `ReactPlayer`:

```kotlin
// . . .
div {
    css {
        display = Display.flex
        marginBottom = 10.px
    }
    EmailShareButton {
        url = props.video.videoUrl
        EmailIcon {
            size = 32
            round = true
        }
    }
    TelegramShareButton {
        url = props.video.videoUrl
        TelegramIcon {
            size = 32
            round = true
        }
    }
}
// . . .
```

Just like that, we've enriched our application with the new feature of social sharing!
Head back to the browser and play around with the buttons to check that they really work.
They open a *share window*, and pre-fill it with the URL of the video.

(If the buttons don't show up or don't seem to be working, you may need to turn off your ad/social blocker.) 

![image-20190729192417600](./assets/image-20190729192417600.png)

[react-share](https://github.com/nygardk/react-share/blob/master/README.md#features) offers a number of other share buttons. If you have a favorite social network you want to include, you can repeat the steps above to include some more share buttons as an exercise.

Our app is taking more and more shape - but it still relies on some basic demo data, instead of "the real thing".
We'll tackle that next.

You can find the state of the project after this section on the `06-packages-from-npm` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js-gradle/tree/06-packages-from-npm) repository.
