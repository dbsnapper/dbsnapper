---
title: "dbsnapper build"
description: "DBSnapper Agent Command Reference for 'dbsnapper build'"
weight: 100
---

# dbsnapper build

## dbsnapper build

Build a database snapshot

### Synopsis

The `dbsnapper build` command creates a database snapshot from a configured target. The command connects to the source database specified in the target configuration and creates a compressed snapshot file that can be loaded later using the `load` command.

The `target_name` is the name of the target defined in the configuration file. The snapshot is built from the connection string specified in the `src_url` parameter of the target configuration.

## Snapshot Creation Process

The build command performs the following operations:

1. **Connects to Source Database** - Uses the `src_url` from target configuration
2. **Creates Database Dump** - Uses appropriate database tools (pg_dump, mysqldump)
3. **Applies Schema Filtering** - PostgreSQL only, based on schema configuration
4. **Creates Compressed Archive** - Saves as `.zip` file in working directory
5. **Uploads to Cloud Storage** - If storage profile is configured
6. **Applies Sanitization** - If sanitization queries are configured

## Output Files

Snapshots are created with timestamps and stored locally:

- **Local Snapshot:** `{timestamp}_{target_name}.zip`
- **Sanitized Snapshot:** `{timestamp}_{target_name}.san.zip` (if sanitization configured)
- **Cloud Storage:** Uploaded with UUID filename if storage profile exists

## Schema Filtering (PostgreSQL Only)

The build command supports advanced schema filtering for PostgreSQL databases to control which schemas are included in the snapshot:

**Schema Configuration Options:**

- **No Configuration:** Defaults to `public` schema only
- **Use Default Schema (`use_default_schema: true`):** Builds only the database's default schema
- **Use All Schemas (`use_default_schema: false`):** Builds all available schemas in the database
- **Include Specific Schemas (`include_schemas`):** Builds only the specified schemas that exist
- **Exclude Specific Schemas (`exclude_schemas`):** Builds all schemas except the excluded ones

**Schema Analysis:**
- The build process dynamically analyzes available schemas in the source database
- Only existing schemas will be included in the snapshot
- Invalid or non-existent schemas are ignored with warnings
- Schema filtering configuration is preserved for load operations

!!! note "PostgreSQL Only"
    Schema filtering is only supported for PostgreSQL databases. MySQL databases use standard mysqldump without schema-level filtering.

## Storage Integration

If a target has a configured storage profile, the build command will:

1. Create the local snapshot file
2. Upload to cloud storage using the storage profile credentials
3. Generate a cloud snapshot entry accessible via DBSnapper Cloud
4. Maintain local copy for immediate use

**Supported Storage Providers:**
- Amazon S3
- Cloudflare R2
- MinIO
- S3-Compatible providers

## Sanitization Integration

The build command integrates with sanitization functionality:

**Sanitization Priority (highest to lowest):**
1. Target-level `override_query` 
2. Global `override.san_query`
3. Target `query_file`

**Sanitization Process:**
- Original snapshot is created first
- Sanitization queries are applied to create `.san.zip` version
- Both original and sanitized versions are available for loading
- Base64-encoded queries are automatically detected and decoded

!!! warning "Sanitization Required for Sharing"
    If you plan to share snapshots or use cloud storage, ensure proper sanitization is configured to remove sensitive data.

## Configuration Examples

Basic target configuration:
```yaml
targets:
  production-db:
    snapshot:
      src_url: "postgresql://user:pass@prod.example.com/myapp"
      dst_url: "postgresql://user:pass@localhost/myapp_dev"
```

Schema filtering configuration:
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

With storage profile and sanitization:
```yaml
targets:
  production-db:
    snapshot:
      src_url: "postgresql://user:pass@prod.example.com/myapp"
      dst_url: "postgresql://user:pass@localhost/myapp_dev"
      storage_profile: "s3-production"
    sanitize:
      query_file: "sanitize-queries.sql"
```

## Safety Features

The build command includes several safety and reliability features:

- **Connection Validation** - Verifies database connectivity before starting
- **Schema Validation** - Checks that specified schemas exist (PostgreSQL)
- **Storage Validation** - Tests cloud storage credentials before upload
- **Error Recovery** - Maintains local snapshots even if cloud upload fails
- **Atomic Operations** - Ensures snapshots are complete before making available

## Examples

Basic snapshot creation:
```bash
dbsnapper build production-db           # Build snapshot of production-db target
dbsnapper build staging-db              # Build snapshot of staging-db target
```

Build with custom configuration:
```bash
dbsnapper build --config custom.yml production-db  # Use custom config file
```

!!! tip "Performance Considerations"
    - Large databases may take significant time to snapshot
    - Consider running during off-peak hours for production databases
    - Cloud upload time depends on file size and network connectivity
    - Use schema filtering to reduce snapshot size when appropriate

!!! warning "Database Permissions"
    The build command requires appropriate database permissions:
    
    **PostgreSQL:** SELECT permissions on all tables, USAGE on schemas
    **MySQL:** SELECT permissions on all databases/tables being snapshotted

```
dbsnapper build <target_name> [flags]
```

### Options

```
  -h, --help   help for build
```

### SEE ALSO

* [dbsnapper](dbsnapper.md)	 - Simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.
* [dbsnapper load](dbsnapper-load.md) - Load a target snapshot or SQL file to a database
* [dbsnapper sanitize](dbsnapper-sanitize.md) - Used to sanitize a snapshot

