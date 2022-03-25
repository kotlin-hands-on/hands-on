# More components

At this point, we know how to create an encapsulated component that can still interact with the rest of the application.
Let's continue splitting our code into components.

### Extracting the video player component

Another naturally self-contained component is the video player (which currently is still a placeholder image).

This time, we can try to identify the props for the video player in advance:
A video player needs to know the talk title, the author of the talk, and the link to the video. 
All this information is already contained in a `Video` object.
We can just pass one of those as a prop, and access its attributes.

Create a new file called `VideoPlayer.kt` and add the following implementation for the `VideoPlayer` component it:

```kotlin
import csstype.*
import react.*
import react.css.css
import react.dom.html.ReactHTML.button
import react.dom.html.ReactHTML.div
import react.dom.html.ReactHTML.h3
import react.dom.html.ReactHTML.img

external interface VideoPlayerProps : Props {
    var video: Video
}

val VideoPlayer = FC<VideoPlayerProps> { props ->
    div {
        css {
            position = Position.absolute
            top = 10.px
            right = 10.px
        }
        h3 {
            +"${props.video.speaker}: ${props.video.title}"
        }
        img {
            src = "https://via.placeholder.com/640x360.png?text=Video+Player+Placeholder"
        }
    }
}
```

Because our `VideoPlayerProps` interface specifies that the `VideoPlayer` component takes a non-null `Video`,
we need to make sure handle this in our `App` component accordingly.

Back in `App.kt`, replace the previous `div` snippet for the video player with our new component:

```kotlin
currentVideo?.let { curr ->
    VideoPlayer {
        video = curr
    }
}
```

By using the [`let` scope function](https://kotlinlang.org/docs/scope-functions.html#let),
we ensure that the `VideoPlayer` component is only added when `state.currentVideo` is not null.

At this point, clicking an entry in the list will bring up the video player,
and populate it with the information from the clicked entry.
Next, let's continue making our application more interactive.

### Adding a button and wiring it

Our application still needs a way to mark a video as (un)watched, to move it between the two lists.
Let's add a button to our `VideoPlayer` component that will do exactly that.

Since this button will move videos between two different lists,
we can already guess that the logic handling the state change will have to *lifted* out of the `VideoPlayer`,
and passed in from the parent as a prop.

Also, we probably want the button to look different based on whether a video has already been watched or not.
This is also information we need to pass as a prop.

Expand the `VideoPlayerProps` interface in `VideoPlayer.kt` to include properties for those two cases:

```kotlin
external interface VideoPlayerProps : Props {
    var video: Video
    var onWatchedButtonPressed: (Video) -> Unit
    var unwatchedVideo: Boolean
}
```

Next, let's add the button to the actual component.
We have some experience building components at this point, so the implementation should not be too surprising.
Add the following snippet to the body of the `VideoPlayer` component, between the `h3` and `img` tags:

```kotlin
button {
    css {
        display = Display.block
        backgroundColor = if (props.unwatchedVideo) NamedColor.lightgreen else NamedColor.red
    }
    onClick = {
        props.onWatchedButtonPressed(props.video)
    }
    if (props.unwatchedVideo) {
        +"Mark as watched"
    } else {
        +"Mark as unwatched"
    }
}
```

This component isn't anything special,
but actually once again shows off the elegance of using Kotlin DSLs:
we use a basic Kotlin `if` expression to change the color of the button dynamically.

### Moving video lists to the application state

With the new `onWatchedButtonPressed` and `unwatchedVideo` props for the `VideoPlayer` component,
we still need to adjust its usage site in the `App` component.
But first, let's think about what it is supposed to do.

When the button is clicked, a video should either be:

- moved from the unwatched list to the watched list, or
- moved from the watched list to the unwatched list.

Lists that can change? That's a prime candidate for more application state!

Back in `App.kt`, add the following `useState` calls to the top of the `App` component:

```kotlin
val App = FC<Props> {
    var currentVideo: Video? by useState(null)
    var unwatchedVideos: List<Video> by useState(listOf(
        Video(1, "Opening Keynote", "Andrey Breslav", "https://youtu.be/PsaFVLr8t4E"),
        Video(2, "Dissecting the stdlib", "Huyen Tue Dao", "https://youtu.be/Fzt_9I733Yg"),
        Video(3, "Kotlin and Spring Boot", "Nicolas Frankel", "https://youtu.be/pSiZVAeReeg")
    ))
    var watchedVideos: List<Video> by useState(listOf(
        Video(4, "Creating Internal DSLs in Kotlin", "Venkat Subramaniam", "https://youtu.be/JzTeAM8N1-o")
    ))
    // . . .
}
```

Since we include all of our demo data in the default values for `watchedVideos` and `unwatchedVideos` directly,
we no longer need the file-level declarations from before.

In `Main.kt`, delete the file-level declarations for `watchedVideos` and `unwatchedVideos`.

Finally, we can return to the task we wanted to do in the first place: changing the call-site for `VideoPlayer`.

Change the code in the `App` component that belongs to the video player to look as follows:

```kotlin
VideoPlayer {
    video = curr
    unwatchedVideo = curr in unwatchedVideos
    onWatchedButtonPressed = {
        if (video in unwatchedVideos) {
            unwatchedVideos = unwatchedVideos - video
            watchedVideos = watchedVideos + video
        } else {
            watchedVideos = watchedVideos - video
            unwatchedVideos = unwatchedVideos + video
        }
    }
}
```

With that, we have implemented the largest part of our application logic.
Go back to the browser, select a video, and press the button a few times.
The video will jump between the two lists.
Sweet!

However, after all this work, our video player is still just a placeholder picture.
But it's time to let others do that kind of heavy lifting.
In the next section, we'll talk about using ready-made React components from `npm` inside our Kotlin/JS app.

You can find the state of the project after this section on the `05-more-components` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js-gradle/tree/05-more-components) repository.
