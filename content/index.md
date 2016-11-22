---
date: 2016-03-08T21:07:13+01:00
title: Sondra
type: index
weight: 0
---

Sondra is an "ORM" and REST-ful webservice framework for Python 3.x, Flask, and RethinkDB with some unique
features. Sondra's goal is to aid full stack developers by letting them focus
on data models and functionality instead of writing workarounds and glue code.
It embraces common "shortcuts" developers take in common full-stack web
applications, e.g. merging "Model" and "Controller" in the oft-used MVC
pattern.

Sondra does not currently support asynchronous access to RethinkDB.  The goal
is to eventually support Tornado as an asynchronous backend for REST and WebSockets.

## Features

* A clear, DRY heirarchical application structure that emphasizes convention over configuration.
* Authentication via JSON Web Tokens (JWT)
* JSON-Schema validation for documents.
* Expose methods on documents, collections, and applications, complete with schemas for call and return.
* A clear, predictable URL scheme for all manner of API calls, covering a broad set of use-cases.
* Self documenting APIs with both human-readable help based on docstrings and schemas for every call.
* Use API idiomatically over HTTP and native Python without writing boilerplate code
