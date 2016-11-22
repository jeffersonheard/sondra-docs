---
toc: true
date: "2016-10-12T16:38:34-04:00"
title: "The Javascript Client"
weight: 4

menu:
  main:
    parent: Topical Guides
    identifier: "Javascript"
    weight: 4
---

Sondra does not wrap raw objects unless requested to. Also, GET, POST, PUT, PATCH, and DELETE
metaphors behave as expected. Therefore a simple XMLHttpRequest or a call to the new standard
Fetch API with raw JSON objects that conform to the schema will work just fine.
[See the tutorial](/basics/tutorial-4) for more information on how these work.

For those who want something more high level, Sondra has a
[Javascript client](https://github.com/jeffersonheard/sondra-client). This client uses ES6 and
Bluebird.js to make a modern, promise-based API for remote communication with a Sondra-based API
server.

## Client API

## Example: Fetch a schema
~~~javascript
import { Sondra, QuerySet } from 'sondra-client';

const api = new Sondra().suite('http', 'localhost', 5000);
const auth = api.app('auth');
const users = auth.collection('users');
const login = auth.method('login');
const jefferson = users.document('jefferson');

users.fetchSchema().then((schema) => {  // we set the format first to make sure that schema overrides it.
  console.log('schema')    
});
~~~

### Results

The resulting schema is quite large. For the purposes of this guide, we have left out much of it.
API (Suite), application, and collection schemas are self describing. Within the suite schema are
references to all the applications under the attribute `applications`. Within the application schema
are references to all the collections under the attribute `collections` as well as to
application-wide methods under `methods`. Within collections are the  document schemas, and these
also contain references to collection-wide methods under the attribute `methods` and to
document-specific (instance) methods under `documentMethods`.

~~~javascript
{
  "type": "object",
  "properties": {
    "username": {
      "type": "string",
      "description": "The user's username",
      "title": "Username"
    },
    "email": {
      "type": "string",
      "format": "email",
      "description": "The user's email address",
      "title": "Email",
      "pattern": "^[^@]+@[^@]+\\.[^@]+$"
    },
     /* ... The actual schema is quite long. */
}
~~~

## Example: Fetch a set of documents

This example retrieves the first page of documents from the API.

~~~javascript

const api = new Sondra().suite('http', 'localhost', 5000);
const auth = api.app('auth');
const users = auth.collection('users');
const login = auth.method('login');

login.call({body: {username: 'jefferson', password: 'mypassword'}})
  .then((rsp) => {
    const { _:token } = rsp;
    const todoApp = todo.app('todo-app').auth(token);
    const items = todoApp.collection('items');
    items.call().then((items) => {
      / ** do something with the item list *//
    });
~~~

## Example: Query a collection

The above example fetches by default the first hundred documents returned by a simple database
query. This is often useful, but more useful is the ability to limit and filter the data. The
following example uses a QuerySet object to limit the data. For more information on querying and
querysets, [see the topical guide on querying](querying).

~~~javascript

import { Sondra, QuerySet } from 'sondra-client';

const api = new Sondra().suite('http', 'localhost', 5000);
const auth = api.app('auth');
const users = auth.collection('users');
const login = auth.method('login');

login.call({body: {username: 'jefferson', password: 'mypassword'}})
  .then((token) => Promise.resolve(api.auth(token._)))
  .then((authorizedApi) => {
    const todoApp = todo.app('todo-app');
    const items = todoApp.collection('items');

    // apply a start and a limit. See sondra.js for a full list of methods supported by QuerySet.
    const q = new QuerySet().start(0).limit(5);  

    items.query(q).call().then((items) => {
      / ** do something with the item list *//
    });
~~~

The Javascript client is complete, and you can do anything within it that you can with Fetch or
XMLHttpRequest. For more information on the API, see the ESDoc in the repository.
