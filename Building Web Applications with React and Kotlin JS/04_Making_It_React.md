# Making It React: Our first Component

### General concept

The basic building blocks in React are called *components*. By combining these components, which can in turn also be composed of other smaller components, we build our application. By structuring our components for reusability and keeping them generic, we'll be able to use them in multiple parts of our application without having to duplicate code or logic.

In fact, the "root" node that we are currently rendering is already a React component. If we were to mark this component in our application, it would look something like this:

![image-20190729153736304](/assets/image-20190729153736304.png)

When we structure our application and split it up into components, we end up with a more structured layout, in which each component handles its own responsibilities:

![image-20190729153826920](/assets/image-20190729153826920.png)

Let's begin the process of structuring our application. We’ll start by explicitly specifying the main component we're rendering into the root element: the `App`. Let's create a new file in our project's `src` folder called `App.kt`. Inside this file, let's create our `App` class which inherits from `RComponent`, a **R**eact **Component**. For now, we can keep its generic parameters default (`RProps` and `RState`). There will be more on those later.

```kotlin
import react.*

class App: RComponent<RProps, RState>() {
    override fun RBuilder.render() {
        // typesafe HTML goes here!
    }
}
```

Let's move all of our typesafe HTML into this render function. Now that all of our HTML is safely packed away in its own explicit component, our `main` function within `Main.kt` should reference our `App` instead of rendering its own HTML. The only thing we need to do now is to tell React to render our App component as a `child` of the `root` object.

```kotlin
fun main() {
    render(document.getElementById("root")) {
        child(App::class) {}
    }
}
```

We'll learn more about how React components work as we create them. If you would like to get an in-depth view of how React and its components operate, feel free to check out their [official documentation and guides](https://reactjs.org/docs/hello-world.html#how-to-read-this-guide).

### Extracting a list component

The first part of our application that lends itself to becoming a reusable component is the *video list(s)*. Since both the "watched video list" and "unwatched video list" have the same functionality (displaying a list of videos), it would make sense not to duplicate them and use a reusable component instead.

Let's create a new file called `VideoList.kt`. The `VideoList` class follows the same pattern as our `App` class from before, inheriting from `RComponent`, and containing the code from our `unwatchedVideos` list.

```kotlin
import react.*
import react.dom.*

class VideoList: RComponent<RProps, RState>() {
    override fun RBuilder.render() {
        for (video in unwatchedVideos) {
            p {
                +"${video.speaker}: ${video.title}"
            }
        }
    }
}
```

This means that the video-list part of our `App` could look something like this:

```kotlin
div {
    h3 {
        +"Videos to watch"
    }
    child(VideoList::class) {}

    h3 {
        +"Videos watched"
    }
    child(VideoList::class) {}
}
```

There is a big problem with this implementation though: our `App` has no control over the content of the list. It is still hard-coded, so we end up seeing the same list twice. We need a mechanism to pass the list *into the component*.

### Adding props

When we reuse our list, we will probably want to fill it with different content. This means that instead of having the list of items stored inside our component’s class, it should be controlled externally, and passed in as an attribute for the component. React calls these attributes props. If props change in React, the framework will take care of re-rendering of the page for us.

In our case, we would like to add a prop which contains the list of talks. Let's restructure our code accordingly. Within `VideoList.kt`, let's add an interface definition like this:

```kotlin
interface VideoListProps: RProps {
    var videos: List<Video>
}
```

We now adjust the class definition of `VideoList` to make use of those props:

```kotlin
class VideoList(props: VideoListProps): RComponent<VideoListProps, RState>(props) {
    override fun RBuilder.render() {
        for (video in props.videos) {
            p {
                key = video.id.toString()
                +"${video.speaker}: ${video.title}"
            }
        }
    }
}
```

Because the content of our lists is now potentially dynamic (i.e. during runtime, the attributes passed into our component might change), we should also add a `key` attribute. This helps the React renderer figure out which parts of the list need to refresh, and which ones can stay the same – a nice free optimization for us! You can find more information about lists and keys in the [official React guide](https://reactjs.org/docs/lists-and-keys.html).

Now, at the usage site within `App`, let's make sure that we instantiate the child components with proper attributes. Replace the two loops underneath the `h3` elements with the respective attributes for `unwatchedVideos` and `watchedVideos` like so:

```kotlin
child(VideoList::class) {
    attrs.videos = unwatchedVideos
}
```

A quick check in the browser window will show us that the lists now render as they are supposed to. We have encapsulated the responsibility of rendering a list of videos into its own component. This shortens the `App`’s source code and makes it easier for us and our fellow developers to read and understand.

### Even smoother usage

If we want to, we can make use of a cool feature called [lambdas with receivers](https://kotlinlang.org/docs/reference/lambdas.html#function-literals-with-receiver) to make the usage site of our component nicer. This snippet does exactly that, and it can be easily reused for other components:

```kotlin
fun RBuilder.videoList(handler: VideoListProps.() -> Unit): ReactElement {
    return child(VideoList::class) {
        this.attrs(handler)
    }
}
```

While it's not necessary to understand what's going on in the snippet above, it might still be interesting to know about: it defines a function called `videoPlayer` as an [extension function](https://kotlinlang.org/docs/reference/extensions.html) on `RBuilder`. It takes one parameter, `handler`, which is an extension function on `VideoListProps` returning `Unit`. This function wraps the call to `child` (as we have done before), and passes the `handler` in order to instantiate the `attrs`.

The main point is that it simplifies the usage site a lot more, allowing us to simply write:

```kotlin
videoList {
    videos = unwatchedVideos
}
```

### Making it interactive

The final goal for the list component is to have it control the video that's currently being played in our video player. To do that, the list first needs to allow the user to interact with its elements. We'll start simple, by `alert`ing the video object the user has selected.

To do this, we modify the code inside our `VideoList` `render` function inside the loop to include a handler that `alert`s the current element:

```kotlin
p {
    attrs {
        onClickFunction = {
            window.alert("Clicked $video!")
        }
    }
    +"${video.speaker}: ${video.title}"
}
```

When IntelliJ IDEA prompts us to add imports for this functionality, we can simply use the `Alt-Enter` quickfixes to have the IDE do the work for us.

Now, when we click on one of the list items in the browser window, we get the corresponding information inside an `alert` window:

![image-20190729161705147](/assets/image-20190729161705147.png)

### Adding state

Instead of simply alerting the user, let's make it so that they can actually select a video. We highlight the selected video with a ▶ triangle. To achieve this, we need to introduce some *state* specific to this component. Just as we did with props, we can do it by defining an interface:

```kotlin
interface VideoListState: RState {
    var selectedVideo: Video?
}
```

There are a few things we need to do to add this state:

- We need to adjust the class definition of `VideoList` to use the `VideoListState` – specifically, we inherit from an `RComponent<..., VideoListState>`.
- For the element of the list that is currently selected, we need to prepend a triangle string. We need to modify the `onClickFunction` to set the contents of `selectedVideo` to be equal to the element in our list. And to ensure that our components are re-rendered when we change the `state`, we need to wrap our variable update in the `setState` lambda.


```kotlin
class VideoList(props: VideoListProps) : RComponent<VideoListProps, VideoListState>(props) {
    override fun RBuilder.render() {
        for (video in props.videos) {
            p {
                key = video.id.toString()
                attrs {
                    onClickFunction = {
                        setState {
                            selectedVideo = video
                        }
                    }
                }
                if(video == state.selectedVideo) {
                    +"▶ "
                }
                +"${video.speaker}: ${video.title}"
            }
        }
    }
}
```

```warning
**State should only ever be modified from within the** **`setState`** **lambda**. This allows the React renderer to detect any changes to the state, and to re-render portions of our UI quickly and efficiently.
```

You can find more details about state in the official [React FAQ](https://reactjs.org/docs/faq-state.html).

You can find the state of the project after this section on the `step-03-first-component` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js/tree/step-03-first-component) repository.