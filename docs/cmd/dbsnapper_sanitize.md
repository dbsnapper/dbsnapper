# dbsnapper sanitize

Sanitize a snapshot using an Ephemeral database and query

## Synopsis

The `dbsnapper sanitize` command sanitizes the specified database snapshot and takes a `target_name` and `snapshot_index` as arguments.
	
	The `target_name` is the name of the target defined in the configuration file.
	
	The `snapshot_index` is the index number of the snapshot to load.

This command will spin up an ephemeral database container, load the specified snapshot into the database, and then sanitize the database using the specified query.

The resulting sanitized database will be dumped to a file in the `working_directory` specified in the conifiguration and will
be associated with the snapshot.


```
dbsnapper sanitize target_name snapshot_index [flags]
```

## Options

```
  -h, --help   help for sanitize
```

## Options inherited from parent commands

```
      --config string   config file (default is $HOME/.config/dbsnapper.yml)
```

## SEE ALSO

* [dbsnapper](/cmd/dbsnapper/)	 - Easy database snapshots for development and testing

