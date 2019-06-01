[![official JetBrains project](https://jb.gg/badges/official.svg)](https://confluence.jetbrains.com/display/ALL/JetBrains+on+GitHub)
[![GitHub license](https://img.shields.io/badge/license-Apache%20License%202.0-blue.svg?style=flat)](https://www.apache.org/licenses/LICENSE-2.0)


## Kotlin Hands-On Labs

[Hands-On labs](https://play.kotlinlang.org/hands-on) are interactive tutorials to learn Kotlin. The labs are all provided under Apache 2 and are open to contributions. 

This readme provides more information on how labs are organised and is targeted at contributors. If you'd like to complete a lab, please use the [website](https://play.kotlinlang.org/hands-on)


### Structure

Each labs consists of:

* Text
* Assets such as images
* Code Samples

Text and assets for all labs are located under this repository under the `lab-name`. The code for each lab has its own repository following the naming convention `kotlin-hands-on-{lab name}`


### Text Structure

Each lab consists of a series of steps. Each step is represented as an individual markdown file with the naming convention `NN_{step-title}` where
`NN` is the step number and `step-title` is the step title. 

Each hands-on lab should start with an `01_Introduction` file that clearly highlights what the hands-on lab is going to cover. Ideally it should also show
a screenshot of the end result (if this is for instance an application). 

The introduction should reference the source code with a last line:

`You can find the code for the hands-on lab on [GitHub](https://github.com/kotlin-hands-on/{lab-name})`


### Style and Formatting

* Use *you* pronoun when referring to the user following the tutorial, i.e. "You first need to click on..." 
* Use bold to highlight UI elements and menu entries. Use | as separators for menu entries, i.e. **File|New|Project...**
* Use `name` notation to refer to folders and files

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

### Code

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


### Coding conventions

* Coding conventions should adapt to the [Kotlin Code Style](https://kotlinlang.org/docs/reference/coding-conventions.html). 
* Packages should be named `com.jetbrains.handson.{lab-name}`


### Referencing Code in steps
 
[TODO] - Instructions on how to validate code so that we make sure it always compiles 

### Contributing

In order to contribute, please send a pull request with the new tutorial folder structure. Once it has been approved, a new repository will be 
created for you where you can reference the code. 

