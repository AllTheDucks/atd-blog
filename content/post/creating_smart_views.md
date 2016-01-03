+++
date = "2016-01-03T15:42:00+10:00"
draft = false
title = "Creating Smart Views for Groups with the Blackboard API"
description = "Creating a Smart View for a Group with the Blackboard API requires jumping through a few hoops."
keywords = ["blackboard", "smart view", "api", "groups"]
+++

*By Shane Argo.*

There has been a fair amount of discussion on the bb_opensrc mailing list and other communities lately about creating a Smart View for a group using the Blackboard API. It's actually fairly simple, but there are some non-obvious things you need to do to get it working. This information should prove useful for creating Smart Views with other (non group) criteria as well.

# Smart View == Custom View #
As is fairly normal for the Blackboard API, the UI has a different name for the same thing. What the Blackboard UI calls a *Grade Center Smart View*, the API calls a *Gradebook Custom View*. 

# Smart View Definitions #
Smart Views are defined using a JSON object. Here is an example block of JSON for a Smart View for a group:

````json
"searchType":"grpmem",
"formula":"1",
"criteria":[
   {
      "fid":"1",
      "cid":"GM",
      "ctype":"GM",
      "cond":"eq",
      "value":"gr_123"
   }
],
"display":{
   "items":"allItem",
   "showhidden":false
}
````

There's a lot of stuff here, but you don't need to worry about most of it.  If you do want to figure out the structure of this JSON, the best way is to create a few Smart Views through the Blackboard UI and then find the resultant JSON in the database.

Looking within the `gradebook_custom_view` table, you'll find the JSON in a column named `json_text`.

If you're trying to create a Smart View for a single group you can just take the JSON from above and vary the value of the `criteria.value` field.

# Aliases #
So what is `criteria.value`? And where does `gr_123` come from? In Blackboard terminology, it's an *alias*. When creating a Smart View, you also need to create an alias for every Group and then use the alias when referencing the Group in the definition JSON.

The name of the alias **must** be "gr_" followed by the internal id of the the group, i.e. the group's `pk1`.

So, we need to firstly create a map for these alias `String`s to the internal group `Id` objects:
````
String alias = String.format("gr_%s", ((PkId) group.getId()).getKey());
Map<String, Id> aliasMap = new HashMap<String, Id>(1);
aliasMap.put(alias, group.getId());
````

and then secondly pass this to the Blackboard API when creating the smart view:
````
gradebookCustomView.setAliases(aliasMap);
````


Following is a full example of creating a new Smart View

````java
public static final String CUSTOM_VIEW_JSON_PATTERN = "\"searchType\":\"grpmem\",\"formula\":\"1\", \"criteria\": [ {\"fid\":\"1\",\"cid\":\"GM\",\"ctype\":\"GM\",\"cond\":\"eq\",\"value\":\"%s\"}], \"display\":{\"items\":\"allItem\",\"showhidden\":false}";

String alias = String.format("gr_%s", ((PkId) group.getId()).getKey());
Map<String, Id> aliasMap = new HashMap<String, Id>(1);
aliasMap.put(alias, group.getId());

GradebookCustomView gradebookCustomView = new GradebookCustomView();
gradebookCustomView.setViewType(GradebookCustomView.SmartViewType.CUSTOM);
gradebookCustomView.setCourseId(course.getId());
gradebookCustomView.setHasUserIds(false);
gradebookCustomView.setJsonText(String.format(CUSTOM_VIEW_JSON_PATTERN, alias));
gradebookCustomView.setAliases(aliasMap);
gradebookCustomView.setTitle(title);

GradebookCustomViewDAO gradebookCustomViewDAO = GradebookCustomViewDAO.get();
gradebookCustomViewDAO.persist(gradebookCustomView);
````