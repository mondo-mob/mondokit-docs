---
layout: default
title: Core
description: Core cloud utility library that is not bound to a cloud provider
parent: Packages
nav_order: 12
---

# MondoKit Core

Core library components that are not bound to any cloud provider. If you are using other libraries in `@mondokit`, then it is unlikely that you will need to depend on
this directly as it is likely referenced, and relevant utilities exported, via cloud-specific libraries such as [@mondokit/gcp-core](./gcp-core.md).

> **Note:** `@mondokit` libs all require ESM. If your project still uses CommonJS, then you can continue using our previous incarnation, [gae-js](https://mondo-mob.github.io/gae-js-docs), until you can make the upgrade.

## Installation

```sh
npm install @mondokit/core
```
