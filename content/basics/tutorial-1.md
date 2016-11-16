---
next: "/basics/tutorial-2"
toc: true
date: "2016-10-12T16:38:34-04:00"
title: "Your first Sondra API"
weight: 1
prev: "/basics"

menu:
  main:
    parent: Tutorial
    identifier: "Getting Started"
    weight: 0
---

### Introduction

This tutorial serves as a basic introduction to Sondra, its data model, and how you
can create an API. The classic example web API is that of a to-do list application. Our tutorial
will build it up in stages, starting with the basic structure of an application, followed by data
modeling, and then we'll add exposed methods to see how those work.

Finally, you'll work through accessing the API via Javascript (ES2015) and Python.

To do this tutorial, you will need the following installed and ready to go:

* Sondra - https://github.com/JeffHeard/sondra.git
* RethinkDB - https://rethinkdb.com
* Python 3 - https://www.python.org
* Your favorite text editor or IDE. I like [Atom](https://www.atom.io/)

### First Steps

First thing's first. Let's make sure Sondra has all its requirements.  Assuming you have a virtual
environment setup already, all you have to do is install requirements.

~~~bash
$ cd sondra
$ pip install -r requirements.txt
$ export PYTHONPATH=$PYTHONPATH:$PWD  # if you want to develop in a different directory or run the examples.
~~~

Let's start with some imports.

~~~python
from sondra.collection import Collection
from sondra.document import Document
from sondra.application import Application
from sondra.suite import Suite
from sondra.schema import S
~~~

These imports serve to introduce us to the basic building blocks of Sondra: Application, Collection,
and Document. We will subclass each to create our application.  The __Document__ subclass defines
the schema of a record in [RethinkDB](https://rethinkdb.com/) and all the methods that operate
directly on a record instance.

The __Collection__ subclass defines the way a document relates to the database. It defines things
such as primary key and indexes and methods that operate on the collection as a whole. Methods
defined on the collection class are conceptually similar to class methods in Python.

The __Application__ subclass defines a group of collections that serve a common purpose.  The
application will often contain schema fragment definitions that are common across multiple related
collections and methods that apply to multiple collections across the application.

The __Suite__ subclass defines the complete API suite, including configurations and database
connections. Applications are added to the Suite.

Finally __S__ is a utility module whose functions return dictionaries that fit the format of
[JSON Schema](http://json-schema.org/) and hyper-schema.

### Modeling Data

So let's start by creating a Document that models a to-do list item:

~~~python
from sondra.collection import Collection
from sondra.document import Document
from sondra.application import Application
from sondra.suite import Suite
from sondra.schema import S

class Item(Document):
    schema = {
      "type": "object",
      "required": ["title"],
      "properties": {
        "title": {"type": "string", "description": "The title of the item"},
        "complete" {"type": "boolean", "default": False},
        "created": {"type": "string", "format": "date-time"}
      }
    }
~~~

Now we've defined a simple item that has a title, a space to mark whether the to-do item is
complete, and a creation date.  Note that schemas are just Python dicts. As long as your dictionary
and all its elements are compatible with the built-in `json` package, your schema will work.
However, this syntax is rather verbose and prone to typo-induced bugs. Therefore the `S` package
provides a bunch of utility functions that generate schema objects. These functions return plain old
dicts, so there's functionally no difference between the two syntaxes, but the S module is safer:

~~~python
class Item(Document):
    schema = S.object(
        required=['Title'],
        properties=S.props(
            ("title", S.string(description="The title of the item")),
            ("complete", S.boolean(default=False)),
            ("created", S.datetime()),
    ))
~~~

That's much shorter! Again, there's no magic to the `S.*` functions. They're just shorthand and
provide a bit more checking.  It's the same schema as above.  One difference is that the first
example does not enforce any order (except in Python 3.6!) on the dictionary elements.  The version
in the most recent example does, because `S.props` uses an `OrderedDict` instead of a plain `dict`
Now let's see how to get it into a collection and an application, and thus into the database. The
next three classes define the collection (RethinkDB table), application (RethinkDB database), and
the suite (the full group of applications, served on a single tree from a single domain):

~~~python
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

### Getting it into RethinkDB

Now that we have all the definitions set up, let's get our data into the database:

~~~python
>>> todo = TodoSuite()
>>> TodoApp(todo)
>>> todo.validate()
>>> todo.ensure_database_objects()

INFO:TodoSuite:Connection established to 'default'
INFO:TodoSuite:Suite base url is: 'http://localhost:5000/api'
INFO:TodoSuite:Docstring processor is {0}
INFO:TodoApp:Registering application todo-app
INFO:TodoSuite:Registered application TodoApp to http://localhost:5000/api/todo-app
INFO:TodoApp:Creating collection for todo-app/items
INFO:TodoSuite:Checking schemas for validity
INFO:TodoSuite:+ todo-app
INFO:TodoSuite:--- items
~~~

The call to `todo.ensure_database_objects()` creates / ensures the existence of tables and indexes and even
databases for everything that's a part of the suite.  It should always be called once when your
application initializes. At the most basic level, you're now ready to expose your API to your
clients now (we'll handle authentication later), and you can play around with it in Python as well.
In Python, Sondra tries to be as pythonic as possible. All levels of the API are exposed as
dictionary-like objects. The keys are dash-cased versions of the classnames (this can be overridden,
  but is the default).  
