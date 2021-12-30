# Migrating from v2.3 to v2.4

Upgrading from `v2.3` to `v2.4` is fairly straightforward if you follow the instructions. Even though we try to ensure backward compatibility (BC) as much as possible, sometimes it is not possible or very complicated to avoid it and still create a good solution to a problem.

::: warning
Upgrading a complex software project always comes at the risk of breaking something, so make sure you have a backup.
:::

## Notes

After upgrading you should check whether your application still works as expected and no plugins are broken. See the following notes on which changes to consider when upgrading from one version to another.

## Upgrade Steps

::: tip
Be sure to complete all of the following steps before you continue to upgrade with the `swipechain update` command to the latest version.
:::

### Step 1. Add `core-state` Package

1. Open `~/.config/swipechain-core/mainnet/plugins.js`
2. Locate the `@swipechain/core-database-postgres` entry.
3. Add this package addition line before it (see below):

   ```js
   "@swipechain/core-state": {}, // Add this line before it
   ```

4. Save the changes. Your configuration file should look like this:

   ```js
   module.exports = {
       "@swipechain/core-event-emitter": {},
       "@swipechain/core-logger-pino": {},
       "@swipechain/core-state": {},
       "@swipechain/core-database-postgres": {},
       ...
       ...
       ...
   }
   ```

### Step 2. Add `core-wallet-api` Package

::: warning
It's especially important to register this plugin as this is what the Desktop and Mobile wallet will use to communicate with nodes.
:::

1. Open `~/.config/swipechain-core/mainnet/plugins.js`
2. Locate the `@swipechain/core-blockchain` entry.
3. Add this package addition line after it (see below):

   ```js
   "@swipechain/core-wallet-api": {}, // Add this line after it
   ```

4. Save the changes. Your configuration file should look like this:

   ```js
   module.exports = {
       ...
       ...
       "@swipechain/core-blockchain": {},
       "@swipechain/core-wallet-api": {},
       ...
       ...
   }
   ```

5. If you are using default firewall don't forget to open port `4040` to make `core-wallet-api` accessible to wallets.

   ```shell
   sudo ufw allow 4040/tcp
   ```

   ::: tip
   You can check your ufw rules by running `sudo ufw status`
   :::

### Step 3. Update `core-p2p` Configuration

1. Open `~/.config/swipechain-core/mainnet/plugins.js`
2. Locate the `@swipechain/core-p2p` entry and replace the block like shown below.

   **Old**

   ```js
   module.exports = {
       ...
       "@swipechain/core-p2p": {
           host: process.env.CORE_P2P_HOST || "0.0.0.0",
           port: process.env.CORE_P2P_PORT || 4001,
       },
       ...
   }
   ```

   **New**

   ```js
   module.exports = {
       ...
       "@swipechain/core-p2p": {
           server: {
               port: process.env.CORE_P2P_PORT || 4001,
           },
       },
       ...
   }
   ```

3. If you have a `whitelist` property in the `core-p2p` entry, make sure to remove this too. Starting with v2.4 this property will filter out any peers that don't match the whitelist.
4. Save the changes.

### Step 4. Update `core-forger` Configuration

1. Open `~/.config/swipechain-core/mainnet/plugins.js`
2. Locate the `@swipechain/core-forger` entry and replace the block like shown below.

   **Old**

   ```js
   module.exports = {
       ...
       "@swipechain/core-forger": {
          hosts: [`http://127.0.0.1:${process.env.CORE_P2P_PORT || 4001}`],
       },
       ...
   }
   ```

   **New**

   ```js
   module.exports = {
       ...
       "@swipechain/core-forger": {
           hosts: [
              {
                 hostname: "127.0.0.1",
                 port: process.env.CORE_P2P_PORT || 4001,
              },
          ],
       },
       ...
   }
   ```

3. Save the changes.

### Step 5. Update `core-json-rpc` to `core-exchange-json-rpc`

::: warning
This only applies if you have the JSON-RPC registered in your `plugins.js` file, otherwise skip this section.
:::

```sh
sed -i 's/CORE_JSONRPC/CORE_EXCHANGE_JSON_RPC/g' ~/.config/swipechain-core/mainnet/.env
sed -i 's/CORE_JSON_RPC/CORE_EXCHANGE_JSON_RPC/g' ~/.config/swipechain-core/mainnet/.env
sed -i 's/CORE_JSONRPC/CORE_EXCHANGE_JSON_RPC/g' ~/.config/swipechain-core/mainnet/plugins.js
sed -i 's/CORE_JSON_RPC/CORE_EXCHANGE_JSON_RPC/g' ~/.config/swipechain-core/mainnet/plugins.js
sed -i 's/core-json-rpc/core-exchange-json-rpc/g' ~/.config/swipechain-core/mainnet/plugins.js
```

### Step 6. Running the Update Command via the `swipechain` CLI

::: warning
Do not run any of the mentioned commands with `sudo` unless explicitly stated.
:::

Make sure that [Step 1](#step-1-add-core-state-package), [Step 2](#step-2-add-core-wallet-api-package), [Step 3](#step-3-update-core-p2p-configuration), [Step 4](#step-4-update-core-forger-configuration) and [Step 5](#step-5-update-core-json-rpc-to-core-exchange-json-rpc) were successfully completed before running the `swipechain update` command via the cli.

**To update to v2.4 run the following command:**

```bash
swipechain update
```

## Developer Related Information

This section addresses developers and lists notable changes during this version upgrade. For more details make sure you checkout the [CHANGELOG]( https://github.com/SwipeChain/swipechain-core/blob/master/CHANGELOG.md) document. The following breaking changes where introduced in v2.4:

### 1. Fee Statistics Removed from `node/configuration` Endpoint

The fee statistic are no longer included in the response of the `node/configuration` endpoint. They can be retrieved through a new dedicated endpoint instead, `node/fees`. More information on that endpoint can be found [here](/api/public/v2/node.html#retrieve-the-fee-statistics).

## Reporting Problems

If you happen to experience any issues please [open an issue](https://github.com/Swipechain/swipechain-core/issues/new?template=Bug_report.md) with a detailed description of the problem, steps to reproduce it and info about your environment.
