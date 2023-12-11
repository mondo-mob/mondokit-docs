---
layout: default
title: GCP Storage
description: Use Cloud Storage in your app
parent: Packages
nav_order: 7
---

# MondoKit GCP Storage

Use Cloud Storage in your app.

## Installation

```sh
npm install @mondokit/gcp-storage
```

## Components

### Configuration

All relevant configuration is within the _required_ `storage` object.

- `storage.defaultBucket`: (required) The default bucket to use for storage
- `storage.apiEndpoint`: (optional) The storage API endpoint to use - rarely used
- `storage.serviceAccountKey`: (optional) User/Service Account Credentials - mainly used for local development (see Local Development below)
- `storage.credentials`: (optional) User/Service Account Credentials - mainly used for local development (see Local Development below). Ignored if `serviceAccountKey` also provided.
- `storage.emulatorHost`: (optional) The emulator host to connect to (only if using emulator)
- `storage.origin`: (optional) specific origin to use for upload urls. Defaults to core `host` configuration.
- `storage.skipDefaultBucketValidation`: (optional) skips checking the default bucket exists. By default, in the background, a check is done with `bucket.exists()` and logs 
  an error if it does not exist, so that we get proactive log notification. This action requires read permissions, however, and there are cases where the application has write
  access to a bucket, without read access. If this is the case, simply use this option to skip the validation.

### StorageProvider

Initialise storage to be accessed elsewhere in your app.

Step 1: Add default bucket to your config

```json
{
  "storage": {
     "defaultBucket": "my-test-bucket"
  }
}
```

Step 2: Initialise storage

```typescript
// On app startup
storageProvider.init();
```

Step 3: Use storage

```typescript
// Anywhere in your app
const storage = storageProvider.get();
const [files] = await storage.bucket("some-bucket").getFiles();
```

### StorageService

Helper service for common tasks

```typescript
// Create new service instance
const storageService = new StorageService();

// Create upload url to default bucket
const uploadUrl = storageService.getDefaultBucketResumableUploadUrl("newfile.txt");

// Use default bucket
const [files] = await storageService.defaultBucket.getFiles();
```

## Local Development

The Storage emulator currently doesn't support a huge amount of Cloud Storage functionality - so often you'll use a
cloud based bucket instead. No special config is required and the Cloud Storage client will use any Application Default
Credentials (ADC) you have configured. However, if you wish to use signed urls this won't work because no `client_email`
is provided as part of these credentials. If you need to support this you have a couple of options:

1) You can configure the `gcp-storage` lib to connect using a service account private key stored in Secrets
   Manager (which is accessible via your Application Default Credentials). This way you can reuse the same secret
   between developers and do not need to store the key on your local machine:
    1) Create a service account with minimal required access and download a key in JSON format
    2) Create a new secret for the credentials. e.g. LOCAL_STORAGE_KEY. You can use either the entire key file or just the private key (i.e. only the `private_key` value from the json key file). 
    3) Delete local key file
    4) Add credentials configuration to your `local.json` app configuration using the auto secret lookup.

        e.g. When entire JSON key file is stored as secret
        ```json
        {
          "secretsProjectId": "project-with-secret",
          "storage": {
             "defaultBucket": "my-test-bucket",
             "serviceAccountKey": "SECRET(LOCAL_STORAGE_KEY)"
          }
        }
        ```

        OR if only private key is stored as secret
        ```json
        {
          "secretsProjectId": "project-with-secret",
          "storage": {
             "defaultBucket": "my-test-bucket",
             "credentials": {
                "clientEmail": "local-storage@project-with-service-account.iam.gserviceaccount.com",
               "privateKey": "SECRET(LOCAL_STORAGE_KEY)"
             } 
          }
        }
        ```

2) Another approach is to download a service account key and then set the GOOGLE_APPLICATION_CREDENTIALS environment
   variable to point to the key file. This environment variable will apply to all the Google client libs you are using,
   e.g. Firestore, Secrets, so roles must be configured accordingly. This can be advantageous in that the service
   account roles can be scoped to match the real roles the application account has when running on GCP.

3) You can also pass any standard Storage client configuration options directly when initialising the Storage provider. e.g.

```typescript
storageProvider.init({
 keyFile: "service-account.json",
});
```
