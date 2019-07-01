# HTML Canvas for Rendering

Now we are ready to update the JS code to implement the `expect`-ed declarations. 
We will be using the [HTML Canvas](https://www.w3schools.com/html/html5_canvas.asp)
on the client-side to render the images.

## Adding Colors

The actual `Color` class for the client-side can be implemented from scratch.
Let's create the `colour-actual.kt` file in the `src/jsMain/kotlin` folder

```kotlin
actual class Color(
        val r: Int,
        val g: Int,
        val b: Int
)
actual fun Colors.newColor(r: Int, g: Int, b: Int) = Color(r, g, b)
actual val Colors.BLACK: Color get() = Color(0, 0, 0)
```

That should be enough to fix compilation of the `jsMain` source set. Now it is
time to support rendering to HTML Canvas

## Adding Canvas Support

Let's add the implementation code to render images into the HTML Canvas. We'll need
to create the `src/jsMain/kotlin/canvas.kt` file with the following contents
```kotlin
fun renderToCanvas(canvas : HTMLCanvasElement, action: (FractalImage) -> Unit) {
  val ctx = canvas.getContext("2d") as CanvasRenderingContext2D
  ctx.clearRect(0.0, 0.0, ctx.canvas.width.toDouble(), ctx.canvas.height.toDouble())

  val imageData = ctx.createImageData(
      ctx.canvas.width.toDouble(),
      ctx.canvas.height.toDouble())

  val image = object:FractalImage {
    override val pixelRect
      get() = Rect(
        left = 0, top = 0,
        right = imageData.width, bottom = imageData.height)

    override fun putPixel(p: Pixel, c: Color) {
      val base = 4 * (p.x + imageData.width * p.y)
      val image : dynamic = imageData.data
      image[base + 0] = c.r
      image[base + 1] = c.g
      image[base + 2] = c.b
      image[base + 3] = 255
    }
  }
  action(image)
  ctx.putImageData(imageData, 0.0, 0.0)
}
```

## Using the Canvas

The very last step is to call the rendering code from the `main.kt` file. 
We add several more lines to the `src/jsMain/kotlin/main.kt` file so that 
the `main()` function will add canvas after `img()` function call:
```kotlin
canvas {
  id = "canvas"
  width = "600"
  height = "600"
}
```

We also add the call to the rendering code at the end of the `main()` function:

```kotlin
renderToCanvas(document.getElementById("canvas") as HTMLCanvasElement) { image ->
  MandelbrotRender.justRender(300, image, MandelbrotRender.initialArea)
}
```

## Running the client-side App

It is now time to run the Gradle `run` task to start the server-side Kotlin/JVM project, and the
`jsRun` task to start the client-side Kotlin/JS application. We should be
able to see two images - the one rendered with Kotlin/JVM on the server-side and the other
one rendered in the browser:

![](./assets/site-full.png)


## Completed Code

We may use the `step-006` branch of the
[github.com/kotlin-hands-on/intro-mpp](https://github.com/kotlin-hands-on/intro-mpp)
repository as the solution of the tasks that we done above. 
We may also just download the
[archive](https://github.com/kotlin-hands-on/intro-mpp/archive/step-006.zip)
from GitHub directly.
   
