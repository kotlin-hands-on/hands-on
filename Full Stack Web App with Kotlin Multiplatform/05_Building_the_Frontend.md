# Building the frontend

To render and manage user interface elements, we will use the popular framework [React](https://reactjs.org/) together with the available [wrappers](https://github.com/JetBrains/kotlin-wrappers/) for Kotlin. While for a very small project like this, using a web framework might seem overkill at first glance, it allows us to re-use this project and its configuration as a jumping-off point for future multiplatform applications that might have higher complexity.

Building web applications with React can be a complex topic in itself, so wherever possible, explanations and details relating to the topic are held light. If you'd like to get a more in-depth view typical workflows and how applications can be structured with React and Kotlin/JS, please refer to the [Building Web Applications with React and Kotlin/JS](https://play.kotlinlang.org/hands-on/Building%20Web%20Applications%20with%20React%20and%20Kotlin%20JS/01_Introduction) hands-on.

Before we render our list, however, we first need to obtain some data from the server that we want to display. For this, we'll build a small API client.

### Writing our API client

Our API client serves as a wrapper around the [`ktor-clients`](https://ktor.io/clients/index.html) library, which allows us to send requests to HTTP endpoints. Ktor clients use Kotlin's coroutines to provide non-blocking networking – and just like the ktor server, the clients support *Features*. One feature we will make use of is `JsonFeature`.

In our configuration, the `JsonFeature` makes use of `kotlinx.serialization` to provide us with a way to create typesafe HTTP requests, by taking care of the automatic conversion between Kotlin objects to their JSON representation and vice versa.

Leveraging these properties, we can create our API wrapper as a set of suspending functions that either accept or return `ShoppingItems`. We implement them as follows in `src/jsMain/kotlin`, in a file called `Api.kt`:

```kotlin
import io.ktor.http.*
import io.ktor.client.*
import io.ktor.client.request.*
import io.ktor.client.features.json.JsonFeature
import io.ktor.client.features.json.serializer.KotlinxSerializer

import kotlinx.browser.window

val endpoint = window.location.origin // only needed until https://github.com/ktorio/ktor/issues/1695 is resolved

val jsonClient = HttpClient {
    install(JsonFeature) { serializer = KotlinxSerializer() }
}

suspend fun getShoppingList(): List<ShoppingListItem> {
    return jsonClient.get(endpoint + ShoppingListItem.path)
}

suspend fun addShoppingListItem(shoppingListItem: ShoppingListItem) {
    jsonClient.post<Unit>(endpoint + ShoppingListItem.path) {
        contentType(ContentType.Application.Json)
        body = shoppingListItem
    }
}

suspend fun deleteShoppingListItem(shoppingListItem: ShoppingListItem) {
    jsonClient.delete<Unit>(endpoint + ShoppingListItem.path + "/${shoppingListItem.id}")
}
```

Now, let's use these functions to get a list on the screen!

### Building the user interface

We've laid the groundwork on the client, and have a clean API to access the data that's being provided by the server. We can now work on displaying this data in a React application.

#### Configuring an entry point for our application

Instead of rendering a simple "Hello, Kotlin/JS" string, it's time we make our application render a functional `App` component. We replace the content inside `src/jsMain/kotlin/Main.kt` with the following:

```kotlin
import react.child
import react.dom.render
import kotlinx.browser.document

fun main() {
    render(document.getElementById("root")) {
        child(App)
    }
}
```

#### Building and rendering the shopping list

The next step is to provide an implementation for the `App` component we would like to render. For our shopping list application, we need to

- keep the "local state" of the shopping list (which elements to display),
- load the shopping list elements from the server (and set the state accordingly), and
- provide React with instructions on how to render the list.

Based on these requirements, we can implement the `App` component as follows. We create and fill the file `src/jsMain/kotlin/App.kt`:

```kotlin
import react.*
import react.dom.*
import kotlinext.js.*
import kotlinx.html.js.*
import kotlinx.coroutines.*

private val scope = MainScope()

val App = functionalComponent<RProps> { _ ->
    val (shoppingList, setShoppingList) = useState(emptyList<ShoppingListItem>())

    useEffect {
        scope.launch {
            setShoppingList(getShoppingList())
        }
    }

    h1 {
        +"Full-Stack Shopping List"
    }
    ul {
        shoppingList.sortedByDescending(ShoppingListItem::priority).forEach { item ->
            li {
                key = item.toString()
                +"[${item.priority}] ${item.desc} "
            }
        }
    }
}

```

As is evident by the code, we use the `kotlinx.html` DSL to define the HTML representation of our application, and use `launch` to obtain our list of `ShoppingListItem`s from the API when the component is first initialized. For more details on the inner workings of React, please check out the corresponding [hands-on](https://play.kotlinlang.org/hands-on/Building%20Web%20Applications%20with%20React%20and%20Kotlin%20JS/01_Introduction).

By making use of React hooks – `useEffect` and `useState`, we can use React's functionality in a concise manner. For more information on how React hooks work, please check out the [official React documentation](https://reactjs.org/docs/hooks-overview.html).

If everything has gone according to plan, we can start our application via the Gradle `run` task, and navigate to `http://localhost:9090/` to see our list, nicely formatted:

![image-20200408185054084](./assets/image-20200408185054084.png)

Next, let's allow our users to add new entries to the shopping list, via a text input field.

#### Adding an input field component

In order to receive input from the user, we need an input component which provides us with a callback for when the user submits their entry to the shopping list.

Create the file `src/jsMain/kotlin/InputComponent.kt`, and fill it with the following definition:

```kotlin
import react.*
import react.dom.*
import kotlinx.html.js.*
import kotlinx.html.InputType
import org.w3c.dom.events.Event
import org.w3c.dom.HTMLInputElement

external interface InputProps : RProps {
    var onSubmit: (String) -> Unit
}

val InputComponent = functionalComponent<InputProps> { props ->
    val (text, setText) = useState("")

    val submitHandler: (Event) -> Unit = {
        it.preventDefault()
        setText("")
        props.onSubmit(text)
    }

    val changeHandler: (Event) -> Unit = {
        val value = (it.target as HTMLInputElement).value
        setText(value)
    }

    form {
        attrs.onSubmitFunction = submitHandler
        input(InputType.text) {
            attrs.onChangeFunction = changeHandler
            attrs.value = text
        }
    }
}
```

While an in-depth explanation of how this component works is outside the scope of this hands-on, it can be summarized as follows: The `InputComponent` keeps track of its internal state (what the user has typed so far), and exposes an `onSubmit` handler that gets called when the user submits the form (usually by pressing the `Enter` key).

To use this `InputComponent` from our application. we add the following snippet to `src/jsMain/kotlin/App.kt`,  at the bottom of the `functionalComponent` block (after the closing brace for the `ul` element):

```kotlin
child(
    InputComponent,
    props = jsObject {
        onSubmit = { input ->
            val cartItem = ShoppingListItem(input.replace("!", ""), input.count { it == '!' })
            scope.launch {
                addShoppingListItem(cartItem)
                setShoppingList(getShoppingList())
            }
        }
    }
)
```

As the user submits their text, we create a new `ShoppingListItem`. Its priority is set to be the number of exclamation points in the input, and its description is the input with all exclamation points removed. This turns `Peaches!! 🍑` into a `ShoppingListItem(desc="Peaches 🍑", priority=2)`.

The generated `ShoppingListItem` gets sent to the server via the API client we built before. Lastly, we update the UI by by obtaining the new list of `ShoppingListItem`s from the server, updating our application state, and letting React do its re-rendering magic.

#### Crossing off items

Always adding items without the possibility to remove items that we have gotten during our last trip to the supermarket would sooner or later result in some long lists. Let's add the ability to remove items from our list through the UI.

Rather than add another UI element (like a "delete" button), we can actually make use of our already existing list: when the user clicks one of the items on the shopping list, we delete it. To achieve this, we can just pass a corresponding handler to the `attrs.onClickFunction` of our list elements. In `src/jsMain/kotlin/App.kt`, add the following to the `li` block (inside the `ul` block):

```kotlin
attrs.onClickFunction = {
    scope.launch {
        deleteShoppingListItem(item)
        setShoppingList(getShoppingList())
    }
}
```

We once again invoke our API client with the element that should be removed, and obtain the updated shopping list from the server once more.

Now seems like a good time to give our application another spin. We can start it using the Gradle `run` task once more, navigate to `http://localhost:9090/`, and can try adding and removing elements in the list.

![](/assets/finished.gif)

At this point, we have pretty much finished building our application! The only current problem is that we can't _persist_ our data, meaning that our shopping list is going to vanish the moment we terminate the server process.

In the next chapter, we'll explore how to use MongoDB as an easy way to store and retrieve our shopping list items even after the server gets shut down.

### Relevant Gradle configuration

To provide all the functionality used in this section, we need to include a number of libraries – both from the Kotlin ecosystem as well as from the JavaScript ecosystem (via `npm`). Please refer to the `jsMain` dependency block in the `build.gradle.kts` file to see the full setup.
