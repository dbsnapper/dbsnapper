# Target Command

The `target` command displays detailed information about a specific target, including all snapshots, database statistics, and table relationships. This command provides comprehensive insight into a target's snapshot history, database structure, and operational status.

## Overview

The target command serves as the primary inspection tool for individual targets, showing:
- Complete snapshot history with creation dates, sizes, and sanitization status
- Database statistics including size, table counts, and relationship mapping
- Snapshot locations (local vs cloud storage)
- Table structure and foreign key relationships
- Database health and connectivity status

## Syntax

```bash
dbsnapper target <target_name> [flags]
```

## Arguments

- `<target_name>`: The name of the target defined in your configuration file

## Options

```bash
      --destdb string   Destination Database URL Override - Will overwrite any existing destination database
  -h, --help            help for target
      --info            Output database and table stats
  -o, --original        Use the original snapshot instead of the sanitized version
  -u, --update          Update the db info for all targets (default: true)
```

## Snapshot Information Display

The target command displays snapshots in a comprehensive table format:

### Snapshot Table Columns

#### INDEX
Zero-based index number for referencing snapshots in other commands.
- **0** = Most recent snapshot
- Higher numbers = Older snapshots
- Used in `load`, `sanitize`, and other commands

#### CREATED
Human-readable creation timestamp showing when the snapshot was built.
- **Format**: `DD-Mon-YY HH:MM`
- **Example**: `25-Aug-07 03:50`
- **Timezone**: Displayed in local timezone

#### NAME
Unique snapshot identifier combining timestamp and target name.
- **Format**: `{timestamp}_{target_name}` or custom names
- **Examples**: 
  - `1754538604_dvdrental-cloud` (standard format)
  - `test_snapshot` (custom name)

#### FILENAME
Actual file name as stored in local working directory or cloud storage.
- **Local Format**: `{timestamp}_{target_name}.zip`
- **Cloud Format**: UUID-based filenames (e.g., `d7786d0b-9c5d-43c6-8744-e92893c4dc9a.zip`)
- **Sanitized Suffix**: `.san.zip` for sanitized snapshots

#### SIZE
Snapshot file size in human-readable format.
- **Units**: Bytes, kB, MB, GB as appropriate
- **Accuracy**: Actual compressed file size
- **Comparison**: Useful for capacity planning and performance estimation

#### SANITIZED?
Indicates whether a sanitized version of the snapshot exists.
- **`true`** = Sanitized snapshot available (preferred for loading)
- **`false`** = Only original snapshot available
- **Priority**: Sanitized snapshots are loaded by default unless `--original` is specified

#### LOCATION
Shows where the snapshot is stored.
- **`local`** = Stored in working directory on local machine
- **`cloud`** = Stored in cloud storage (S3, R2, etc.)
- **Hybrid**: Some snapshots may exist in both locations

## Database Statistics (`--info` flag)

The `--info` flag provides comprehensive database analysis and health information:

### Database Stats Table

#### Core Database Information
```
+---------------------+---------+
|     STATISTIC       |  VALUE  |
+---------------------+---------+
| Database Size       | 8613011 |
| Database Status     | OK      |
| Database Messages   |         |
```

- **Database Size**: Total database size in bytes
- **Database Status**: Connection and health status (`OK`, `ERROR`, etc.)
- **Database Messages**: Any warnings or status messages

#### Table Analysis
```
| Subset Tables       |       0 |
| Tables Connected    |       0 |
| Tables Disconnected |      14 |
| Tables Upstream     |       0 |
| Tables Downstream   |       0 |
| Total Tables        |      14 |
```

- **Subset Tables**: Tables included in subset operations
- **Tables Connected**: Tables with foreign key relationships
- **Tables Disconnected**: Standalone tables without relationships
- **Tables Upstream**: Tables referenced by others
- **Tables Downstream**: Tables that reference others
- **Total Tables**: Complete table count

### Table Details

The `--info` flag shows individual table information:

```
+------+---------------------------+------+
| TYPE |           TABLE           | ROWS |
+------+---------------------------+------+
| X    | public.users              |   -1 |
| X    | temp_schema.cache_entries |   -1 |
| X    | analytics.reports         |   -1 |
```

- **TYPE**: Table classification (`X` = Excluded/Disconnected, others TBD)
- **TABLE**: Full schema-qualified table name
- **ROWS**: Row count (-1 indicates not calculated)

### Relationship Mapping

Foreign key relationships are displayed in detail:

```
+------------------------+------+-------------------------+--------------+---------------------+------------+
|          NAME          | TYPE |         FKTABLE         |  FKCOLUMNS   |      REFTABLE       | REFCOLUMNS |
+------------------------+------+-------------------------+--------------+---------------------+------------+
| Relationships          |      |                         |              |                     |            |
|                        | X    | analytics.events        | user_id      | public.users        | id         |
|                        | X    | public.posts            | user_id      | public.users        | id         |
| Excluded Relationships |      |                         |              |                     |            |
```

- **Relationships**: Active foreign key constraints
- **Excluded Relationships**: Relationships ignored in subset operations
- **FKTABLE**: Table containing the foreign key
- **FKCOLUMNS**: Foreign key column names
- **REFTABLE**: Referenced table
- **REFCOLUMNS**: Referenced column names

## Location Types and Storage

### Local Snapshots
- **Storage**: Working directory (default: `~/.dbsnapper`)
- **File Format**: `{timestamp}_{target_name}.zip`
- **Advantages**: Fast access, no network dependency
- **Use Cases**: Development, local testing, offline scenarios

### Cloud Snapshots
- **Storage**: S3-compatible cloud storage (AWS S3, Cloudflare R2, etc.)
- **File Format**: UUID-based names for security
- **Advantages**: Team sharing, backup, cross-environment access
- **Requirements**: Storage profile configuration and authentication

## Sanitization Status and Priority

### Sanitization Indicators
- **`SANITIZED? = true`**: Sanitized snapshot exists
- **`SANITIZED? = false`**: Only original snapshot available

### Loading Priority
```bash
# Default behavior - loads sanitized version if available
dbsnapper load target_name 0

# Force original snapshot
dbsnapper load target_name 0 --original
```

## Example Usage

### Basic Target Inspection
```bash
# View all snapshots for a target
dbsnapper target production-db

# View snapshots with database statistics
dbsnapper target production-db --info
```

### Database Analysis and Planning
```bash
# Get comprehensive database analysis
dbsnapper target analytics-db --info
# Use output for subset planning, schema analysis, capacity planning
```

### Snapshot History Review
```bash
# Review snapshot history for target
dbsnapper target user-data
# Identify suitable snapshots for specific needs
```

## Example Output

### Basic Target View
```bash
dbsnapper target production-app
```

**Output:**
```
DBSnapper v2.7.0-dev - Target Snapshots
[CLOUD] DBSnapper Cloud: Enabled
[TARGET] Target: production-app
[CONFIG] Config: ~/.config/dbsnapper/dbsnapper.yml
──────────────────────────────────────────────────

+-------+-----------------+--------------------------+--------------------------------+--------+------------+----------+
| INDEX |     CREATED     |           NAME           |            FILENAME            |  SIZE  | SANITIZED? | LOCATION |
+-------+-----------------+--------------------------+--------------------------------+--------+------------+----------+
|     0 | 25-Aug-07 09:15 | 1754558129_production-app| 1754558129_production-app.zip  | 2.3 GB | true       | local    |
|     1 | 25-Aug-06 09:15 | 1754471729_production-app| b84d2c91-4e32-4a91-b123-...   | 2.2 GB | true       | cloud    |
|     2 | 25-Aug-05 09:15 | 1754385329_production-app| a72f1b82-3d21-4b82-a012-...   | 2.1 GB | false      | cloud    |
|     3 | 25-Aug-04 09:15 | 1754298929_production-app| 9e8c4a73-2c12-4c73-9901-...   | 2.0 GB | false      | cloud    |
+-------+-----------------+--------------------------+--------------------------------+--------+------------+----------+
```

### Detailed Database Analysis
```bash
dbsnapper target production-app --info
```

**Output:**
```
DBSnapper v2.7.0-dev - Target Snapshots
[CLOUD] DBSnapper Cloud: Enabled
[TARGET] Target: production-app
[CONFIG] Config: ~/.config/dbsnapper/dbsnapper.yml
──────────────────────────────────────────────────

Database Stats:
+---------------------+-----------+
|      STATISTIC      |   VALUE   |
+---------------------+-----------+
| Database Size       | 2461820152|
| Database Status     | OK        |
| Database Messages   |           |
| Subset Tables       |         3 |
| Tables Connected    |        12 |
| Tables Disconnected |         8 |
| Tables Upstream     |         5 |
| Tables Downstream   |         7 |
| Total Tables        |        20 |
+---------------------+-----------+

Table Stats:
+------+----------------------+--------+
| TYPE |        TABLE         |  ROWS  |
+------+----------------------+--------+
| ✓    | public.users         | 125000 |
| ✓    | public.orders        |  45000 |
| ✓    | public.products      |   1500 |
| X    | public.audit_logs    | 890000 |
| X    | temp.cache_entries   |  12000 |
+------+----------------------+--------+

Relationships:
+------------------------+------+-------------------+-----------+-----------------+------------+
|          NAME          | TYPE |      FKTABLE      | FKCOLUMNS |    REFTABLE     | REFCOLUMNS |
+------------------------+------+-------------------+-----------+-----------------+------------+
| Active Relationships   |      |                   |           |                 |            |
|                        | ✓    | public.orders     | user_id   | public.users    | id         |
|                        | ✓    | public.order_items| order_id  | public.orders   | id         |
|                        | ✓    | public.order_items| product_id| public.products | id         |
+------------------------+------+-------------------+-----------+-----------------+------------+

[Snapshot listing follows...]
```

## Use Cases

### Development Environment Planning
```bash
# Analyze production database before creating development snapshot
dbsnapper target production-db --info
# Review table sizes, relationships for subset planning
dbsnapper subset production-db  # Based on analysis
```

### Snapshot Selection for Operations
```bash
# Find appropriate snapshot for specific date/time
dbsnapper target user-data
# Select snapshot by index for loading
dbsnapper load staging-env 3  # Load snapshot at index 3
```

### Database Health Monitoring
```bash
# Regular database health checks
dbsnapper target production-db --info
# Monitor table growth, relationship changes over time
```

### Sanitization Planning
```bash
# Identify which snapshots have sanitized versions
dbsnapper target sensitive-data
# Plan sanitization for snapshots that need it
dbsnapper sanitize sensitive-data 2  # Sanitize specific snapshot
```

### Cloud Storage Management
```bash
# Review cloud vs local snapshot distribution
dbsnapper target distributed-app
# Plan pull operations for cloud snapshots
dbsnapper pull distributed-app 5  # Pull specific cloud snapshot
```

### Capacity Planning and Analysis
```bash
# Analyze database growth trends
dbsnapper target analytics-db --info
# Plan storage and backup strategies based on sizes
```

## Performance Considerations

### Update Behavior
- **With Update (default)**: Connects to database for current information
- **Without Update (`--update=false`)**: Uses cached data for faster response

### Information Depth
- **Basic View**: Fast snapshot listing
- **With `--info`**: Comprehensive analysis (slower due to database queries)

### Optimization Strategies
```bash
# Fast snapshot review for automation
dbsnapper target production-db --update=false

# Comprehensive analysis for planning
dbsnapper target production-db --info --update

# Focus on recent snapshots for quick decisions
dbsnapper target production-db | head -20
```

## Integration with Other Commands

### Snapshot Selection Workflow
```bash
# 1. Review available snapshots
dbsnapper target production-db

# 2. Choose appropriate snapshot by index
dbsnapper load development-db 2

# 3. Verify loaded snapshot
dbsnapper target development-db --info
```

### Sanitization Workflow
```bash
# 1. Check sanitization status
dbsnapper target sensitive-db

# 2. Sanitize unsanitized snapshots
dbsnapper sanitize sensitive-db 3

# 3. Verify sanitization completed
dbsnapper target sensitive-db
```

### Cloud Management Workflow
```bash
# 1. Review cloud snapshot availability
dbsnapper target shared-data

# 2. Pull needed snapshots locally
dbsnapper pull shared-data 1

# 3. Confirm local availability
dbsnapper target shared-data
```

## Troubleshooting

### Common Issues

**Target Not Found**
```
Error: target 'unknown-target' not found
```
- Verify target name with `dbsnapper targets`
- Check configuration file for target definition
- Ensure correct configuration file is being used

**Database Connection Failed**
```
Error: failed to connect to source database
```
- Check database server availability
- Verify connection string in target configuration
- Test connectivity with database client tools
- Use `--update=false` to skip database connections

**No Snapshots Found**
```
No snapshots found for target: test-target
```
- Build first snapshot: `dbsnapper build test-target`
- Check working directory for snapshot files
- Verify storage profile configuration for cloud targets

**Permission Denied**
```
Error: insufficient privileges to analyze database
```
- Check database user permissions
- Ensure user has SELECT privileges on system catalogs
- Verify authentication credentials

**Incomplete Statistics with `--info`**
```
Database Stats show -1 for row counts
```
- Run with `--update` to refresh statistics
- Check database user has privileges to query system tables
- Some statistics may require elevated database privileges

### Recovery Steps

1. **Verify Configuration**: Check target configuration and connection strings
2. **Test Connectivity**: Ensure database servers are accessible
3. **Check Permissions**: Verify database user privileges
4. **Use Minimal Options**: Try without `--info` or `--update` flags
5. **Review Logs**: Check for detailed error messages

## Performance and Optimization

### Command Execution Speed
- **Fastest**: `dbsnapper target name --update=false` (cached data only)
- **Medium**: `dbsnapper target name` (snapshot list + basic database check)
- **Slowest**: `dbsnapper target name --info` (full database analysis)

### Large Snapshot Lists
For targets with many snapshots, consider:
```bash
# Limit output to recent snapshots
dbsnapper target production-db | head -20

# Use grep to find specific snapshots
dbsnapper target production-db | grep "25-Aug-07"

# Focus on sanitized snapshots only
dbsnapper target production-db | grep "true"
```

!!! tip "Snapshot Selection"
    Use the INDEX column values directly in other commands. Index 0 is always the most recent snapshot.

!!! note "Database Analysis"
    The `--info` flag provides valuable insights for subset planning, schema analysis, and capacity planning. Use it when detailed database understanding is needed.

!!! warning "Update Behavior"
    The `--update` flag (default: true) connects to the source database. Use `--update=false` for faster execution when current database state isn't needed.

## Related Commands

- [`targets`](targets.md) - List all available targets
- [`build`](build.md) - Create new snapshots for targets
- [`load`](load.md) - Load target snapshots into databases
- [`sanitize`](sanitize.md) - Create sanitized versions of snapshots
- [`pull`](pull.md) - Retrieve cloud snapshots locally

## See Also

- [Snapshot Management](../snapshot/introduction.md) - Snapshot concepts and workflows
- [Target Configuration](../configuration.md) - How to configure targets
- [Database Engines](../database-engines/introduction.md) - Supported databases
- [Cloud Storage](../dbsnapper-cloud/storage_profiles.md) - Cloud storage integration