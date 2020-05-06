# Addendum: Modern React with Hooks

React 16.8 introduced [Hooks](https://reactjs.org/docs/hooks-intro.html) as a novel way of using state and other React features without writing a class. The great news is that the Kotlin wrappers for React also supports working with Hooks!

To give you an idea about this new way of writing React components, as well as how these concepts translate into Kotlin, we'll take a look at a few isolated examples of the two most commonly used Hooks that are built into React – the *State* and *Effect* Hooks. Just like all other hooks, these two are used in the context of functional components.

### Function components

Conceptually, React's function components themselves aren't very complex. They simply contain the instructions of how to render a React component in a single function based on the properties passed in, and don't rely on a surrounding class structure. A very simple function component could look something like this in Kotlin:

```kotlin
external interface WelcomeProps: RProps {
    var name: String
}

val welcome = functionalComponent<WelcomeProps> { props -> 
    h1 {
        +"Hello, ${props.name}"
    }
}
```

Just like with class components, we define our properties as an `external interface`. We pass it to our `functionalComponent` builder as a generic argument. Inside the builder, we can write code that's very similar to our `render` function in class components – we have access to `RBuilder` functions, and can use the passed in `props`.

Similar to how we use class components, we can use the `child` function to add this `welcome` component to a parent component:

```kotlin
child(welcome) {
    attrs.name = "Kotlin"
}
```

If we find this call site too cumbersome, we can of course create an `RBuilder` extension function  (just like we did in chapter 4)  which takes a handler function, allowing us to write `welcome { name = "Kotlin" }` whenever we are using the component:

```kotlin
fun RBuilder.welcome(handler: WelcomeProps.() -> Unit) = child(welcome) {
    attrs {
        handler()
    }
}
```

However, just like this, function components are somewhat constrained. They only really start unleashing their full potential when combined with Hooks.

### The State Hook

We can use the State Hook when we want to keep track of state without the need for a class component. Let's consider the following implementation of a counter inside a `functionalComponent` for a moment:

```kotlin
val counter = functionalComponent<RProps> {
    val (count, setCount) = useState(0)
    button {
        attrs.onClickFunction = { setCount(count + 1) }
        +count.toString()
    }
}
```

There are a few key properties to note about this snippet that relate to the use of the State hook:

- `useState` is called with an initial value `0` – meaning the type is automatically inferred to be `Int`. We could have also specified the type explicitly, which is especially useful when working with nullable values (`useState<String?>(null)`).
- Calling `useState` returns a pair (that is directly destructured):
  1. a reference to our current state (here, `count` of type `Int`), and
  2. a function that can be used to set the state (here, `setCount` of type `RSetState<Int>`).
- Unlike properties in a React class component, the `setCount` function can be called without having to use a `setState` lambda.

React's inner workings take care of making sure the lifecycle is handled properly: it automatically makes sure that the count variable is only initialized with its initial value once, and that subsequent renderings use the correct updated state. This makes our life as users of the framework somewhat easier – we don't have to worry about providing separate initializer functions, for example.

To learn more about the State Hook, check out the [official React documentation](https://reactjs.org/docs/hooks-state.html).

### The Effect Hook

We can use the Effect Hook when we want to perform side effects in a functional component – like querying an API or establishing a connection. Consider the following implementation of a `functionalComponent` which `fetch`es a random fact and displays it in an `h3` tag:

```kotlin
val randomFact = functionalComponent<RProps> {
    val (randomFact, setRandomFact) = useState<String?>(null)
    useEffect(listOf()) {
        GlobalScope.launch {
            val fortyTwoFact = window.fetch("http://numbersapi.com/42").await().text().await()
            setRandomFact(fortyTwoFact)
        }
    }
    h3 { +(randomFact ?: "Fetching...") }
}
```

To keep track of the API result, we make use of the State Hook which we introduced in the previous section. We then call `useEffect`, run the call to our API, and use the `setRandomFact` function provided to us by the State Hook to update the stored result.

Note that `useEffect` is called with its first parameter, it's _dependencies_. Dependencies simply describe the props or state which needs to change in order for the Effect Hook to be run again. In our case, we only want to make the call to the external API once, regardless of other state changes in our application, so we pass an `emptyList()`.

If we omitted `emptyList()`, our Effect Hook would be called after each invocation of `setRandomFact`, resulting in an endless loop.

To learn more about these intricacies and other intricacies of the Effect Hook, as well as how it relates to the "classical" React lifecycle, check out the [official React documentation](https://reactjs.org/docs/hooks-effect.html).

### Homework: Trying out Hooks

If you want, you can try converting some of the components we have in our application – like the `videoList` – to a functional component that uses the new Hooks API. While the `useState` hook will likely come in handy for almost all components, the `useEffect` hook will be especially important for interacting with external APIs, as we have implemented in chapter 8.

Class-based components and function components can happily coexist in the same React application – so you can also consider writing a _new_ functional component and adding it to the application.