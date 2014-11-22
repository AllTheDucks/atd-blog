+++
date = "2014-05-26T21:29:25+11:00"
draft = false
title = "Add the version from bb-manifest to your war"
description = "If you’re using Gradle to build and package your Building Blocks, it’s remarkably easy to add a version number to the . war file that Gradle creates."
keywords = ["cool tricks", "blackboard", "gradle"]
author = "wiley"
+++

If you’re using Gradle to build and package your Building Blocks, it’s remarkably easy to add a version number to the . war file that Gradle creates.

If you set the version property in build.gradle, like so: ````version=1.0.5````

You’ll end up with a war file named: ````myproject-1.0.5.war````

This seems a bit redundant when we’ve already set the version in /WEB-INF/bb-manifest.xml. So using the following snippets, you can just grab the version from bb-manifest.xml, and stuff it into the version property.

````
version = getB2Version()
````


````
String getB2Version() {
  File mfFile = new File(file(webAppDir), 'WEB-INF/bb-manifest.xml');
  def manifest = new XmlSlurper().parse(mfFile);
  return manifest.plugin.version['@value'];
}
````

Have a look at the JSHack Version 1 source code to see it in Action.
[build.gradle for JSHack, on projects.oscelot.org](http://projects.oscelot.org/gf/project/jshack/scmsvn/?action=browse&path=%2Ftrunk%2Fb2%2Fbuild.gradle&revision=208&view=markup)