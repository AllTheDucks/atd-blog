+++
date = "2014-11-22T23:09:00+10:00"
draft = false
title = "Using Blackboard's ORM Framework"
description = "You can use Blackboard's built-in ORM framework for accessing the database instead of using Hibernate, JDBC, etc."
keywords = ["blackboard", "orm", "b2"]
+++

If you've ever needed to access the Blackboard database from a building block, you'll know that it can be a bit painful.  Historically, Blackboard hasn't made it easy to use a pre-defined datasource, so using Hibernate or JPA has invariably meant hard-coding connection strings, either in your java code, or ORM config files.  Alternatively, you could wrap all the Bb DB Access objects to make them look like a normal Datasource, but this was a bit of a pain too, and JPA just doesn't want to play ball unless you're using JNDI.  I think this has changed somewhat since I last looked at it, but I haven't been willing to spend the hours required to investigate.

Anyway, there is another option.  Believe it or not, Blackboard has had a built-in ORM framework for a long time, they've just never made it public.  Danny Thomas recently gave a presentation about it at the Australian Bb Dev Con, and we at All the Ducks have done a bit more digging to figure out and document how to use it. 

In this, part one, we'll show you how to write code to fetch pre-defined Blackboard Legacy Entities.  We'll use *Legacy* to refer to the old-style Blackboard entities which are loaded via DbLoader classes.

So, step one, you need to find the Entity you want to use.  For our example, we're going to use ````blackboard.data.announcement.Announcement````.  These *legacy* entities have an associated ````MAP```` field, which contains the mapping from Database Columns to java field names. 


Next you need to extend the ````blackboard.persist.dao.impl.SimpleDAO```` class. This contains a few convenience methods that you can use to fetch Entities, such as ````loadAll```` and ````loadById````.  
At this point, your code will look something like the following.


````
public class MyAnnouncementDAO extends SimpleDAO<Announcement> {

    public MyAnnouncementDAO() {
        super(AnnouncementDbMap.MAP);
    }
}
````

From this point, you can call methods like ````myAnnouncementDao.loadById(id)```` or ````myAnnouncementDao.loadAll()````.  Your code might look something like the following. 
````
MyAnnouncementDAO myDao = new MyAnnouncementDAO();

List<Announcement> announcements = myDao.loadAll();
````


Now, a method like ````loadAll```` is, apart from being quite dangerous from a performance/system stability perspective, not actually all that useful.  Let say, for example that we wanted all the announcements with a start date in the last 2 days.  The method signature might look something like this;
````
public List<Announcement> loadAnnouncementsSinceDate(Calendar cal);
````

To implement this, we need to use Blackboard's ````Query```` classes.   For this method we'll use the ````SimpleSelectQuery````.

The final code will look like the following. 

````
public List<Announcement> loadAnnouncementsSinceDate(Calendar cal) {
    SimpleSelectQuery query = new SimpleSelectQuery(this.getDAOSupport().getMap());

    Criteria criteria = query.getCriteria();

    criteria.add(criteria.greaterThan("RestrictionStartDate", cal));

    return getDAOSupport().loadList(query);
}
````
Let's break this down a bit.  When dealing with *legacy* entities, SimpleSelectQuery expects the entity's MAP as a constructor argument.  We've already set this onto the DAO, and it's referenced by the DAOSupport class, so we can just retrieve it from there.

You then need to add extra ````Criteria```` to the query, in the form of ````blackboard.platform.query.Criteria```` objects.   First you need to get the Criteria from your new query, then call instance methods on it to add extra Criteria instances.

For the purposes of our method, we want to fetch all records where the *RestrictionStartDate* is greater than the date represented by the *cal* object.  The field name, in this case *RestrictionStartDate*, is the same as the getter/setter method with the *get* or *set* prefix removed.

Finally, use the DAOSupport object to load the list of entities using the query.

If you've used the Blackboard API for any length of time, you'll notice that there are two sets of ORM infrastructure.  One for the DBLoader/DBPersister classes, and another for the DAO style classes.   The DAO classes use Annotations instead of the MAP classes to declare the table/column mappings, and you can use these annotations in your own Entities.   In a future post we'll investigate how you can do this. 

For now, here's the full code for the DAO described in this post. 


````
package edu.myinst.myproject.dao;

import blackboard.data.announcement.Announcement;
import blackboard.persist.announcement.impl.AnnouncementDbMap;
import blackboard.persist.dao.impl.SimpleDAO;
import blackboard.persist.impl.SimpleSelectQuery;
import blackboard.platform.query.Criteria;

import java.util.Calendar;
import java.util.List;


public class AnnouncementDAO extends SimpleDAO<Announcement> {

    public AnnouncementDAO() {
        super(AnnouncementDbMap.MAP);
    }

    /**
     * Loads any announcements with a start_date more recent than the specified
     * date.
     *
     * @return
     */
    public List<Announcement> loadAnnouncementsSinceDate(Calendar cal) {
        SimpleSelectQuery query = new SimpleSelectQuery(this.getDAOSupport().getMap());

        Criteria criteria = query.getCriteria();

        criteria.add(criteria.greaterThan("RestrictionStartDate", cal));

        return getDAOSupport().loadList(query);
    }

}
````




