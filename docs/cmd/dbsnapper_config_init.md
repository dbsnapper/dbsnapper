---
title: "dbsnapper config init"
description: "DBSnapper Agent Command Reference for 'dbsnapper config init'"
weight: 100
---

# dbsnapper config init

## dbsnapper config init

Generate dbsnapper configuration file and create working directory

### Synopsis

The `dbsnapper config init` command generates a dbsnapper configuration file and working directory. 
	
It also generates a `secret_key` used to encrypt sensitive data in the configuration file.

```
dbsnapper config init [flags]
```

### Options

```
  -h, --help   help for init
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper config](dbsnapper_config.md)	 - Configuration commands

