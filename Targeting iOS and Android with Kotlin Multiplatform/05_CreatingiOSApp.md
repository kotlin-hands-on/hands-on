# Creating iOS Application

We can start by opening Xcode and selecting the *Create a new Xcode project* option. In 
the dialog, we need to choose the iOS target and select the *Single View App* and click next. Fill in the next page with the defaults, 
and use `KotlinIOS` as the *Product Name*. Let's select _Swift_ as the language (it is possible to use
Objective-C too). Use the `com.jetbrains.handson.mpp.mobile` string for the _Organization Identifier_ field.
Now the _Next_ button should be available, let's click it to move on.
In the file dialog, shown after clicking the _Next_ button, we need to select the root folder
of our project, click the _New Folder_ button and create a folder called `native` in it. 
The folder should be selected now, and we can click the _Create_
button to complete the dialog. We will use relative paths in the configuration files later in this tutorial. 

The created iOS application is ready to run on the iOS simulator or iOS device. For the device to run
it may require an Apple developer account and a developer certificate. Xcode does its
best to get us through the process. 

Let's make sure we can run the application on the iPhone simulator or device by clicking the play button
from the XCode window title bar. 

The `step-006` branch of the 
[github.com/kotlin-hands-on/mpp-ios-android](https://github.com/kotlin-hands-on/mpp-ios-android/tree/step-006)
repository contains a possible solution for the tasks that we have done above.  We can also download the
[archive](https://github.com/kotlin-hands-on/mpp-ios-android/archive/step-006.zip) from GitHub directly or
check out the repository and select the branch.
