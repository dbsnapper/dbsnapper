---
title: "dbsnapper targets"
description: "DBSnapper Agent Command Reference for 'dbsnapper targets'"
weight: 100
---

# dbsnapper targets

## dbsnapper targets

Use to list all targets

```
dbsnapper targets [flags]
```

### Options

```
      --destdb string   Destination Database URL Override - Will Overwrite any existing destination database!!!
  -h, --help            help for targets
  -o, --original        Use the original snapshot instead of the sanitized version.
  -u, --update          Update the db info for all targets (default true)
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper](dbsnapper.md)	 - Simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.

