# Creating a project

We will be using IntelliJ IDEA Community Edition for this tutorial, though using Ultimate edition is possible as well. The Kotlin plugin 1.3.x or higher should be installed in the IDE. This can be verified via the Language & Frameworks | Kotlin Updates section in the Settings (or Preferences) of the IDE. Native part of this project is written using Mac OS X, but don't worry if you are using another platform, the platform affects only directory names in this particular tutorial.




A multiplatform sample library is now created and imported into IntelliJ IDEA. Let's go to any .kt file and rename the package with the IntelliJ IDEA action Refactor | Rename action to org.jetbrains.base64 Let's just check everything is right with the project so far, the project structure should be:


```
└── src
    ├── commonMain
    │   └── kotlin
    ├── commonTest
    │   └── kotlin
    ├── jsMain
    │   └── kotlin
    ├── jsTest
    │   └── kotlin
    ├── jvmMain
    │   └── kotlin
    ├── jvmTest
    │   └── kotlin
    ├── macosMain
    │   └── kotlin
    └── macosTest
        └── kotlin
```


And the `kotlin` folder should contain an `org.jetbrains.base64` subfolder.
