# Adding interop to build process

In order to use header files, we need to make sure they are generated as part of the build process. For this, add the `compilations` entry to the `build.gradle` file

```groovy
macosX64("macos") {
        // C Interop support
        compilations.main {
            cinterops {    
                libcurl    
            }              
        }                  

        binaries {
            executable {
                // ...
            }
        }
    }
```

The format is adding `cinterops` and then an entry for each `def` file. By default, if the name of the file is used but all this 
can be overriden with additional parameters

```groovy
libcurl {
    defFile project.file("libcurl.def")
    packageName 'com.jetbrains.handson.http'
    compilerOpts '-I/path'
    includeDirs.allHeaders("path")
}
```

For full details on the options available, see the [documentation](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html#cinterop-support).

Once done, you can now build the project and access the headers from Kotlin as we'll see in the next step. 
