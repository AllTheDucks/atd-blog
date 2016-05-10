+++
date = "2016-05-10T12:23:00+10:00"
draft = false
title = "Disabling Blackboard's SQL Query Cache for Development"
description = "When developing Building Blocks for Blackboard that use 'External Queries', the cache can be a real pain but it can be disabled for development purposes."
keywords = ["blackboard", "external queries", "sql", "cache", "development"]
+++

*By Shane Argo.*

Blackboard has an API for loading SQL queries from an external file, which can be very useful. This article is not going to cover the API itself, but I will hopefully get a chance to blog about it in the future.

The API provides a number of advantages over inlining the query in your java code but, by default, the query is cached. It gets loaded once, on first use, and then cached in memory for future reuse. The introduction of this caching is one of the reasons that Blackboard now recommends a rolling restart of Blackboard after updating a building block.

This caching is a good thing, you don't want to have to read the query from the file over and over in a production environment. (Although, it would be nice if the cache was flushed when updating a Building Block.) Having said that, it is a pain to have to restart Blackboard in a development environment just to have the query reloaded. Needless to say, this slows down development to a snails pace.

Fortunately, there is a configuration item that can be set which turns off this caching altogether.

In your development environment navigate to the `config` directory in the Blackboard installation home. In this directory, you'll find a file named `external-sql.properties.bb`. (Make sure you are looking at the one ending in `.bb`). Its contents will look like this:

```
resource.loader=internalsql, report

# internal sql loader
internalsql.resource.loader.class=blackboard.persist.impl.external.ExternalSqlLoader
internalsql.resource.loader.path=@@bbconfig.basedir@@/config/internal/sql
internalsql.resource.loader.cache=true

# reporting loader
report.resource.loader.class=blackboard.persist.impl.external.ExternalSqlLoader
report.resource.loader.path=@@bbconfig.base.shared.dir@@/reporting/reportdefs
report.resource.loader.cache=true
```

See that `internalsql.resource.loader.cache` property? That's the one. We want to change it to `false`:

```
internalsql.resource.loader.cache=false
```

Now you'll need to run `tools/admin/PushConfigUpdates.sh` (or `tools\admin\PushConfigUpdates.bat` in Windows) to persist the changes.

Of course, this means that the SQL will be loaded from the disk every time it's needed, which is the desired effect, but clearly is not a good idea in a production environment.

