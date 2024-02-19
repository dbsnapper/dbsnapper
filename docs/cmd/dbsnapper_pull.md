---
title: "dbsnapper pull"
description: "DBSnapper Agent Command Reference: 'dbsnapper pull' - Retrieve a snapshot and store it locally"
---
Retrieve a snapshot and store it locally

### Synopsis


The `dbsnapper pull` command downloads a snapshot from the DBsnapper Cloud or finds it locally. the `pull` command takes  `target_name` and `snapshot_index` as arguments.

The `target_name` is the name of the target defined in the configuration file.

The `snapshot_index` is the index number of the snapshot to load.

!!! note
	If a sanitized snapshot exists at this `snapshot_index`, it will be loaded
	unless the `--original` flag is set.

	

```
dbsnapper pull target_name snapshot_index [flags]
```

### Options

```
  -o, --original   Use the original snapshot instead of the sanitized version.
```

