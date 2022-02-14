# Working together: Composing components

The list component we built in the previous section works nicely on its own.
However, there is a small problem: each list keeps track of the selected video on its own.
That means, when we have two lists (one for watched and one for unwatched videos), we can select two videos.
That's not really helpful, since we only have one video player:

![image-20190729172131420](./assets/image-20190729172131420.png)

Clearly, a list can't keep track of both which video is selected inside itself, and inside a sibling list.
Really, the selected video is not part of the _list_ state, but of the _application_ state.
This means we need to *lift* the state out of the individual components.

### Lifting state

React makes sure that props can only be passed from a parent component to its children.
This prevents components from being hard-wired together in strange, criss-cross ways.

If a component wants to change the state of a sibling component, it needs to do so via its parent.
At that point, the state also no longer belongs to any of the child components, but to the overarching parent component.

The process of migrating state from components to their parents is called *lifting state*. So, let's lift!

We add `currentVideo` as state to our `App` component. 

In `App.kt`, add the following to the top of the definition of the `App` component:

```kotlin
val App = FC<Props> {
    var currentVideo: Video? by useState(null)
    // . . .
}
```

Our `VideoList` component does not need to keep track of the state anymore -
it will receive the current video as a prop, instead.
We've essentially just made it stateless, again.

Make sure to remove the `useState` call in `VideoList`.
This is going to break some of our code, which we'll fix in a second.

Next, let's prepare the `VideoList` component to receive the selected video as a prop. 
We expand the `VideoListProps` interface to contain the `selectedVideo`:

```kotlin
external interface VideoListProps : Props {
    var videos: List<Video>
    var selectedVideo: Video?
}
```

We can easily fix the condition for the triangle highlight to use `props` instead of `state`.
Change the triangle-related code to:

```kotlin
if(video == props.selectedVideo) {
    +"▶ "
}
```

The second issue is a little more tricky: We can't assign a value to a prop -
so our `onClick` function won't work the way it currently does.
We need to change the state of a parent component again.
How can we do that?
Well, by lifting some more!

### Passing handlers

In React, state always flows from parent to child.
So, how do we change _application_ state from one of the child components?
We move the logic for handling user interaction to the parent component.
We then pass the logic in as a prop. Remember that in Kotlin, variables can have the [type of a function](https://kotlinlang.org/docs/reference/lambdas.html#function-types).

We can expand the `VideoListProps` interface so that it contains a variable `onSelectVideo`,
which is a function taking a `Video` and returning `Unit`:

```kotlin
external interface VideoListProps : Props {
    var videos: List<Video>
    var selectedVideo: Video?
    var onSelectVideo: (Video) -> Unit
}
```

In our `VideoList` component, we now use the new prop in the `onClick` handler:

```kotlin
onClick = {
    props.onSelectVideo(video)
}
```

With this part out of the way, we can go back to our `App` component.
Here, we pass the `selectedVideo`, and a handler for `onSelectVideo` for each of our two video lists:

```kotlin
VideoList {
    videos = unwatchedVideos
    selectedVideo = currentVideo
    onSelectVideo = { video ->
        currentVideo = video
    }
}
```

Repeat the snippet above for the "watched videos" list as well.

With these changes being applied, we can switch back to the browser and inspect the behavior of the app.
When selecting a video in either of the two lists, the selection now jumps properly, without duplication.
Elegant!

You can find the state of the project after this section on the `04-composing-components` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js-gradle/tree/04-composing-components) repository.
