---
layout: default
title: Datastore
parent: Packages
nav_order: 5
---

# MondoKit GCP DATASTORE

Use Cloud Datastore (or Firestore in Datastore Mode) as your app db including [DataLoader](https://github.com/graphql/dataloader) implementation GraphQL
and request level caching.

## Installation

```sh
npm install @mondokit/gcp-datastore
```

## Components

### DatastoreProvider
Initialise Datastore to be accessed elsewhere in your app.

```
// On app startup
datastoreProvider.init();

// Anywhere else in your app
const datastore = datastoreProvider.get();
const key = datastore.key(["MyItem", "id123"]);
const [doc] = await datastore.get(key);
```

### DatastoreLoader
Dataloader implementation to help batch and cache db requests. Used internally by DatastoreRepository

```
// Apply middleware to create a new dataloader on each request
app.use(datastoreLoader());
```

### DatastoreRepository
Access your collections through typed repositories.

Step 1: Define your entity

```
// Define your class schema
const demoItemSchema = t.type({
  id: t.string,
  name: t.string,
});

// Define your class type
type DemoItem = t.TypeOf<typeof demoItemSchema>;

// Initialise repository for the collection we want to access data in
const repository = new DatastoreRepository<DemoItem>("demo-items", {validator: demoItemSchema });

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
const totalCount = await repository.reindexInBatches();
const totalCount = await repository.reindexInBatches({ batchSize: 1000 });

// Reindex (re-save) all items in batches with "quiet" mode (no logs written)
const totalCount = await repository.reindexInBatches({ quiet: true });

// Reindex (re-save) all items in batches, with a transformation per item
const totalCount = await repository.reindexInBatches({ transform: ({id, name}) => ({ id, name: `UPDATED ${name}` })});

// Reindex (re-save) all items at once (WARNING: Small datasets only)
const items = await repository.reindex();

// Reindex (re-save) all items at once (WARNING: Small datasets only), with a transformation per item (WARNING: Small datasets only)
const items = await repository.reindex(({id, name}) => ({ id, name: `UPDATED ${name}` }));

```

### @Transactional

Annotate functions to make them transactional.

NOTE: Requires `"experimentalDecorators": true` set in your `tsconfig.json`

```typescript
class UserService {
  constructor(
    public userRepo: DatastoreRepository<User>,
  ) {}

  @Transactional()
  async addCredits(userId: string, credits: number): Promise<User> {
    const user = this.userRepo.get(userId);
    user.credits = user.credits + credits;
    return this.userRepo.save(user);
  }
}
```
