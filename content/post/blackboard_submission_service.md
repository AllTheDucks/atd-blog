+++
date = "2016-01-19T19:50:00+10:00"
draft = true
title = "Blackboard Extensions and the Submission Service"
description = "Stepping through the process for adding the required fields for a Blackboard extension to your bb-manifest file."
keywords = ["blackboard", "extension", "submission-serivce", "bb-manifest"]
+++

Extensions are one of many available Blackboard APIs which are there to provide useful interfaces to interact with the system. This post will walk through the process for implementing one such extension provided by Blackboard, namely the submission service. The submission service provides event handlers for submittable items (assignments, tests, et.al). For example when a student submits an assignment, one of the handlers in this extension fires. So let's jump into the code!

## Quick Setup

Start by grabbing the jar file, it can be obtained from the Blackboard development VM. For the submission service, the jar file is located at `/usr/local/blackboard/content/vi/BBLEARN/plugins/bb-submission-services/webapp/WEB-INF/libext/submission-services-api-1.1-SNAPSHOT.jar`. Move this file onto the compile-time classpath for your building block. If you are using gradle, add the following to your dependencies:

```java
providedCompile files('lib/submission-services-api-1.1-SNAPSHOT.jar')
```

 The changes below ensure the copied jar file is added to the run-time classpath of Blackboard. This means that this step is only for use in development.

## bb-manifest

Implementing Blackboard extensions also requires some modifications to the `bb-manifest.xml` file. This building block requires extension points, so start by [adding `<webapp-type value="javaext" />`](https://docs.alltheducks.com/blackboard/bb-manifest-ref.html#toc_18).

To have the Submission Service building block's libraries added to your building block's classpath at run time, you need to specify it as a dependency:

```xml
<requires>
    <bbversion value="9.1"/>
    <plugin-versions>
        <plugin-version min="1.1" handle="submission-services" vendor="bb"/>
    </plugin-versions>
</requires>
```

Finally, the last addition to `bb-manifest.xml` is registering your implementation of the extension points. Take a look at the `bb-manifest.xml` file for the service you are implementing. Within the `<extension-defs>` tag, the `namespace` for the extension points and the `id` of each extension point are the main values needed from the services' `bb-manifest.xml`. You must know these values for Blackboard to know which interfaces of the service you are implementing.

In our `bb-manifest.xml` we need something that looks like this:
```xml
<extension-defs>
  <definition namespace="com.alltheducks.submissionservice">
      <extension id="atditemSubmissionEventHandler"
                 point="blackboard.plugin.submission-services.submittableItemEventHandler"
                 class="com.alltheducks.submissionservice.handler.SubmittableItemEventHandlerImpl"
                 singleton="true" />
      <extension id="atditemSubmissionEventHandler"
                 point="blackboard.plugin.submission-services.itemSubmissionEventHandler"
                 class="com.alltheducks.submissionservice.handler.ItemSubmissionEventHandlerImpl"
                 singleton="true" />
      <extension id="atdsubmissionServiceTagHandler"
                 point="blackboard.plugin.submission-services.submissionServiceTagHandler"
                 class="com.alltheducks.submissionservice.handler.SubmissionServiceTagHandlerImpl"
                 singleton="true" />
      <extension id="atdsubmissionServiceCxHandler"
                 point="blackboard.plugin.submission-services.submissionServiceCxHandler"
                 class="com.alltheducks.submissionservice.handler.SubmissionServiceCxHandlerImpl"
                 singleton="true" />
  </definition>
</extension-defs>
```

Remember the `namespace` and `id` values? These are the values used for the `point` for each extension being implemented. You may also note that we are implementing methods for the providers rather than the consumer. In this instance, Blackboard consumes the extension that you provide. It also doesn't seem to matter what you put for the extension `id` or the `namespace` for the extensions.

## A Quick Note on Implementing Methods

For these services, most methods take one or more `Id` parameters. For the majority of them, these parameters have names like `param1`, `var` or simply `id`. It is very a good idea to log or print the types of Blackboard Ids these are once the methods get called and record them in documentation above the method name:

```java
public class ItemSubmissionEventHandlerImpl implements ItemSubmissionEventHandler {

    /**
     * @param id A Course Id
     * @param id1 A GradeableItem Id
     * @return
     * @throws SubmissionServiceException
     */
    @Override
    public boolean submittableItemExists(Id id, Id id1) throws SubmissionServiceException {
        System.out.println(id.toString());
        System.out.println(id1.toString());
        return true;
    }

  }
  ```

  After checking the types of Id being passed to the method, it's also a good idea to rename the parameters as well, to avoid further confusion.
