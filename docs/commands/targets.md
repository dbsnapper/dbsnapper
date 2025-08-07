# Targets Command

The `targets` command lists all available database targets from local configuration, cloud sources, and shared targets. This is the primary discovery command for understanding what databases are available for snapshot operations, providing a comprehensive overview of your DBSnapper environment.

## Overview

The targets command aggregates targets from multiple sources and displays them in a unified table format. It shows essential information about each target including connection details, storage profiles, and target types, helping users quickly identify available databases for snapshot operations.

## Syntax

```bash
dbsnapper targets [flags]
```

## Arguments

No arguments required - lists all available targets by default.

## Options

```bash
      --destdb string   Destination Database URL Override - Will overwrite any existing destination database
  -h, --help            help for targets  
  -o, --original        Use the original snapshot instead of the sanitized version
  -u, --update          Update the db info for all targets (default: true)
```

## Target Sources and Types

DBSnapper aggregates targets from three different sources, clearly identifying each type in the output:

### Local Targets (`local`)
- **Source**: Local configuration file (`~/.config/dbsnapper/dbsnapper.yml`)
- **Configuration**: Manually defined in the `targets` section
- **Availability**: Always available regardless of network connectivity
- **Control**: Full control over configuration and settings

### Cloud Targets (`cloud`)
- **Source**: DBSnapper Cloud API
- **Configuration**: Defined via DBSnapper Cloud web interface
- **Availability**: Requires internet connectivity and authentication
- **Control**: Managed through cloud interface, shared with team members

### Shared Targets (`share`)
- **Source**: Okta SSO integration via DBSnapper Cloud
- **Configuration**: Shared by other organization members
- **Availability**: Requires cloud authentication and SSO setup
- **Control**: Read-only access to shared configurations
- **Limitation**: Only sanitized snapshots are accessible

## Target Information Display

The targets command displays information in a tabular format with these columns:

### NAME
The unique identifier for each target, used in other DBSnapper commands.

### TYPE
- `local` - Locally configured target
- `cloud` - Cloud-managed target  
- `share` - Shared target via SSO

### SRC
The source database connection string where snapshots are built from.
- **Format**: `protocol://host:port/database`
- **Protocols**: `pgl://` (PostgreSQL), `myl://` (MySQL), `pgd://` (PostgreSQL Docker), `myd://` (MySQL Docker)

### SIZE
Current database size information (when available).
- **Updates**: Controlled by `--update` flag
- **Performance**: Disable updates for faster listing

### DST
The destination database connection string where snapshots are loaded to.
- **Optional**: May be empty for shared targets
- **Override**: Can be overridden with `--destdb` flag

### STORAGE PROFILE
Associated cloud storage configuration for snapshot storage.
- **Empty**: Local-only snapshots
- **Named**: Cloud storage integration enabled

## Target Update Behavior

The `--update` flag controls whether database information is refreshed:

### With Update (Default: `--update=true`)
```bash
dbsnapper targets
dbsnapper targets --update
```
- Queries each database to get current size and schema information
- **Slower execution** due to database connections
- **Current information** reflecting real-time database state
- **Use for**: Planning operations, capacity planning, health checks

### Without Update (`--update=false`)
```bash
dbsnapper targets --update=false
```
- Uses cached information from previous queries
- **Faster execution** with no database connections required
- **Cached information** may be outdated
- **Use for**: Quick target discovery, automated scripts, offline environments

## Cloud Integration

### Authentication Required
Cloud and shared targets require DBSnapper Cloud authentication:

```bash
# Set up cloud authentication first
dbsnapper auth token <your_api_token>

# Then list all targets including cloud and shared
dbsnapper targets
```

### Cloud Mode Control
```bash
# Disable cloud integration for faster local-only listing
dbsnapper targets --nocloud
```

## Example Output

### Multi-Source Target Listing
```bash
dbsnapper targets
```

**Output:**
```
DBSnapper v2.7.0-dev - List Targets
[CLOUD] DBSnapper Cloud: Enabled
[CONFIG] Config: ~/.config/dbsnapper/dbsnapper.yml
──────────────────────────────────────────────────
Listing all targets, update = true

+----------------------+-------+----------------------------------+--------+----------------------------------+------------------+
|        NAME          | TYPE  |               SRC                |  SIZE  |               DST                | STORAGE PROFILE  |
+----------------------+-------+----------------------------------+--------+----------------------------------+------------------+
| production-db        | cloud | pgl://prod:5432/app              | 2.3 GB | pgl://staging:5432/app_snap      | aws-s3-prod      |
| analytics-db         | cloud | pgl://analytics:5432/data        | 847 MB | pgl://dev:5432/analytics_snap    | aws-s3-analytics |
|                      | ----- |                                  |        |                                  |                  |
| dev-postgres         | local | pgl://localhost:5432/myapp       | 45 MB  | pgl://localhost:5432/myapp_test  |                  |
| dev-mysql            | local | myl://localhost:3306/webapp      | 123 MB | myl://localhost:3306/webapp_test |                  |
| docker-test          | local | pgd://postgres17:5432/testdb     | 12 MB  | pgd://postgres17:5432/testdb_snap|                  |
|                      | ----- |                                  |        |                                  |                  |
| team-shared-prod     | share |                                  |        | pgl://local:5432/shared_prod     | r2-shared-prod   |
| marketing-data       | share |                                  |        | pgl://local:5432/marketing       | s3-marketing     |
+----------------------+-------+----------------------------------+--------+----------------------------------+------------------+
```

### Local-Only Target Listing
```bash
dbsnapper targets --nocloud
```

**Output:**
```
DBSnapper v2.7.0-dev - List Targets
[CLOUD] DBSnapper Cloud: Disabled
[CONFIG] Config: ~/.config/dbsnapper/dbsnapper.yml
──────────────────────────────────────────────────

+----------------------+-------+----------------------------------+--------+----------------------------------+------------------+
|        NAME          | TYPE  |               SRC                |  SIZE  |               DST                | STORAGE PROFILE  |
+----------------------+-------+----------------------------------+--------+----------------------------------+------------------+
| dev-postgres         | local | pgl://localhost:5432/myapp       | 45 MB  | pgl://localhost:5432/myapp_test  |                  |
| dev-mysql            | local | myl://localhost:3306/webapp      | 123 MB | myl://localhost:3306/webapp_test |                  |
| docker-test          | local | pgd://postgres17:5432/testdb     | 12 MB  | pgd://postgres17:5432/testdb_snap|                  |
+----------------------+-------+----------------------------------+--------+----------------------------------+------------------+
```

## Example Usage

### Basic Target Discovery
```bash
# List all available targets with current information
dbsnapper targets

# Quick target listing without database updates
dbsnapper targets --update=false

# Local targets only (no cloud connectivity required)
dbsnapper targets --nocloud
```

### Target Validation and Planning
```bash
# Get current database sizes for capacity planning
dbsnapper targets --update

# Check available targets before building snapshots
dbsnapper targets
dbsnapper build production-db
```

### Automation and Scripting
```bash
# Fast target enumeration for automated scripts
dbsnapper targets --update=false --nocloud

# Pipeline validation - ensure required targets exist
if dbsnapper targets --nocloud | grep -q "production-db"; then
    echo "Production target available"
    dbsnapper build production-db
fi
```

## Use Cases

### Development Environment Setup
```bash
# Discover available development targets
dbsnapper targets --nocloud
# Choose appropriate target for development database refresh
dbsnapper load dev-postgres
```

### Production Database Management
```bash
# View production targets with current sizes
dbsnapper targets | grep cloud
# Plan snapshot operations based on database sizes
```

### Team Collaboration
```bash
# Discover shared targets from team members
dbsnapper targets | grep share
# Access shared sanitized snapshots
dbsnapper load team-shared-prod
```

### CI/CD Pipeline Integration
```bash
# Fast target validation in automated pipelines
targets_count=$(dbsnapper targets --update=false --nocloud | grep -c local)
if [ "$targets_count" -gt 0 ]; then
    echo "Targets available, proceeding with snapshot operations"
fi
```

### Capacity Planning and Monitoring
```bash
# Get current database sizes for all targets
dbsnapper targets --update | tee target_sizes.log
# Monitor database growth over time
```

## Database Connection Protocols

The targets command displays various database connection protocols:

### PostgreSQL
- **`pgl://`** - Local PostgreSQL instance
- **`pgd://`** - PostgreSQL Docker container

### MySQL  
- **`myl://`** - Local MySQL instance
- **`myd://`** - MySQL Docker container

### Connection String Format
```
protocol://[user[:password]@]host[:port]/database[?options]
```

**Examples:**
```
pgl://localhost:5432/myapp
myl://user:pass@prod-host:3306/webapp?tls=true
pgd://postgres17:5432/testdb
```

## Cloud Storage Integration

### Storage Profile Display
The `STORAGE PROFILE` column indicates cloud storage configuration:

- **Empty**: Snapshots stored locally only
- **Profile Name**: Cloud storage enabled with specified profile

### Common Storage Profile Types
- **AWS S3**: `aws-s3-prod`, `s3-backups`
- **Cloudflare R2**: `r2-snapshots`, `r2-shared-prod`  
- **Custom**: Organization-specific naming conventions

## Performance Considerations

### Update Behavior Impact
- **With Updates**: Slower but current information
- **Without Updates**: Faster but potentially stale information

### Cloud Integration Impact
- **Cloud Enabled**: Additional network requests for cloud/shared targets
- **Cloud Disabled**: Local targets only, faster execution

### Optimization Strategies
```bash
# For frequent target checking in scripts
dbsnapper targets --update=false --nocloud

# For planning and capacity management
dbsnapper targets --update

# For team collaboration discovery
dbsnapper targets | grep -E "(cloud|share)"
```

## Troubleshooting

### Common Issues

**No Targets Found**
```
No targets found
```
- Check configuration file exists: `~/.config/dbsnapper/dbsnapper.yml`
- Verify targets are defined in configuration
- Run `dbsnapper config init` if starting fresh

**Cloud Targets Not Appearing**
```
Only local targets shown, no cloud targets
```
- Verify cloud authentication: `dbsnapper auth token <token>`
- Check internet connectivity
- Ensure cloud mode not disabled: remove `--nocloud` flag

**Database Connection Timeouts**
```
Error: timeout connecting to database
```
- Use `--update=false` to skip database connections
- Verify database servers are running and accessible
- Check network connectivity and firewall settings

**Shared Targets Not Visible**
```
No shared targets in listing
```
- Verify Okta SSO integration is configured
- Check organization membership and permissions
- Ensure shared targets have been created by team members

**Permission Denied Errors**
```
Error: permission denied accessing target
```
- Check database credentials in target configuration
- Verify database user has required permissions
- Test connectivity with database client tools

### Recovery Steps

1. **Verify Configuration**: Check configuration file exists and is valid
2. **Test Connectivity**: Verify database and cloud connectivity
3. **Check Authentication**: Ensure cloud authentication is set up correctly
4. **Use Minimal Flags**: Try `--update=false --nocloud` for basic operation
5. **Validate Targets**: Use `dbsnapper config validate` to check target definitions

## Integration with Other Commands

### Target Selection for Operations
```bash
# List targets to find the one you need
dbsnapper targets

# Use target name in other commands
dbsnapper build production-db
dbsnapper load dev-postgres
dbsnapper sanitize team-data
```

### Target Details Investigation
```bash
# List all targets to find interesting ones
dbsnapper targets

# Get detailed information about specific target
dbsnapper target production-db
```

### Snapshot Management Workflow
```bash
# 1. Discover available targets
dbsnapper targets

# 2. Build snapshot from desired target
dbsnapper build production-db

# 3. Load snapshot to development target
dbsnapper load dev-postgres
```

!!! tip "Quick Discovery"
    Use `dbsnapper targets --update=false` for fast target discovery in scripts and automation where current database sizes aren't needed.

!!! note "Target Types"
    The three target types (local, cloud, share) represent different sources and access levels:
    - **Local**: Full control, locally configured
    - **Cloud**: Team-managed via web interface  
    - **Share**: Read-only access to team snapshots

!!! warning "Database Updates"
    The `--update` flag (default: true) connects to each database to get current information. Disable with `--update=false` for faster execution when current sizes aren't needed.

## Related Commands

- [`target`](target.md) - View detailed information about a specific target
- [`build`](build.md) - Create snapshots from targets
- [`load`](load.md) - Load snapshots into target databases
- [`config init`](config-init.md) - Initialize configuration with targets

## See Also

- [Target Configuration](../configuration.md) - How to configure targets
- [DBSnapper Cloud](../dbsnapper-cloud/introduction.md) - Cloud integration and shared targets
- [Storage Profiles](../dbsnapper-cloud/storage_profiles.md) - Cloud storage configuration
- [Database Engines](../database-engines/introduction.md) - Supported database types