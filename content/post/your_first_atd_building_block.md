+++
date = "2014-05-16T21:29:25+11:00"
draft = false
title = "Creating Your First Building Block, The All The Ducks Way"
description = ""
keywords = ["blackboard", "development", "tutorial", "b2", "atdb2"]
author = "shane"
+++


This is the fourth post in a series about how All the Ducks builds a Blackboard building block. To summarise the series so far:

* I began by [describing how we structure our building block project](/post/structure_of_an_all_the_ducks_blackboard_building_block_project/)
* then moved on to [the tools we use during development](/post/blackboard_building_block_development_tools/); and
* finally [we used Gradle to build a web application that could be deployed to a standard container, without needing Blackboard](/post/foundation_of_gradle_for_blackboard_building_blocks/.
This time we will take the project we created at the end of the last post and expand it to build an actual building block that can be installed into Blackboard.

#### Creating the building block subproject

The vernacular in the Blackboard community is to abbreviate “Building Block” to “b2″. This is because Bb is already used as an abbreviation for Blackboard, so I will use  b2 from this point forward. Lets create a subproject for the b2. You may recall from the last post that to do this we need to create a subdirectory, and include it in the *settings.gradle* file. So create a directory in the root of the project called *b2* and then add this to the *settings.gradle* file:

````
include 'b2'
````

#### Configuring the b2 subproject

Create the *build.gradle* file in the *b2* subdirectory. I mentioned at the end of the last post that a b2 is just a WAR with an additional file which has some metadata for Blackboard to use. So, like the *test-web* subproject, we need to apply the WAR plugin. Add this to the new *build.gradle*:

````
apply plugin: 'war'
````

As with the *test-web* subproject, we will create this directory structure:

````
src/main/webapp
````
This is the directory where we’ll put all the resources for the web application. These files are included in the final application as is. This is opposed to things like Java files which need to be compiled prior to getting added to the application. The extra file for Blackboard that I referred to a couple of paragraphs up, is a file named *bb-manifest.xml* within a directory named *WEB-INF*. This file doesn’t require any preprocessing, so we will put it into the webapp directory we’ve just created. Create the *WEB-INF* directory and then create the file with this as the contents:

````
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
    <plugin>
        <name value="Sample b2" />
        <handle value="sample-b2" />
        <description value="A sample building block, demonstrating how All the Ducks builds a b2." />
        <version value="1.0" />
        <requires>
            <bbversion value="9.1" />
        </requires>
        <vendor>
            <id value="atd" />
            <name value="All the Ducks" />
            <url value="http://www.alltheducks.com" />
            <description value="All the Ducks provides services and products to the EdTech community." />
        </vendor>
    </plugin>
</manifest>
````

A lot of this is fairly self explanatory. The most important parts at this point and the plugin’s handle and the vendor’s id. This is used to uniquely identify your b2. Any b2s with the same combination of these two attributes will be considered different versions of the same b2. These attributes are also used in the URLs of the pages of your application. It is advised no to use any spaces or complex characters in these two values.

#### Adding a web.xml file

I’ve mentioned a couple of times already that a Blackboard building block is a standard Java WAR file, with the additional bb-manifest.xml file. There is one other file that is usually included in a standard WAR file; this is the *web.xml* file.

In the last tutorial, we didn’t add one of these to your *test-web* project. This is because Jetty will not complain if this file is missing and it was easier for the purpose of the tutorial to leave it out. Now however, we will need to create one, as Blackboard is not as forgiving.

The *web.xml* file is expected to exist in the *WEB-INF* directory, like the *bb-manifest.xml*. So lets create it there. At this stage, even though Blackboard requires this file to exist, we don’t have any configuration we need to do in it. So we’ll make the contents of the file the bare minimum:

````
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
</web-app>
````
Don’t worry if it looks crazy to you, this isn’t important right now.

#### Creating the hello world page

Okay, we’re almost there. We need to include a hello world page in the b2 so that it does something, however useless it is at this point. Create a file named *index.jsp* in this directory:

````
src/main/webapp
````
Remember this directory is for files in your building block that are not preprocessed, which includes html files. Lets add some html to that file:

````
<html>
 <head>
 <title>Hello World from your first b2!</title>
 </head>
 <body>
 <h1>Hello World from your first b2!</h1>
 </body>
</html>
````
#### Build your first b2

Finally, we are ready to build your b2. We now have two subprojects which will create WAR files. So now we need to be specific about which WAR file we want to build:

````
./gradlew b2:war
````
(Remember in windows to exclude the “./” from the beginning of the command.) This will create the war file for your first b2 in the following location:

````
b2/build/libs/b2.war
````
#### Install and Test

Lets see how we did. Install the building block into a Blackboard instance and activate it. Now we need to see if it’s worked. The URL your application will be mapped to will follow this pattern:

````
http://your.blackboard.domain/webapps/<vendor id>-<b2 handle>-<instance id>/
````
(Be sure to note the dashes) The *vendor id* and *b2 handle* were defined in your *bb-manifest.xml* file as discussed earlier. If you didn’t change that file, they will be *atd* and *sample-b2* respectively. The *instance id* is the id of your specific Blackboard environment. Depending on how old the instance is, it will probably be BBLEARN or bb_bb60. This can be customised at install time, so if neither of these work, you’ll need to consult the person who installed Bb at your institution. So the URL of your webapp will look something like this:

````
http://your.blackboard.domain/webapps/atd-sample-b2-BBLEARN/
````

Hopefully, you should see your hello world page.

#### Summary

In this tutorial, we’ve added a subproject to your project with the purpose of building a useable Blackboard b2. We added some meta data to this b2, which Blackboard requires, and added a basic page HTML page. Finally we built the b, installed it to Blackboard and viewed the page. Next time, we’ll bring the common web resources subproject in. This sub project will simplify development and reduce duplication.