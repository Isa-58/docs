# Creating and Using Snapshots

Node operators are most likely familiar with creating snapshots of their database and restoring their nodes using a snapshot to lower downtime. In Solar V1, Postgres' native SQL dump was used to create snapshots. Solar V2 has packages to optimize snapshots and increase the ease of use.

## Summary

The `core-snapshots` package facilitates the process of creating, verifying, and applying blockchain backups. This suite of packages can be used to create regular database backups in a serialization format understood by all Solar Core nodes.

To ensure that snapshots are usable across the network, it is often helpful to establish a common standard of data serialization. If all nodes agree on a single ruleset governing how blockchain data maps into a database representation, it becomes much easier to compare, validate, and verify snapshots created by different nodes. `core-snapshots` offers three such standards, also known as codecs, covering a wide range of use cases:

- The `lite` codec utilizes a MessagePack encoding format with Solar-specific key-value pairs. This encoding format is faster than the `solar` format, but results in larger backup file sizes, as keys are stored alongside their respective values
- The `solar` codec uses Solar's serialize and deserialize standards in creating the backup. As the Solar serialization protocol does not include key data, this codec results in smaller database backup sizes. However, this density comes at the expense of performance, as Solar serialization (currently) cannot match MessagePack's encoding and decoding speed.
- The `msgpack` codec uses MessagePack without any Solar-specific standards. As this codec has no specific knowledge of Solar serialization, this option is both the fastest and the most inefficient regarding snapshot file size.

With all options, the tradeoff to keep in mind is performance vs. filesystem impact. If you have an external storage solution for backups and limited computational resources, using `msgpack` will ensure maximum performance across all potential use cases. Alternatively, if your node setup is robust and backup creation speed is not a relevant factor, the `solar` codec might be the right choice.

If you're unsure of which to choose, use the `lite` codec. Generally speaking, it offers the best tradeoff between performance and impact.

## Usage

The `@solar-network/core` is a command-line interface designed to help node operators and developers automate their backup creation workflow. While the commands themselves can be found with `solar snapshot --help`, the source code behind these commands can be found in the `packages/core/src/commands/snapshot` [file](https://github.com/solar-network/solar-core/blob/develop/packages/core/src/commands/snapshot).

## Create a Snapshot

Calling the `dump` CLI command prompts your node to create a backup and save it in the data directory specified at runtime. The folder name will follow the format `{data}/snapshots/{network}/{startblock}-{endblock}` and contains `transactions.lite`, `blocks.lite` and `meta.json`.

### Creating a New Snapshot

To create a snapshot, run the following command:

```bash
solar snapshot:dump
```

The command will generate snapshot files in your configured folder. By default, this folder will be in `~/.local/share/solar-core/NETWORK_NAME/snapshots`. Files names follow the pattern: `{TABLE}.{CODEC}` For example, running `yarn dump:devnet` will create the following files in the folder `~/.local/share/solar-core/devnet/snapshots/0-331985/`:

- blocks.lite
- transactions.lite

The codec used can be specified using the `—codec` flag, for example:

```bash
solar snapshot:dump --codec solar
```

The folder `0-331985` indicates that the snapshot includes data between block 0 and block 331985.

Using the optional `—start` and `—end` flags will specify a lower and uppers bounds for the snapshot, allowing you to customize your backups to your specific needs.

> Click [here](/guidebook/core/cli.md#snapshot-dump) to see the flags that can be added to the `solar snapshot:dump` command at runtime or type `solar snapshot:dump --help`.

### Append Data to an Existing Snapshot

Appending data to existing snapshots can help manage snapshot size and improve snapshot import speeds. The command is the same as creating a snapshot with an additional parameter for `--blocks`. This flag allows you to specify the existing snapshot blocks/folder you want to append to.

To use the `--blocks` flag, provide the `0-331985` blocks number or folder name as an argument:

```bash
solar snapshot:dump --blocks 0-331985
```

Note that all appends create new backup folders and leave the original snapshot intact. To preserve hard disk space, remove old backups if you are sure your appended snapshot is valid.

## Restoring a Snapshot

The `restore` command allows you to restore your Solar Core node with data from a backup you previously created.

Restoring new snapshots **should not be done while your node is still running**, as running a blockchain node without blockchain data can lead to unexpected behavior.

Snapshot filename must be specified:

```bash
solar snapshot:restore --blocks 0-331985
```

If you want to restore from block 1, e.g., empty database first, you should run the yarn truncate:NETWORK_NAME command.

```bash
solar snapshot:truncate
```

Alternatively, add the `—truncate` flag to the `restore` command to truncate and restore with one command:

```bash
solar snapshot:restore --blocks 0-331985 --truncate
```

By default, the block height is set to last finished round (blocks are deleted at the end). If you have more snapshots files following each other, then you can disable this behavior with the `--skip-revert-round` flag. If this flag is present, block height will not be reverted at the end of restore to last completed round.

If you want to do additional `crypto.verify` check for each block and transaction a flag `--signatureVerify` can be added to the restore command

```bash
solar snapshot:restore --blocks 0-331985 --truncate --signatureVerify
```

Please note that this will increase the restore time drastically.

> Click [here](/guidebook/core/cli.md#snapshot-restore) to see the flags that can be added to the `solar snapshot:restore` command at runtime or type `solar snapshot:restore --help`.

## Verify Existing Snapshot

Verifying a snapshot with the `verify` command involves running checks on a snapshot to ensure all signatures are cryptographically valid and that block hashes follow each other in a logical progression to create a valid blockchain.

Please note that the `verify` command does not interact with the database in any way. It is therefore safe, and a good idea, to verify all snapshots before importing them into your Solar Core node.

```bash
solar snapshot:verify --blocks 0-331985
```

You can also verify the chaining process and skip signature verification with `--skip-sign-verify` option.

```bash
solar snapshot:verify --blocks 0-331985 --skip-sign-verify
```

Note that database verification is run by default whenever a node boots up. Although this procedure ensures network consistency, importing an invalid snapshot will increase the amount of time it takes your node to sync with the network. Trust, but verify — even with your snapshots.

> Click [here](/guidebook/core/cli.md#snapshot-verify) to see the flags that can be added to the `solar snapshot:verify` command at runtime or type `solar snapshot:verify --help`.

## Performing a Rollback

The `rollback` command can be used to roll your blockchain database back to a specific height. This can be useful to manually alter your blockchain structure in case the rollback features included in Solar Core are not accurate enough for your use case.

The following command will rollback the chain to block height of 350000:

```bash
solar snapshot:rollback --height 350000
```

If the `--height` argument is not set, the command will rollback the chain to the last completed round.

Rollback command also makes a backup of forged transactions, ensuring that no local history is accidentally deleted in a rollback. Transactions are stored next to the snapshot files (in `~/.local/share/solar-core/NETWORK_NAME/snapshots/`). The file is named `rollbackTransactionBackup.startBlockHeight.endBlockHeight.json`.

For example: `rollbackTransactionBackup.53001.54978.json` contains transactions from block 53001 to block 54978.

> Click [here](/guidebook/core/cli.md#snapshot-rollback) to see the flags that can be added to the `solar snapshot:rollback` command at runtime or type `solar snapshot:rollback --help`.

## Implementation

Behind the scenes, `core-snapshots` uses NodeJS streams to process export and import commands. The export and import pipelines are modified based on various runtime options, such as whether the output should be gzipped or not. Here is a condensed version of the business logic behind the `export` command:

```js
exportTable: async (table, options) => {
    const snapFileName = utils.getPath(
      table,
      options.meta.folder,
      options.codec,
    )
    const codec = codecs.get(options.codec)
    const gzip = zlib.createGzip()
    await fs.ensureFile(snapFileName)

    try {
      const snapshotWriteStream = fs.createWriteStream(
        snapFileName,
        options.blocks ? { flags: 'a' } : {},
      )
      const encodeStream = msgpack.createEncodeStream(
        codec ? { codec: codec[table] } : {},
      )
      const qs = new QueryStream(options.queries[table])

      const data = await options.database.db.stream(qs, s => {
        if (options.meta.skipCompression) {
          return s.pipe(encodeStream).pipe(snapshotWriteStream)
        }

        return s
          .pipe(encodeStream)
          .pipe(gzip)
          .pipe(snapshotWriteStream)
      })
      logger.info(
        `Snapshot: ${table} done. ==> Total rows processed: ${
          data.processed
        }, duration: ${data.duration} ms`,
      )

      return {
        count: utils.calcRecordCount(table, data.processed, options.blocks),
        startHeight: utils.calcStartHeight(
          table,
          options.meta.startHeight,
          options.blocks,
        ),
        endHeight: options.meta.endHeight,
      }
    } catch (error) {
      app.forceExit('Error while exporting data via query stream', error)
    }
  },
```

You can find more information about streams and pipes in the [NodeJS documentation](https://nodejs.org/api/stream.html).

### Default Parameters

```js
{
  codec: 'lite',
  chunkSize: 50000,
}
```
