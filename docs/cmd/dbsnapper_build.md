---
title: "dbsnapper build"
description: "DBSnapper Agent Command Reference for 'dbsnapper build'"
weight: 100
---

# dbsnapper build

## dbsnapper build

Build a database snapshot

### Synopsis

The `build` command creates a database snapshot.	

The `target_name` is the name of the target defined in the configuration file.

The snapshot is built from the connection string specified in the `src_url` parameter of the target configuration.

## Schema Filtering (PostgreSQL Only)

The build command supports schema filtering for PostgreSQL databases to control which schemas are included in the snapshot:

**Schema Configuration Options:**
- **No Configuration:** Defaults to `public` schema only
- **Use Default Schema (`use_default_schema: true`):** Builds only the default schema
- **Use All Schemas (`use_default_schema: false`):** Builds all available schemas in the database
- **Include Specific Schemas (`include_schemas`):** Builds only the specified schemas
- **Exclude Specific Schemas (`exclude_schemas`):** Builds all schemas except the excluded ones

**Note:** Schema filtering is only supported for PostgreSQL. MySQL databases use standard dump without filtering.

## Configuration Example

```yaml
targets:
  my-target:
    snapshot:
      src_url: "postgresql://user:pass@localhost/mydb"
      dst_url: "postgresql://user:pass@localhost/mydb_copy"
      schema_config:
        use_default_schema: false         # Build all schemas
        # OR
        include_schemas: ["public", "app_data"]  # Build only these schemas
        # OR  
        exclude_schemas: ["temp_schema"]   # Build all except these schemas
```

**Important Notes:**
- Schema filtering is analyzed dynamically based on available schemas in the source database
- Only existing schemas will be included in the snapshot
- Build and load operations work together - load will respect what was built



```
dbsnapper build <target_name> [flags]
```

### Examples

```
dbsnapper build sakila
```

### Options

```
  -h, --help   help for build
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper](dbsnapper.md)	 - Simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.

