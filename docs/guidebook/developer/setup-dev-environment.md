# How to Setup Your Dev Environment

The best means of getting started with Solar development is to spin up your local network, complete with 51 test delegates. Here we will show you how to do precisely that: we will cover the tools you will need to set up your testnet, along with how to use those tools to get your network up and running.

## Installation

The following development tools need to be installed on your machine to develop an application using Solar Core:

- [NodeJS](https://nodejs.org/en/) As Solar Core is written exclusively in NodeJS, the server-side framework for JavaScript, installing Node is a necessity for core development.

- [Git](https://git-scm.com/). As the most popular version control software in existence, Git is a staple of many developer workflows, and Solar is no exception. Downloading Git will allow you to download the latest version of Solar Core, as well as update your local copy of Core to incorporate the latest code changes as they are released.

- [Docker](https://www.docker.com/). Docker is a container management tool that's rapidly becoming the industry standard. If you are unfamiliar with containers or why managing them is important, fear not: when you install Solar Core, a robust Docker setup is provided for you out of the box. However, to use that setup, you are going to need a Docker installation on your machine already.

- [Yarn](https://yarnpkg.com/en/). Yarn is a package manager that seeks to build upon the foundation of Node's npm. Although yarn is not a strict requirement, in many cases it works faster and more elegantly than npm. Most Solar developers use yarn, and as such, you will see `yarn` commands often used throughout our documentation.

## Spinning Up Your First Testnet

With the above dependencies installed, you are all set to start your first testnet. Here are the steps to doing so:

1. Navigate to a directory where you want to install Solar Core. This could be in a `projects` folder, or an `solar` folder, or something else entirely — the choice is up to you. Just make sure that you're in this folder before installing Core.

2. Once in the directory that you have created, install Solar Core using Git:

```bash
git clone git@github.com:Solar/core.git
```

3. Move inside the newly-created `core` directory.

4. Run `yarn setup`. This command will install all necessary JavaScript dependencies to get you up and running with Solar Core.

5. Although we have now installed all required JavaScript dependencies, there are a couple more missing pieces that we will need to fill in to get our testnet running. In particular, Solar Core makes use of the database technology [PostgreSQL](https://www.postgresql.org/). Installing Postgres the traditional way can be somewhat cumbersome. Fortunately, Docker has our backs here, and we've got your back with Docker.

6. Run `yarn docker solar`. This command will generate all necessary docker files. From `core`, navigate to `docker/development/testnet`. From within this directory, run the following command: `docker-compose up -d postgres`. This will install Postgres with the necessary settings to work with Solar Core.

7. Return to the `core` directory.

8. From `core`, navigate to `packages/core`. This is the starting point for all Solar nodes, `relays` and `forgers` alike.

9. From `core/packages/core`, run the command `yarn full:testnet`.

### Network Boot

This last command, `yarn full:testnet`, is where the magic happens. Let us do a quick walkthrough of what happens when this command is run:

1. The `full:testnet` command is run within `core`, which as of the time of writing executes the following command in `npm`: `cross-env CORE_PATH_CONFIG=./bin/config/testnet CORE_ENV=test yarn solar core:run --networkStart`

2. As seen in the previous step, the `./bin/run` file is called with the `core:run` command. That command looks like this:

::: tip
Take a look at the following pieces of code to get a better understand of what commands are executed under the hood and what flags can be used to manipulate behaviour and pass in data.

- [core:run](https://github.com/solar-network/solar-core/blob/develop/packages/core/src/commands/core/run.ts)
- [BaseCommand](https://github.com/solar-network/solar-core/blob/develop/packages/core/src/commands/command.ts#L20-L61)
  :::

3. Based on the command config and the options passed by the `full:testnet` command as seen in the links above, we can see that the network sets the `config` directory to `bin/config/testnet` and the `networkStart` option to `true`, which starts our testnet from scratch with a new genesis block.

```js
import { app } from "@solar-network/core-container";
import { CommandFlags } from "../../types";
import { BaseCommand } from "../command";

export class RunCommand extends BaseCommand {
    public static description: string = "Run the core (without pm2)";

    public static flags: CommandFlags = {
        ...BaseCommand.flagsNetwork,
        ...BaseCommand.flagsBehaviour,
        ...BaseCommand.flagsForger,
    };

    public async run(): Promise<void> {
        const { flags } = await this.parseWithNetwork(RunCommand);

        await this.buildApplication(app, flags, {
            options: {
                "@solar-network/core-p2p": this.buildPeerOptions(flags),
                "@solar-network/core-blockchain": {
                    networkStart: flags.networkStart,
                },
                "@solar-network/core-forger": await this.buildBIP38(flags),
            },
        });
    }
}
```

5. We'll delve more into the inner mechanics of `core-container` [here](/guidebook/core/node-lifecycle.html#bootstrapping-our-container). For now, we are going to take a brief look at the `setUp` method, found in the `core-container` package:

```js
public async setUp(version: string, variables: any, options: any = {}) {
    // Register any exit signal handling
    this.registerExitHandler(["SIGINT", "exit"]);

    // Set options and variables
    this.options = options;
    this.variables = variables;

    this.setVersion(version);

    // Register the environment variables
    const environment = new Environment(variables);
    environment.setUp();

    // Mainly used for testing environments!
    if (options.skipPlugins) {
        this.isReady = true;
        return;
    }

    // Setup the configuration
    this.config = await configManager.setUp(variables);

    // Setup the plugins
    this.plugins = new PluginRegistrar(this, options);
    await this.plugins.setUp();

    this.isReady = true;
}
```

1. After setting up environment variables based on the passed-in configuration, all Core plugins are loaded using the `options` key of the second argument to `container.setUp`. You can find the installed plugins in the `plugins.js` file located in the `core` package at `bin/config/testnet`.

This last step is where the meat-and-potatoes of Solar Core is loaded. During this step, the Postgres database is set up, all Solar-specific tables and fields are migrated, the genesis block is created, 51 forging delegates are created and set up to forge blocks — all the blockchain goodness you would expect from of a fully-formed testnet.

A full walkthrough of the node setUp process will be accessible in the Guidebook shortly, and further posts in the tutorials will guide you through some of the most common functions you will want to perform with your testnet. However, by following the steps above, you will be up and running with your very own network.
