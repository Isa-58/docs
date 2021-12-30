---
title: "Webhooks"
---

# Webhooks

::: tip
You can find the source code of this package at [packages/core-webhooks](https://github.com/solar-network/solar-core/tree/develop/packages/core-webhooks).
:::

## Installation

```bash
yarn add @solar-network/core-webhooks
```

## Alias

`webhooks`

## Configuration

```ts
export const defaults = {
  enabled: process.env.CORE_WEBHOOKS_ENABLED,
  server: {
    host: process.env.CORE_WEBHOOKS_HOST || "0.0.0.0",
    port: process.env.CORE_WEBHOOKS_PORT || 4004,
    whitelist: ["127.0.0.1", "::ffff:127.0.0.1"]
  }
};
```
