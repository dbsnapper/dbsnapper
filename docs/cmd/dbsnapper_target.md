---
title: "dbsnapper target"
description: "DBSnapper Agent Command Reference for 'dbsnapper target'"
weight: 100
---

# dbsnapper target

## dbsnapper target

Used to list snapshots for a target

### Synopsis

 The `target` command lists all snapshots for a `target_name`.

The `target_name` is the name of the target defined in the configuration file.

Specify the `--cloud` flag to only list cloud snapshots.


```
dbsnapper target target_name [flags]
```

### Options

```
      --destdb string   Destination Database URL Override - Will Overwrite any existing destination database!!!
  -h, --help            help for target
      --info            Output database and table stats using
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

