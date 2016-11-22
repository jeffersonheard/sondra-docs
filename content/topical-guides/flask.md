---
toc: true
date: "2016-10-12T16:38:34-04:00"
title: "The Flask App"
weight: 0

menu:
  main:
    parent: Topical Guides
    identifier: "Flask"
    weight: 0
---

The following boilerplate, based on the Todo app in the tutorials, will give you what you need to
create a basic Flask application to serve your API. Typically, this would be served via uWSGI and a
webserver to serve static and uploaded content. Since Sondra is only an API server and ORM, these
topics are not covered as part of the guide.

This boilerplate includes compression. Most suites will also require the CORS middleware, which
can be configured in the suite, and is applied by default.

~~~python
from sondra.auth import Auth  # this is a sample Auth application. You may want to use another service for auth.
from sondra.flask import api_tree, init  
from todo import TodoApp, TodoSuite
from flask.ext.compress import Compress

app = Flask(__name__)
Compress(app)  # I can't think of a reason you wouldn't want to do this, especially for large payloads.

# passing a parameter into the Suite will cause all databases to be prefixed with that string.
# This can be useful for creating multiple test platforms using the same database server and
# codebase without configuration changes.
if len(sys.argv) > 1:
    app.suite = TodoSuite(sys.argv[1])
else:
    app.suite = TodoSuite()

# this adds cross-origin capabilities, sets max content length, and others.
init(app)

# Register all the applications.
auth = Auth(app.suite)
core = TodoApp(app.suite)

# Create all databases and tables.
app.suite.validate()
app.suite.ensure_database_objects()

# Attach the API to the /api/ endpoint.
app.register_blueprint(api_tree, url_prefix='/api')

if __name__ == '__main__':
    app.run()
~~~
