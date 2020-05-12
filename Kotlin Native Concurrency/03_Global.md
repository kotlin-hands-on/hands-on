## 3) Global State

Before we get into actual concurrent code, lets chat a bit about global state. Some things in Kotlin can be referenced globally, from any thread. Kotlin/Native state rules apply to everything, so global state has some special rules applied out of the box.

There are two types of global state we need to be concerned with: objects and properties.

### object

All global `object` instances are frozen, but they can be referenced from any thread.

Find `3) Global State` in `SampleMacos.kt` and uncomment `cantChangeThis()`.

```kotlin
fun cantChangeThis(){
    println("i ${DefaultGlobalState.i}")
    DefaultGlobalState.i++
    println("i ${DefaultGlobalState.i}") //We won't get here
}

object DefaultGlobalState{
    var i = 5
}
```

This will compile fine[^1], but all global `object` instances are frozen by default, so you'll see your old friend `InvalidMutabilityException`.

If you need that state to be mutable you do have some options. You can use atomics, which we'll talk about later, but you can also make that object thread local. That means each thread has it's own copy, and according to rule #1, that means it can be mutable.

Run `canChangeThreadLocal()`.

```kotlin
fun canChangeThreadLocal(){
    println("i ${ThreadLocalGlobalState.i}")
    ThreadLocalGlobalState.i++
    println("i ${ThreadLocalGlobalState.i}")
}

@ThreadLocal
object ThreadLocalGlobalState{
    var i = 5
}
```

Notice the annotation `@ThreadLocal`. That tells the runtime that each thread has an instance defined. Running `canChangeThreadLocal()` will print

```
i 5
i 6
```

Keep in mind that if you were accessing `ThreadLocalGlobalState` from another thread, the value of `i` would be different. *Each thread gets it's own copy of an object annotated with `@ThreadLocal`.*

We haven't explained how to leave the main thread yet, but to see how `@ThreadLocal` behaves with multiple threads, run `threadLocalDifferentThreads()`. In the following code, the lambda passed to `background` runs in a different thread:

```kotlin
fun threadLocalDifferentThreads(){
    println("main thread: i ${ThreadLocalGlobalState.i}")
    ThreadLocalGlobalState.i++
    println("main thread: i ${ThreadLocalGlobalState.i}")
    background {
        println("other thread: i ${ThreadLocalGlobalState.i}")
    }
}
```

The result will be:

```
main thread: i 5
main thread: i 6
other thread: i 5
```

Companion objects are also globally referencable, and will also be frozen. You can run `companionAlsoFrozen()` to see this in action.

```kotlin
fun companionAlsoFrozen(){
    println("i ${GlobalStateCompanion.i}")
    GlobalStateCompanion.i++
    println("i ${GlobalStateCompanion.i}") //We won't get here
}

class GlobalStateCompanion{
    companion object {
        var i = 5
    }
}
```

### Properties

Global properties have a relatively strange rule. Global properties are accessible only from the main thread, but are mutable.

To be clear, global *functions* are fine. We're just talking about global *properties*.

Run `globalCounting()`

```kotlin
fun globalCounting(){
    globalCounterData.i++
    println("count ${globalCounterData.i}")
    globalCounterData.i++
    println("count ${globalCounterData.i}")
}

var globalCounterData = SomeMutableData(33)
```

You'll see

```
count 34
count 35
```

To see what happens when you access a global property from a background thread, run `globalCountingFail()`.

```kotlin
data class SomeMutableData(var i:Int)

var globalCounterData = SomeMutableData(33)

fun globalCountingFail(){
    background {
        try {
            globalCounterData.i++
            println("count ${globalCounterData.i}")
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}
```

Running that code results in the following exception:

```
kotlin.native.IncorrectDereferenceException: Trying to access top level value not marked as @ThreadLocal or @SharedImmutable from non-main thread
        at 0   KNConcurrencySamples.kexe   (blah blah blah)
        at 1   KNConcurrencySamples.kexe   (blah blah blah)
(etc)

```

You'll see `IncorrectDereferenceException` less often than `InvalidMutabilityException`, but it's another concurrency related exception. It basically means you're not allowed to touch that state, as in the case here. The code compiles, but the runtime says you can't touch that global property from a background thread.

For native mobile developers, this should be very familiar. You're not allowed to edit the UI state from a background thread. This is again an example of how Kotlin/Native's runtime is simply formalizing a rule we're all generally familiar with.

Like with `object`, you can annotate a global property with `@ThreadLocal`. This will give each thread a copy, and obviously will allow you to access that state from other threads. You can also annotate global properties with `@SharedImmutable`, which will make that property available to all threads, but it will be frozen, like `object` is by default. Run `globalCountingSharedFail()`.

```kotlin
@SharedImmutable
val globalCounterDataShared = SomeMutableData(33)

fun globalCountingSharedFail(){
    globalCounterDataShared.i++
}
```

---

[^1]: Kotlin 1.4 will include better warnings for situations like this, but they will still compile.
