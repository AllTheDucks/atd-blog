+++
date = "2015-10-15T10:30:00+10:00"
draft = false
title = "Introduction to Blackboard's Schema.xml"
description = "Blackboard's Building Block API provides a mechanism for creating and managing custom database entities."
keywords = ["blackboard", "database", "b2", "tables"]
+++

*By Shane Argo.*

For a very long time, there was no way for Blackboard building blocks to create and manage database entities. Instead, if you wanted to persist and load data in your building blocks, you had to use hacky solutions. For example, you could store data in:

* files in the shared content;
* manually created database tables in the Blackboard database; or
* an external database.

All of these solutions had their own pitfalls and were often messy, difficult and unmaintainable.

Then the Blackboard development team made the decision to start using the building block platform to implement core features and ran into this problem. One of the first examples of this is the Self and Peer Assessment Tool, which stores all of it's data in files in the shared content.

It wasn't long after this, that the Schema API came along. This is a database independent way of creating and managing tables and other database entities. I am pleased to say that it works quite well. I am even more pleased to say that Blackboard decided to make the API public, even if it is less than optimally documented.

# Structure #

To create and manage database entities in your building blocks, you'll need to define them in a series of files in your building block's WAR package. So let's start by looking at the structure of these directories and files. Firstly, everything related to the Schema API is stored in the root of the package in a directory aptly named *schema*.

Within this directory, there is one directory per database that your building block will use. Usually this will only be the core database, normally named BBLEARN or bb_bb60, but may also be the stats database. The directory names here can be anything, so choose something that is meaningful to you. Blackboard normally names the core database directory *instance*, and the stats database directory *stats*.

This is what our directory structure looks like so far:

* / (WAR file root)
    - schema
        + instance
        + stats

Now we need to tell blackboard about these directories by referencing them within the bb-manifest.xml file. Add this under &lt;manifest&gt;/&lt;plugin&gt;:

````
<schema-dirs>
    <schema-dir dir-name="instance" /> <!-- BBLEARN / bb_bb60 etc.-->
    <schema-dir dir-name="stats" database="stats" /> <!-- stats db -->
</schema-dirs>
````

You do not need to include a &lt;schema-dir&gt; entry for databases you are not using.

From now on, I'll focus exclusively on the instance directory, but everything applies equally well to the stats database and, theoretically, any other database.

# Schema.xml #

Within the instance directory, there is one required file - schema.xml. And, this is the file that does most of the heavy lifting and will be the primary focus of this post.

So let's check out this file, starting with an example. I am going to follow on using the [Santa's list example from my last post]({{< ref "post/blackboard_ORM_framework_part_two.md" >}}):

````
<?xml version="1.0" encoding="utf-8"?>
<schema xmlns="http://www.blackboard.com/bb-schema"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.blackboard.com/bb-schema http://fibbba.medu.com/xsd/bb-schema.xsd"
        name="atd_santaslist" license="Course Delivery">

    <table name="atd_santaslist_gift">
        <column name="pk1" data-type="id" nullable="false" identity="true" />
        <column name="user_pk1" data-type="id" nullable="false" />
        <column name="descr" data-type="nvarchar(100)" nullable="false" />
        <column name="count" data-type="int" default="1" nullable="false" />

        <primary-key name="atd_santaslist_gift_pk">
            <columnref name="pk1" />
        </primary-key>

        <foreign-key name="atd_santaslist_gift_fk1" reference-table="users" on-delete="cascade">
            <columnref name="user_pk1" />
        </foreign-key>

        <index name="atd_santaslist_gift_ak1" unique="true">
            <columnref name="user_pk1" />
            <columnref name="descr" />
        </index>
    </table>

</schema>
````

Okay. So the first seven lines is just jumping through the XML hoops and can pretty much be copied and pasted, but be sure to update the name property of the &lt;schema&gt; tag.

# Tables #

````
<table name="atd_santaslist_gift">
````

Next, we define a table for storing the gifts on Santa's list using the &lt;table&gt; tag. We give the table a name using the *name* attribute. 

## Columns ##

````
<column name="pk1" data-type="id" nullable="false" identity="true" />
<column name="user_pk1" data-type="id" nullable="false" />
<column name="descr" data-type="nvarchar(100)" nullable="false" />
<column name="count" data-type="int" default="1" nullable="false" />
````

The table has four columns: pk1, user_pk1, descr and count; [just like the bean we defined last time]({{< ref "post/blackboard_ORM_framework_part_two.md" >}}). 

Columns must have a *data-type* defined. During our [exploration using grep]({{< ref "post/schema_xml_datatypes.md" >}}), we discovered that the possible data-types are:

* bigint
* char(?)
* datetime
* float
* id
* image
* int
* integer
* ntext
* numeric
* numeric(?)
* nvarchar(?)
* text
* varchar(?)

Columns may also be defined as either nullable or not nullable using the *nullable* attribute. They can also be defined as an identity, meaning that Blackboard will automatically handle the automatic incrementing.

Columns can be given default values using the *default* attribute, but it requires a bit of thought. Whatever you put in *default* ends up in the SQL without any consideration of the data-type. It is up to you to ensure that the default value given is appropriate for the data-type, including the use of quotes. This is hard to explain, so I'll use an example. 

If you are defining the default value of an integer, it'd be pretty straightforward; you'd use something like `default="4"`. On the other-hand, if you wanted to define a default value for a string you'd use something like `default="'bar'"`. Notice the single quotes inside the value. Again, this value is injected directly into the SQL so to understand this, lets have a look at what these examples and one without quotes would like inside a contrived update statement:

1. `default="4"` : `UPDATE example SET foo = 4`
2. `default="'bar'"` : `UPDATE example SET foo = 'bar'`
3. `default="bar"` : `UPDATE example SET foo = bar`

Example one and two are valid SQL, example three is not. (Or worse, it is valid, but refers to another column named *bar*)

## Value Constants ##

Columns can be constrained to specific values using the &lt;value-constraint&gt; and &lt;accepted-value&gt; tags. Within a column definition, define a &lt;value-constraint&gt with a unique *name*, and then within that define one &lt;accepted-value&gt; per acceptable value. 

````
<column name="sack" data-type="varchar(100)" default="'red_sack'" nullable="false">
    <value-constraint name="atd_santaslist_">
        <accepted-value value="blue_sack"/>
        <accepted-value value="red_sack"/>
        <accepted-value value="green_sack"/>
    </value-constraint>
</column>
````

Note that the values don't suffer from the same issues as default values. Notice how the *value* attribute on the &lt;accepted-value&gt; tag doesn't have the quotes, but the default value does.

## Primary Keys ##

````
<primary-key name="atd_santaslist_gift_pk">
    <columnref name="pk1" />
</primary-key>
````

All tables should have a primary key defined, which we've done with the &lt;primary-key&gt; tag. All primary keys must have a unique name, again defined using the *name* attribute. The columns that make up the primary key are defined using one &lt;columnref&gt; tag per column.

## Foreign Keys ##

````
<foreign-key name="atd_santaslist_gift_fk1" reference-table="users" on-delete="cascade">
    <columnref name="user_pk1" />
</foreign-key>
````

Foreign keys are defined in much the same way as primary keys except they are defined using the &lt;foreign-key&gt; tag and are required to specify what table the foreign references with the *reference-table* attribute. You may also choose to define what to do when the foreign data is deleted with the *on-delete* attribute. Its values can be either *cascade* which will delete the rows in this table too or *setnull* which will set the field to null.

You have to be very careful with foreign keys as you can actually break core Blackboard functionality if not done correctly. Specifically, you need to be aware not to prevent the deleting of the foreign data. This scenario can arise if you don't define the *on-delete* attribute or you define it as *setnull* on a non-nullable column when referencing core Blackboard tables.

## Indexes ##

````
<index name="atd_santaslist_gift_ak1" unique="true">
    <columnref name="user_pk1" />
    <columnref name="descr" />
</index>
````

You can define additional indexes on the table to either speed up queries or for uniqueness constraints. These are set up very similar to primary and foreign keys except they are defined using the &lt;index&gt; tag. As with the keys, you need to define a unique *name* for the index. If you want to enforce uniqueness, then define the *unique* property as *true*.

# Important Notes #

Two very important thing to note are:

* **All** database entities **must** start with the vendor id, followed by an underscore followed by the building block handle and a trailing underscore. As an example, here our vendor id is *atd*, and the handle is *santaslist* therefore, all entities started with *atd_santaslist_*. Entities that do not follow this naming standard are ignored and not created or updated. When I say *all* entities, this includes keys and indexes too.
* Due to restrictions in Oracle no entities name can be over **32 characters** in length, including the above mentioned required prefix. This can make naming things a bit of a challenge sometimes, especially if your building block handle is quite long.

# Booleans #

You may have noticed that *boolean* is conspicuously missing from the list of available data types. Experienced Blackboard admins, who've spent any amount of time in the database, may have noticed that Blackboard never uses booleans. Instead they use a single *char* and constrain it to either *Y* or *N*. Blackboard always names these columns with the suffix *_ind*.

This would have been a legacy from a time when no databases agreed on the implementation of boolean. Lets look at an example of a boolean column in the schema.xml:

````
<column name="naughty_ind" data-type="char(1)" default="'N'" nullable="false">
    <value-constraint name="atd_santaslist_naughy_con">
        <accepted-value value="Y"/>
        <accepted-value value="N"/>
    </value-constraint>
</column>
````

# Blackboard's Behavior #

Now that we've defined our tables in the schema.xml file, and added the references to the bb-manifest.xml file, Blackboard will automatically create these tables for you when installing the building block. It will even update them if you modify the structure, though you may need to be careful about things like supplying default values for nullable columns. *Make sure you test*.

# What else can this do? #
There are many other things that can be done with the Schema API including [defining stored procedures]({{< ref "post/b2_stored_procedures.md" >}}), functions, triggers and views. It can also be used to define SQL scripts that run before or after upgrading the building block to do more complex upgrade tasks if required. I hope to cover these more in the future.

