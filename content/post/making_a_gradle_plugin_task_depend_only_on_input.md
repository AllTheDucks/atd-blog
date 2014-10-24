+++
date = "2014-01-09T13:04:55+11:00"
draft = false
title = "Making a Gradle Plugin Task depend only on input"
author = "shane"
keywords = ["build tool", "gradle", "plugin"]
+++

Recently, I have been doing some development of a plugin for the (amazing) build tool [Gradle](http://www.gradle.org/). Specifically, the All the Ducks developed [Glovr Gradle plugin](https://github.com/AllTheDucks/glovr), which adds support for the Google Closure [Plovr](http://plovr.com/) tool to Gradle. We will inevitably blog about this plugin more as we prepare for release of version 1.

I was adding support for the [Google Closure Linter](https://code.google.com/p/closure-linter/) to the tool when I ran into a little stumbling block.

As some background, when defining tasks on a Gradle Plugin, if you specify the inputs and outputs of the task, then Gradle will automatically determine if the outputs are up to date and ultimately if the task can be skipped. This is really handy and is, in my opinion, one of the many features that makes Gradle so good.


![Gradle tasks, up to date.](/images/uptodate.png "Gradle tasks, up to date.")

The problem was, the Linter doesn’t have any file outputs. I wanted the task to be skipped only if the input files have changed, but Gradle will not attempt to determine if the task can be skipped unless you specify both the inputs and outputs of the file. The solution was actually very simple.

The Gradle TaskOutputs interface allows you to specify a Closure (distinct from the Google Closure Tools) for determining if the outputs are up to date using the upToDateWhen method. So I simply defined a closure that always returns true (the code below is in written Groovy):

````
project.task('gjslint') {
    inputs.dir theInputsDirectory
    outputs.upToDateWhen( { return true } );
    ...
}
````

Simple! Now when Gradle attempts to determine if the task needs to be re-executed, the inputs directory will be scanned as it normally would, but the closure will be executed to determine if the outputs are up to date and always determine that they are.