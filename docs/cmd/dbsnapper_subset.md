---
title: "dbsnapper subset"
description: "DBSnapper Agent Command Reference for 'dbsnapper subset'"
weight: 100
---

# dbsnapper subset

## dbsnapper subset

Used to subset a target based on the configuration in targets.<target>.subset

### Synopsis

The `dbsnapper subset` command subsets the specified target resulting in a relationally-complete subset of the source database.
	
The `target_name` is the name of the target defined in the configuration file.
	


```
dbsnapper subset target_name [flags]
```

### Options

```
  -h, --help   help for subset
```

### SEE ALSO

* [dbsnapper](dbsnapper.md)	 - Simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.

