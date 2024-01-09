---
layout: default
title: GCP BigQuery
parent: Packages
description: Simplify BigQuery client initialisation and common BigQuery operations
nav_order: 9
---

# MondoKit GCP BigQuery

Simplify BigQuery client initialisation and common BigQuery operations.

> **Note:** `@mondokit` libs all require ESM. If your project still uses CommonJS, then you can continue using our previous incarnation, [gae-js](https://mondo-mob.github.io/gae-js-docs), until you can make the upgrade.

Find the source code at [gcp-bigquery on GitHub](https://github.com/mondo-mob/mondokit/tree/main/packages/gcp-bigquery).

## Installation

```sh
npm install @mondokit/gcp-bigquery
```

## Quick Start

### Using BigQuery Provider
To have a global BigQuery instance available to your entire application, initialise the `bigQueryProvider`.
This will initialise the BigQuery client based on your current configuration. The instance can
then be recalled anywhere within your application as required.

```typescript
// On app startup
bigQueryProvider.init();

// Elsewhere in the app
const bigQuery = bigQueryProvider.get();
```

### Manual connect to BigQuery
To manage your own BigQuery instances, use `connectBigQuery` to connect to BigQuery based on your
current application configuration.

```typescript
class CustomBigQueryService {
  private readonly bigQuery: BigQuery;

  constructor() {
    this.bigQuery = connectBigQuery()
  }
}
```

### Edit configuration
No configuration is required if you are happy with the default conventions. 

The following options are available under the `bigQuery` namespace.

| Property     | Description                                                                         | Required |
|--------------|-------------------------------------------------------------------------------------|----------|
| projectId    | the BigQuery projectId to connect to. Will default to the application's project id. | N        |

e.g.
```json
{
  "bigQuery": {
    "projectId": "my-bigquery-project"
  }
}
```
