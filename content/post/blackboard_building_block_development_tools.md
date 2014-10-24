+++
date = "2014-05-13T13:24:33+11:00"
draft = false
title = "Blackboard Building Block development tools"
keywords = ["atdb2", "b2", "b2deploy", "closure", "git", "glovr", "gradle", "idea", "jetty", "junit", "mockito", "plovr", "starting block", "tutorial"]
author = "shane"
+++

This is the second post in a series about how All the Ducks builds a Blackboard Building block. [The first described how we structure a project](/post/structure_of_an_all_the_ducks_blackboard_building_block_project/).

In this post, I want to document the tools we use to develop Blackboard building blocks. The truth is, that we’d use a lot of these tools for any software development.

Whilst it is entirely possible, software is not normally developed using only a text editor and a compiler. Usually as a project grows in size and it becomes necessary to support development with a number of tools.

It is my intention in this series of posts to slowly introduce the following tools, beginning with the bare bones “text editor and compiler” project.

#### Source Control – [Git](http://git-scm.com/)

I think source control is one of those tools that you don’t realise you need until you start using it. The power to view history, compare changes, rewind, branch is invaluable as your project grows especially as the number of developers increase. Enter git.

Git, is a source control tool that is used extensively by the open source community. It can be difficult to get your head around the commands. There are more tutorials on git on the internet than grains of sand on the earth. The only way to learn with this one is to dive in head first. It will begin to make sense.

#### Build Tool – [Gradle](http://www.gradle.org/)

Again, as your project expands your project is going to require more work to build. This can be a huge time waster, eating into time you could be using to write code and it often the biggest stumbling block for new developers.

Gradle is a relative newcomer to the build tool scene, but in our opinion it is the best. It’s competitors often require a huge amount of XML (or similar) configuration just to get started. The Gradle team has a philosophy of “convention over configuration”. What this means is that if you set up your project using the generally accepted conventions of the development community,  then Gradle should be able to figure out for itself how to build your project.

It is almost certain you will want to include a library or two (or twenty) into your project. Gradle makes this a breeze.

All the Ducks’ Javascript Hacks Blackboard building block project uses Gradle and it reduces the build from the these steps:

1. Compiling the API classes
1. Create the API JAR
1. Compile the “Common Web” classes
1. Compile the “B2″ classes
1. Run the Google JS Linter on the project Javascript
1. Run the Google Closure JS Compiler
1. Package the outputs of the above steps in to the final WAR
to a single command:

````
./gradlew jshack-b2:war
````

This is ignoring that commands to perform the seven steps above are fairly complex. Add into this also that there are other “tasks” you may wish to perform in the project which could require similarly complex steps.

This is pretty amazing when you consider that without downloading anything (except the Java JDK) a new developer can build the building block with the above command. They don’t even need to download Gradle!

It will download all the project dependencies, compile the code, and stitch it all together. I could go on all day about this tool, but I wont.

#### Web Application Container – [Jetty](http://www.eclipse.org/jetty/)

In the [previous post](http://blog.alltheducks.com/2014/05/b2-project-structure/), I introduced the structure that we use with our building blocks. This includes a test component without any dependencies on the Blackboard APIs. This is where the Jetty Application Container comes in.

Using the Jetty Gradle plugin, we can compile the WAR and have it running with one command. No need to install anything. To reiterate, one line of configuration in Gradle and you can now click around in your web application.

Sorry, somehow I ended up raving about Gradle in the Jetty section.

#### Test Deployment – Starting Block / B2Deploy

Obviously, at some point you are going to have to deploy your building block in an actual Blackboard environment. Using the Starting Block building block and B2Deploy Ant Task this is greatly simplified.

The Starting Block building block is developed by Blackboard and allows the remote uploading and activation of a building block using a HTTP POST. If this means nothing to you, it doesn’t matter. I cannot link the the starting block directly, as it is Blackboard proprietary software, but you can download it from Behind the Blackboard. Please, never ever install this into an environment available from an untrusted location (i.e. your production environment).

The B2Deploy Ant task is an extension to the Ant build tool which generates the HTTP POST for you, meaning you can deploy with a single command. Thankfully, Gradle is compatible with Ant tasks, allowing this extension to be used with it.

#### Testing – [JUnit](http://junit.org/) & [Mockito](https://code.google.com/p/mockito/)

Testing is a vital part of any large scale project. Adding tests to your project while it is still small makes life a lot easier.

JUnit is probably the most prolific of the Java testing frameworks out there. There are heaps of tutorials on the internet and I will have a future post about how we use it in our building block projects.

Mockito makes testing unbelievably simple. With minimal setup, you can run code and “ask questions” about the expected outcome. I find it to be an easy to read and write way of setting up tests.

#### IDE – [Intellij IDEA](http://www.jetbrains.com/idea/)

I’d never found an Java IDE I was happy with until I came across IDEA. It always seems to know what I am thinking. Add in it’s first class support for Gradle, and you’ve got a winner.

At the end of the day, your IDE is a personal choice.

#### Javascript Framework – [Google Closure](https://developers.google.com/closure/) & [Plovr](http://plovr.com/) & [Glovr](https://github.com/AllTheDucks/glovr)

Blackboard uses the Javascript library PrototypeJS. For basic Javascript requirements this is probably enough and you really shouldn’t go adding any heavy client-side dependencies on top of this.

Sometimes projects will get more complex and this is when Google’s Closure becomes extremely useful. It’s not only a robust and flexible Javascript library but also a set of tools for optimisation, code linter, templating system and stylesheet language.

The final output of the Closure compiler is a minified, obfuscated and optimised file with no dependency on heavy client-side libraries. It does this by picking and choosing the classes your code requires.

Plovr is a separately developed tool which executes the Javascript compiler on demand, making development easier. Glovr is a plugin for Gradle we developed which integrates Plovr.