+++
date = "2016-06-04T15:50:00+10:00"
draft = false
title = "Enums and Blackboard's ORM Framework"
description = "Using Enums with Blackboard's built-in ORM framework."
keywords = ["blackboard", "orm", "enum", "b2", "beans"]
+++

*By Shane Argo.*

A while back I wrote about [using Blackboard's ORM framework with your own beans]({{< ref "post/blackboard_ORM_framework_part_two.md" >}}). This is going to be a really quick post about how you can map database values to Java Enums.

This is done using the `@EnumValueMapping` annotation, which is only [mentioned breifly in Blackboard's documention](https://en-us.help.blackboard.com/Learn/9.1_2014_04/Administrator/080_Developer_Resources/020_Develop/Developer's_Guides/Date_Management_Developer's_Guide) but is pretty simple to use. Let's start with an example:

```java
@EnumValueMapping(values={"TOP", "BOTTOM", "LEFT", "RIGHT"})
enum Position {
    TOP, BOTTOM, LEFT, RIGHT
}
```

By annotating your enum with the `@EnumValueMapping` annotation, you can give each of the values a string representation or an integer representation like this:

```java
@EnumValueMapping(values={"1", "2", "3", "4", "5"}, integer = true)
enum StarRating {
    ONE, TWO, THREE, FOUR, FIVE
}
```

The values in the annotation are assigned to the enum values in order.

Once you've annotated your enum like this, you can use it as the type of a field just like in the [ORM post]({{< ref "post/blackboard_ORM_framework_part_two.md" >}}).