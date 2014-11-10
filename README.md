Hoodie, meet PouchDB
====================

Up until now, Hoodie uses a custom solution on top of localStorage for data persistence and offline sync. But we can do better. [PouchDB](http://pouchdb.com/) is a database for browsers. And it syncs. Plus, it has one of the friendliest and most dedicated community behind it. A match made in heaven.


## The Plan™

We will create an isolated module that implements the `hoodie.store` API. Once finished, we will replace our self-cooked localStorage solution with it, but that will be another plan™. For now:

- [x] Create hoodie.store dream api
- [ ] Create test setup with own CouchDB backend [#8](https://github.com/hoodiehq/wip-hoodie-store-on-pouchdb/issues/8)
- [ ] Implement sync methods [#19](https://github.com/hoodiehq/wip-hoodie-store-on-pouchdb/issues/19)
- [ ] Implement store methods [#7](https://github.com/hoodiehq/wip-hoodie-store-on-pouchdb/issues/7)
- [ ] Implement events [#26](https://github.com/hoodiehq/wip-hoodie-store-on-pouchdb/issues/26)
- [ ] Implement `.hasLocalChanges()` [#29](https://github.com/hoodiehq/wip-hoodie-store-on-pouchdb/issues/29) & `.changedObjects()` [#30](https://github.com/hoodiehq/wip-hoodie-store-on-pouchdb/issues/30)
- [ ] var todoStore = hoodie.store('todo') [#31](https://github.com/hoodiehq/wip-hoodie-store-on-pouchdb/issues/31)

And along the way, we create [PouchDB plugins](http://pouchdb.com/api.html#plugins) whenever feasible.


## Dream API

The API is a merge of Hoodie's current `hoodie.store`
and `hoodie.remote` API.

```js
// all methods return native promises
hoodie.store.add(object)
hoodie.store.add([object1, object2])
hoodie.store.find(id)
hoodie.store.find(object) // with id property
hoodie.store.findOrAdd(id, object)
hoodie.store.findOrAdd(object)
hoodie.store.findOrAdd([object1, object2])
hoodie.store.findAll()
hoodie.store.findAll(filterFunction)
hoodie.store.update(id, changedProperties)
hoodie.store.update(id, updateFunction)
hoodie.store.update(object)
hoodie.store.updateOrAdd(id, object)
hoodie.store.updateOrAdd(object)
hoodie.store.updateOrAdd([object1, object2])
hoodie.store.updateAll(changedProperties)
hoodie.store.updateAll(updateFunction)
hoodie.store.remove(id)
hoodie.store.remove(object)
hoodie.store.removeAll()
hoodie.store.removeAll(filterFunction)
hoodie.store.clear()

// sync methods, return native promises
hoodie.store.pull() // pulls changes one-time
hoodie.store.push() // pushes changes one-time
hoodie.store.sync() // pulls and pushes changes one-time
hoodie.store.connect() // starts continuous replication
hoodie.store.disconnect() // stops continuous replication and all pending requests


// returns true or false
hoodie.store.hasLocalChanges()
// returns all objects witch local changes that have not been synced yet.
hoodie.store.changedObjects()

// events
hoodie.store.on(event, handler)
hoodie.store.off(event /*, handler */)

// returns a custom store
hoodie.store(typeOrOptions)
```


## Events

```js
hoodie.store.on('add', function(object, options) {})
hoodie.store.on('update', function(object, options) {})
hoodie.store.on('remove', function(object, options) {})
hoodie.store.on('change', function(eventName, object, options) {})
hoodie.store.on('sync', function(object) {})
hoodie.store.on('clear', function() {})
```


## Custom stores

```js
// example
var todoStore = hoodie.store('todo')
todoStore.add({name: 'Rule the world'})
todoStore.on('change', renderList)
```

When calling `hoodie.store()` as a function, a custom
store is returned. When a string is passed, the returned
store is automatically "namespaced". The code above would
store a new object with the properties:

- `type`: todo
- `id`: uuid567
- `name`: Rule the world

The `renderList` callback would only be called if objects
with `type: 'todo'` are changed.


## Differences to current hoodie.store API / out of scope

- `type` is no longer a required parameter and it’s no longer part    of the _id properties in CouchDB.
- returned Promises only expose `.then` and `.catch`. We will add the jQuery-esque callbacks later.
- no more namespaces events a la `type:change` on `hoodie.store`. Instead, create a scoped store using `hoodie.store(‘type’).on(‘change’)`
- we don’t create Hoodie’s special properties for now (`createdAt`, `updatedAt`, `createdBy`)


## Internals

`hoodie.store` creates a local PouchDB with the name “hoodie-store”. The remote database is called “user/hoodieid” (`<hoodieid>` will later be replaced with the random hoodie.id()). The first time one of the sync methods is called and it fails, `hoodie.store` tries to create the database and then do the sync method, and only if that fails, the returned promise gets rejected.


## Test setup

```
npm start
# starts a local server at localhost:9000
# It serves /index.html and all other static assets, and
# proxies /hoodie/store to its own CouchDB in admin-party
# mode so anonymous users can create databases.

npm test
# tests all the things
```
