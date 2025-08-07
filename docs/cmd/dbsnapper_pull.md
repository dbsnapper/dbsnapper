---
title: "dbsnapper pull"
description: "DBSnapper Agent Command Reference for 'dbsnapper pull'"
weight: 100
---

# dbsnapper pull

## dbsnapper pull

Retrieve a snapshot and store it locally

### Synopsis

The `dbsnapper pull` command downloads a snapshot from the DBsnapper Cloud or finds it locally. the `pull` command takes  `target_name` and `snapshot_index` as arguments.

The `target_name` is the name of the target defined in the configuration file.

The `snapshot_index` is the index number of the snapshot to load.

!!! note
	If a sanitized snapshot exists at this `snapshot_index`, it will be loaded
	unless the `--original` flag is set.

	

```
dbsnapper pull <target_name> [snapshot_index] [flags]
```

### Options

```
  -h, --help       help for pull
  -o, --original   Use the original snapshot instead of the sanitized version.
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper](dbsnapper.md)	 - Simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.

