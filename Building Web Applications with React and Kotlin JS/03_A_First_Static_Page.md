# A first static page with React

Let's start our story with a classic little *Hello World*-type program!

Replace the code inside `src/main/kotlin/Main.kt` with the following:

```kotlin
import kotlinx.browser.document
import react.*
import emotion.react.css
import react.dom.render
import csstype.Position
import csstype.px
import react.dom.html.ReactHTML.h1
import react.dom.html.ReactHTML.h3
import react.dom.html.ReactHTML.div
import react.dom.html.ReactHTML.p
import react.dom.html.ReactHTML.img
import kotlinx.serialization.Serializable

fun main() {
    val container = document.getElementById("root") ?: error("Couldn't find root container!")

    render(Fragment.create {
        h1 {
            +"Hello, React+Kotlin/JS!"
        }
    }, container)
}
```

If you don't have the continuous Gradle build (as described in Chapter 2) still running in the background, run the build from your IDE (or execute `./gradlew run` from the command line), and have a look!

![image-20190729142055566](./assets/image-20190729142055566.png)

Congratulations, you've just written your first website using pure Kotlin/JS and React!

In this first snippet, you can see the `render` from [kotlin-react-dom](https://github.com/JetBrains/kotlin-wrappers/tree/master/kotlin-react-dom) render a first HTML element inside a [fragment](https://reactjs.org/docs/fragments.html) to an element called `#root`.
This container element is defined in `/src/main/resources/index.html`, which was included in the template.

The content we render is pretty simple - it is just an `<h1>` headline. However, we use Kotlin's typesafe DSL to render this HTML snippet.

### Building typesafe HTML

The Kotlin [wrappers](https://github.com/JetBrains/kotlin-wrappers/blob/master/kotlin-react/README.md) for React come with a [domain-specific language (DSL)](https://kotlinlang.org/docs/type-safe-builders.html) that lets us write HTML in pure Kotlin code.
In this way, it's similar to [JSX](https://reactjs.org/docs/introducing-jsx.html) from the JavaScript world.
However, with this markup being Kotlin, we get all the benefits of a statically typed language -- from autocomplete to type-checking. 

That hopefully means more time to write cool applications, and less time hunting misspelled attributes!

##### About the plus

When you look at the Kotlin snippet above, you may notice the `+` sign in front of our string literal.
It gives you a bit of a hint of how this DSL works under the hood: `h1` is really a function that takes a lambda parameter.
When we write `+`, we are really invoking the function `unaryPlus` (through [operator overloading](https://kotlinlang.org/docs/reference/operator-overloading.html)). That function then takes care of appending the string to the enclosing HTML element.

Simply said, you can think of the `+` as "append my string inside this element."

#### Converting classic HTML

With the pictures shown in the introduction to this tutorial, we already have a pretty good idea of what our website should look like. We can start by translating our (mental) draft from HTML to the Kotlin DSL.

The raw HTML of our page would look something like this:

```xml
<h1>KotlinConf Explorer</h1>
<div>
    <h3>Videos to watch</h3>
    <p>John Doe: Building and breaking things</p>
    <p>Jane Smith: The development process</p>
    <p>Matt Miller: The Web 7.0</p>
    <h3>Videos watched</h3>
    <p>Tom Jerry: Mouseless development</p>
</div>
<div>
    <h3>John Doe: Building and breaking things</h3>
    <img src="https://via.placeholder.com/640x360.png?text=Video+Player+Placeholder">
</div>
```

Let's translate this code into Kotlin. The conversion should be rather straightforward. If you'd like to challenge yourself, you can even try doing it without peeking at the sample solution below!

```kotlin
h1 {
    +"Hello, React+Kotlin/JS!"
}
div {
    h3 {
        +"Videos to watch"
    }
    p {
        +"John Doe: Building and breaking things"
    }
    p {
        +"Jane Smith: The development process"
    }
    p {
        +"Matt Miller: The Web 7.0"
    }

    h3 {
        +"Videos watched"
    }
    p {
        +"Tom Jerry: Mouseless development"
    }
}
div {
    h3 {
        +"John Doe: Building and breaking things"
    }
    img {
        src = "https://via.placeholder.com/640x360.png?text=Video+Player+Placeholder"
    }
}
```

Type or paste the above code as the contents of your `Fragment.create` call inside the `main` function,
replacing our previous `h1` tag.

After saving the file, let's go back to the browser.
We will see a page that looks like this:

![image-20190729143514676](./assets/image-20190729143514676.png)

### Using Kotlin language constructs in markup

There are some useful side effects to writing HTML in Kotlin using this DSL.
For example, we can manipulate our website using regular Kotlin constructs.
We could use loops, conditions, collections, string interpolation, and more.
We'll see how that works in practice in just a bit.

Next, let's replace the hardcoded list of videos with a list of real Kotlin objects.
We'll create a simple data class to hold together the attributes of a video called `Video`.

Add the following snippet to the `Main.kt` file (or another file of our choice):

```kotlin
data class Video(
    val id: Int,
    val title: String,
    val speaker: String,
    val videoUrl: String
)
```

Then, let's fill up the two lists for unwatched videos and watched videos respectively. For now, we can just have these declarations at file-level inside our `Main.kt`:

```kotlin
val unwatchedVideos = listOf(
    Video(1, "Opening Keynote", "Andrey Breslav", "https://youtu.be/PsaFVLr8t4E"),
    Video(2, "Dissecting the stdlib", "Huyen Tue Dao", "https://youtu.be/Fzt_9I733Yg"),
    Video(3, "Kotlin and Spring Boot", "Nicolas Frankel", "https://youtu.be/pSiZVAeReeg")
)

val watchedVideos = listOf(
    Video(4, "Creating Internal DSLs in Kotlin", "Venkat Subramaniam", "https://youtu.be/JzTeAM8N1-o")
)
```

To use these videos in our page, we can now write a Kotlin `for`-loop to iterate over the collection of unwatched videos. Replace the three `p` tags under "Videos to watch" with the following snippet:

```kotlin
for (video in unwatchedVideos) {
    p {
        +"${video.speaker}: ${video.title}"
    }
}
```

We apply the same process to modify the code for the single tag following "Videos watched", as well:

```kotlin
for (video in watchedVideos) {
    p {
        +"${video.speaker}: ${video.title}"
    }
}
```

Wait for the browser to reload. If everything went well, we should see the same layout as before (but now, using proper Kotlin objects behind the scenes!)
To _really_ make sure our loop is actually looping, you can of course add some more videos to the list on your own at this point.

### Writing typesafe CSS

Our little "app" now already has the ability to show watched and unwatched videos in a list.
Unfortunately, it still looks a bit boring.
To fix that, we could include a CSS file in our `index.html` template.
But we can also use this chance to play with another Kotlin DSL - this time for CSS.

The [kotlin-react-css](https://github.com/JetBrains/kotlin-wrappers/tree/master/kotlin-react-css) library allows us to specify CSS attributes – even dynamic ones – right alongside our HTML.
Conceptually, that makes it similar to [CSS-in-JS](https://reactjs.org/docs/faq-styling.html#what-is-css-in-js) – but for Kotlin! Like before, the benefit of using a DSL is that we can use Kotlin code constructs to express our formatting rules.

The template project for this tutorial already includes everything we need to use `kotlin-react-css`.
The relevant block in our build configuration is this:

```kotlin
dependencies {
    //...
    //Kotlin React CSS (chapter 3)
    implementation("org.jetbrains.kotlin-wrappers:kotlin-react-css:17.0.2-pre.298-kotlin-1.6.10")
    //...
}
```

In the previous segment, we used HTML elements like `div` and `h3`. With `kotlin-react-css` in our project, we can also specify a `css` block inside these elements, where we can define our styles.

Let's use CSS to move the video player to the top right corner of the page.
Adjust the code for our mock video player (the last `div` in our snippet) to look like this:

```kotlin
div {
    css {
        position = Position.absolute
        top = 10.px
        right = 10.px
    }
    h3 {
        +"John Doe: Building and breaking things"
    }
    img {
        src = "https://via.placeholder.com/640x360.png?text=Video+Player+Placeholder"
    }
}
```

The example given is very minimalistic. Feel free to experiment with some other styles. For example, you could change the `fontFamily`, or add a splash of `color` to your UI.

You can find the state of the project after this section on the `02-first-static-page` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js-gradle/tree/02-first-static-page) repository.
