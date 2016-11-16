---
next: "/basics/tutorial-3"
toc: true
date: "2016-10-12T16:38:34-04:00"
title: "Expose it to the Web"
weight: 2
prev: "/basics/tutorial-1"

menu:
  main:
    parent: Tutorial
    identifier: "Expose it to the Web"
    weight: 1
---

Now how do we make it into an API and serve it to the web?  Add this code to the file we've been
building and run it as a module in Python:

~~~python
from flask import Flask
from flask.ext.compress import Compress
from sondra.flask import api_tree, init

# Create the Flask instance and the suite.
app = Flask(__name__)
Compress(app)  # This is not necessary, but I find it generally helpful.
app.debug = True
app.suite = TodoSuite()
init(app)

# Register all the applications.
TodoApp(app.suite)

# Create all databases and tables.
app.suite.validate()  # remember this call?
app.suite.ensure_database_objects()  # and this one?

# Attach the API to the /api/ endpoint.
app.register_blueprint(api_tree, url_prefix='/api')

if __name__ == '__main__':
    app.run()

~~~

It's probably pretty obvious what's going on here.  This is the typical look of a Flask app, after
all. The additional things we have to do are, in order:

1. Attach the suite to the Flask app as `app.suite`.
2. `init` the app. This sets up CORS if it's been configured and makes sure that logging is handled correctly.
3. Ensure the application objects exist in the database.
4. Register the `api_tree` blueprint with Flask, typically at `/api`.
