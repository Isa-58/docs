---
title: "How to Write a Core Plugin"
---

# How to Write a Core Plugin

In this guide, you will find information to enable you to write a proper Solar Core plugin, for use in your own Solar deployments; both in the case of Solar Core nodes a `BridgeChain` nodes.

[[toc]]

## Basic Structure & Properties of a Plugin

Plugins are very simple to write. At their core they are an object with a register property, that is a function with the signature async function. Additionally the plugin object has a required `pkg` property and several optional properties including version.

```ts
import { Container } from "@solar-network/core-interfaces";
import { LogManager } from "@solar-network/core-logger";
import { defaults } from "./defaults";
import { PinoLogger } from "./driver";

export const plugin: Container.PluginDescriptor = {
  pkg: require("../package.json"),
  defaults,
  alias: "logger",
  extends: "@solar-network/core-logger",
  async register(container: Container.IContainer, options) {
    const logManager: LogManager = container.resolvePlugin("log-manager");
    await logManager.makeDriver(new PinoLogger(options));

    return logManager.driver();
  }
};
```

### pkg

This property contains all the information about your plugin that is needed to register it like the name and version, usually this will be simply your `package.json` as it already has all of that information.

### Defaults

All of the settings that your plugin provides should come with a default value so the user needs to configure as little as possible. An exception to this rule would be things like addresses, public keys or passphrases as those are things the user should configure so he knows the values.

### Alias

In the above example you've probably noticed the `alias: "logger"` line. This serves as an alias to allow us quick access to the plugin via `container.resolvePlugin("logger")` instead of having to type the exact name of the logger we are using, e.g. `container.resolvePlugin("@solar-network/core-logger-pino")`.

::: warning
Aliases should be used with caution if you are using a lot of plugins as you might overwrite something that you did not intend to overwrite which can cause unwanted behaviours.
:::

### Extends

This property will be rarely used and only be seen in a few plugins like logger or database implementations. The `extends` property tells the container that we first need to load the plugin that was defined via the `extends` property before we can continue with registering our plugin.

Plugins that are loaded via `extends` usually don't do anything on their own as they just provide an abstract, a factory or interfaces that should be used by plugins that provide concrete implementations.

### Register

As we've seen above, the **register** method accepts two parameters, **container** and **options**.

The **container** parameter is an instance of the application container to provide you easy access to other plugins like configuration or database connections.

The **options** parameter is whatever options the user passes to your plugin when registering.

**register** should be an async function that returns once your plugin has completed whatever steps are necessary for it to be ready. Alternatively your register plugin should throw an error if an error occurred while registering your plugin.

## Setup

Many components are required to have a proper environment setup for the development of your Solar Core plugin.

You can view instructions on how to setup your development environment in the [here](./setup-dev-environment.md).

### Plugin Skeleton

Make sure you are in the Solar Core folder cloned from the official [repo](https://github.com/solar-network/core).

Add a submodule for the plugin skeleton.

```bash
cd plugins/
git submodule add -f https://github.com/solar-network/core-plugin-skeleton
cd core-plugin-skeleton
```

## Configuration

We need to make some changes to the skeleton first. Make sure to modify the default names for the files:

- **core-plugin-skeleton/** (folder name)
- **README.md** (header)
- **package.json** (many fields)
- **src/defaults.ts** (settings)
- **src/index.ts** (exports)
- **src/plugin.ts** (alias)

The name of our plugin is **demo-plugin**. Make sure to change the name in your package.json accordingly. It is recommended to scope your packages with a prefix like `@your-vendor/` to distinguish it from other npm packages. Check [https://docs.npmjs.com/misc/scope](https://docs.npmjs.com/misc/scope) for more information.

After having changed the name of the plugin, make sure to run `yarn bootstrap` to expose the package name for scoped package installation.

```bash
yarn bootstrap
```

::: warning
`yarn bootstrap` takes a long time, just let it finish obtaining all dependencies before continuing.
:::

#### Adding Dependencies

If your package relies on any dependencies you should install them via [lerna add](https://github.com/lerna/lerna/tree/master/commands/add) the plugin you are developing.

```bash
lerna add dependency-name --scope=@vendor/demo-plugin --dev
```

Once everything is set up and configured, we can move on to developing the plugin.

## Implementation

The file we'll be writing our vendor code in is called `demo.ts` and it's located in the `src/` folder of the plugin skeleton.

The sample code we will use for this demo is

```ts
import { app } from "@solar-network/core-container";

export class Demo {
  public log(message) {
    app.resolvePlugin("logger").log(message);

    return {
      success: true
    };
  }

  public exit() {
    app.resolvePlugin("logger").info("Exiting from the demo plugin");

    return {
      success: true
    };
  }
}
```

Before writing tests, it is essential to correctly set up the registration and deregistration of our plugin in the `plugin.ts` file of the `src/` folder.

```ts
import { Container } from "@solar-network/core-interfaces";
import { defaults } from "./defaults";
import { Demo } from "./demo";

export const plugin: Container.PluginDescriptor = {
  pkg: require("../package.json"),
  defaults,
  alias: "demo-plugin",
  async register(container: Container.IContainer, options) {
    return new Demo();
  },
  async deregister(container: Container.IContainer, options) {
    return container.resolvePlugin("demo-plugin").exit();
  }
};
```

### Testing

All testing happens in the root `__tests__` directory.

Basically there are 4 main folders in the `__tests__` directory : `e2e`, `functional`, `integration`, `unit`. Each corresponds to a type of tests.

Say you want to write unit tests for your plugin. Then you will create a directory inside the `unit` subfolder with your plugin name, and write your tests inside. You will run them with `yarn test unit/<yourPluginName>`.

You can read about testing details in [the testing documentation](../../../guidebook/testing.md).

## Conclusion

In the end, you should be able to write your plugin for Solar Core, with full interoperability with the existing core packages and other dependencies that might be required for your project.
