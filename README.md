[![official JetBrains project](https://jb.gg/badges/official.svg)](https://confluence.jetbrains.com/display/ALL/JetBrains+on+GitHub)
[![GitHub license](https://img.shields.io/badge/license-Apache%20License%202.0-blue.svg?style=flat)](https://www.apache.org/licenses/LICENSE-2.0)


## Kotlin Hands-On Labs

[Hands-On labs](https://play.kotlinlang.org/hands-on) are interactive tutorials to learn Kotlin.
Each lab corresponds to one or more sample projects, and explains how to create them step-by-step.   
The labs are all provided under the Apache 2.0 license and are open to contributions. 

This readme provides more information on how labs are organised and is targeted at contributors.
If you'd like to complete a lab, please use the [website](https://play.kotlinlang.org/hands-on).


### Structure

Each labs consists of:

* Text
* Assets such as images
* Code samples

Text and assets for all labs are located under this repository under the `lab-name`.
The sample projects corresponding to each lab are separate repositories under the `https://github.com/kotlin-hands-on` organization.  

### Hands-On Initiator Command Line tool

There's a [command line tool](https://github.com/kotlin-hands-on/hands-on-init) that can make the process of
initializing everything much easier.


### Text Structure

Each lab consists of a series of steps.
Each step is represented as an individual markdown file with the naming convention `NN_{step-title}` where
`NN` is the step number and `step-title` is the step title. 

Each hands-on lab should start with an `00_description.md` file which is the text used for the card displayed on the
list of tutorials, and a `01_Introduction.md` file that clearly highlights what the hands-on lab is going to cover.
Ideally it should also show a screenshot of the end result (if this is for instance an application).

### Style and Formatting

* Use *we* (not *you*) pronoun when referring to the user following the tutorial, i.e. "We first need to click on..." 
* Use bold to highlight UI elements and menu entries. Use | as separators for menu entries, i.e. **File|New|Project...**
* Use `name` notation to refer to folders and files
* Use plain language. Avoid fancy and elaborate words. 
* Use simple sentences. Remove words if they don’t affect the meaning.
* Use keywords that users can relate to. 
* Put statements in the positive form.
* Avoid jargon and slang.
* Avoid nested structure.
* Avoid acronyms (e.g., i.e., a.k.a., etc.).
* Use Oxford comma (serial comma) - the final comma in a list of things. Example: _Running, testing, and packaging._

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


#### Prompt styles

You can use four different prompt modes: `note`, `warning`, `tip`, `todo`.

```
`​`​`note
Lorem Ipsum is simply dummy text of the printing and typesetting industry. 
Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, 
when an unknown printer took a galley of type and scrambled it to make.
`​`​`
```

### Assets

All assets, excluding code, should be placed in a subfolder named `assets` under each lab folder. 
The asset should be referenced relative to this:

```
![](./assets/path/ImageName.png)
```

### Videos

The following syntax allows inlining videos in the text:

```
![Video description](video link) 
```

### Sample Projects

The code for the lab is located in its own repository in
[https://github.com/kotlin-hands-on](https://github.com/kotlin-hands-on) organization using the naming convention `{lab-name}`

Please make sure you use the appropriate `.gitignore` file:

```
.gradle/
gradle/
*.iml
.idea/
build/
out/
.DS_Store
```

It should contain a README.md with the following contents:

* License (by default this would be Apache 2.0 License)
* [JetBrains GitHub Label](https://confluence.jetbrains.com/display/ALL/JetBrains+on+GitHub)
* The following text:

This repository is the code corresponding to the hands-on lab `{link to hands-on lab}`. 

The lab should reference the project(s) containing the source code:

`You can find the code for the hands-on lab on [GitHub](https://github.com/kotlin-hands-on/{lab-name})`

If the lab describes creating the project step-by-step, the state of the project after each step should be present
in the project as a separate commit. After each step, the correct commit should be referred in the text. `master` branch
should contain only the initial state of the project, and the final state with the complete commit history should be
stored in the `final` branch.  

### Automatic compilation and testing of sample projects

We build sample projects on TeamCity to make sure the projects always compile. 
If a project contains two branches, both branches should be compiled on TeamCity.
The tests in each branch should be checked automatically except special branches
that contain failing tests for the readers to fix.

The TeamCity build configurations can be found here:

[https://teamcity.jetbrains.com/project/Kotlin_HandsOnLabs?projectTab=overview&mode=builds](https://teamcity.jetbrains.com/project/Kotlin_HandsOnLabs?projectTab=overview&mode=builds)

### Coding conventions

* Coding conventions should adapt to the [Kotlin Code Style](https://kotlinlang.org/docs/reference/coding-conventions.html). 
* Packages should be named `com.jetbrains.handson.{lab-name}` 

### Contributing

We'll be really happy to get content contributions from you! 

If you want to contribute and develop a new lab, the best first step is to create an issue in this project describing
your idea to make sure something similar isn't planned already.
When your idea gets approved, a new repository will be created for you where you can reference the code.

If you're not a native speaker, but want to contribute, feel free to do that.
We (JetBrains) can proofread the final lab text. 

Feel free to contribute small changes to existing labs directly to the corresponding projects. If you're planning on contributing more significant changes to existing labs, we recommend you get in touch with us first via an issue.
