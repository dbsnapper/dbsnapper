---
title: "dbsnapper config check"
description: "DBSnapper Agent Command Reference for 'dbsnapper config check'"
weight: 100
---

# dbsnapper config check

## dbsnapper config check

Check the configuration and required dependencies

### Synopsis

The `dbsnapper config check` command validates that you have the required dependencies to run dbsnapper.
It can also automatically discover and configure database client tools.


```
dbsnapper config check [flags]
```

### Options

```
      --auto-discover   Automatically discover database client tools
  -h, --help            help for check
      --save            Save discovered tool paths to configuration file (use with --auto-discover)
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper config](dbsnapper_config.md)	 - Configuration commands

