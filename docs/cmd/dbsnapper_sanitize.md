---
title: "dbsnapper sanitize"
description: "DBSnapper Agent Command Reference: 'dbsnapper sanitize' - Used to sanitize a snapshot"
---
Used to sanitize a snapshot

### Synopsis


The `dbsnapper sanitize` command sanitizes the specified database snapshot and takes a `target_name` and `snapshot_index` as arguments.
	
The `target_name` is the name of the target defined in the configuration file.
	
The `snapshot_index` is the index number of the snapshot to load.

<!-- prettier-ignore-start -->
!!! note

    The target configuration must specifiy a `query_file` configuration parameter that specifies
    the name of the file (located in the `working_directory`) that contains the sanitization query

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
  -n, --new         Create a new snapshot set for the target
```

