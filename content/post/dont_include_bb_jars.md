+++
date = "2016-06-09T12:40:00+10:00"
draft = false
title = "Don't include Blackboard JARs in your WAR"
description = ""
keywords = ["blackboard", "b2", "development", "java"]
+++

*By Shane Argo*

Every building block in Blackboard has its own [classpath](https://en.wikipedia.org/wiki/Classpath_(Java)). This essentially means that every building block can define it's own dependencies without conflicting with others. 

This is generally a good thing, but also comes with a pretty major drawback. If multiple building blocks use the same library (JAR), each will load thier own copy of the classes into memory. To get around this, Blackboard provides a number of common libraries. Specifically, all the libraries in Blackboard's `systemlib` directory.

This directory includes JARs such as `bb-plaform.jar`, `bb-cms-admin.jar`, and `bb-taglibs.jar` but also includes several useful general purpose libraries such as `commons-lang-2.4.jar`, `jsoup-1.8.3.jar`, and `guava-17.0.jar`.

What this means is that you can use these libraries in your own building blocks without needing to include them in your WAR. In fact, you **should make sure that you don't** include them in you WAR.

Including simple utility libraries, like the [Apache Common Lang](https://commons.apache.org/proper/commons-lang/) will probably just be a waste of memory. But, with more complex libraries, things can get pretty hairy.

Including Blackboard's Libraries is even worse. This can cause a number of issues but usually results in an issue related to permissions. Blackboard utilises [Java Security Permissions](https://commons.apache.org/proper/commons-lang/) to grant access to different resources. This is what you are using when you add a `<permission>` to your `bb-manifest.xml` file. Any code that is loaded by your building block is restricted to only doing the things that have been granted to it. 

So now what happens when you try to execute a method from the Blackboard APIs? The Blackboard code tries to do something with a resource which it normally has access to, such as a file in the Blackboard home directory, but your building block hasn't been granted this access. Because of this, the most common way this issue manifests itself is with a security exception.

To illustrate this, I've just added the `bb-platform.jar` to a building block. This is the error I got:

````
java.security.AccessControlException: access denied ("java.util.PropertyPermission" "*" "read,write")
  at java.security.AccessControlContext.checkPermission(AccessControlContext.java:372)
  at java.security.AccessController.checkPermission(AccessController.java:559)
  at java.lang.SecurityManager.checkPermission(SecurityManager.java:549)
  at blackboard.platform.security.BbSecurityManager.actuallyCheckPermission(BbSecurityManager.java:115)
  at blackboard.platform.security.BbSecurityManager.checkPermission(BbSecurityManager.java:105)
  at java.lang.SecurityManager.checkPropertiesAccess(SecurityManager.java:1265)
  at java.lang.System.getProperties(System.java:624)
  at blackboard.platform.service.ServiceManagerImpl.logSystemProperties(ServiceManagerImpl.java:734)
  at blackboard.platform.service.ServiceManagerImpl.init(ServiceManagerImpl.java:220)
````

The error message that you get might be different, but the lesson is this. If you are getting an unexpected security exception, then the first thing to check is that you are not including any of the Blackboard JARs in your WAR.

