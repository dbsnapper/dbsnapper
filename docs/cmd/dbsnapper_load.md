# dbsnapper load

Load a target snapshot to a database

## Synopsis

The `dbsnapper load` command loads a snapshot to a database and takes a `target_name` and `snapshot_index` as arguments.
The `target_name` is the name of the target defined in the configuration file.

The `snapshot_index` is the index number of the snapshot to load.

!!! note
If a sanitized snapshot at this exists at this `snapshot_index`, it will be loaded
unless the `--original` flag is set.

The snapshot is loaded to the database specified in the `dst_url` parameter of the target configuration.

!!! danger
This is a destructive operation and any database that exists at `dst_url` will be dropped and recreated!

```
dbsnapper load target_name snapshot_index [flags]
```

## Options

```
  -h, --help       help for load
  -o, --original   Use the original snapshot instead of the sanitized version.
```

## SEE ALSO

- [dbsnapper]](cmd/dbsnapper/) - Easy database snapshots for development and testing
