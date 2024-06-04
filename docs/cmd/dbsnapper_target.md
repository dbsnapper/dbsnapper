---
title: "dbsnapper target"
description: "DBSnapper Agent Command Reference: 'dbsnapper target' - Used to list snapshots for a target"
---
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
      --noui            Disable the Terminal UI
  -o, --original        Use the original snapshot instead of the sanitized version.
  -u, --update          Update the db info for all targets (default true)
```

