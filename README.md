[![official JetBrains project](https://jb.gg/badges/official.svg)](https://confluence.jetbrains.com/display/ALL/JetBrains+on+GitHub)
[![GitHub license](https://img.shields.io/badge/license-Apache%20License%202.0-blue.svg?style=flat)](https://www.apache.org/licenses/LICENSE-2.0)


## Kotlin Hands-On Labs

Hands-On labs are interactive tutorials to learn Kotlin. They are published at [TODO]. The labs are all provided under Apache 2 and are open to contributions. 

This readme provides more information on how labs are organised and is targeted at contributors. If you'd like to complete a lab, please use the
[TODO WEBSITE]

### Structure

Each labs consists of:

* Text
* Assets such as images
* Code Samples

Text and assets for all labs are located under this repository under the `lab-name`. The code for each lab has its own repository following the naming convention `kotlin-hands-on-{lab name}`


#### Text Structure

Each lab consists of a series of steps. Each step is represented as an individual markdown file with the naming convention `NN_{step-title}` where
`NN` is the step number and `step-title` is the step title. 

#### Assets

All assets, excluding code, should be placed in a subfolder named `assets` under each lab folder and the asset should be referenced relative to this. 

#### Code

The code for the lab is located in its own repository using the naming convention `kotlin-hands-on-{lab-name}`

#### Referencing Code in steps
 
[TODO] - Instructions on how to validate code so that we make sure it always compiles 

### Contributing

In order to contribute, please send a pull request with the new tutorial folder structure. Once it has been approved, a new repository will be 
created for you where you can reference the code. [TODO - Confirm process]




