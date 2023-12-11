---
layout: default
title: Home
nav_order: 1
description: "Simplify building NodeJS applications on cloud platforms"
permalink: /
---

# MondoKit

Simplify building NodeJS applications on cloud platforms

## Installation

Install the core component:
```sh
npm install @mondokit/gcp-core
```

Then depending on your use case install the components you want to use. e.g.

```sh
npm install @mondokit/gcp-firestore
npm install @mondokit/gcp-firebase-auth
```

## Usage
Here's an example Express app configuration that uses the core library as well as both Firestore and Firebase Auth.

```
// Create a request aware logger
const logger = createLogger("mondokit-demo");

// Create your express app as normal
const app = express();

// Add the core logging and async storage middlewares
app.use(mondokitApp);

// Add firestore support (with dataloader for graphql support)
firestoreProvider.init();
app.use(firestoreLoader());

// Add firebase auth support
const firebaseAdmin = admin.initializeApp({ projectId: <my-gcp-project-id> });
app.use(verifyFirebaseUser(firebaseAdmin));

// Add handlers as required
app.get(
  "/demo-items",
  requiresUser(),                   // <-- Util middleware that enforces a valid user
  asyncMiddleware(async (req, res) => { // <-- Util middleware that lets you write async handlers
    const repository = new FirestoreRepository<DemoItem>("demo-items");
    logger.info("listing demo-items from firestore");
    const list = await repository.query();
    res.send(`Hello firestore ${JSON.stringify(list)}`);
  })
);

app.use((err, req, res, next) => {
  // your error handling logic goes here
  logger.error("Error", err);
  res.status(500).send("Bad stuff happened")
});
```

## History

These libraries have most recently evolved from [GAE JS](https://github.com/mondo-mob/gae-js), authored by the same team. They have been re-branded,
given they are not just useful for Google App Engine, updated for ESM, and cleaned up with deprecations removed.

## Components

### gcp-core ([documentation](packages/gcp-core.md))

#### Async Local Storage Support

This library relies on Async Local Storage for passing data around. Yes it's still an Experimental NodeJS
API but it's really useful and the general push is to mark it as stable soon (https://github.com/nodejs/node/issues/35286).

#### Logging
Create request aware Bunyan loggers for Cloud Logging.
All of your logs from a single request will be correlated together in Cloud Logging.

#### Configuration
Extendable, typed and environment aware configuration loader

#### Configuration Secrets
Seamlessly load secrets stored in "Google Cloud Secret Manager"

#### Serving static resources with etags
Serve static assets with strong etags to workaround GAE build wiping out your file timestamps

#### Authentication/Authorization
Middleware to protect your routes to authenticated users or specific user roles

#### Search
Framework for adding search capability to your data layer

#### Other stuff
A few other (hopefully) useful things to help you along the way and stop reinventing the wheel

### gcp-bigquery ([documentation](packages/gcp-bigquery.md))
#### Use BigQuery in your app
Simplifies client initialisation and common BQ tasks

### gcp-datastore ([documentation](packages/gcp-datastore.md))

#### Use Cloud Datastore (or Firestore in Datastore mode)
Access your collections through typed repositories, backed by a DataLoader implementation to support GraphQL.

#### Simple transaction support
Use annotations on your methods to make them transactional

### gcp-datastore-backups ([documentation](packages/gcp-datastore-backups.md))
#### Automate Datastore Exports
Run/schedule full or partial exports of Datastore into Google Cloud Storage
#### Import into BigQuery
Schedule Extract and Load jobs to export from Datastore into BigQuery

### gcp-firebase-auth ([documentation](packages/gcp-firebase-auth.md))
#### Use Firebase Auth to authenticate your users
Middleware to verify Firebase Auth tokens and set user into the request

### gcp-firestore ([documentation](packages/gcp-firestore.md))

#### Use Firestore in Native mode
Access your collections through typed repositories, backed by a DataLoader implementation to support GraphQL.

#### Simple transaction support
Use annotations on your methods to make them transactional

### gcp-firestore-backups ([documentation](packages/gcp-firestore-backups.md))
#### Automate Firestore Exports
Run/schedule full or partial exports of Firestore into Google Cloud Storage
#### Import into BigQuery
Schedule Extract and Load jobs to export from Firestore into BigQuery

### gcp-firestore-migrations ([documentation](./packages/gcp-firestore-migrations.md))
#### Run migrations on a firestore database
Bootstrap migrations to be run when server starts or create an endpoint to trigger them. Status of migrations and mutex lock is managed with Firestore.

### gcp-google-auth ([documentation](./packages/gcp-google-auth.md))
#### Google Auth utilities
Utilities extending on [Google Auth Library](https://github.com/googleapis/google-auth-library-nodejs#readme), such as middleware to validate Google JWT.

### gcp-storage ([documentation](./packages/gcp-storage.md))
#### Use Cloud Storage in your app
Simplifies client initialisation and common storage tasks

### gcp-tasks ([documentation](./packages/gcp-tasks.md))
#### Use Cloud Tasks in your app
Simplifies client initialisation and common task operations

