## 5) Debugging

### InvalidMutabilityException

The issue you will most likely encouter is `InvalidMutabilityException`, when attempting to edit frozen state. Fixing this means figuring out how the state was frozen. The Kotlin/Native runtime provides a function `ensureNeverFrozen()` which can be called on any object. If called on a frozen object, it will fail immediately. If called on an unfrozen object, if that object is later being frozen, the freezing process will fail.

See `ensureNeverFrozenFailNow()` and `ensureNeverFrozenFailLater()` for examples.

```kotlin
fun ensureNeverFrozenFailNow() {
    val sd = SomeData("a", 1)
    sd.freeze()
    sd.ensureNeverFrozen() //This will fail
}

fun ensureNeverFrozenFailLater() {
    val sd = SomeData("a", 1)
    sd.ensureNeverFrozen()
    sd.freeze() //This will fail
}
```

If you recall, we said the pattern that has emerged for concurrency functions lambda arguments that get frozen and run on another thread. That will catch and freeze whatever data is captured in the lambda, and can be the cause of something getting frozen unintentionally. By calling `ensureNeverFrozen()` on an object before that happens, you can see what the cause is.

In `ensureNeverFrozenBackground()`, the call to `background` will fail because freezing the lambda argument fails.

```kotlin
fun ensureNeverFrozenBackground() {
    val sd = SomeData("a", 1)
    sd.ensureNeverFrozen()
    background {
        println("Won't get here $sd")
    }
}
```

Going back to `captureTooMuch()`, we get the `InvalidMutabilityException` when we try to increment, but that's not super helpful. At that point it's too late. We want to know **when and how** our data got frozen in the first place. Obviously, in the `captureTooMuch()` example, it's not hard to visually track down the issue, but in more complex examples it will be.

Run `captureTooMuchInit()`. The class `CountingModelEnsure` is a model that we want to make sure is never frozen, so we call `ensureNeverFrozen()` in the init block. This can be good practice for objects you're sure should never be frozen, especially during early development of a project.

```kotlin
fun captureTooMuchInit() {
    val model = CountingModelEnsure()
    model.increment()
    println("I have ${model.count}") //We won't even get here
}

class CountingModelEnsure{

    init {
        ensureNeverFrozen()
    }

    var count = 0

    fun increment(){
        count++
        background {
            saveToDb(count)
        }
    }

    private fun saveToDb(arg:Int){
        //Do some db stuff
        println("Saving $arg to db")
    }
}
```

That will get us an exception that traces right back to the root issue:

```
Uncaught Kotlin exception: kotlin.native.concurrent.FreezingException: freezing of sample.CountingModelEnsure.$increment$lambda-0$FUNCTION_REFERENCE$8@a1e01d98 has failed, first blocker is sample.CountingModelEnsure@a1e01d18
        at 0   KNConcurrencySamples.kexe           (...)
        at 1   KNConcurrencySamples.kexe           (...)
        
        (skip a few lines...)
        
        at 13  KNConcurrencySamples.kexe           0x000000010604208b kfun:sample.background(kotlin.Function0<kotlin.Unit>) + 363 ([path]/kotlin/sample/BackgroundSupport.kt:10:25)
        at 14  KNConcurrencySamples.kexe           0x0000000106043182 kfun:sample.CountingModelEnsure.increment() + 258 ([path]/kotlin/sample/Debugging.kt:42:9)
        at 15  KNConcurrencySamples.kexe           0x0000000106042e50 kfun:sample.captureTooMuchInit() + 272 ([path]/kotlin/sample/Debugging.kt:28:11)
        at 16  KNConcurrencySamples.kexe           0x0000000106043b64 kfun:sample.main() + 36 ([path]/kotlin/sample/SampleMain.kt:37:5)
        at 17  KNConcurrencySamples.kexe           0x0000000106043c2a Konan_start + 138 ([path]/kotlin/sample/SampleMain.kt:3:1)
        (etc...)        
```

The stack trace is a bit verbose, but you'll be able to walk back to the source issue.

As an aside, this is probably a good time to mention that calling `freeze()` may actually fail, and `ensureNeverFrozen()` is why. This may be something that pops up, as `freeze()` is generally assumed to work without issue.

### Other Issues

In the normal course of events, assuming you're not using `Worker` directly or trying to transfer mutable state between threads, there aren't many other exceptions you'll run into that relate to concurrency. `IncorrectDereferenceException` is likely the only one you'll encounter. This will occasionally come up when you use a library or some new shared code that was tested on the main thread, but you run it on a background thread for the first time.

The real risk is moving unfrozen state between threads outside of Kotlin/Native. For example, in the following iOS example (this code is **not** in the sample application), we have a function that takes a param of `SomeData`, a data class. Creating that in Swift, unfrozen, then calling into a Kotlin function from a different thread, throws an exception.

#### Kotlin code

```kotlin
fun callFromOutside(sd: SomeData){
    println("Passed $sd")
}

data class SomeData(val s:String)
```

#### Swift code

```swift
//Currently in main thread
let sd = SomeData(s: "ttt")
DispatchQueue.global(qos: .background).async {
  BreedModelKt.callFromOutside(sd: sd)
}
```

Calls like this will also result in `IncorrectDereferenceException`. The `SomeData` instance is unfrozen and mutable, so being accessed from multiple threads is not OK. You'll need to freeze it before crossing threads, either in Kotlin or by exposing a method to freeze data to Swift. The same rules will apply if integrating Kotlin/Native and any native platform code, including C interop code.

If you're crossing threads in native code, you need to be very careful with what state is touched by different threads, and any of that needs to be frozen.