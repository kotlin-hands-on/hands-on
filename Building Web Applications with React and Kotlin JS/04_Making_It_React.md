# Making It React: Our first Component

### General concept

The basic building blocks in React are called *[components](https://reactjs.org/docs/components-and-props.html)*.
Components themselves can also be composed of other, smaller components. We combine them to build our application.

If we structure our components to be generic and reusable, we can use them in multiple parts of our app. That means we prevent code duplication.

The content of our `render` function pretty much already describes a very basic component. If we mark that component on a UI screenshot, it would look like this:

![image-20190729153736304](./assets/image-20190729153736304.png)

By decomposing our application into individual components, we end up with a more structured layout. In that case, each component handles its own responsibilities:

![image-20190729153826920](./assets/image-20190729153826920.png)

Let's begin the process of structuring our application. We’ll start by explicitly specifying the main component we're rendering into the root element: the `App`.

Create a new file `src/main/kotlin/App.kt`. Inside this file, add the following snippet, and move the typesafe HTML from `Main.kt` into the snippet:

```kotlin
import kotlinx.coroutines.async
import react.*
import react.dom.*
import kotlinx.browser.window
import kotlinx.coroutines.*
import kotlinx.serialization.decodeFromString
import kotlinx.serialization.json.Json
import react.css.css
import csstype.Position
import csstype.px
import react.dom.html.ReactHTML.h1
import react.dom.html.ReactHTML.h3
import react.dom.html.ReactHTML.div
import react.dom.html.ReactHTML.p
import react.dom.html.ReactHTML.img

val App = FC<Props> {
    // typesafe HTML goes here, starting with the first h1 tag!
}
```

In this snippet, we're using the `FC` function to create a [**f**unction **c**omponent](https://reactjs.org/docs/components-and-props.html#function-and-class-components).
We'll talk more about the generic parameter `Props` later.

Our user interface is now packed away in a function component!
Let's make our `main` function render it.
Change your `Main.kt` file to look as follows:

```kotlin
fun main() {
    val container = document.getElementById("root") ?: error("Couldn't find root container!")
    render(App.create(), container)
}
```

With that, we have finished extracting our first component, and have instructed our program to create an instance of the `App` component and render it to the container we specified. 
We'll learn a bit more about how React components work as we create them.

For a more in-depth view of React's concepts, feel free to check out their [official documentation and guides](https://reactjs.org/docs/hello-world.html).

### Extracting a list component

Looking at our interface, we can identify a reusable component: the video list(s).
Both the "watched" and "unwatched" video lists have the same functionality: they display a list of videos. Instead of duplicating their logic, we can turn them into a reusable component instead, and use that component twice.

The `VideoList` component follows the same pattern as the `App` component. It uses the `FC` builder function, and contains the code from our `unwatchedVideos` list.

Create a new file called `VideoList.kt` and add the following code:

```kotlin
import kotlinx.browser.window
import react.*
import react.dom.*
import react.dom.html.ReactHTML.p

val VideoList = FC<Props> {
    for (video in unwatchedVideos) {
        p {
            +"${video.speaker}: ${video.title}"
        }
    }
}
```

In `App.kt`, we could then use the `VideoList` component by invoking the component without parameters:

```kotlin
// . . .
div {
    h3 {
        +"Videos to watch"
    }
    VideoList()

    h3 {
        +"Videos watched"
    }
    VideoList()
}
// . . .
```

Of course, this has one big problem.
The `App` component has no control over the content that is shown by the `VideoList` component.
It is hard-coded, so we see the same list twice.

Instead, we need a mechanism to pass the list *into the component*.

### Adding props

To properly reuse our `VideoList` component, we need to be able to fill it with different types of content.
To do so, we need to be able to pass the list of items as an attribute to the component.
In React, these attributes are called _props_.

The magic behind React is that when the props of a component change, the framework automatically takes care of re-rendering the component.

For our `VideoList`, we want to add a prop containing the list of videos to be shown.
To do so, let's define an `external interface` that holds all the props which can be passed to a `VideoList` component.

Add the following definition to your `VideoList.kt` file:

```kotlin
external interface VideoListProps : Props {
    var videos: List<Video>
}
```

We now adjust the class definition of `VideoList` to make use of the `props`, which are passed into the `FC` block as a parameter:

```kotlin
val VideoList = FC<VideoListProps> { props ->
    for (video in props.videos) {
        p {
            key = video.id.toString()
            +"${video.speaker}: ${video.title}"
        }
    }
}
```

You may wonder about the `key` attribute, which is also new in this snippet.
This attribute helps the React renderer figure out what it needs to do when the value of `props.videos` changes.
It uses the key to determine _which parts_ of a list need to refresh, and which ones stay the same. This small optimization is a best practice when working with lists and React. 
For more information about lists and keys, refer to the [official React guide](https://reactjs.org/docs/lists-and-keys.html).

In our `App` component, let's make sure we instantiate the child components with proper attributes.

In `App.kt`, replace the two loops underneath the `h3` elements with an invocation of `VideoList` together with the attributes for `unwatchedVideos` and `watchedVideos`. In the Kotlin DSL, you assign them inside a block belonging to the `VideoList` component, like so:

```kotlin
VideoList {
    videos = unwatchedVideos // repeat with watchedVideos
}
```

We can have a quick look in the browser to confirm that the lists render correctly now.

We now have a reusable component that can render a list of videos.
The code for our `App` component got smaller, and we got rid of some duplication in our code. Nice!

### Making it interactive

A static list of video titles doesn't help our users much, though.
Eventually, when the user clicks on a list entry, our video player should actually play that video.
Let's start working towards that. 
We'll start simple: When the user selects a video from the list, we'll pop open an `alert`.

In the code of our `VideoList`, add an `onClick` handler function that triggers an alert with the current video:

```kotlin
// . . .
p {
    key = video.id.toString()
    onClick = {
        window.alert("Clicked $video!")
    }
    +"${video.speaker}: ${video.title}"
}
// . . .
```

Let's go back to the browser, and check that our code works.
When we click on one of the list items in the browser window, we get the corresponding information inside an `alert` window:

![image-20190729161705147](./assets/image-20190729161705147.png)

```note
Defining an `onClick` function directly as a lambda is concise and very useful for prototyping. However, [due to the way equality currently works in Kotlin/JS](https://youtrack.jetbrains.com/issue/KT-15101), it is not the most performance-optimized way of passing click handlers. If you want to optimize rendering performance, consider storing your functions in a variable and passing them.
```

### Adding state

Let's add actual selection functionality next, instead of just alerting the user. We'll highlight the selected video with a "▶" triangle.

To achieve this, we need to introduce some *state* specific to this component.
Next to props, state is one of the other core concepts in React.
In modern React (using the so-called Hooks API), state is expressed using the [`useState` hook](https://reactjs.org/docs/hooks-state.html). 

Add the following line of code to the top of the `VideoList` declaration:

```kotlin
val VideoList = FC<VideoListProps> { props ->
    var selectedVideo: Video? by useState(null)
// . . .
```

Even though this is just one new line of code, it introduces a few more advanced features to our code.
Let's briefly talk about them.

On the top level, this line of code specifies the following:
Our functional component `VideoList` keeps state (a value that is independent of the current function invocation).
The state is nullable, and of type `Video?`.
Its default value is `null`.

By using the somewhat magical `useState` function from the React framework,
you instruct the framework to keep track of state across multiple invocations of the function.
For example, even though we specify a default value,
React makes sure that the default value is only assigned in the beginning.
When the state changes, the component will re-render based on the new state.

The `by` keyword indicates that `useState` acts as a [delegated property](https://kotlinlang.org/docs/delegated-properties.html). Just like any other variable, you read and write values.
The implementation behind `useState` takes care the machinery required to make state work.

To learn more about the State Hook, check out the [official React documentation](https://reactjs.org/docs/hooks-state.html).

With this new knowledge, let's continue our modification of the `VideoList` component:
- When the user clicks a video, we need to assign its value to the `selectedVideo` variable.
- When we render the list entry that is currently selected, we need to prepend the triangle.

Change your implementation of `VideoList` to look as follows:

```kotlin
val VideoList = FC<VideoListProps> { props ->
    var selectedVideo: Video? by useState(null)
    for (video in props.videos) {
        p {
            key = video.id.toString()
            onClick = {
                selectedVideo = video
            }
            if (video == selectedVideo) {
                +"▶ "
            }
            +"${video.speaker}: ${video.title}"
        }
    }
}
```

You can find more details about state in the official [React FAQ](https://reactjs.org/docs/faq-state.html).

Let's check back in the browser, and click an item in the list. Looks like everything works... almost!

You can find the state of the project after this section on the `03-first-component` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js-gradle/tree/03-first-component) repository.
