+++
date = "2014-11-19T15:48:00+10:00"
draft = false
title = "Lessons Learned: B2s and Stored Procedures"
description = "Blackboard building blocks can define stored procedures (and other database objects) but there are some gotchas."
keywords = ["blackboard", "stored procedures", "mssql"]
+++

# Stored Procedures in B2s #

When writing a Blackboard building block, you can define your own database objects, including tables, keys and stored procedures/functions. There's far too much functionality to talk about in this blog post, but the gist of it is this:

At the root of your B2, create a directory named schema and within it create a directory for each database you want to create database objects in. This is likely to be the main database (referred to as *instance*), but it may also be the stats database. As an example:

````
/schema/instance
/schema/stats
````

Now modify your bb-manifest.xml file and add the following element:

````
<schema-dirs>
    <schema-dir dir-name="instance" />
    <schema-dir dir-name="stats" database="stats" />
</schema-dirs>
````

Obviously, if you're not using the stats db you can leave out that line. Also, if you do not define the database, it refers to the main (or *instance*) database. From here I will be focusing on a single database, but everything applies equally to both.

Within, the ````schema/instance```` directory, create [a schema.xml file](http://www.edugarage.com/display/BBDN/Schema+Definitions). There is a lot of stuff that isn't this documentation (as of 19th October 2014), including the topic of this post, defining stored procedures. There are a couple of ways of doing this:


**Option 1**
````
/schema/instance/stored_procedures/my_procedure.sql.db-mssql
/schema/instance/stored_procedures/my_procedure.sql.db-oracle
/schema/instance/stored_procedures/my_procedure.sql.db-pgsql
````


**Option 2**
````
/schema/instance/stored_procedures.db-mssql/my_procedure.sql
/schema/instance/stored_procedures.db-oracle/my_procedure.sql
/schema/instance/stored_procedures.db-pgsql/my_procedure.sql
````

The first has a single "stored procedures" directory and a file per database, the second has a directory per database and a file in each for each stored procedure. Blackboard understands both, so this is pretty much a personal preference thing.  I do, however, think it's nice having the meaningful ".sql" file extension with the second. Depending on the option you choose you must reference these within your schema.xml file:


**Option 1**
````
<stored-procedures>
    <stored-procedure name="my_procedure.sql.db-mssql" comment="My stored procedure" availability="SqlServer" />
    <stored-procedure name="my_procedure.sql.db-oracle" comment="My stored procedure" availability="Oracle" />
    <stored-procedure name="my_procedure.sql.db-pgsql" comment="My stored procedure" availability="PostgreSQL" />
</stored-procedures>
````


**Option 2**
````
<stored-procedures>
    <stored-procedure name="my_procedure.sql" comment="My stored procedure" availability="SqlServer" />
    <stored-procedure name="my_procedure.sql" comment="My stored procedure" availability="Oracle" />
    <stored-procedure name="my_procedure.sql" comment="My stored procedure" availability="PostgreSQL" />
</stored-procedures>
````

Great! Now blackboard will handle the creation of these database objects for you. **But**...

# Gotchas #

**Oracle's limit on name length**

Oracle has a limit of 32 characters for the length of the name of any object (table, column, sequence, procedures, etc.) in the database. Lesson Learned? **Keep everything under this limit**.

**MSSQL's inability to "CREATE OR REPLACE"**

Oracle and PostgreSQL both support the ````CREATE OR REPLACE```` syntax when defining procedures/functions. This is great for building blocks, as you don't know if the user is upgrading, or installing the building block for the first time. Unfortunately, MSSQL doesn't support it. In the normal world, there are ways to do it, like checking for the object's existence and dropping it if it's found prior to creating. However, this will not work with B2s (for complicated reasons, specifically the way MSSQL handles CREATE). Thankfully, Blackboard has accounted for this. For MSSQL, Blackboard will drop the stored procedure for you before running your creation script. But, how does it know the name of your procedure without parsing your SQL? The answer is the *file name*. 

Lesson Learned? **Make sure your file name matches your procedure name!** If your file name doesn't match the procedure name, everything will work exactly as normal, until you try to upgrade the building block on an MSSQL instance.

## Story Time ##

I always develop using the fantastic Blackboard Dev VM, which is backed by PostgreSQL, which doesn't have either of these Gotchas. So it's quite easy to let a database object's name sneak over the 32 character limit, and not know until you deploy to an Oracle instance.

I developed a building block where I made this mistake. When the B2 was deployed to a test Oracle instance, the error was caught. I went back and renamed the procedure to keep it under this limit, but didn't rename the SQL files.

Of course, now I hit the second gotcha, but I didn't know about it at the time. It wasn't until I'd spent a long time trying to figure out why the install (actually, reinstall because the issue only occurs during upgrades/reinstalls) was failing on MSSQL server that I figured this one out. And, thus this blog post was born.

