---
next: "/basics"
toc: true
date: "2016-10-12T16:38:34-04:00"
title: "Adding and Accessing Data"
weight: 4
prev: "/basics/tutorial-2"

menu:
  main:
    parent: Tutorial
    identifier: "Adding and Accessing Data"
    weight: 3
---

## Adding data in Python

The simplest way to add data is through Python. Assuming our suite is in an object called
`api`, this is how we add data:

~~~python
>>> api['todo-app']['items'].create({
...     "title": "Remember the milk.",
...     "created": datetime.utcnow()
... })
<examples.todo.Item object at 0x10f611160>

>>> next(iter(api['todo-app']['items']))
'642478b1-03bb-45e2-9f6d-e90e90d1b7cf'
~~~

The second line demonstrates that Sondra indeed follows the dict pattern closely.  Getting an
iterator over the collection returns a generator that yields keys. This is done via a single
cursor, and is thus very efficient.  You can also use `.keys()` and `.values()` and even `.items()`
as you expect to, yielding primary keys, Document objects, and pairs as they are supposed to.  

~~~python
>>> items = api['todo-app']['items']
>>> len(items)
1
>>> items.keys()
KeysView(<examples.todo.Items object at 0x110d40a90>)
>>> items.values()
ValuesView(<examples.todo.Items object at 0x110d40a90>)
>>> items.items()
ItemsView(<examples.todo.Items object at 0x110d40a90>)
>>> '642478b1-03bb-45e2-9f6d-e90e90d1b7cf' in items
True
>>> items['642478b1-03bb-45e2-9f6d-e90e90d1b7cf'] in items
True
>>> first_item = items['642478b1-03bb-45e2-9f6d-e90e90d1b7cf']
>>> first_item
<examples.todo.Item object at 0x10f611160>
>>> first_item.obj
OrderedDict([('complete', False), ('created', '2016-10-13T18:45:08.255000+00:00'),
    ('title', 'Remember the milk.'), ('id', '642478b1-03bb-45e2-9f6d-e90e90d1b7cf')])
>>> first_item == '642478b1-03bb-45e2-9f6d-e90e90d1b7cf'
True
~~~

The last line deserves some explanation. An item is always equal to its primary key, even though we
are really comparing strings and objects. This may be counterintuitive, but it makes for fewer
queries internally and is thus more efficient than the obvious implementation.

Another thing to note is that Sondra takes care to map data to its proper datatype whenever possible.
There's a little "magic" in this, but it is explained deeper in the documentation.  For example,
let's look at our "created" attribute:

~~~python
>>> first_item['created']

datetime.datetime(2016, 10, 20, 16, 21, 54, 546629)
~~~

Sondra uses "value handlers", found in the `sondra.documents.valuehandlers` section of the sondra
code to translate between Python datatypes, JSON datatypes, and RethinkDB datatypes. To determine
the datatype, you need look no further than the schema itself. The following shows what schema
attributes  Sondra uses to treat values specially:

- `{... "format": "date-time"}` - Handled as a Python datetime, JSON string, RethinkDB date.
- `{... "geo": true}` - Handled as a Python dict-like object (a geometry object if Shapely is installed,
  otherwise it will fall back to a dict), JSON object (as GeoJSON geometry), RethinkDB geometry
  type.

Sondra is compatible with the base datetime library and with [arrow](http://crsmithdev.com/arrow/)

## Adding and accessing data over HTTP(S)

For this section, we're going to assume you will get around to authentication. A basic
authentication and authorization app exists in the Sondra codebase, but you will probably want to
use other, more well-tested frameworks to handle authentication and authorization of web requests in
Flask. This is just fine and well supported. For now, we will assume that a POST will work without
authorization, so you can follow the tutorial without reading extra material.

Sondra is fairly standard when it comes to REST metaphors, but for reference, here is how HTTP
commands map to operations on the API:

- `GET` - **Retrieve**
  - Retrieve a document,
  - list of documents, or
  - make a simple method call
- `POST` - **Add or Replace**
  - Add items to a collection.
  - Replace an existing document.
  - Make a more complex method call.
  - As a special case, you can use JSON to map any of the other commands to a POST. More details in
    the source code or the detailed documentation to come.
- `PUT` - **Replace**
  - Replace items in a collection.
  - Replace an existing document.
- `PATCH` - **Update in Place**
  - Make updates to items in a collection.
  - Make updates to a single document. (Merges dicts to create updates)
- `DELETE` - **Delete**
  - Delete items from a collection. Delete a single document.

We'll use the Python requests library. By default, Flask exposes itself over port 5000 on localhost,
so that's the assumption we will make. We will also make the assumption that you are *not* changing
the default api base path, which is simply `api`.  First, let's get the record we've already
created:

~~~python
>>> rsp = requests.get('http://localhost:5000/api/todo-app/items')
>>> rsp.json

[{'id': '642478b1-03bb-45e2-9f6d-e90e90d1b7cf', 'complete': False,
  'created': '2016-10-13T18:45:08.255000+00:00', 'title', 'Remember the milk.'}]

>>> requests.get('http://localhost:5000/api/todo-app/items/642478b1-03bb-45e2-9f6d-e90e90d1b7cf').json()

{'id': '642478b1-03bb-45e2-9f6d-e90e90d1b7cf', 'complete': False,
 'created': '2016-10-13T18:45:08.255000+00:00', 'title', 'Remember the milk.'}
~~~

Now how about adding data?  Let's actually use an example in Javascript, since that's how an API
like this will most commonly be used.  Here we're using the "fetch" API, but you could do the same
with a standard XMLHttpRequest.

~~~Javascript
fetch('http://localhost:5000/api/todo-app/items', {
  method: 'POST',
  mode: 'cors',
  body: JSON.stringify({
    title: 'Remember the cheese',
    complete: false,
    created: new Date()
  })
});
~~~

### Replace the document on the server

~~~Javascript
fetch('http://localhost:5000/api/todo-app/items/642478b1-03bb-45e2-9f6d-e90e90d1b7cf', {
  method: 'POST',
  mode: 'cors',
  body: JSON.stringify({
    title: 'Remember the milk',
    complete: true,
    created: new Date()
  })
});
~~~

This will actually replace the document with the given ID. If you omit any properties from the
document, they will be omitted in the replacement, since this effectively destroys the old copy and
puts a new one in its place on the database.  What if we just want to say that the item has been
completed?  After all, we don't want to update the "created" field, since now the data is wrong.

### Update properties on the document on the server.

~~~Javascript
fetch('http://localhost:5000/api/todo-app/items/642478b1-03bb-45e2-9f6d-e90e90d1b7cf', {
  method: 'PATCH',
  mode: 'cors',
  body: JSON.stringify({complete: true})
});
~~~

The `PATCH` method only updates the properties that you give to the request, leaving the others
unchanged.
