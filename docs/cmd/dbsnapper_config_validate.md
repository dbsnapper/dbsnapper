---
title: "dbsnapper config validate"
description: "DBSnapper Agent Command Reference for 'dbsnapper config validate'"
weight: 100
---

# dbsnapper config validate

## dbsnapper config validate

Validate current database tools configuration

### Synopsis

The `dbsnapper config validate` command validates that your currently configured
database client tools are properly installed and executable.

This command checks the tools specified in your configuration file and verifies
that they can be executed with their --version flag.

Examples:
  # Validate current configuration
  dbsnapper config validate
  
  # Validate and output as JSON
  dbsnapper config validate --json

```
dbsnapper config validate [flags]
```

### Options

```
  -h, --help   help for validate
      --json   Output results as JSON
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper config](dbsnapper_config.md)	 - Configuration commands

