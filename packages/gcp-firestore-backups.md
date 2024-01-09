---
layout: default
title: GCP Firestore Backups
description: Perform scheduled Firestore exports from your app
parent: Packages
nav_order: 3
---

# MondoKit GCP Firestore Backups

Module to perform scheduled Firestore exports in your app.

> **Note:** `@mondokit` libs all require ESM. If your project still uses CommonJS, then you can continue using our previous incarnation, [gae-js](https://mondo-mob.github.io/gae-js-docs), until you can make the upgrade.

Find the source code at [gcp-firestore-backups on GitHub](https://github.com/mondo-mob/mondokit/tree/main/packages/gcp-firestore-backups).

Backups can be configured to perform full or partial exports of Firestore into Google Cloud Storage.

As well as regular backups you can configure automatic export and import into BigQuery.
This is a full import/export and any existing data in BigQuery will be deleted for the requested kinds.

Firestore exports are long-running operations. This module keeps track of operations it starts in a `backup-operations` firestore collection.
After an operation is started it will queue tasks to update the status of the operation until it is complete.
Once complete the backup can be auto-imported into BigQuery.

## Installation

```sh
npm install @mondokit/gcp-firestore-backups
```

## Quick Start

### Create queue to handle backup tasks

By default this library uses the task queue named `backup-queue`, so create that if it doesn't exist.
Alternatively add configuration for the queue you wish to use.

e.g. add to `queue.yaml` (or add equivalent to terraform, etc)
```yaml
queue:
  - name: backup-queue
    mode: push
    rate: 1/s
    retry_parameters:
      task_retry_limit: 20
      min_backoff_seconds: 120
      max_doublings: 5
```

### Create bucket to store backup data

By default this library will use a bucket called `firestore-backup-[PROJECT_ID]`, so create that if it doesn't exist.
This must be a regional bucket in the same region as your firestore instance.
Alternatively you can add configuration for the bucket you wish to use.

### Add IAM Roles

Authorise the App Engine Default Service account to perform datastore exports
In Google Cloud Console IAM & Admin add the following IAM role: 
- `Datastore -> Cloud Datastore Import Export Admin`

### Add cron and task routes to your app

e.g. at the root application
```typescript
  app.use("/crons", appEngineCron, firestoreBackupCronRoutes());
  app.use("/tasks", appEngineTask, firestoreBackupTaskRoutes());
```

or as part of a sub-application/sub-router
```typescript
  const cronsController = Router();
  cronsController.use(firestoreBackupCronRoutes())
  cronsController.get("/some-other-cron", asyncHandler(async () => {}))

  app.use("/crons", appEngineCron, cronsController);

  const tasksController = Router();
  tasksController.use(firestoreBackupTaskRoutes())
  tasksController.post("/some-other-task", asyncHandler(async () => {}))

  app.use("/tasks", appEngineTask, tasksController);
```

### Add Cron schedule

Add cron schedules to your `cron.yaml` for the backup jobs you wish to run. e.g.

```yaml
cron:
  - description: "Firestore export"
    url: /crons/backups/firestore?type=EXPORT&name=FullBackup
    schedule: every day 03:00
    timezone: Australia/NSW

  - description: "Firestore export and load to BigQuery"
    url: /crons/backups/firestore?type=EXPORT_TO_BIGQUERY&name=ExportToBigQuery&targetDataset=backup_data&collectionIds=demo-items
    schedule: every day 03:00
    timezone: Australia/NSW
```

#### Endpoint options

Default endpoint: `GET /crons/backups/firestore`

Query params:

| Property      | Description                                                                    | Required                               |
|---------------|--------------------------------------------------------------------------------|----------------------------------------|
| type          | The type of export to perform. Must be one of "EXPORT" or "EXPORT_TO_BIGQUERY" | Y                                      |
| name          | name for the datastore export                                                  | N                                      |
| collectionIds | the firestore collections to be exported                                       | N for EXPORT, Y for EXPORT_TO_BIGQUERY |
| targetDataset | the target dataset for import in BigQuery                                      | Y for EXPORT_TO_BIGQUERY               |


### Edit configuration (if required)

No configuration is required if you are happy with the default conventions. 

The following options are available under the `firestoreBackup` namespace.

| Property     | Description                                                                                   | Required |
|--------------|-----------------------------------------------------------------------------------------------|----------|
| bucket       | the GCS bucket to export to. Default is `firestore-backup-[PROJECT_ID]`                       | N        |
| folderFormat | folder format to use - using Luxon date formatting options. Default `yyyy/MM/yyyyMMdd-HHmmss` | N        |
| queue        | the queue to send backup tasks to. Default `backup-queue`                                     | N        |
| taskPrefix   | tasks handler url prefix. Default `/tasks`                                                    | N        |
| taskService  | the app engine service to direct task requests to. Defaults to `default` service              | N        |
| timeZone     | the timezone to use when formatting backup output folder. Defaults to `"Australia/Sydney"     | N        |

e.g.
```json
{
  "projectId": "my-project-dev",
  "firestoreBackup": {
    "bucket": "my-backup-bucket",
    "queue": "admin-queue",
    "taskPrefix": "/tasks/admin",
    "taskService": "backup-service",
    "timeZone": "Australia/Sydney"
  }
}
```
