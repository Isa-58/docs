---
title: "JSON-RPC"
---

::: warning
If you are looking to just broadcast transactions you should take a look at [Create a transactions](/api/public/v2/transactions.html#create-a-transaction) for the public API instead.
:::

# JSON-RPC

::: warning
This package is deprecated and is no longer maintained and supported.
:::

::: tip
You can find the source code of this package at [core-plugins-deprecated/core-json-rpc]( https://github.com/solar-network/solar-core-plugins-deprecated/tree/master/core-json-rpc).
:::

## Installation

```bash
yarn add @solar-network/core-json-rpc
```

## Alias

`json-rpc`

## Configuration

```ts
export const defaults = {
  enabled: process.env.CORE_JSON_RPC_ENABLED,
  host: process.env.CORE_JSON_RPC_HOST || "0.0.0.0",
  port: process.env.CORE_JSON_RPC_PORT || 8080,
  allowRemote: false,
  whitelist: ["127.0.0.1", "::ffff:127.0.0.1"],
  database: {
    uri:
      process.env.CORE_JSON_RPC_DATABASE ||
      `sqlite://${process.env.CORE_PATH_DATA}/json-rpc.sqlite`,
    options: {}
  }
};
```
