---
title: "dbsnapper sanitize"
description: "DBSnapper Agent Command Reference for 'dbsnapper sanitize'"
weight: 100
---

# dbsnapper sanitize

## dbsnapper sanitize

Used to sanitize a snapshot

### Synopsis

The `dbsnapper sanitize` command sanitizes the specified database snapshot and takes a `target_name` and `snapshot_index` as arguments.
	
The `target_name` is the name of the target defined in the configuration file.
	
The `snapshot_index` is the index number of the snapshot to load.

<!-- prettier-ignore-start -->
!!! note

    The target configuration must specifiy a `query_file` configuration parameter that specifies
    the path to the file containing the sanitization query. Absolute paths are used as-is, while relative 
    paths are resolved relative to the `working_directory`

<!-- prettier-ignore-end -->


This command will use the database specified in the `sanitize: dst_url:` configuration parameter, load the specified snapshot into the database, 
and then sanitize the database using the specified query. If `sanitize: dst_url:` is not specified, the command will use an ephemeral database via docker container.

If you would like to create a new snapshot set (unsanitized and sanitized snapshots), you can use the `-n` flag. 
This will create a new snapshot set for the target.

If you want to force ephemeral sanitization, you can use the `-e` flag. 
This will force the command to use an ephemeral database for sanitization, overriding the `sanitize: dst_url` configuration parameter.

The resulting sanitized database will be dumped to a file in the `working_directory` specified in the conifiguration and will
be associated with the snapshot.


```
dbsnapper sanitize [flags] target_name snapshot_index
```

### Options

```
  -e, --ephemeral   Create a snapshot using an ephemeral database via docker containers
  -h, --help        help for sanitize
  -n, --new         Create a new snapshot set for the target
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper](dbsnapper.md)	 - Simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.

