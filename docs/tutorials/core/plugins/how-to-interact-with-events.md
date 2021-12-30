---
title: "How to Interact With Events"
---

# How to Interact With Events

[[toc]]

## Getting Started

Core provides a package called [core-event-emitter](https://github.com/solar-network/solar-core/tree/develop/packages/core-event-emitter/src) which exposes an instance of the [Node.js EventEmitter](https://nodejs.org/api/events.html). This plugin should not be installed manually but rather be resolved from the app container.

## Listening for Events

A common use-case is that your plugin will listen to events that core emitted in order to process the data for monitoring.

```ts
import { app } from "@solar-network/core-container";
import { EventEmitter, Logger } from "@solar-network/core-interfaces";

const logger = app.resolvePlugin<Logger.ILogger>("logger");
const emitter = app.resolvePlugin<EventEmitter.EventEmitter>("event-emitter");

emitter.on("forger.missing", delegate => {
  // This will be a wallet object that contains information like the address, username, public key, votes, etc.
  logger.warn(`${delegate.username} just missed a block.`);

  // Here we could for example send a telegram notification that we just missed a block
});
```

This example will resolve the `logger` and `event emitter` from the app container and register a listener for the `forger.missing` event. Every time the `forger.missing` event is emitted this listener will be triggered and log the message `USERNAME just missed a block.`.

::: tip
In a real world use-case you should send notifications via telegram, email, SMS or whatever else you prefer to use for notifications instead of doing simple logging for missed blocks if you intend to write a monitoring tool.
:::

## Emitting Your Own Events

A less common use-case is that your plugin will emit events that can be listened to by other plugins _(or your own for internal use, core-snapshots does this)_ rather then your plugin itself listening and reacting to events that core emitted. The steps are basically the same as for the listener.

```ts
import { app } from "@solar-network/core-container";
import { EventEmitter, Logger } from "@solar-network/core-interfaces";

const logger = app.resolvePlugin<Logger.ILogger>("logger");
const emitter = app.resolvePlugin<EventEmitter.EventEmitter>("event-emitter");

emitter.on("my-plugin.emit", ({ message }) => {
  logger.info(`Received a new message: ${message}`);
});

emitter.emit("my-plugin.emit", { message: "Hello World" });
```

This example will create a listener for the `my-plugin.emit` event that will log the message `Received a new message: Hello World` when `my-plugin.emit` is emitted later on.

## Conclusion

This guide should give you a basic idea of how to interact with the event emitter, to learn more about the emitter itself you can check out the [Node.js EventEmitter](https://nodejs.org/api/events.html) documentation to build feature-rich plugins.
