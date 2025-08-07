# Build Command

The `build` command creates database snapshots from your configured targets. This is the primary command for capturing database state for later restoration, sharing, or sanitization.

## Overview

The build command connects to a source database, creates a comprehensive snapshot of its structure and data, and stores it locally. If cloud storage is configured, the snapshot is also uploaded to your cloud storage provider for sharing and backup.

## Syntax

```bash
dbsnapper build <target_name> [flags]
```

## Arguments

- `<target_name>`: The name of the target defined in your configuration file

## Options

### Command-Specific Flags

```bash
  -h, --help   Show help for the build command
```

### Global Flags

```bash
      --config string   Config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

## Target Configuration

Before using the build command, you need a target properly configured in your `dbsnapper.yml` file. A target defines the database connection and snapshot settings.

### Required Configuration

**Minimal Target Setup**:
A target entry is broken into sections that reflect the operation being performed. A minimal build target configuration would specify source and destination URLs under the `snapshot` section.

```yaml
targets:
  my-target:
    snapshot:
      src_url: "postgresql://user:pass@host:5432/dbname"
      dst_url: "postgresql://user:pass@dev:5432/dbname_copy"
```

### Complete Target Configuration

**Complete Target with All Options**:

```yaml
targets:
  my-target:
    name: "My Database Target" # Optional: Display name
    cpus: 4 # Optional: CPU cores for PostgreSQL operations
    snapshot:
      src_url: "postgresql://user:pass@host:5432/dbname"
      dst_url: "postgresql://user:pass@dev:5432/dbname_copy"
      schema_config: # Optional: PostgreSQL schema filtering
        include_schemas: ["public", "app_data"]
    storage_profile: "s3-storage" # Optional: Cloud storage
    sanitize: # Optional: Sanitization configuration
      dst_url: "postgresql://user:pass@san:5432/dbname_sanitized"
      query_file: "./sanitize.sql"
```

### Configuration Options

| Option            | Description                            | Required | Default            |
| ----------------- | -------------------------------------- | -------- | ------------------ |
| `src_url`         | Source database connection string      | Yes      | -                  |
| `dst_url`         | Destination database connection string | Yes      | -                  |
| `name`            | Human-readable target name             | No       | Target key         |
| `cpus`            | CPU cores for PostgreSQL operations    | No       | 1                  |
| `schema_config`   | PostgreSQL schema filtering options    | No       | Public schema only |
| `storage_profile` | Cloud storage profile name             | No       | None               |

## Example Usage

### Basic Build

```bash
dbsnapper build myapp-prod
```

### Build with Schema Filtering

```bash
# Builds only the schemas configured in schema_config
dbsnapper build postgres-target
```

## How It Works

1. **Connection**: DBSnapper connects to the source database using the `src_url` from your target configuration
2. **Schema Analysis**: For PostgreSQL databases, analyzes available schemas and applies filtering based on `schema_config`
3. **Snapshot Creation**: Creates a complete backup including schema, data, indexes, and constraints
4. **Local Storage**: Saves the snapshot as a compressed ZIP file in your working directory
5. **Cloud Upload**: If a storage profile is configured, uploads the snapshot to your cloud storage
6. **Metadata**: Records snapshot information for future reference

## Output Files

### Local Snapshots

- **Format**: `<timestamp>_<target_name>.zip`
- **Location**: Working directory specified in configuration file
- **Example**: `1752444532_dvdrental.zip`

### Sanitized Local Snapshots

- **Format**: `<timestamp>_<target_name>.san.zip`
- **Location**: Working directory specified in configuration file
- **Example**: `1752444532_dvdrental.san.zip`

### Cloud Snapshots

- **Name**: `{timestamp}_{target_name}` (user-visible identifier)
- **Storage**: Unique UUID filename in your cloud bucket
- **Access**: Available through DBSnapper Cloud interface

## Schema Filtering (PostgreSQL Only)

The build command supports advanced schema filtering for PostgreSQL databases, allowing you to control exactly which schemas are included in your snapshots.

### Schema Configuration Options

| Configuration                        | Behavior                           | Use Case                                 |
| ------------------------------------ | ---------------------------------- | ---------------------------------------- |
| No `schema_config`                   | Builds `public` schema only        | Default behavior for simple applications |
| `use_default_schema: true`           | Builds default schema (`public`)   | Explicit default schema only             |
| `use_default_schema: false`          | Builds **all available schemas**   | Multi-schema applications                |
| `include_schemas: ["public", "app"]` | Builds only specified schemas      | Selective schema inclusion               |
| `exclude_schemas: ["temp", "logs"]`  | Builds all schemas except excluded | Exclude temporary/log schemas            |

### Schema Configuration Examples

**Include Specific Schemas**:

```yaml
targets:
  myapp-prod:
    snapshot:
      src_url: "postgresql://user:pass@prod:5432/myapp"
      dst_url: "postgresql://user:pass@dev:5432/myapp_snap"
      schema_config:
        include_schemas: ["public", "app_data", "reports"]
```

**Exclude Temporary Schemas**:

```yaml
targets:
  myapp-prod:
    snapshot:
      src_url: "postgresql://user:pass@prod:5432/myapp"
      dst_url: "postgresql://user:pass@dev:5432/myapp_snap"
      schema_config:
        exclude_schemas: ["temp_data", "debug_logs", "analytics"]
```

**Build All Schemas**:

```yaml
targets:
  myapp-prod:
    snapshot:
      src_url: "postgresql://user:pass@prod:5432/myapp"
      dst_url: "postgresql://user:pass@dev:5432/myapp_snap"
      schema_config:
        use_default_schema: false # Include all available schemas
```

**Advanced Schema-Table Filtering**:

```yaml
targets:
  myapp-prod:
    snapshot:
      src_url: "postgresql://user:pass@prod:5432/myapp"
      dst_url: "postgresql://user:pass@dev:5432/myapp_snap"
      schema_config:
        include_schemas: ["public", "app_data"]
        schema_table_filters:
          public: ["users", "orders", "products"]
          app_data: ["analytics", "reports"]
```

### Important Notes

- **PostgreSQL Only**: Schema filtering only works with PostgreSQL databases. MySQL targets ignore `schema_config`
- **Dynamic Analysis**: The build process analyzes available schemas in the source database at runtime
- **Existing Schemas Only**: Only schemas that actually exist in the source database will be included
- **Load Consistency**: The load command will respect the schemas that were captured during build
- **Validation**: Schemas cannot appear in both `include_schemas` and `exclude_schemas` lists

!!! warning "MySQL Limitation"
    Schema filtering is not supported for MySQL databases. MySQL targets will use standard mysqldump without schema filtering, regardless of `schema_config` settings.

## CPU Configuration (PostgreSQL Only)

The build command supports configuring the number of CPU cores used for PostgreSQL dump operations via the `-j` flag. This feature allows you to optimize performance based on your system resources.

### CPU Configuration Priority

The CPU configuration follows a three-level priority system:

1. **Target-level CPU configuration** (highest priority)
2. **Global default CPU configuration** (medium priority)
3. **Default value** (lowest priority): Defaults to 1 CPU

### CPU Configuration Examples

**Target-Level CPU Configuration**:

```yaml
targets:
  myapp-prod:
    cpus: 4 # Use 4 CPUs for this target
    snapshot:
      src_url: "postgresql://user:pass@prod:5432/myapp"
      dst_url: "postgresql://user:pass@dev:5432/myapp_snap"
```

**Global Default CPU Configuration**:

```yaml
defaults:
  cpus: 2 # Default to 2 CPUs for all targets

targets:
  myapp-prod:
    # Will use 2 CPUs from global default
    snapshot:
      src_url: "postgresql://user:pass@prod:5432/myapp"
      dst_url: "postgresql://user:pass@dev:5432/myapp_snap"
```

### CPU Validation

- The system automatically detects the maximum available CPUs on your machine
- If configured CPU count exceeds available CPUs, a warning is displayed and the count is capped
- Operations continue with the adjusted CPU count

**Example Warning**:

```
CPU configuration warning: requested CPU count (16) exceeds maximum available CPUs (8), using 8 CPUs
```

!!! note "Database Engine Support"
    CPU configuration is currently supported for PostgreSQL only. MySQL operations ignore CPU configuration settings.

## Storage Profiles

For cloud storage, configure a storage profile:

```yaml
storage_profiles:
  aws-s3:
    type: "s3"
    region: "us-west-2"
    bucket: "my-snapshots"
    # Additional S3 configuration...
```

## Best Practices

### Timing

- **Production Databases**: Schedule builds during low-traffic periods
- **Development**: Build regularly to maintain fresh snapshots
- **Testing**: Build before major changes for quick rollback

### Security

- **Credentials**: Use environment variables or secure credential management
- **Network**: Ensure secure network connections to production databases
- **Access**: Limit build permissions to authorized personnel

### Performance

- **Large Databases**: Monitor build times and storage requirements
- **Network**: Consider bandwidth impact for cloud uploads
- **Resources**: Ensure adequate disk space for local snapshots

## Troubleshooting

### Common Issues

**Connection Failed**

```
Error: failed to connect to database
```

- Verify `src_url` connection string
- Check network connectivity
- Confirm database credentials

**Schema Configuration Errors**

```
Error: schema 'analytics' cannot be in both include and exclude lists
```

- Remove the schema from one of the lists
- Use either `include_schemas` OR `exclude_schemas` for any given schema

**Schema Not Found Warnings**

```
Warning: schema 'non_existent' specified in include_schemas but not found in database
```

- Verify schema names are correct
- Check that schemas exist in the source database
- Review available schemas in your database

**Insufficient Storage**

```
Error: no space left on device
```

- Free up disk space in working directory
- Consider cleanup of old snapshots

**Cloud Upload Failed**

```
Error: failed to upload to cloud storage
```

- Verify storage profile configuration
- Check cloud credentials and permissions
- Confirm bucket exists and is accessible

## Related Commands

- [`load`](load.md) - Load snapshots into databases
- [`sanitize`](../sanitize/sanitize.md) - Create sanitized versions of snapshots
- [`targets`](targets.md) - List available targets
- [`pull`](pull.md) - Download cloud snapshots

## See Also

- [Snapshot Configuration](../snapshot/configuration.md)
- [Cloud Storage Setup](../cloud-storage-engines/introduction.md)
- [Target Configuration](../configuration.md)