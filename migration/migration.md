---
layout: default
title: Migration from gae-js
nav_order: 3
permalink: /migration-from-gae-js
---

# Migration from gae-js

Before you can migrate to using all `@mondokit` libraries, you need to ensure your project is ESM packaged and, if using typescript, on version 5 or higher. This library will _not work_ with `CommonJS` (`reequire`).
Performing the migration from `CommonJS` to `ESM` is beyond this guide, as there are plenty of great guides available already online.

If you cannot upgrade to `ESM` then you will need to defer upgrading to this library and continue with `gae-js`. Note that `gae-js` will work in an `ESM` build, so you can do the upgrade and make sure everything runs fine first.

## Minimum requirements

Through internal dependency updates, the minimum NodeJS version supported is 18. Some libraries may work with NodeJS 14.

## Unsupported packages

* [@mondomob/gae-js-search](https://mondo-mob.github.io/gae-js-docs/packages/gae-js-gae-search.html) - this package utilises the deprecated and soon to be unsupported App Engine Search API. Although Google have extended grace periods a few times on this, they have made it clear that it will not stay around. As such we do not aim to maintain library support for the solution for proxying to a Java search service.


## Package mapping

In most cases, you can simply search and replace your imports of `@mondomob/gae-js` with `@mondokit/gcp` as the suffixes have remained the same for the libs.

The only exceptions to this are:
* the [unsupported packages](#unsupported-packages)
* [@mondomob/gae-js-migrations](https://mondo-mob.github.io/gae-js-docs/packages/gae-js-migrations.html) is specific to `Firestore` and thus has been renamed to [@mondokit/gcp-firestore-migrations](../packages/gcp-firestore-migrations.md)

## Breaking changes per package
In addition to migration steps outlined in [Steps to migrate](#steps-to-migrate), the following specific breaking changes need to be considered.
* [@mondokit/gcp-core](../packages/gcp-core.md)
  * The default logger needs to be _explicitly_ set to use the `GCP` one, given the logger now resides in our [core](../packages/core.md) package that is cloud agnostic. Add the following to your initialisation file:

    ```typescript
    import { defaultLoggerProvider, googleCloudLogger } from "@mondokit/gcp-core";
    // ...
    defaultLoggerProvider.set(googleCloudLogger());
    ```
  * Names with `gaeJs` or `GaeJs` have been renamed. Search/replace all instances below:
    * `gaeJsApp` => `gcpApp`
    * `GaeJsCoreConfiguration` => `GcpCoreConfiguration`
    * `gaeJsCoreConfigurationSchema` => `gcpCoreConfigurationSchema`
    * `gaeJsCron` => `appEngineCron`
    * `gaeRequestLogger` => `gcpRequestLogger`
  * Environment variable names changed. Check your code, CI, runtime configuration and make the following replacements:
    * `GAEJS_ENVIRONMENT` => `MONDOKIT_ENVIRONMENT`
    * `GAEJS_CONFIG_DIR` => `MONDOKIT_CONFIG_DIR`
    * `GAEJS_OVERRIDES` => `MONDOKIT_CONFIG_OVERRIDES`
  * Removed deprecated items
    * `asArray` has been deprecated for a while. Replace with [lodash castArray](https://lodash.com/docs/4.17.15#castArray) instead, as it's exactly what this function did.
* [@mondokit/gcp-tasks](../packages/gcp-tasks.md)
  * Search/replace all instances:
    * `gaeJsTask` => `appEngineTask`
* [@mondokit/gcp-storage](../packages/gcp-storage.md)
  * Search/replace all instances:
    * `GaeJsStorageConfiguration` => `GcpStorageConfiguration`
    * `gaeJsStorageConfigurationSchema` => `gcpStorageConfigurationSchema`
* [@mondokit/gcp-bigquery](../packages/gcp-bigquery.md)
  * Search/replace all instances:
    * `GaeJsBigQueryConfiguration` => `GcpBigQueryConfiguration`
    * `gaeJsBigQueryConfigurationSchema` => `gcpBigQueryConfigurationSchema`
* [@mondokit/gcp-firestore](../packages/gcp-firestore.md) 
  * removed the `@Transactional` annotation as it was based on the original _experimental decorators_. Rather than update this, we have kept the library lightweight and omitted it altogether.
  * Search/replace all instances:
    * `GaeJsFirestoreConfiguration` => `GcpFirestoreConfiguration`
    * `gaeJsFirestoreConfigurationSchema` => `gcpFirestoreConfigurationSchema`
* [@mondokit/gcp-datastore](../packages/gcp-datastore.md) 
  * removed the `@Transactional` annotation as it was based on the original _experimental decorators_. Rather than update this, we have kept the library lightweight and omitted it altogether.
  * Search/replace all instances:
    * `GaeJsDatastoreConfiguration` => `GcpDatastoreConfiguration`
    * `gaeJsDatastoreConfigurationSchema` => `gcpDatastoreConfigurationSchema`
* [@mondokit/gcp-datastore-backups](../packages/gcp-datastore-backups.md)
  * Search/replace all instances:
    * `GaeJsDatastoreBackupConfiguration` => `GcpDatastoreBackupConfiguration`
    * `gaeJsDatastoreBackupConfigSchema` => `gcpDatastoreBackupConfigSchema`

## Steps to migrate

For each package, the following steps should suffice:

1. If possible, upgrade your `@google-cloud` lib dependencies where you have them explicitly defined. Check the changelog entries for each lib to see breaking changes. The latest lib may not be a hard requirement for migrating to `@mondokit` libs.
2. Validate nothing breaks
3. `npm uninstall @mondomob/gae-js-xxx` lib
4. `npm i @mondokit/gcp-xxx` lib (see [package mapping](#package-mapping))
5. Search/replace your js/ts files to fix imports based on your package mapping
6. Fire it up!

