# Using platform APIs from Kotlin

A fun element to manipulate in the DOM and to show off the capabilities of the browser APIs in Kotlin is a `<canvas>` element. Let's adjust the contents of `/src/main/resources/index.html` to add a 200x200 canvas to the page.

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello, Kotlin/JS!</title>
</head>
<body>
<canvas width="200" height="200" id="myCanvas"></canvas>
</body>
<script src="handson.js"></script>
</html>
```

We can now write some code that actually interacts with the canvas. For example, we can have some code 

```kotlin
fun main() {
    val canvas = document.getElementById("myCanvas") as HTMLCanvasElement
    val ctx = canvas.getContext("2d") as CanvasRenderingContext2D
    with(ctx) {
        repeat(30) {
            beginPath()
            fillStyle = listOf("red", "green", "blue").random()
            rect(randomCoordinate(), randomCoordinate(), 20.0, 20.0)
            fill()
            closePath()
            console.log("Done drawing a $fillStyle square!")
        }
    }
}

fun randomCoordinate() = Random.nextDouble(0.0, 200.0)
```

We can once again launch our application through the `run` gradle task or via `./gradlew run` from the Gradle wrapper. The website should now greet us with some wonderful mosaic-looking image every time we reload the page.

![img](/assets/img.png)

Now would be a great time to wander off the path of the tutorial, try your own DOM manipulation techniques, or explore some of the APIs with the help of your IDEs autocomplete feature. But there is of course always more to explore.

Stick around if you'd like to see how debugging, tests, and other functionality works in Kotlin/JS with the `kotlin.js` Gradle plugin.