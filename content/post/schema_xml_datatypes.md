+++
date = "2014-07-10T20:56:01+11:00"
author = "shane"
draft = false
title = "schema.xml data types and using grep to explore"
keywords = ["database", "blackboard", "b2", "schema.xml", "grep"]
+++

I’m currently in the process of developing a relatively complex Blackboard building block which includes persistence of data into custom tables in the database.

For those who don’t know, this is done by defining all the database entities in a file named schema.xml in the building block package. The specifics of this is worthy of a post of it’s own, but this isn’t what I wish to focus on for this post.

The only documentation of this capability can be found on [the rather out-dated EduGarage](http://www.edugarage.com/display/BBDN/Schema+Definitions). This document is both limited and, in at least one place, wrong. (For reference, when enumerating the possible values of on-delete in the foreign-key element they are listed as “setnull” and “delete”, but they should be “setnull” and “cascade”.

This document lists only four data-types, but surely there are more? What about storing dates? Floating Points? Longs?  Surely this should be possible. I happen to know that this is used by Blackboard in their own building blocks, so I decided to do some exploration.

I SSHed into the developer VM (available from Blackboard’s support portal) and changed to the plugins directory:

````
cd /usr/local/blackboard/content/vi/BBLEARN/plugins
````

And then used the *nix grep utility to search for all unique references to data-types in files named schema.xml:

````
[vagrant@bbdev plugins]$ grep -PIroh --include=schema.xml \
 "data-type=\"[^\"(]*(\"|\()" . | sort | uniq
data-type="bigint"
data-type="char(
data-type="datetime"
data-type="float"
data-type="id"
data-type="image"
data-type="int"
data-type="integer"
data-type="ntext"
data-type="numeric"
data-type="numeric(
data-type="nvarchar(
data-type="NVARCHAR(
data-type="text"
data-type="varchar(
````

Well, there we go. Turns out there are a lot more available.

I chose to search for and include the opening bracket so that I’d know which data types allowed for, or required, lengths. We can gather that it probably isn’t case sensitive either, given that a capitalised version of nvarchar has been used at least once.

If you want to see more about how these are used, instead of just what is available, remove the o and h from the grep switches, add an n, and don’t filter the results with sort and uniq:

````
[vagrant@bbdev plugins]$ grep -PIrn --include=schema.xml "data-type=\"id" .
````
The result will be a list of every single file and the line number that the data type is referenced in.

This turns out to be quite a useful technique for learning about features of the Blackboard API. For example, when I wanted to see how one of the JSP tags in the Blackboard libraries was used, so I searched “*.jsp” files for it.

**Bonus**: The core database entities for the core Blackboard components are created the same way. They can be found here:

````
/usr/local/blackboard/system/database/vi/
````