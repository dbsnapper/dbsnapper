---
title: "dbsnapper build"
description: "DBSnapper Agent Command Reference: 'dbsnapper build' - Build a database snapshot"
---
Build a database snapshot

### Synopsis


The `build` command creates a database snapshot.	

The `target_name` is the name of the target defined in the configuration file.

The snapshot is built from the connection string specified in the `src_url` parameter of the target configuration. 



```
dbsnapper build <target_name>
```

### Examples

```
dbsnapper build sakila
```

