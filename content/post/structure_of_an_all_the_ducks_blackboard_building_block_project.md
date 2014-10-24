+++
date = "2014-05-09T13:17:37+11:00"
draft = false
title = "Structure of an All the Ducks Blackboard Building Block Project"
author = "shane"
keywords = ["atdb2", "b2", "tutorial"]
+++

This is the first in a series of posts about how All the Ducks builds a blackboard building block. Sometimes tutorials will step you through the bare minimum to get the desired result, but there is often a long way between this and a “real world” application. Instead, we have chosen to describe the real world development processes that we use.

This is the structure we used for [Javascript Hacks](http://projects.oscelot.org/gf/project/jshack/) (JS Hacks). At first glance, this structure seems pretty complicated, but we’ve found that it creates a good separation between the components and makes development quicker and easier to test.

The structure allows the building of two different web archives (WARs). The first WAR is fairly obvious, it is the Blackboard Building Block (B2). However, the second is one without any dependencies on the Blackboard libraries. This WAR can be loaded into any application container and tested independent of a functioning Blackboard Environment.

Our building blocks are broken into four components: api, common-web, test-web and b2.

#### API (api)

This component is a pure Java library, i.e. resulting artifact is a Java jar. It contains all of the classes and interfaces which do the heavy lifting in the application. This component should not have any dependencies on Blackboard libraries. All the of the classes and associated methods can be tested on this component independent of Blackboard.

As an example, in JS Hacks, this component contains (among many other things) the models and the code for loading the hacks from disk and enumerating a list of hacks.

#### Common Web Resources (common-web)

This component is a partial web archive (WAR). There is no complete archive produced from this component. Instead, it is combined with one of the other two remaining components to produce a complete WAR. Any resource in this project will be included in both the building block WAR and the test WAR. It will depend on the API JAR.

The common-web component will contain any web resource useful to both WARs. For example, this component in JS Hacks contains web services and stripes actions.

#### Test Web Archive (test-web)

The artifact produced from this component will be a functional web application which can be loaded into any web application container (for example, [Tomcat] or [Jetty]).

The resources in this component are likely to be the application context and JSP. It will also include the implementation of some interfaces.

The JS Hacks project implements an interface in this component which returns a directory for storing and retrieving hacks and the application context defines this implementation for use in this WAR.

#### Building Block (b2)

Finally, the b2 component will create the actual Blackboard building block WAR ready for deployment to a Blackboard environment.

As with the test-web component this will contain the application context, JSP and some interface implementations.

In this component in JS Hacks, the same interface for returning a data directory is implemented using the Blackboard APIs. Additionally, the same JSPs that are contained in the test-web component are included here, this time using the Blackboard tag libraries.

#### Summary

This project structure simplifies testing and development by separating most of the code from the Blackboard APIs. Automated testing using a library like JUnit is greatly simplified. Development is made easier as most can be done without the need for a Blackboard environment to deploy too.

This structure is particularly useful when combined with a number of tools and libraries, detailed in coming posts.

