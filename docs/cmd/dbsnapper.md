# dbsnapper

Easy database snapshots for development and testing

## Synopsis

DBSnapper is a database snapshot creation and management tool for developers. 
DBSnapper simplifies the build, sanitizaiton and loading of database snapshots.

DBSnapper allows you to use sanitized real-world data to speed up your application development and testing.

You can specify your database targets and other configuration properties in the `~/.config/dbsnapper/dbsnapper.yml` configuration file.


## Options

```
      --config string   config file (default is $HOME/.config/dbsnapper.yml)
  -h, --help            help for dbsnapper
```

## SEE ALSO

* [dbsnapper build](/cmd/dbsnapper_build/)	 - Build a database snapshot
* [dbsnapper config](/cmd/dbsnapper_config/)	 - Configuration commands
* [dbsnapper load](/cmd/dbsnapper_load/)	 - Load a target snapshot to a database
* [dbsnapper target](/cmd/dbsnapper_target/)	 - List snapshots for a target
* [dbsnapper targets](/cmd/dbsnapper_targets/)	 - List all targets

