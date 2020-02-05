# A first static page with React

Every good programming story starts with a *Hello World*-derivative – so, let's more from our colored page to exactly that!

Change the code inside `src/Main.kt` to look as follows:

```kotlin
import react.dom.*
import kotlin.browser.document

fun main() {
    render(document.getElementById("root")) {
        h1 {
            +"Hello, React+Kotlin/JS!"
        }
    }
}
```

If you don't have the continous Gradle build (as described in Chapter 2) still running in the background, run the build from your IDE or execute `./gradlew run` from the command line in the project root, and observe the magic happening in your web browser:

![image-20190729142055566](./assets/image-20190729142055566.png)

Congratulations, you've just written your first website using pure Kotlin and React! Let's try to understand what's actually happening here. The render function instructs [kotlin-react-dom](https://github.com/JetBrains/kotlin-wrappers/tree/master/kotlin-react-dom) to render out a component (more on those later) into an element in the website. Going back to the `/src/main/resources/index.html`, we can see that there's a container element called `root`, into which we render our content. The content we render is pretty simple, and uses a typesafe DSL to render HTML.

### Building typesafe HTML

Kotlin's support for *Domain Specific Languages* (DSLs), a feature provided to us through [kotlin-react](https://github.com/JetBrains/kotlin-wrappers/blob/master/kotlin-react/README.md), allows us to describe a markup language like HTML using a syntax that is easy to read (and hopefully also to write) for those familiar with HTML.

By writing pure Kotlin code, we can use all the benefits that a statically typed language brings with it, from autocomplete to type-checking. This way, we can hopefully spend less time in the browser dev tools hunting down a misspelled attribute, and more time making neat applications!

##### About the plus

When you first encounter the pure Kotlin-snippet above, the only thing that really sticks out about the example is the `+` sign in front of our string literal. `h1` is really a function that takes a lambda parameter. When we write `+`, we are really invoking the function `unaryPlus` (through [operator overloading](https://kotlinlang.org/docs/reference/operator-overloading.html)) which takes care of appending the string to the enclosed HTML element.

Simply put, you can think of the `+` as "append my string inside this element."

#### Converting classic HTML

Since we already have a pretty good idea of what our website will look like, we can simply translate our (mental) draft into a Kotlin representation for HTML. If you're comfortable writing simple HTML, you should have no problem getting started with the typesafe variant in Kotlin. We want to build a layout that looks something like this in raw HTML:

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

Let’s translate this code into Kotlin. The conversion should be rather straightforward. If you'd like to test yourself, you can even try doing it without peeking at the sample solution below!

```kotlin
h1 {
    +"KotlinConf Explorer"
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
       attrs {
           src = "https://via.placeholder.com/640x360.png?text=Video+Player+Placeholder"
       }
    }
}
```

Type or paste the above code as the contents of your `render` call. If IntelliJ IDEA complains about missing imports, simply invoke the corresponding quick-fixes using `Alt-Enter`. Once we've saved our file, let's wait for the browser to reload. We will be greeted by a page that looks like this:

![image-20190729143514676](./assets/image-20190729143514676.png)

### Using Kotlin language constructs in markup

While writing HTML in Kotlin just for the sake of it is a noble idea, there are actually a lot more benefits to writing your HTML directly inside Kotlin. A big advantage of using this domain specific language is that we can manipulate our website content using language constructs we are already familiar with. Whether it's conditions, loops, collections, or string interpolation, we can expect them to work the same in HTML as they would in Kotlin.

Now, instead of hardcoding the list of videos, let's actually define them as a list of Kotlin objects and display those objects instead. We'll create a simple class to hold together the attributes of a video (we can do this in `Main.kt` or a file of our choice):

```kotlin
data class Video(val id: Int, val title: String, val speaker: String, val videoUrl: String)
```

Then, let's fill up the two lists for unwatched videos and watched videos respectively. For now, we can just have these declarations at file-level inside our `Main.kt`:

```kotlin
val unwatchedVideos = listOf(
        Video(1, "Building and breaking things", "John Doe", "https://youtu.be/PsaFVLr8t4E"),
        Video(2, "The development process", "Jane Smith", "https://youtu.be/PsaFVLr8t4E"),
        Video(3, "The Web 7.0", "Matt Miller", "https://youtu.be/PsaFVLr8t4E")
)

val watchedVideos = listOf(
        Video(4, "Mouseless development", "Tom Jerry", "https://youtu.be/PsaFVLr8t4E")
)
```

We don't have to learn any kind of special template syntax to pull this information back into our HTML. We can simply write Kotlin code to loop over the collection and add an HTML element for each of them! This means that instead of three `p`-tags, we can now write the following:

```kotlin
for(video in unwatchedVideos) {
    p {
        +"${video.speaker}: ${video.title}"
    }
}
```

We can use the exact same process to modify the code for the `watchedVideos` as well. A short wait and browser reload later, we should see that this change is rendering the equivalent list as above. Feel free to experiment and add more videos to either of the lists to verify that the loop is actually looping.

### Writing typesafe CSS

We have now built our first "feature," but our application, unfortunately, still looks a bit bland and unappealing. While it’s possible to simply include a stylesheet in the template HTML, let’s use this opportunity to play with Kotlin DSLs again – this time for CSS.

[kotlin-styled](https://github.com/JetBrains/kotlin-wrappers/tree/master/kotlin-styled) provides wonderful typesafe wrappers for [styled-components](https://www.styled-components.com/) that allow us to quickly and safely define styles [globally](https://github.com/JetBrains/kotlin-wrappers/blob/master/kotlin-styled/README.md#global-styles) or for individual elements of our DOM. It wraps the styled-components library and allows us to build constructs that look like [CSS-in-JS](https://reactjs.org/docs/faq-styling.html#what-is-css-in-js). Since we are writing pure Kotlin code, we can, for example, express conditions concisely for our formatting rules.

We do not need to do perform any extra steps to start using the functionality, because we have already added the necessary dependencies in our Gradle configuration. The relevant block is:

```kotlin
dependencies {
    //...
    //Kotlin Styled (chapter 3)
    implementation("org.jetbrains:kotlin-styled:1.0.0-pre.90-kotlin-1.3.61")
    implementation(npm("styled-components"))
    implementation(npm("inline-style-prefixer"))
    //...
}
```

Instead of just writing out our HTML elements like `div` or `h3`, we can now use their `styled` counterparts, e.g. `styledDiv` or `styledH3`. This allows us to specify `css` styles in the body. For example, to move the video player to the top right corner of the page, we can adjust the code to look like this:

```kotlin
styledDiv {
    css {
        position = Position.absolute
        top = 10.px
        right = 10.px
    }
    h3 {
        +"John Doe: Building and breaking things"
    }
    img {
        attrs {
            src = "https://via.placeholder.com/640x360.png?text=Video+Player+Placeholder"
        }
    }
}
```

At this point, IntelliJ IDEA will complain about a lot of unresolved references, which we can remedy by adding the appropriate imports to the top of the file:

```kotlin
import kotlinx.css.*
import styled.*
```

or, alternatively, by using the quick-fixes via `Alt-Enter`.

Feel free to style away at the app to your heart's content and experiment around with it – the example given is rather minimalistic. You can even try playing about with the CSS grids to make the app responsive (however, these topics are a little too advanced for this hands-on). Try making the headline use a `fontFamily` that is `sans-serif`, for example, or define some more beautiful `color`s in your code.

You can find the state of the project after this section on the `step-02-first-static-page` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js-gradle/tree/step-02-first-static-page) repository.