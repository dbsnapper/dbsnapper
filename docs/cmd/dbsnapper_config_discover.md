---
title: "dbsnapper config discover"
description: "DBSnapper Agent Command Reference for 'dbsnapper config discover'"
weight: 100
---

# dbsnapper config discover

## dbsnapper config discover

Discover and configure database client tools

### Synopsis

The `dbsnapper config discover` command automatically discovers database client tools
on your system and optionally saves the configuration to your dbsnapper configuration file.

This command will find tools like pg_dump, pg_restore, psql, mysql, and mysqldump
in common locations and validate that they are executable.

Examples:
  # Discover tools and show results
  dbsnapper config discover
  
  # Discover tools and save to configuration file
  dbsnapper config discover --save
  
  # Discover tools and output as JSON
  dbsnapper config discover --json
  
  # Discover, save, and output as JSON
  dbsnapper config discover --save --json

```
dbsnapper config discover [flags]
```

### Options

```
  -h, --help   help for discover
      --json   Output results as JSON
      --save   Save discovered tool paths to configuration file
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper config](dbsnapper_config.md)	 - Configuration commands

