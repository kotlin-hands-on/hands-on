# Adding interop to build process

In order to use header files, we need to make sure they are generated as part of the build process. For this, add the following entry to the `build.gradle` file

<div class="highlight-snippet" mode="groovy" theme="idea">

```groovy
macosX64("macos") {
        compilations.main { // NL
            cinterops {     // NL
                libcurl     // NL
            }               // NL
        }                   // NL
        binaries {
            executable {
                // ...
            }
        }
    }
```

</div>

The new lines added are marked with `// NL`. The format is adding `cinterops` and then an entry for each `def` file. By default, if the name of the file is used but all this 
can be overriden with additional parameters

<div class="highlight-snippet" mode="groovy" theme="idea">

```groovy
libcurl {
    defFile project.file("libcurl.def")
    packageName 'com.jetbrains.handson.http'
    compilerOpts '-I/path'
    includeDirs.allHeaders("path")
}
```

</div>

For full details on the options available, see the [documentation](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html#cinterop-support).

Once done, you can now build the project and access the headers from Kotlin as we'll see in the next step. 
