# Setting up

### Prerequisites

To get started, let's make sure we have installed an up-to-date version of all the tools we’ll be using in this hands-on tutorial. Please refer to each tool’s individual documentation on how to download and install it on its respective website. The tools we’ll need are:

- `npm` (version `6.10` or above) – [Download/Install](https://www.npmjs.com/get-npm)
- `yarn` (version `1.16.0` or above) – [Download/Install](https://yarnpkg.com/lang/en/docs/install/#mac-stable)
- IntelliJ IDEA (version `2019.2.2` or above) with the Kotlin plugin (`1.3.50` or above) – [Download/Install](https://www.jetbrains.com/idea/download/)

### Setting up the project

We are going to set up our project using `create-react-kotlin-app`, *CRKA* for short. [CRKA](https://github.com/JetBrains/create-react-kotlin-app) is a zero-configuration tool used to create React apps with Kotlin (like `create-react-app` for regular React apps). It includes a [list of logical defaults](https://github.com/JetBrains/create-react-kotlin-app#whats-inside) that will allow us to accomplish all the tasks we are going to encounter on our learning journey without having to write a single line of configuration.

We can use CRKA without prior installation through `yarn`. To create a new application with its folder structure and appropriate configuration, let's run the following command:

```yarn create react-kotlin-app confexplorer```

`yarn` will pick up its work, and after a while, we will be greeted by a success message in our terminal.

![image-20190730162247914](/assets/image-20190730162247914.png)

CRKA ships by default with `webpack-dev-server`, allowing us to try our application locally from within the terminal. To start the development server, we can follow the instructions in the terminal, i.e. type the following commands:

```
cd confexplorer
yarn start
```

CRKA will start compiling the demo application that ships with a new project. After a few seconds, you should see the following confirmation in the command line:

![image-20190730162459994](/assets/image-20190730162459994.png)

The development server automatically opens a web browser as well, pointing at http://localhost:3000 and displaying the compiled application.

![image-20190730162757059](/assets/image-20190730162757059.png)

During development, feel free to keep the terminal open and leave the development server running. It will watch for the changes you make to your application, automatically recompile your Kotlin code, and reload the page while it is running.

Let's open the project in IntelliJ IDEA. To open the project, go to `File > Open` and navigate to the root of our project. When we first open the project, a prompt to upgrade the project to a new version might appear. We can do this without any hesitation.

![image-20190729140409230](/assets/image-20190729140409230.png)

Before we continue, feel free to have a look around, change a string or two, save, and wait for the automatic reload of your browser window. If you've accidentally terminated `yarn start`, you can just restart it from the terminal in IntelliJ IDEA.

### Cleaning up

Since it's in our interest to understand things *from scratch*, let's remove the example code from our repository. You can always refer back to the example code by creating a new project using the steps above.

So, let's delete all the files inside the `src` folder, and start from scratch! We'll explore what each bit of code does, and build our application from the ground up.

This is a blank slate of endless possibilities. Let’s get started!

You can find the state of the project after this section on the `step-01-empty-project` branch in the [GitHub](https://github.com/kotlin-hands-on/web-app-react-kotlin-js/tree/step-01-empty-project) repository.