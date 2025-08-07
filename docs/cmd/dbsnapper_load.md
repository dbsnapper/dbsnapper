---
title: "dbsnapper load"
description: "DBSnapper Agent Command Reference for 'dbsnapper load'"
weight: 100
---

# dbsnapper load

## dbsnapper load

Load a target snapshot or SQL file to a database

### Synopsis

The `dbsnapper load` command loads a snapshot or SQL file to a database and takes a `target_name` and optional arguments.

The `target_name` is the name of the target defined in the configuration file.

## Snapshot Loading

Load database snapshots created with the `build` command:

- `dbsnapper load target_name` - Load most recent snapshot to dst_url
- `dbsnapper load target_name 2` - Load snapshot by index to dst_url  
- `dbsnapper load target_name --original` - Load original (non-sanitized) snapshot
- `dbsnapper load target_name --destdb <url>` - Load to custom destination database

The `snapshot_index` is the index number of the snapshot to load (0 = most recent).

## SQL File Loading

Load SQL files directly to target databases:

- `dbsnapper load target_name file.sql` - Load SQL file to dst_url (default)
- `dbsnapper load target_name file.sql --src_url` - Load SQL file to src_url
- `dbsnapper load target_name file.sql --destdb <url>` - Load to custom destination
- `dbsnapper load target_name file.sql --no-drop` - Execute against existing database
- `dbsnapper load target_name file.sql --dry-run` - Preview execution without running

SQL files are automatically detected by the `.sql` extension (case-insensitive).

## Execution Modes

**Replace Mode (default):** Drops and recreates the target database before loading.

**Append Mode (--no-drop):** Executes SQL against the existing database without dropping it.

## Schema Filtering (PostgreSQL Only)

When loading snapshots, the load command respects the schema configuration for PostgreSQL databases:

**Schema Configuration Options:**
- **No Configuration:** Defaults to `public` schema only
- **Use Default Schema (`use_default_schema: true`):** Loads only the default schema
- **Use All Schemas (`use_default_schema: false`):** Loads all schemas present in the snapshot
- **Include Specific Schemas (`include_schemas`):** Loads only specified schemas that exist in the snapshot
- **Exclude Specific Schemas (`exclude_schemas`):** Loads all schemas except the excluded ones

**Important Notes:**
- Only schemas that exist in the snapshot will be loaded (no empty schemas created)
- If requested schemas don't exist in the snapshot, warnings will be displayed
- The load operation analyzes snapshot contents to determine available schemas
- Schema filtering applies to PostgreSQL snapshot loading only, not SQL file loading or MySQL
- MySQL databases use standard restore without schema filtering

## Safety Features

The command includes multiple safety features:

- **Interactive Confirmation:** Prompts before destructive operations
- **Dry Run Mode:** Preview SQL execution without making changes  
- **File Validation:** Validates SQL files before execution
- **Transaction Support:** Wraps SQL execution in transactions for rollback capability

To skip confirmations in automated environments, set `CI=true` or `DBSNAPPER_NO_CONFIRM=true`.

!!! note "Sanitized Snapshots"
	If a sanitized snapshot exists at the specified index, it will be loaded
	unless the `--original` flag is set.

!!! warning "Destructive Operation"
	This is a destructive operation and any database that exists at the target URL 
	will be dropped and recreated! Use `--no-drop` flag with SQL files to 
	execute against existing database.

!!! tip "Database URL Priority"
	Database URL resolution follows this priority:
	1. `--destdb` flag (highest priority)
	2. `--src_url` flag (determines src_url vs dst_url)
	3. Configuration overrides
	4. Target configuration defaults

## Examples

Load snapshots:
```bash
dbsnapper load production-db              # Load latest snapshot
dbsnapper load production-db 5            # Load snapshot at index 5
dbsnapper load production-db --original   # Load original snapshot
```

Load SQL files:
```bash
dbsnapper load dev-db schema.sql          # Load schema to dst_url
dbsnapper load dev-db data.sql --src_url   # Load data to src_url  
dbsnapper load dev-db migration.sql --dry-run  # Preview migration
```

Safety examples:
```bash
dbsnapper load prod-db update.sql --no-drop    # Execute without dropping
DBSNAPPER_NO_CONFIRM=true dbsnapper load ...   # Skip confirmations
```

Schema filtering configuration example:
```yaml
targets:
  my-target:
    snapshot:
      schema_config:
        use_default_schema: false         # Load all schemas from snapshot
        # OR
        include_schemas: ["public", "app_data"]  # Load only these schemas
        # OR  
        exclude_schemas: ["temp_schema"]   # Load all except these schemas
```



```
dbsnapper load target_name [snapshot_index|file.sql] [flags]
```

### Options

```
      --destdb string   Destination Database URL Override - Will Overwrite any existing destination database!!!
      --dry-run         Show execution plan without running (SQL files only)
  -h, --help            help for load
      --no-drop         Skip dropping/recreating database (SQL files only)
      --original        Use the original snapshot instead of the sanitized version (snapshots only)
      --src_url         Load SQL file to source database instead of destination
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper](dbsnapper.md)	 - Simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.

