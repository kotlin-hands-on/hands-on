# Project Description

In the tutorial we will learn how to create a Kotlin Multiplatform project
to set up code reuse between client side JavaScript application and the
server-side JVM applications. Throughout the tutorial we will
implement the [Mandelbrot Set](https://en.wikipedia.org/wiki/Mandelbrot_set)
rendering algorithm and introduce both server-side and client-side
image rendering.
We'll see how our JVM-only implementation can be turned into a fully functional
Kotlin multiplatform project.


## Basics

Fractals and the [Mandelbrot Set](https://en.wikipedia.org/wiki/Mandelbrot_set)
are good assignments at a computer science classes. All we need to know right now
is that we map every pixel of an image to 2D coordinates (e.g. `0.03, 0.045`)
and run computations to find the color of that pixel. The best picture
is visible at 2D area `[-2.0 .. 2.0] x [-2.0 .. 2.0]`. For example

![](./assets/mandelbrot-full.png)

The main fun and requirement for a Mandelbrot rendering tool is
allow an easy zooming, e.g. 

![](./assets/mandelbrot-zoom1.png)

or 

![](./assets/mandelbrot-zoom2.png)
