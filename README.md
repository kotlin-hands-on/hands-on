[![official JetBrains project](https://jb.gg/badges/official.svg)](https://confluence.jetbrains.com/display/ALL/JetBrains+on+GitHub)
[![GitHub license](https://img.shields.io/badge/license-Apache%20License%202.0-blue.svg?style=flat)](https://www.apache.org/licenses/LICENSE-2.0)


## Kotlin Hands-On Labs

[Hands-On labs](https://play.kotlinlang.org/hands-on) are interactive tutorials to learn Kotlin.
Each lab corresponds to one or more sample projects, and explains how to create them step-by-step.   
The labs are all provided under Apache 2 and are open to contributions. 

This readme provides more information on how labs are organised and is targeted at contributors.
If you'd like to complete a lab, please use the [website](https://play.kotlinlang.org/hands-on).


### Structure

Each labs consists of:

* Text
* Assets such as images
* Code Samples

Text and assets for all labs are located under this repository under the `lab-name`.
The sample projects corresponding to each lab are separate repositories under the `https://github.com/kotlin-hands-on` organization.  

### Text Structure

Each lab consists of a series of steps.
Each step is represented as an individual markdown file with the naming convention `NN_{step-title}` where
`NN` is the step number and `step-title` is the step title. 

Each hands-on lab should start with an `01_Introduction` file that clearly highlights what the hands-on lab is going to cover.
Ideally it should also show a screenshot of the end result (if this is for instance an application). 

### Style and Formatting

* Use *we* (not *you*) pronoun when referring to the user following the tutorial, i.e. "We first need to click on..." 
* Use bold to highlight UI elements and menu entries. Use | as separators for menu entries, i.e. **File|New|Project...**
* Use `name` notation to refer to folders and files

If you're not a native speaker, but want to contribute, feel free to do that.
We (JetBrains) can proofread the final lab text.

#### Code styles

Support modes: `kotlin` | `js` | `java` | `groovy` | `xml` | `c` | `shell` | `swift` | `obj-c`

```java
`​`​`java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World");
    }
}
`​`​`
```

Runnable modes: `run-kotlin` | `run-kotlin-js`

```kotlin
`​`​`run-kotlin
fun main() {
    println("Hello world")
}
`​`​`
```


#### Promt styles

Support 3 promt modes: `note`, `warning`, `tip`, `todo`.

```
`​`​`note
Lorem Ipsum is simply dummy text of the printing and typesetting industry. 
Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, 
when an unknown printer took a galley of type and scrambled it to make.
`​`​`
```

### Assets

All assets, excluding code, should be placed in a subfolder named `assets` under each lab folder and the asset should be referenced relative to this.

### Videos

The following syntax allows inlining videos in the text:

![Video description](video link) 

### Sample Projects

The code for the lab is located in its own repository using the naming convention `{lab-name}`

The description for the repository should be `{lab-name} Hands-On Lab`. Please make sure you use the appropriate .gitignore file:

```
.gradle/
build/
*.iml
out/
.DS_Store
.idea/
gradle/
```

It should contain a README.md with the following contents:

* License (by default this would be Apache 2.0 License)
* [JetBrains GitHub Label](https://confluence.jetbrains.com/display/ALL/JetBrains+on+GitHub)
* Title which would be `{lab-name} Hands-On Lab`
* The following text:

This repository is the code corresponding to the hands-on lab `{link to hands-on lab}`. 

If the lab describes creating the project step-by-step, the state of the project after each step should be present
in the project and stored in a separate branch or branches. If one branch is enough, it can be called `solutions`
(see [Introduction to Coroutines and Channels](https://play.kotlinlang.org/hands-on/Introduction%20to%20Coroutines%20and%20Channels/01_Introduction) as an example).
If each steps modifies the whole project, intermediate steps should be stored in different branches
(see [Introduction to Kotlin Multiplatform](https://github.com/kotlin-hands-on/intro-kotlin-mutliplatform) as an example).

The lab should reference the project(s) containing the source code:

`You can find the code for the hands-on lab on [GitHub](https://github.com/kotlin-hands-on/{lab-name})` 

If the project contains several branches, the lab text should describe these branches, give the detailed instructions on
installing the project and choosing the correct initial branch. After each step, the correct branch should be referred.

### Automatic compilation and testing of sample projects

We build sample projects at TeamCity to make sure the projects always compile. 
If a project contains several branches with intermediate state, all the branches should be compiled at TeamCity.
The tests in each branch should be checked automatically except special branches that contain failing tests for the readers to fix.

The TeamCity build configurations can be found here:

`https://teamcity.jetbrains.com/project/Kotlin_HandsOnLabs?projectTab=overview&mode=builds`.

### Coding conventions

* Coding conventions should adapt to the [Kotlin Code Style](https://kotlinlang.org/docs/reference/coding-conventions.html). 
* Packages should be named `com.jetbrains.handson.{lab-name}` 

### Contributing

We'll be really happy to get the content contributions from you! 

If you want to contribute and develop a new lab, the best first step is to contact Hadi Hariri or Svetlana Isakova in
[the KotlinLang slack](http://kotlinlang.slack.com/) and share your idea to make sure something similar isn't planned already.
When your idea gets approved, a new repository will be created for you where you can reference the code. 

Feel free to contribute the small changes to existing labs directly to the corresponding projects, and better contact us
before contributing more significant changes to existing labs. 
