+++
date = "2015-10-12T23:16:00+10:00"
draft = false
title = "Using Blackboard's ORM Framework with your own beans"
description = "You can use Blackboard's built-in ORM framework with your own custom beans."
keywords = ["blackboard", "orm", "b2", "beans"]
+++

Last time, (yeah, I know that was almost a year ago) Wiley discussed [using Blackboard's ORM to access legacy Blackboard entities]({{< ref "post/blackboard_ORM_framework_part_one.md" >}}). This time around I want to talk about how you can use the ORM to perform [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations on your own custom beans.

For those new to development altogether, this all means we can interact with the database, modifying and reading data purely with Java. This simpilfies things a lot, taking care of things like supporting different databases.

In this post, I'm going to be writing code to help Santa keep track of his gifts. Firstly, we'll define a "bean" (just a plain java class) to represent an entry in his list:

````
public class Gift {

    private Id userId;
    private String description;
    private int count;

    public Id getUserId() {
        return userId;
    }

    public void setUserId(Id userId) {
        this.userId = userId;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }
    
}
````

Now we have representation of what an entry on Santa's list is going to look like. It's going to have a reference to the recipient's user ID, the name of the gift they are to receive and the how many of that gift the recipient will receive. 

We can now begin annotating the bean so that it can be used with Blackboard's ORM. All entities in Blackboard must have a unique ID. This takes the form of a ````backboard.persist.Id````. To construct one of these ````Id```` objects you need to have a ````blackboard.persist.DataType```` defined. This might sound like a lot of work, but Blackboard provides mechanisms to simplify it.

The first step is to have your bean extend the ````blackboard.data.AbstractIdentifiable```` class. This class comes with an ````Id```` field already, but expects the ````DataType```` to be defined on a static field named ````DATA_TYPE````:

````
public class Gift extends AbstractIdentifiable {

    public static final DataType DATA_TYPE = new DataType(Gift.class);

    ...

}
````

We now need to tell the ORM how to map the bean and its fields to the database tables. We *could* create a map, with the same structure as the legacy Blackboard entities, defining mappings between field names and table columns, the types of those columns and other metadata. The main issue with this is that all that data is outside of the bean, hidden away in the map.

Thankfully, Blackboard provide a bunch of annotations which you can use to decorate your bean and its fields, and a static method to interpret these annotations and dynamically create the map. Let's first look at the annotations, which are all in the ````blackboard.persist.impl.mapping.annotation```` package. Here is our complete ````Gift```` bean:

````
@Table("atd_santaslist_gifts")
public class Gift extends AbstractIdentifiable {

    public static final DataType DATA_TYPE = new DataType(Gift.class);

    @Column({"user_pk1"})
    @RefersTo(User.class)
    private Id userId;

    @Column({"descr"})
    private String description;

    @Column({"count"})
    private int count;

    public Id getUserId() {
        return userId;
    }

    public void setUserId(Id userId) {
        this.userId = userId;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }

}
````

Okay. These annotations are fairly self explanatory. The ````Table```` maps the bean to the database table. Likewise, ````Column```` maps the field to the column. The ````RefersTo```` annotation tells Blackboard what the ````DataType```` of the ````Id```` object is.

The static method that Blackboard provides to interpret the annotations and generate the dynamic map is ````blackboard.persist.impl.mapping.annotation.AnnotationMappingFactory.getMap(Class<?>)````.

You can now use this bean just [like we did with the legacy Blackboard beans in the previous post]({{< ref "post/blackboard_ORM_framework_part_one.md" >}}). Here is a ````SimpleDAO```` implementation for our ````Gift```` bean:

````
public class GiftDao extends SimpleDAO<Gift> {

    private static final DbObjectMap GIFT_EXT_MAP = AnnotationMappingFactory.getMap(Gift.class);

    public GiftDao() {
        super(GIFT_EXT_MAP);
    }

}
````

Here I store the map in a constant named ````GIFT_EXT_MAP````. This looks a bit tider, but also ensures that the map is constructed when Java loads the class, instead of once every time a new SimpleDAO is created, which is what would have happened if we'd called ````getMap```` within the constructor.

This ````GiftDao```` can now be added to however you wish. Obviously, this all assumes that the tables already exist in the database. I hope to cover this in a coming post. (I'll try not to take 11 months this time.)
