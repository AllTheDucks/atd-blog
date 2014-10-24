+++
date = "2014-05-15T13:42:08+11:00"
draft = false
title = "Foundation of Gradle for Blackboard Building Blocks"
author = "shane"
keywords = ["atdb2","gradle","jetty","tutorial"]
+++

This is the third post in a series about how All the Ducks builds a building block. In the last post, [I summarised the tools we use](/post/blackboard building block development tools/) and introduced [Gradle](http://www.gradle.org/) as our build tool of choice. In this post, we’ll use Gradle to build a ‘Hello World’ building block.

Some familiarity with your terminal or command prompt will be necessary to complete this tutorial, but it should all be very basic. Essentially, if you know how to navigate (change directory) you should be okay.

#### Installing Gradle

One of the great features of Gradle is that a project can be setup with a wrapper which means there is no need to install Gradle to build it. Having said that, you do need to install Gradle in order to add the wrapper to the project. What this means is you’ll need to install Gradle to initialise the project, but no one else will to build it.

[Plenty of guides exist which provide good instructions for installation](http://codetutr.com/2013/03/23/how-to-install-gradle/), so I am not going to go into detail here.

1. You’ll need the Java JDK installed.
1. Unzip the Gradle package into a location of your choosing.
1. Add the ‘bin’ directory from inside the unzipped contents to the PATH environment variable for your OS.

If you already have a terminal or command prompt open, you’ll need to reopen it to pick up the new PATH environment variable.

Once you’ve done this, you should be able to run the following command:

````
gradle -v
````
and get output similar to this:

````
--------------------------------------------
Gradle 1.12
--------------------------------------------

Build time: 2014-04-29 09:24:31 UTC
Build number: none
Revision: a831fa866d46cbee94e61a09af15f9dd95987421

Groovy: 1.8.6
Ant: Apache Ant(TM) version 1.9.3 compiled on December 23 2013
Ivy: 2.2.0
JVM: 1.7.0 51 (Oracle Corporation 24.51-b03)
OS: Linux 3.13.0-24-generic amd64
````

Don’t worry too much if it varies in the details, so long as you don’t get an error.

#### Initialise Gradle in your project

Now you are ready to set up your project with Gradle. Open your terminal or command prompt to the root directory of your project (create one if you haven’t already) and execute the following command:

````
gradle init
````

and you should see output similar to this:

````
:wrapper
:init

BUILD SUCCESSFUL

Total time: 2.199 secs
````

Any action in gradle is called a task. Tasks can depend on other tasks. Gradle is extremely smart; it is able to determine which of the dependent tasks need to be executed in order to complete the task you’ve asked for.

In this case, we’ve asked to execute the init task and this task depends on the wrapper task. Gradle is telling us that it has executed this wrapper task, and then the init task as requested.

The file structure of your project should now looks like this:

* sample
  * build.gradle
  * gradle
  * .gradle
  * gradlew
  * gradlew.bat
  * settings.gradle

*gradle* and *.gradle* are directories which contain files to support Gradle. For now, and probably forever, you can ignore them.
*gradlew* and *gradlew.bat* are used to execute the gradle wrapper. From this point forward you’ll use this script instead of using the instance of Gradle you installed on your computer. (The former is for *nix systems such as Linux and OSX and the latter is for Windows Systems.)
*build.gradle* is a configuration file with the instructions for building your project. There will eventually be one of these for each subproject as well.
*settings.gradle* is a configuration file specific to the project as a whole. Only one of these will ever exist in the entire project.

Now lets try out the wrapper. From the root directory of your project execute this command on Linux or OSX:

````
./gradlew tasks
````

or this, if you are on a Windows system:

````
gradlew tasks
````

From this point onwards, I will always include the “./” at the front of the command, but if you are on Windows you’ll need to exclude it.

*tasks* is a task that discovers and lists the tasks available to the project. The output of the above commands should be similar to this:

````
:tasks

--------------------------------------------
All tasks runnable from root project
--------------------------------------------

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Help tasks
----------
dependencies - Displays all dependencies declared in root project 'sample'.
dependencyInsight - Displays the insight into a specific dependency in root project 'sample'.
help - Displays a help message
projects - Displays the sub-projects of root project 'sample'.
properties - Displays the properties of root project 'sample'.
tasks - Displays the tasks runnable from root project 'sample'.

To see all tasks and more detail, run with --all.

BUILD SUCCESSFUL

Total time: 1.772 secs
````

As you can see from the above output, other than the “help” tasks, the only tasks available in the project at the moment is the init and *wrapper* tasks which we we’ve already executed. Gradle needs a little bit more configuration before we’ll get some more tasks.

Create the test application

In the first post in this series I discussed the creation of a test application which will run independent of Blackboard APIs. This package could be loaded into and Java web application container and you could use your tool without needing a full Blackboard instance running. Let’s put this into practice.

This is what the *test-web* subproject will be for. So we’ll create a subdirectory in the root project directory for this subproject and call it *test-web*. Edit the *settings.gradle* file. It’ll contain a lot of stuff added by the init task. Remove everything except this line:

````
rootProject.name = 'sample'
````

This line, fairly obviously, names the root project. All the other stuff is simply examples and links to documentation. Next we need to tell Gradle about the subproject we’ve just created. Add this to the top of the *settings.gradle* file add this:

````
include 'test-web'
````

This includes the *test-web* subproject into the root project. Let’s make sure it’s worked. Execute the following command to see the project structure:

````
./gradlew projects
````
Let’s now change to the *test-web* directory and create a new build.gradle file. This file will describe how to build this subproject, as will the *build.gradle* files in each respective subproject’s subdirectory.

Edit the new *build.gradle* file for the subproject and add this to it:

````
apply plugin: 'war'
````
This is does exactly what it says; it applies the WAR plugin to this subproject. Lets again run the *tasks* task. From the project root:

````
./gradlew tasks
````

For brevity, I’m not going to include the output here, but you should see a number of new tasks related to the compiling of Java classes and the creation of the WAR.

Within the *test-web* directory create this directory structure:

````
src/main/webapp
````

I said it in the last post, and I’ll say it again, one of the reasons Gradle is so great is because it often requires very little configuration if you use industry conventions. This directory structure follows the conventions and therefore we have no need to tell Gradle of it’s existence.

Within the *webapp* directory, create a file named *index.jsp* with this as the contents:

````
<html>
    <head>
        <title>Hello World!</title>
    </head>
    <body>
        <h1>Hello World!</h1>
    </body>
</html>
````
Now let’s create the WAR:

````
./gradlew war
````
This will have created the war file at:

````
test-web/build/libs/test-web.war
````
This file could be installed into and Java Web Application container, but setting this up and making it deploy could be time consuming. Not with Gradle.

#### Viewing the test application

Let’s modify the build.gradle file in the test-web subproject. Add this to the file:

````
apply plugin: 'jetty'
````
This is the [Jetty Application Container](http://www.eclipse.org/jetty/) plugin. It’s going to do all the hard word for you. Execute the following command:

````
./gradlew jettyRun
````
Now open your web browser and navigate to this URL:

````
http://localhost:8080/test-web
````
What do you see? Hopefully your test web application!

#### Summary

In this tutorial, we’ve installed Gradle, initialised our project with it, created a basic test web application and loaded into a application container to be viewed.

I think this was a pretty small amount of work for what we have achieved, but the benefit is even clearer when someone else on your team attempts to build your project.

Without installing anything (okay, they need the Java JDK) they can run a single (and simple) command to build the application:

````
./gradlew war
````
and a similarly simple command to view the application in a running Java application container:

````
./gradlew jettyRun
````
Gradle will download itself, project dependencies, jetty; whatever it needs to get it working. This is a very small amount of configuration when you consider this fact.

Next time, we’ll create a WAR file which can be deployed to Blackboard, also known as a build block.