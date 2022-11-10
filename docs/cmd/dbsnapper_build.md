# dbsnapper build

Build a database snapshot

## Synopsis

The `build` command creates a database snapshot.	

The `target_name` is the name of the target defined in the configuration file.

The snapshot is built from the connection string specified in the `src_url` parameter of the target configuration. 



```
dbsnapper build target_name [flags]
```

## Options

```
  -h, --help   help for build
```

## SEE ALSO

* [dbsnapper](/cmd/dbsnapper/)	 - Easy database snapshots for development and testing

