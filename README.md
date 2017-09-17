# orbit-db-store

[![npm version](https://badge.fury.io/js/orbit-db-store.svg)](https://badge.fury.io/js/orbit-db-store)

Base class for [orbit-db](https://github.com/haadcode/orbit-db) data stores. You generally don't need to use this module if you want to use `orbit-db`. This module contains shared methods between all data stores in `orbit-db` and can be used as a base class for a new data model.

### Used in
- [orbit-db-kvstore](https://github.com/haadcode/orbit-db-kvstore)
- [orbit-db-eventstore](https://github.com/haadcode/orbit-db-eventstore)
- [orbit-db-feedstore](https://github.com/haadcode/orbit-db-feedstore)
- [orbit-db-counterstore](https://github.com/haadcode/orbit-db-counterstore)
- [orbit-db-docstore](https://github.com/shamb0t/orbit-db-docstore)

### Requirements
- Node.js >= 6.0
- npm >= 3.0

### events

  Store has an `events` ([EventEmitter](https://nodejs.org/api/events.html)) object that emits events that describe what's happening in the database.

  - `load` - (dbname, hash)

    Emitted before loading the database history. *hash* is the hash from which the history is loaded from.

    ```javascript
    db.events.on('load', (id, hash) => ... )
    db.load()
    ```

  - `ready` - (dbname)

    Emitted after fully loading the database history.

    ```javascript
    db.events.on('ready', (id) => ... )
    db.load()
    ```

  - `progress.load` - (id, hash, entry, progress, total)

    Emitted for each entry during load.

    *Progress* is the current load count. *Total* is the maximum load count (ie. length of the full database). These are useful eg. for displaying a load progress percentage.

    ```javascript
    db.events.on('load', (id, hash, entry, progress, total) => ... )
    db.load()
    ```

  - `replicated` - (dbname)

    Emitted after the database was synced with an update from a peer database.

    ```javascript
    db.events.on('replicated', (id) => ... )
    ```

  - `write` - (id, hash, entry)

    Emitted after an entry was added locally to the database. *hash* is the IPFS hash of the latest state of the database. *entry* is the Entry that was added.

    ```javascript
    db.events.on('write', (id, hash, entry) => ... )
    ```

  - `close` - (id)
    
    Emitted after the database was closed.

    ```javascript
    db.events.on('close', (id) => ... )
    db.close()
    ```

### API

#### Public methods

##### `load()`

Load the database using locally persisted state.

##### `saveSnapshot()`

Save the current state of the database locally. Returns a *Promise* that resolves to a IPFS Multihash as a Base58 encoded string. The the database can be loaded using this hash.

##### `loadFromSnapshot(hash, onProgressCallback)`

Load the state of the database from a snapshot. *hash* is the IPFS Multihash of the snapshot data. Returns a *Promise* that resolves when the database has been loaded.

##### `close()`

Uninitialize the store. Emits `close` after the store has been uninitialized.

##### `drop()`

Remove the database locally. This doesn't remove or delete the database from peers who have replicated the database.

##### `sync(heads)`

Sync this database with entries from *heads* where *heads* is an array of ipfs-log Entries. Usually, you don't need to call this method manually as OrbitDB takes care of this for you.

#### Properties

##### `type`

Remove all items from the local store. This doesn't remove or delete any entries in the distributed operations log.

```javascript
console.lo(db.type) // "eventlog"
```

#### Private methods

##### `_addOperation(data)`

Add an entry to the store. Takes `data` as a parameter which can be of any type.

```javascript
this._addOperation({
  op: 'PUT',
  key: 'greeting',
  value: 'hello world!'
});
```

### Creating Custom Data Stores
You can create a custom data stores that stores data in a way you need it to. To do this, you need to import `orbit-db-store` to your custom store and extend your store ckass from orbit-db-store's `Store`. Below is the `orbit-db-kvstore` which is a custom data store for `orbit-db`.

*TODO: describe indices and how they work*

```javascript
const Store         = require('orbit-db-store');
const KeyValueIndex = require('./KeyValueIndex');

class KeyValueStore extends Store {
  constructor(ipfs, id, dbname, options) {
    Object.assign(options || {}, { Index: KeyValueIndex });
    super(ipfs, id, dbname, options)
  }

  get(key) {
    return this._index.get(key);
  }

  set(key, data) {
    this.put(key, data);
  }

  put(key, data) {
    return this._addOperation({
      op: 'PUT',
      key: key,
      value: data,
      meta: {
        ts: new Date().getTime()
      }
    });
  }

  del(key) {
    return this._addOperation({
      op: 'DEL',
      key: key,
      value: null,
      meta: {
        ts: new Date().getTime()
      }
    });
  }
}

module.exports = KeyValueStore;
```

## Contributing

See [orbit-db's contributing guideline](https://github.com/haadcode/orbit-db#contributing).

## License

[MIT](LICENSE) ©️ 2016 Haadcode
