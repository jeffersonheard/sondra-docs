---
toc: true
date: "2016-10-12T16:38:34-04:00"
title: "Working with Applications"
weight: 1

menu:
  main:
    parent: Topical Guides
    identifier: "Applications"
    weight: 1
---

Applications are the highest level grouping for apis. A Sondra application subclass corresponds to a
single database in the RethinkDB backend. Like collections and documents, you can attach methods to
the application class and expose them as API endpoints.

## Using Applications to Structure your API

### Sample definition of your API
~~~python

from sondra.collection import Collection
from sondra.document import Document
from sondra.application import Application
from sondra.suite import Suite
from sondra.schema import S

class Items(Collection):
    document_class = Item  # this is the class that as instances per-record
    indexes = ["title", "complete"]  # the fields to build indexes on.
    order_by = ["created"]  # Sondra treats "format": "date-time" as a RethinkDB Date.

class TodoApp(Application):
    collections = (Items,)  # For now we'll just define the collections

class TodoSuite(Suite):
    cross_origin = True  # append CORS headers
    debug = True  # extra logging
~~~


## Application Object Lifecycle

1. One creates an application object and assigns it to a particular suite in the constructor.
2. The application ensures that its database exists.
3. The application creates all its collections and registers them with itself.

### Signals

Unless otherwise noted, all signals are sent with the application instance as their first and only
argument.

* `pre_init` - Sent after the Application registers with the suite, but before it initializes any collections.
* `post_init` - Sent after the Application object initializes its collections.
* `pre_registration` - Sent before the Application registers with the suite.
* `post_registration` - Sent after the Application registers with the suite.
* `pre_create_database` - Sent before the Application's database is created / ensured to exist.
* `post_create_database` - Sent after the Application's database is guaranteed to exist, but before tables and indexes are checked.
* `pre_create_tables` - Sent after the Application's database is guaranteed to exist, but before the tables and indexes are checked.
* `post_create_tables` - Sent as soon as all tables are created.
* `pre_delete_database` - Sent before databases and all tables will be deleted.
* `post_delete_database` - Sent after the database has been deleted.
* `pre_delete_tables` - Sent before tables will be deleted.
* `post_delete_tables` - Sent after tables have been deleted.
