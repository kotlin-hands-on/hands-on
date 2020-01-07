# Tests and platform test runners 
In the dependencies, let's make sure we have the following line:

```kotlin
implementation(kotlin("test-js"))
```

We can now fine tune the behaviour.

```kotlin
target {
    browser {
        testTask {
            useKarma {
                useChromeHeadless()
                useFirefox()
            }
        }
    }
}
```

To drive home the point and actually see some tests in action, let's create a file called `src/test/kotlin/AppTest.kt` and fill it with this content:

```kotlin
import kotlin.test.Test
import kotlin.test.assertEquals

class AppTest {
    @Test
    fun thingsShouldWork() {
        assertEquals(listOf(1,2,3).reversed(), listOf(3,2,1))
    }

    @Test
    fun thingsShouldBreak() {
        assertEquals(listOf(1,2,3).reversed(), listOf(1,2,3))
    }
}
```

After running the `browserTest` task via the IDE or via the command line:

```./gradlew browserTest```

We can find a nicely formatted test report in `build/reports/tests/browserTest/index.html`. Let's open the file in a browser of our choice.

![image-20191203213055870](/assets/image-20191203213055870.png)

As expected, one test breaks, giving us a total of 50% successful tests. We can explore the failed tests by navigating through the provided hyperlinks.

![image-20191203213154776](/assets/image-20191203213154776.png)

Feel free to click around and discover things like where to find the standard output or other interesting statistics that might come in handy. Once you fixed the failing test, re-run the tests and find yourself at 100% and colored in a delightful green hue.