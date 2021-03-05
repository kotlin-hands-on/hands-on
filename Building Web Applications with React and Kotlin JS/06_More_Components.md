# More components

Now that we have learned how to create a separate, encapsulated component that can still interact with the rest of the application, let's do the same for the remaining component(s).

### Extracting the video player component

Another element that lends itself to becoming a self-contained unit is our video player, currently still visualized by a placeholder image. This time, we can try to identify the props for the video player in advance. Author, talk title, and video link need to be passed to the `VideoPlayer` component. This information is already contained within a `Video` object, so we can pass it as a prop and use its attributes accordingly. When we write the whole component including its respective interfaces, we should end up with a file called `VideoPlayer.kt` that contains the following:

```kotlin
import kotlinx.css.*
import kotlinx.html.js.onClickFunction
import react.*
import react.dom.*
import styled.*

external interface VideoPlayerProps : RProps {
    var video: Video
}

@JsExport
class VideoPlayer : RComponent<VideoPlayerProps, RState>() {
    override fun RBuilder.render() {
        styledDiv {
            css {
                position = Position.absolute
                top = 10.px
                right = 10.px
            }
            h3 {
                +"${props.video.speaker}: ${props.video.title}"
            }
            img {
                attrs {
                    src = "https://via.placeholder.com/640x360.png?text=Video+Player+Placeholder"
                }
            }
        }
    }
}

fun RBuilder.videoPlayer(handler: VideoPlayerProps.() -> Unit): ReactElement {
    return child(VideoPlayer::class) {
        this.attrs(handler)
    }
}
```

To play nice with the compiler and its never-ending quest for `null`-safety, our usage site in `App.kt`, replacing the previous snippet, looks like this – replacing the previous `styledDiv` that contained the video player:

```kotlin
state.currentVideo?.let { currentVideo ->
    videoPlayer {
        video = currentVideo
    }
}
```

Finishing up, this means that the video player will only be shown if the `currentVideo` in our application's state is set.

### Adding a button and wiring it

So far, we haven't really had a way to move our videos from the list of unwatched videos to the list of watched videos (and vice versa). Let's add a button to our `VideoPlayer` component that will do exactly that.

The fact that we want to move things between two different lists (an effect that happens outside the `VideoPlayer` component) suggests that once again, the handler for our button needs to be *lifted* and passed in from the outside.

Since we want the button to have different text depending on whether the video has been watched or not, we need to give the button some more information to work with – specifically, the status of the video passed in. So, we expand our `VideoPlayerProps` interface:

```kotlin
external interface VideoPlayerProps : RProps {
    var video: Video
    var onWatchedButtonPressed: (Video) -> Unit
    var unwatchedVideo: Boolean
}
```

After we’ve built the previous components, the implementation of this button shouldn't come as a big surprise now. A nice thing to note: we can use props to adjust CSS properties! In this case, we color the button dynamically based on the state of the video. Add the following HTML DSL snippet to the `render` function of `VideoPlayer`, between the `h3` and `img` tags.

```kotlin
styledButton {
    css {
        display = Display.block
        backgroundColor = if(props.unwatchedVideo) Color.lightGreen else Color.red
    }
    attrs {
        onClickFunction = {
            props.onWatchedButtonPressed(props.video)
        }
    }
    if(props.unwatchedVideo) {
        +"Mark as watched"
    }
    else {
        +"Mark as unwatched"
    }
}
```

### Moving video lists to the application state

Before we adjust the usage site for the `VideoPlayer`, let's think about what it is supposed to do.

When the button is clicked, a video should either be:

- moved from the unwatched list to the watched list, or
- moved from the watched list to the unwatched list.

Now that these lists can actually change, it's time to move them into our application state! Again, the interface gets expanded with a couple more lines:

```kotlin
external interface AppState : RState {
    var currentVideo: Video?
    var unwatchedVideos: List<Video>
    var watchedVideos: List<Video>
}
```

We can fill the state with some predefined values from within the `init` method. We do this by adding an override to the body of our `App` class:

```kotlin
override fun AppState.init() {
    unwatchedVideos = listOf(
            Video(1, "Building and breaking things", "John Doe", "https://youtu.be/PsaFVLr8t4E"),
            Video(2, "The development process", "Jane Smith", "https://youtu.be/PsaFVLr8t4E"),
            Video(3, "The Web 7.0", "Matt Miller", "https://youtu.be/PsaFVLr8t4E")
    )
    watchedVideos = listOf(
            Video(4, "Mouseless development", "Tom Jerry", "https://youtu.be/PsaFVLr8t4E")
    )
}
```

We can delete the original file-level declarations for `unwatchedVideos` and `watchedVideos` in `Main.kt`, and follow the compiler errors to replace all references to (`un`)`watchedVideos` in the `videoList` invocations and replace them with `state.`(`un`)`watchedVideos` inside `App.kt`. Now, writing the `videoPlayer` call site is rather simple:

```kotlin
videoPlayer {
    video = currentVideo
    unwatchedVideo = currentVideo in state.unwatchedVideos
    onWatchedButtonPressed = {
        if (video in state.unwatchedVideos) {
            setState {
                unwatchedVideos -= video
                watchedVideos += video
            }
        } else {
            setState {
                watchedVideos -= video
                unwatchedVideos += video
            }
        }
    }
}
```

Go back to your browser, select a video, hit the button a few times, and watch the element jump between the two lists!

Just like that, we've now implemented the largest chunk of the custom logic for our application. Feel free to play around with the styles for the button, adjust it to your heart's content, and maybe even try extracting the button as its own reusable component!

Now it's time to kick back and let others do the heavy lifting. Let's talk about using ready-made and freely available React components from within Kotlin.

You can find the state of the project after this section on the `step-05-more-components` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js-gradle/tree/step-05-more-components) repository.
