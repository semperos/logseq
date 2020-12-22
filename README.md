# Logseq

[![latest release version](https://img.shields.io/github/v/release/logseq/logseq)](https://github.com/logseq/logseq/releases)
[![License](https://img.shields.io/github/license/logseq/logseq?color=blue)](https://github.com/logseq/logseq/blob/master/LICENSE.md)
[![Twitter follow](https://img.shields.io/badge/follow-%40logseq-blue.svg?style=flat&logo=twitter)](https://twitter.com/logseq)
[![discord](https://img.shields.io/discord/725182569297215569?label=discord&logo=Discord&color=blue)](https://discord.gg/KpN4eHY)
[![total](https://opencollective.com/logseq/tiers/badge.svg?color=blue)](https://opencollective.com/logseq)

[![Contributors](https://opencollective.com/logseq/tiers/sponsors.svg?avatarHeight=24&width=600)](https://opencollective.com/logseq)
[![Contributors](https://opencollective.com/logseq/tiers/backers.svg?avatarHeight=24&width=600)](https://opencollective.com/logseq)

A local-first, non-linear, outliner notebook for organizing and sharing your personal knowledge base.

Use it to organize your todo list, to write your journals, or to record your unique life.

## Why Logseq?

[Logseq](https://logseq.com) is an open-source platform for knowledge sharing and management. It focuses on privacy, longevity, and user control.

The server will never store or analyze your private notes. Your data are plain text files and we currently support both Markdown and Emacs Org mode (more to be added soon).

In the unlikely event that the website is down or cannot be maintained, your data is, and will always be yours.

![Image of logseq](https://cdn.logseq.com/%2F8b9a461d-437e-4ca5-a2da-18b51077b5142020_07_25_Screenshot%202020-07-25%2013-29-49%20%2B0800.png?Expires=4749255017&Signature=Qbx6jkgAytqm6nLxVXQQW1igfcf~umV1OcG6jXUt09TOVhgXyA2Z5jHJ3AGJASNcphs31pZf4CjFQ5mRCyVKw6N8wb8Nn-MxuTJl0iI8o-jLIAIs9q1v-2cusCvuFfXH7bq6ir8Lpf0KYAprzuZ00FENin3dn6RBW35ENQwUioEr5Ghl7YOCr8bKew3jPV~OyL67MttT3wJig1j3IC8lxDDT8Ov5IMG2GWcHERSy00F3mp3tJtzGE17-OUILdeuTFz6d-NDFAmzB8BebiurYz0Bxa4tkcdLUpD5ToFHU08jKzZExoEUY8tvaZ1-t7djmo3d~BAXDtlEhC2L1YC2aVQ__&Key-Pair-Id=APKAJE5CCD6X7MP6PTEA)

## Feature requests
Please go to https://discuss.logseq.com/c/feature-requests/7.

## How can I use it?

1. Make sure you have registered a [GitHub account](https://github.com/join) and already created a repository (could be an old one). _Currently we only support GitHub, but more sync  options (e.g. Gitlab, Dropbox, Google Drive, WebDAV, etc.) will be added soon._

2. Visit our website <https://logseq.com/>.

3. Click the "Login with GitHub" button in the upper-right corner.

4. Following the on-screen instructions, install Logseq app on your selected repository.

5. Start writing and have fun!

## Credits

Logseq is hugely inspired by [Roam Research](https://roamresearch.com/), [Org Mode](https://orgmode.org/), [Tiddlywiki](https://tiddlywiki.com/), [Workflowy](https://workflowy.com/) and [Cuekeeper](https://github.com/talex5/cuekeeper), hats off to all of them!

Logseq is also made possible by the following projects:

- [Clojure & ClojureScript](https://clojure.org/) - A dynamic, functional, general-purpose programming language
- [DataScript](https://github.com/tonsky/datascript) - Immutable database and Datalog query-engine for Clojure, ClojureScript and JS
- [OCaml](https://ocaml.org/) & [Angstrom](https://github.com/inhabitedtype/angstrom), for the document [parser](https://github.com/mldoc/mldoc)
- [isomorphic-git](https://isomorphic-git.org/) - A pure JavaScript implementation of Git for node and browsers
- [sci](https://github.com/borkdude/sci) - Small Clojure Interpreter

![Logseq Credits](https://asset.logseq.com/static/img/credits.png)

## Learn more

- Our blog: https://logseq.com/blog - Please be sure to visit our [About page](https://logseq.com/blog/about) for the latest updates of the app
- Twitter: https://twitter.com/logseq
- Discord: https://discord.gg/KpN4eHY - Where we answer questions, disucss workflows and share tips
- Github: https://github.com/logseq/logseq - everyone is encouraged to report issues!

- - - -

The following is for developers and designers who want to build and run Logseq locally and contribute to this project.

## Set up development environment

TODO: Integration the following code to import/export the IndexedDB data as JSON:

```js
'use strict';

/**
 * Export all data from an IndexedDB database
 * @param {IDBDatabase} idbDatabase - to export from
 * @param {function(Object?, string?)} cb - callback with signature (error, jsonString)
 */
function exportToJsonString(idbDatabase, cb) {
  const exportObject = {};
  const objectStoreNamesSet = new Set(idbDatabase.objectStoreNames);
  const size = objectStoreNamesSet.size;
  if (size === 0) {
    cb(null, JSON.stringify(exportObject));
  } else {
    const objectStoreNames = Array.from(objectStoreNamesSet);
    const transaction = idbDatabase.transaction(
        objectStoreNames,
        'readonly'
    );
    transaction.onerror = (event) => cb(event, null);

    objectStoreNames.forEach((storeName) => {
      const allObjects = [];
      transaction.objectStore(storeName).openCursor().onsuccess = (event) => {
        const cursor = event.target.result;
        if (cursor) {
          allObjects.push(cursor.value);
          cursor.continue();
        } else {
          exportObject[storeName] = allObjects;
          if (
            objectStoreNames.length ===
            Object.keys(exportObject).length
          ) {
            cb(null, JSON.stringify(exportObject));
          }
        }
      };
    });
  }
}

/**
 * Import data from JSON into an IndexedDB database. This does not delete any existing data
 *  from the database, so keys could clash.
 *
 * Only object stores that already exist will be imported.
 *
 * @param {IDBDatabase} idbDatabase - to import into
 * @param {string} jsonString - data to import, one key per object store
 * @param {function(Object)} cb - callback with signature (error), where error is null on success
 * @return {void}
 */
function importFromJsonString(idbDatabase, jsonString, cb) {
  const objectStoreNamesSet = new Set(idbDatabase.objectStoreNames);
  const size = objectStoreNamesSet.size;
  if (size === 0) {
    cb(null);
  } else {
    const objectStoreNames = Array.from(objectStoreNamesSet);
    const transaction = idbDatabase.transaction(
        objectStoreNames,
        'readwrite'
    );
    transaction.onerror = (event) => cb(event);

    const importObject = JSON.parse(jsonString);
    objectStoreNames.forEach((storeName) => {
      let count = 0;
      const aux = Array.from(importObject[storeName]);
      if (importObject[storeName] && aux.length > 0) {
        aux.forEach((toAdd) => {
          const request = transaction.objectStore(storeName).add(toAdd);
          request.onsuccess = () => {
            count++;
            if (count === importObject[storeName].length) {
              // added all objects for this store
              delete importObject[storeName];
              if (Object.keys(importObject).length === 0) {
                // added all object stores
                cb(null);
              }
            }
          };
          request.onerror = (event) => {
            console.log(event);
          };
        });
      } else {
        delete importObject[storeName];
        if (Object.keys(importObject).length === 0) {
          // added all object stores
          cb(null);
        }
      }
    });
  }
}

/**
 * Clears a database of all data.
 *
 * The object stores will still exist but will be empty.
 *
 * @param {IDBDatabase} idbDatabase - to delete all data from
 * @param {function(Object)} cb - callback with signature (error), where error is null on success
 * @return {void}
 */
function clearDatabase(idbDatabase, cb) {
  const objectStoreNamesSet = new Set(idbDatabase.objectStoreNames);
  const size = objectStoreNamesSet.size;
  if (size === 0) {
    cb(null);
  } else {
    const objectStoreNames = Array.from(objectStoreNamesSet);
    const transaction = idbDatabase.transaction(
        objectStoreNames,
        'readwrite'
    );
    transaction.onerror = (event) => cb(event);

    let count = 0;
    objectStoreNames.forEach(function(storeName) {
      transaction.objectStore(storeName).clear().onsuccess = () => {
        count++;
        if (count === size) {
          // cleared all object stores
          cb(null);
        }
      };
    });
  }
}

if (typeof module !== 'undefined' && module.exports) {
  // We are running on Node.js so need to export the module
  module.exports = {
    exportToJsonString: exportToJsonString,
    importFromJsonString: importFromJsonString,
    clearDatabase: clearDatabase,
  };
}
```

### 1. Requirements

- [Node.js](https://nodejs.org/en/download/) & [Yarn](https://classic.yarnpkg.com/en/docs/install/)
- [Java & Clojure](https://clojure.org/guides/getting_started)

### 2. Compile to JavaScript

``` bash
git clone https://github.com/logseq/logseq
yarn
yarn watch
```

### 3. Open the browser

Open <http://localhost:3001>.

### 4. Build a release

``` bash
yarn release
```

## Alternative: Docker based development environment

### 1. Fetch sources

``` bash
git clone https://github.com/logseq/logseq
```

### 2. Build Docker image

``` bash
cd logseq
docker build -t logseq-docker .
```

### 3. Run Docker container

``` bash
docker run -v $(pwd):/home/logseq/logseq -p 3001:3001 -p 9630:9630 -p 8701:8701 --rm -it logseq-docker /bin/bash
```

### 4. Inside the container compile as described above

``` bash
cd logseq
yarn
yarn watch
```
