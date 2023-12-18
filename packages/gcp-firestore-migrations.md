---
layout: default
title: GCP Firestore Migrations
description: Set up and run migrations, with state stored in firestore and safety across clusters
parent: Packages
nav_order: 4
---

# MondoKit GCP Firestore Migrations

Set up and run migrations, with state stored in firestore and a mutex lock to ensure only one is run at a time

> **Note:** `@mondokit` libs all require ESM. If your project still uses CommonJS, then you can continue using our previous incarnation, [gae-js](https://mondo-mob.github.io/gae-js-docs), until you can make the upgrade.

## Installation

```sh
npm install @mondokit/gcp-firestore-migrations
```

## Components

### runMigrations
Run Migrations as a function e.g /migrate route handler

### bootstrapMigrations
Bootstrap Migrations to be run before application starts

#### Migration Files
The [AutoMigration](https://github.com/mondo-mob/mondokit/blob/main/packages/gcp-firestore-migrations/src/auto-migration.ts) interface 

Use naming convention with date and index number to version migrations

   `migrations/v_20220122_001_addUsers.ts`

```typescript
const userRepository = new TimestampedRepository<User>("users");

export const v_20220122_001_addUsers: AutoMigration = {
    id: "v_20220122_001_addUsers",   
    migrate: async ({ logger }) => {
      logger.info("Adding users");
    
      const createdUsers = await userRepository.save([
        {
          ...newTimestampedEntity("user1"),
          name: "User 1",
        },
        {
          ...newTimestampedEntity("user2"),
          name: "User 2",
        },
      ]);
      logger.info(`Creating ${createdUsers.length} new users`);
    },
    
    
    // Optional function that skips if returning true. For example to skip in a certain environment.
    // skip: () => true,
    
    // Optional options to override from defaults, or those defined globally
    // options: { disableTimestampUpdate: false },
}
```

#### Application Startup (index.ts)
```typescript
import { bootstrap } from "@mondokit/gcp-core";
import { bootstrapMigrations } from "@mondokit/gcp-firestore-migrations";
import { firestoreLoader, firestoreProvider, newTimestampedEntity } from "@mondokit/gcp-firestore";
import {v_20220122_001_addUsers} from "./migrations/v_20220122_001_addUsers"

// Add firestore support
firestoreProvider.init();
app.use(firestoreLoader());

// After firestore initialised
const migrations: AutoMigration[] = [
    v_20220122_001_addUsers
];

await bootstrap([bootstrapMigrations(migrations)]);
// OR with options
// await bootstrap([bootstrapMigrations(migrations, { disableTimestampUpdate: true })]);
```


### Migration options (either global or per migration)

All properties are optional. These options can be specified globally (via the `bootstrapMigrations` or `runMigrations` functions) or overridden for an individual
migration file (via the `AutoMigration` interface with the `options` property). Any options set via `AutoMigration` options will take preference over global settings or defaults.

| Property               | Type      | Default | Description                                                                                                        |
|------------------------|-----------|---------|--------------------------------------------------------------------------------------------------------------------|
| disableTimestampUpdate | `boolean` | `false` | If enabled, this will skip automatically updating timestamp values via [TimestampedRepository](./gcp-firestore.html) |
