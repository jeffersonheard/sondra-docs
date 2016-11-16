---
next: "/basics"
toc: true
date: "2016-10-12T16:38:34-04:00"
title: "Accessing Schemas"
weight: 3
prev: "/basics/tutorial-2"

menu:
  main:
    parent: Tutorial
    identifier: "Accessing Schemas"
    weight: 2
---

### Schema Endpoints

Let's look at this in a browser. Run the module in a terminal window, open your favorite browser, and
surf to http://localhost:5000/api;schema.

~~~json
{
    "title": "Sondra-Based API",
    "definitions": {
        "filterOps": {
            "enum": [
                "with_fields",
                "count",
                "max",
                "min",
                "avg",
                "sample",
                "sum",
                "distinct",
                "contains",
                "pluck",
                "without",
                "has_fields",
                "order_by",
                "between"
            ]
        },
        "timedelta": {
            "type": "object",
            "required": [
                "start",
                "end"
            ],
            "properties": {
                "hours": {
                    "type": "integer"
                },
                "days": {
                    "type": "integer"
                },
                "seconds": {
                    "type": "number"
                },
                "minutes": {
                    "type": "integer"
                }
            }
        },
        "spatialOps": {
            "enum": [
                "distance",
                "get_intersecting",
                "get_nearest"
            ]
        }
    },
    "type": "object",
    "id": "http://localhost:5000/api;schema",
    "applications": {
        "todo-app": "http://localhost:5000/api/todo-app"
    },
    "description": "*No description provided.*"
}
~~~

That's pretty neat! A few basic datatypes exist in the definitions, mainly to define how filtering
and query-sets work. We'll get into those later.  For now, we'll focus on what we just created.  The
suite schema doesn't show us the data-type that we defined, but it does show a list of applications
and their endpoints in `.applications`. Note that our application is no longer CamelCase, but has
been turned into kabob-case. This is the default, but it can be configured otherwise. All levels
of the API are turned from CamelCase or underscore_separation into kabob-case by default, for
consistency, and because this is more typical of url patterns than the other case types.

### Application Schema

Let's check out the schema to todo-app.  We'll use the url
in the schema followed by `;schema`. Sondra is very regular about the way it treats urls. Formatting
and pragmas are handled using url parameters. This is different than changing the endpoint for the
file-type, which is typical of many other frameworks. The reason for this is that often, there are
multiple ways to render data and keep it in the same file type.  For instance, the `schema` and
`json` types allow you to control the ordering and whether data is pretty-printed with lines and
indentations. Other formats allow other parameters as we'll see later on.

~~~json
{
  "id": "http://localhost:5000/api/todo-app;schema",
  "definitions": {},
  "collections": {
    "items": "http://localhost:5000/api/todo-app/items"
  },
  "type": "object",
  "title": "Todo App",
  "methods": {},
  "description": "*No description provided.*"
}
~~~

This is the application schema.  Once again, this is rather succinct. There's not a lot of
information in here, but if there were methods defined at the application level we would see them
show up here.  One can also add application level schema fragment definitions in the class and
they would show up here.

### Collection Schema

Let's move on to the items themselves. Go to http://localhost:5000/api/todo-app/items;schema;indent=2.

~~~json
{
  "type": "object",
  "properties": {
    "title": {
      "type": "string",
      "title": "Title",
      "description": "The title of the item"
    },
    "complete": {
      "type": "boolean",
      "title": "Complete",
      "default": false
    },
    "created": {
      "type": "string",
      "title": "Created",
      "format": "date-time"
    },
    "id": {
      "type": "string",
      "title": "ID",
      "description": "The primary key."
    }
  },
  "methods": {
    "count": {
      "id": "count",
      "oneOf": [
        {
          "$ref": "#/definitions/method_request"
        },
        {
          "$ref": "#/definitions/method_response"
        }
      ],
      "title": "Object Count",
      "definitions": {
        "method_request": {
          "type": "null",
          "title": "Object Count",
          "side_effects": false,
          "description": "The number of objects in the collection."
        },
        "method_response": {
          "type": "object",
          "title": "Object Count",
          "properties": {
            "\_": {
              "type": "number"
            }
          },
          "description": "The number of objects in the collection."
        }
      },
      "description": "The number of objects in the collection."
    },
    "key-list": {
      "id": "key-list",
      "oneOf": [
        {
          "$ref": "#/definitions/method_request"
        },
        {
          "$ref": "#/definitions/method_response"
        }
      ],
      "title": "Keys",
      "definitions": {
        "method_request": {
          "type": "null",
          "title": "Keys",
          "side_effects": false,
          "description": "*No description provided*"
        },
        "method_response": {
          "type": "array",
          "title": "Keys",
          "items": {
            "type": "string"
          },
          "description": "*No description provided*"
        }
      },
      "description": "*No description provided*"
    },
    "autocomplete": {
      "id": "autocomplete",
      "oneOf": [
        {
          "$ref": "#/definitions/method_request"
        },
        {
          "$ref": "#/definitions/method_response"
        }
      ],
      "title": "Autocomplete",
      "definitions": {
        "method_request": {
          "type": "object",
          "title": "Autocomplete",
          "properties": {
            "partial": {
              "type": "string",
              "title": "Partial",
              "description": "A regular expression to match on any and all of the autocomplete fields."
            }
          },
          "side_effects": false,
          "description": "Search results based on a partial input as a regex"
        },
        "method_response": {
          "type": "array",
          "title": "Autocomplete",
          "items": {
            "type": "object",
            "properties": {
              "k": {
                "type": "string"
              },
              "v": {
                "type": "string"
              }
            }
          },
          "description": "Search results based on a partial input as a regex"
        }
      },
      "description": "Search results based on a partial input as a regex"
    },
    "key-map": {
      "id": "key-map",
      "oneOf": [
        {
          "$ref": "#/definitions/method_request"
        },
        {
          "$ref": "#/definitions/method_response"
        }
      ],
      "title": "Key Map",
      "definitions": {
        "method_request": {
          "type": "null",
          "title": "Key Map",
          "side_effects": false,
          "description": "*No description provided*"
        },
        "method_response": {
          "type": "object",
          "title": "Key Map",
          "description": "*No description provided*"
        }
      },
      "description": "*No description provided*"
    }
  },
  "definitions": {
    "filterOps": {
      "enum": [
        "with_fields",
        "count",
        "max",
        "min",
        "avg",
        "sample",
        "sum",
        "distinct",
        "contains",
        "pluck",
        "without",
        "has_fields",
        "order_by",
        "between"
      ]
    },
    "timedelta": {
      "type": "object",
      "required": [
        "start",
        "end"
      ],
      "properties": {
        "hours": {
          "type": "integer"
        },
        "days": {
          "type": "integer"
        },
        "seconds": {
          "type": "number"
        },
        "minutes": {
          "type": "integer"
        }
      }
    },
    "spatialOps": {
      "enum": [
        "distance",
        "get_intersecting",
        "get_nearest"
      ]
    }
  },
  "template": "{id}",
  "documentMethods": {
    "rel": {
      "id": "\*rel",
      "oneOf": [
        {
          "$ref": "#/definitions/method_request"
        },
        {
          "$ref": "#/definitions/method_response"
        }
      ],
      "title": "Related Documents",
      "definitions": {
        "method_request": {
          "type": "object",
          "properties": {
            "app": {
              "type": "string",
              "title": "App",
              "description": "The slug of the application ``coll`` is in."
            },
            "coll": {
              "type": "string",
              "title": "Coll",
              "description": "The slug of the collection to search for documents in."
            },
            "related_key": {
              "type": "string",
              "title": "Related Key",
              "description": "The name of the key to search for this document in.If none, defaults to the first matching foreign key element."
            }
          },
          "required": [
            "app",
            "coll"
          ],
          "title": "Related Documents",
          "description": "Reverse relation.  Get a query set of all documents in a collection that have foreign keys that point to this document.",
          "side_effects": false
        },
        "method_response": {
          "type": "object",
          "properties": {},
          "title": "Related Documents",
          "description": "Reverse relation.  Get a query set of all documents in a collection that have foreign keys that point to this document."
        }
      },
      "description": "..."
    }
  },
  "primary_key": "id",
  "description": "No Description Provided.",
  "title": "Item",
  "id": "http://localhost:5000/api/todo-app/items;schema"
}
~~~

That's a lot more information!. This may look very dense, but it contains all the methods and inherits
all the schema from the application and suite objects as well. This allows buggy JSON schema parsers
to find references without having to fetch other documents in the course of resolving references.

Let's remove the methods for a moment and see what we have:

~~~json
{
  "template": "{id}",
  "primary_key": "id",
  "description": "No Description Provided.",
  "title": "Item",
  "id": "http://localhost:5000/api/todo-app/items;schema",
  "type": "object",
  "required": ["title"],
  "properties": {
    "title": {
      "type": "string",
      "title": "Title",
      "description": "The title of the item"
    },
    "complete": {
      "type": "boolean",
      "title": "Complete",
      "default": false
    },
    "created": {
      "type": "string",
      "title": "Created",
      "format": "date-time"
    },
    "id": {
      "type": "string",
      "title": "ID",
      "description": "The primary key."
    }
  }
}
~~~

JSON-Schema tells us what we're looking at here, but just to be clear, all Sondra has done is
translated the code into JSON for the client. There are a few properties that are extra to JSON
Schema, such as `template` and `primary_key`. These are defined by Sondra. Template describes how
one would create a visual representation for a record. Primary key lets you know what data field to
use to address documents as endpoints. If you had a document with id `00ab-00ac-00ad-00ae` then you
would access it via the API as `http://localhost:5000/api/todo-app/items/00ab-00ac-00ad-00ae`.
