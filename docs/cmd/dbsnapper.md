---
title: "dbsnapper"
description: "DBSnapper Agent Command Reference for 'dbsnapper'"
weight: 1
---

# dbsnapper

## dbsnapper

Simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.

### Synopsis

DBSnapper is a database snapshot creation and management tool for developers. 
	DBSnapper makes it easy to snapshot, subset, sanitize and share your database to speed up application development and testing with real-world data.
	
	You can specify your database targets and other configuration properties in the `~/.config/dbsnapper/dbsnapper.yml` configuration file.
	

### Options

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
  -h, --help            help for dbsnapper
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper auth](dbsnapper_auth.md)	 - Login to DBSnapper Cloud
* [dbsnapper build](dbsnapper_build.md)	 - Build a database snapshot
* [dbsnapper config](dbsnapper_config.md)	 - Configuration commands
* [dbsnapper dev](dbsnapper_dev.md)	 - Development utilities (requires feature flag)
* [dbsnapper load](dbsnapper_load.md)	 - Load a target snapshot or SQL file to a database
* [dbsnapper mcp](dbsnapper_mcp.md)	 - Start the MCP (Model Context Protocol) server
* [dbsnapper pull](dbsnapper_pull.md)	 - Retrieve a snapshot and store it locally
* [dbsnapper sanitize](dbsnapper_sanitize.md)	 - Used to sanitize a snapshot
* [dbsnapper subset](dbsnapper_subset.md)	 - Used to subset a target based on the configuration in targets.<target>.subset
* [dbsnapper target](dbsnapper_target.md)	 - Used to list snapshots for a target
* [dbsnapper targets](dbsnapper_targets.md)	 - Use to list all targets

