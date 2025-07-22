# Load Command

The `load` command restores database snapshots and executes SQL files in target databases. This versatile command supports both snapshot restoration and direct SQL file execution with comprehensive safety features and execution modes.

## Overview

The load command handles two primary operations:
1. **Snapshot Loading**: Restores database snapshots created with the `build` command
2. **SQL File Loading**: Executes SQL files directly against target databases

Both operations support advanced configuration options, safety features, and schema filtering for PostgreSQL databases.

## Syntax

```bash
dbsnapper load <target_name> [snapshot_index|file.sql] [flags]
```

## Arguments

- `<target_name>`: The name of the target defined in your configuration file
- `[snapshot_index|file.sql]`: Optional snapshot index (default: 0 for most recent) or SQL file path

## Options

```bash
      --destdb string   Destination Database URL Override - Will overwrite any existing destination database
      --dry-run         Show execution plan without running (SQL files only)
      --no-drop         Skip dropping/recreating database (SQL files only)
      --original        Use the original snapshot instead of the sanitized version (snapshots only)
      --source          Load SQL file to source database instead of destination
  -h, --help           Show help for the load command
```

## Snapshot Loading

Load database snapshots created with the `build` command:

### Basic Snapshot Operations

```bash
dbsnapper load target_name              # Load most recent snapshot to dst_url
dbsnapper load target_name 2            # Load snapshot by index to dst_url
dbsnapper load target_name --original   # Load original (non-sanitized) snapshot
dbsnapper load target_name --destdb <url>  # Load to custom destination database
```

### How Snapshot Loading Works

1. **Snapshot Selection**: Identifies the snapshot to load based on index (0 = most recent)
2. **Database Preparation**: Drops and recreates the destination database
3. **Schema Analysis**: For PostgreSQL, analyzes snapshot contents and applies schema filtering
4. **Data Restoration**: Restores schema, data, indexes, and constraints from the snapshot
5. **Verification**: Confirms successful restoration

### Snapshot Priority System

DBSnapper automatically prioritizes sanitized snapshots for safety:

1. **Sanitized Snapshot**: Loads `{timestamp}_{target_name}.san.zip` if available
2. **Original Snapshot**: Loads `{timestamp}_{target_name}.zip` if no sanitized version exists  
3. **Override**: Use `--original` flag to force loading the original snapshot

## SQL File Loading

Load SQL files directly to target databases with advanced execution modes and safety features:

### Basic SQL File Operations

```bash
dbsnapper load target_name file.sql                    # Load SQL file to dst_url (default)
dbsnapper load target_name file.sql --source           # Load SQL file to src_url
dbsnapper load target_name file.sql --destdb <url>     # Load to custom destination
dbsnapper load target_name file.sql --no-drop         # Execute against existing database
dbsnapper load target_name file.sql --dry-run         # Preview execution without running
```

### SQL File Detection

SQL files are automatically detected by the `.sql` extension (case-insensitive):
- `schema.sql`, `data.SQL`, `migration.Sql` are all valid
- Non-SQL files will be treated as snapshot indices

### Execution Modes

**Replace Mode (default)**
- Drops and recreates the target database before loading
- Ensures clean database state
- Use for complete database refreshes

**Append Mode (`--no-drop`)**
- Executes SQL against the existing database without dropping it
- Preserves existing data and structure
- Use for incremental updates, migrations, or data additions

### Database URL Resolution Priority

Database URL resolution follows this priority order:

1. `--destdb` flag (highest priority)
2. `--source` flag (determines src_url vs dst_url)
3. Configuration overrides
4. Target configuration defaults

## Schema Filtering (PostgreSQL Only)

When loading snapshots, the load command respects schema configuration for PostgreSQL databases:

### Schema Configuration Behavior

| Configuration | Load Behavior | Notes |
|---------------|---------------|-------|
| No `schema_config` | Loads `public` schema only | Default behavior |
| `use_default_schema: true` | Loads default schema from snapshot | Explicit default |
| `use_default_schema: false` | Loads **all schemas** present in snapshot | Multi-schema applications |
| `include_schemas: ["public", "app"]` | Loads only listed schemas from snapshot | Selective inclusion |
| `exclude_schemas: ["temp", "logs"]` | Loads all schemas except excluded ones | Exclude temporary data |

### Schema Configuration Example

```yaml
targets:
  postgres-target:
    snapshot:
      schema_config:
        # Option 1: Load all schemas from snapshot
        use_default_schema: false
        
        # Option 2: Load only specific schemas
        # include_schemas: ["public", "app_data"]
        
        # Option 3: Load all except excluded schemas  
        # exclude_schemas: ["temp_schema", "debug_logs"]
```

### Important Schema Notes

- **PostgreSQL Only**: Schema filtering only applies to PostgreSQL snapshot loading
- **Snapshot Contents**: Only schemas that exist in the snapshot will be loaded
- **No Empty Schemas**: No empty schemas are created if they don't exist in the snapshot
- **Warnings Displayed**: If requested schemas don't exist in the snapshot, warnings are shown
- **MySQL Limitation**: MySQL databases use standard restore without schema filtering
- **SQL Files Excluded**: Schema filtering does not apply to SQL file loading

## Safety Features

The load command includes comprehensive safety features:

### Interactive Confirmation
- Prompts before destructive operations
- Shows operation details and target information
- Can be disabled for automation

### Dry Run Mode
- Preview SQL execution without making changes (`--dry-run`)
- Shows execution plan and affected objects
- Available for SQL file loading only

### File Validation
- Validates SQL files before execution
- Checks file existence and readability
- Prevents execution of invalid files

### Transaction Support
- Wraps SQL execution in transactions for rollback capability
- Ensures atomicity of operations
- Provides rollback on errors

### Confirmation Control

Skip confirmations in automated environments:

```bash
# Environment variable (recommended for CI/CD)
export CI=true
dbsnapper load target_name file.sql

# Alternative environment variable
export DBSNAPPER_NO_CONFIRM=true
dbsnapper load target_name file.sql
```

## Example Usage

### Snapshot Loading Examples

```bash
# Basic snapshot operations
dbsnapper load production-db              # Load latest snapshot
dbsnapper load production-db 5            # Load snapshot at index 5
dbsnapper load production-db --original   # Load original (non-sanitized) snapshot

# Custom destination
dbsnapper load prod-db --destdb "postgresql://user:pass@test-host:5432/test_db"
```

### SQL File Loading Examples

```bash
# Basic SQL file operations
dbsnapper load dev-db schema.sql          # Load schema to dst_url
dbsnapper load dev-db data.sql --source   # Load data to src_url
dbsnapper load dev-db migration.sql --dry-run  # Preview migration

# Advanced SQL file operations
dbsnapper load prod-db update.sql --no-drop    # Execute without dropping database
dbsnapper load staging-db backup.sql --destdb "postgres://user:pass@host/custom_db"
```

### Automation Examples

```bash
# CI/CD pipeline usage
CI=true dbsnapper load production-db
DBSNAPPER_NO_CONFIRM=true dbsnapper load staging-db migration.sql --no-drop

# Batch processing
for file in migrations/*.sql; do
  dbsnapper load dev-db "$file" --no-drop
done
```

## Configuration Requirements

### Basic Target Configuration

```yaml
targets:
  myapp-dev:
    name: "Development Database"
    snapshot:
      src_url: "postgresql://user:pass@prod:5432/myapp"
      dst_url: "postgresql://user:pass@dev:5432/myapp_dev"
```

### Advanced Target Configuration with Schema Filtering

```yaml
targets:
  postgres-target:
    name: "PostgreSQL with Schema Control"
    snapshot:
      src_url: "postgresql://user:pass@prod:5432/myapp"
      dst_url: "postgresql://user:pass@dev:5432/myapp_dev"
      schema_config:
        include_schemas: ["public", "app_data", "reports"]
        # Exclude temporary and debug schemas from loading
```

## Use Cases

### Development Environment Setup

```bash
# Refresh development database with latest production snapshot
dbsnapper build myapp-prod
dbsnapper load myapp-dev
```

### Testing with Clean Data

```bash
# Load sanitized snapshot for testing
dbsnapper load test-env 0
```

### Database Migrations

```bash
# Preview migration before applying
dbsnapper load staging-db migration_v2.sql --dry-run

# Apply migration without dropping database
dbsnapper load staging-db migration_v2.sql --no-drop
```

### Feature Branch Testing

```bash
# Use specific snapshot state for consistent testing
dbsnapper load feature-branch 3
```

### Schema Management

```bash
# Load only application schemas, excluding logs and temp data
# (configured via schema_config in target definition)
dbsnapper load filtered-target
```

## Troubleshooting

### Common Issues

**Snapshot Not Found**
```
Error: snapshot index 5 not found for target myapp-dev
```
- List available snapshots with `dbsnapper target myapp-dev`
- Use valid snapshot index (0 = most recent)

**Database Connection Failed**
```
Error: failed to connect to destination database
```
- Verify `dst_url` or `--destdb` connection string
- Check database server availability
- Confirm credentials and permissions

**Permission Denied**
```
Error: insufficient privileges to drop/create database
```
- Ensure database user has CREATE/DROP privileges
- Check database server permissions
- Verify connection string includes correct credentials

**SQL File Not Found**
```
Error: SQL file not found: missing.sql
```
- Verify file path is correct
- Check file exists and is readable
- Use absolute or relative paths as appropriate

**Schema Configuration Errors**
```
Warning: schema 'non_existent' specified but not found in snapshot
```
- Verify schema names in configuration
- Check that schemas exist in the original snapshot
- Review schema configuration syntax

### Recovery Steps

1. **Verify Configuration**: Check target configuration and connection strings
2. **Test Connectivity**: Ensure database servers are accessible
3. **Check Permissions**: Verify database user privileges
4. **Validate Files**: Confirm SQL files exist and are readable
5. **Use Dry Run**: Preview operations with `--dry-run` flag

!!! danger "Destructive Operation"
    The load command will **drop and recreate** the destination database by default. All existing data in the destination database will be permanently lost unless using `--no-drop` flag with SQL files.

!!! note "Sanitized Snapshots Priority"
    If a sanitized snapshot exists at the specified index, it will be loaded unless the `--original` flag is set. This ensures sensitive data protection by default.

!!! tip "Database URL Priority"
    Database URL resolution follows this priority:
    1. `--destdb` flag (highest priority)
    2. `--source` flag (determines src_url vs dst_url)  
    3. Configuration overrides
    4. Target configuration defaults

## Related Commands

- [`build`](build.md) - Create new database snapshots
- [`sanitize`](../sanitize/sanitize.md) - Create sanitized versions of snapshots
- [`target`](../cmd/dbsnapper_target.md) - View target details and available snapshots
- [`targets`](../cmd/dbsnapper_targets.md) - List all available targets

## See Also

- [Configuration](../configuration.md) - Target and schema configuration
- [Database Engines](../database-engines/introduction.md) - Supported databases
- [Snapshot Management](../snapshot/introduction.md) - Snapshot concepts and workflows
- [Sanitization](../sanitize/introduction.md) - Data sanitization features