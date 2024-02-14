# dbsnapper

Easy database snapshots for development and testing

## Synopsis

DBSnapper is a database snapshot creation and management tool for developers. 
DBSnapper simplifies the build, sanitizaiton and loading of database snapshots.

DBSnapper allows you to use sanitized real-world data to speed up your application development and testing.

You can specify your database targets and other configuration properties in the `~/.config/dbsnapper/dbsnapper.yml` configuration file.


## Options

```
  -h, --help   help for dbsnapper
```

## SEE ALSO

* [dbsnapper auth](dbsnapper_auth.md)	 - Login to DBSnapper Cloud
* [dbsnapper build](dbsnapper_build.md)	 - Build a database snapshot
* [dbsnapper config](dbsnapper_config.md)	 - Configuration commands
* [dbsnapper load](dbsnapper_load.md)	 - Load a target snapshot to a database
* [dbsnapper pull](dbsnapper_pull.md)	 - Retrieve a snapshot and store it locally
* [dbsnapper subset](dbsnapper_subset.md)	 - Subset a target based on the configuration in targets.<target>.subset
* [dbsnapper target](dbsnapper_target.md)	 - List snapshots for a target
* [dbsnapper targets](dbsnapper_targets.md)	 - List all targets

