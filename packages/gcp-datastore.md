---
layout: default
title: GCP Datastore
description: Use Google Cloud Datastore (or Firestore in Datastore Mode) as your app db, including DataLoader
parent: Packages
nav_order: 5
---

# MondoKit GCP Datastore

Use Google Cloud Datastore (or Firestore in Datastore Mode) as your app db including [DataLoader](https://github.com/graphql/dataloader).

> **Note:** `@mondokit` libs all require ESM. If your project still uses CommonJS, then you can continue using our previous incarnation, [gae-js](https://mondo-mob.github.io/gae-js-docs), until you can make the upgrade.

Find the source code at [gcp-datastore on GitHub](https://github.com/mondo-mob/mondokit/tree/main/packages/gcp-datastore).

## Installation

```sh
npm install @mondokit/gcp-datastore
```

## Components

### DatastoreProvider
Initialise Datastore to be accessed elsewhere in your app.

```typescript
// On app startup
datastoreProvider.init();

// Anywhere else in your app
const datastore = datastoreProvider.get();
const key = datastore.key(["MyItem", "id123"]);
const [doc] = await datastore.get(key);
```

### DatastoreLoader
Dataloader implementation to help batch and cache db requests. Used internally by DatastoreRepository

```typescript
// Apply middleware to create a new dataloader on each request
app.use(datastoreLoader());
```

### DatastoreRepository
Access your collections through typed repositories.

Step 1: Define your entity

```typescript
// Define your class schema
const demoItemSchema = t.type({
  id: t.string,
  name: t.string,
});

// Define your class type
type DemoItem = t.TypeOf<typeof demoItemSchema>;

// Initialise repository for the collection we want to access data in
const repositoryDirect = new DatastoreRepository<DemoItem>("demo-items", {validator: demoItemSchema });

// OR define a custom class first
class DemoItemRepository extends DatastoreRepository<DemoItem> {
  constructor() {
    super("demo-items", { validator: demoItemSchema });
  }
}
const repository = new DemoItemsRepository();

// Save an item
await repository.save({ id: "id123", name: "test item" });

// Get an item
const item = await repository.get("id123");

// Query items
const list = await demoItemsRepository.query();

// Reindex (re-save) all items in batches (200 by default)
const totalCount1 = await repository.reindexInBatches();
const totalCount2 = await repository.reindexInBatches({ batchSize: 1000 });

// Reindex (re-save) all items in batches with "quiet" mode (no logs written)
const totalCount3 = await repository.reindexInBatches({ quiet: true });

// Reindex (re-save) all items in batches, with a transformation per item
const totalCount4 = await repository.reindexInBatches({ transform: ({id, name}) => ({ id, name: `UPDATED ${name}` })});

// Reindex (re-save) all items at once (WARNING: Small datasets only)
const items1 = await repository.reindex();

// Reindex (re-save) all items at once (WARNING: Small datasets only), with a transformation per item (WARNING: Small datasets only)
const items2 = await repository.reindex(({id, name}) => ({ id, name: `UPDATED ${name}` }));

```

