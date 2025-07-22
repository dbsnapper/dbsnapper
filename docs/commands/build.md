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

```bash
  -h, --help   Show help for the build command
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
- **Format**: `{timestamp}_{target_name}.zip`
- **Location**: Current working directory
- **Example**: `1752444532_dvdrental.zip`

### Cloud Snapshots
- **Name**: `{timestamp}_{target_name}` (user-visible identifier)
- **Storage**: Unique UUID filename in your cloud bucket
- **Access**: Available through DBSnapper Cloud interface

## Schema Filtering (PostgreSQL Only)

The build command supports advanced schema filtering for PostgreSQL databases, allowing you to control exactly which schemas are included in your snapshots.

### Schema Configuration Options

| Configuration | Behavior | Use Case |
|---------------|----------|----------|
| No `schema_config` | Builds `public` schema only | Default behavior for simple applications |
| `use_default_schema: true` | Builds default schema (`public`) | Explicit default schema only |
| `use_default_schema: false` | Builds **all available schemas** | Multi-schema applications |
| `include_schemas: ["public", "app"]` | Builds only specified schemas | Selective schema inclusion |
| `exclude_schemas: ["temp", "logs"]` | Builds all schemas except excluded | Exclude temporary/log schemas |

### Configuration Examples

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
        use_default_schema: false  # Include all available schemas
```

### Important Notes

- **PostgreSQL Only**: Schema filtering only works with PostgreSQL databases. MySQL targets ignore `schema_config`
- **Dynamic Analysis**: The build process analyzes available schemas in the source database at runtime
- **Existing Schemas Only**: Only schemas that actually exist in the source database will be included
- **Load Consistency**: The load command will respect the schemas that were captured during build
- **Validation**: Schemas cannot appear in both `include_schemas` and `exclude_schemas` lists

!!! warning "MySQL Limitation"
    Schema filtering is not supported for MySQL databases. MySQL targets will use standard mysqldump without schema filtering, regardless of `schema_config` settings.

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

## Configuration Requirements

Your target must be properly configured in your `dbsnapper.yml` file:

```yaml
targets:
  myapp-prod:
    name: "Production Database"
    src_url: "postgresql://user:pass@host:5432/myapp"
    storage_profile: "aws-s3"  # Optional: for cloud storage
```

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
- [`targets`](../cmd/dbsnapper_targets.md) - List available targets
- [`pull`](../cmd/dbsnapper_pull.md) - Download cloud snapshots

## See Also

- [Snapshot Configuration](../snapshot/configuration.md)
- [Cloud Storage Setup](../cloud-storage-engines/introduction.md)
- [Target Configuration](../configuration.md)