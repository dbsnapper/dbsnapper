# Config Check Command

The `config check` command validates your DBSnapper configuration and system dependencies, ensuring all required components are properly installed and configured. This comprehensive health check tool serves as both a diagnostic utility and an automated setup assistant.

## Overview

The config check command performs a thorough validation of your DBSnapper environment:
- **Configuration file validation**: Verifies syntax and structure
- **Database tool detection**: Confirms required client tools are available
- **Docker integration**: Validates Docker setup and required images
- **Cloud connectivity**: Tests DBSnapper Cloud authentication and connection
- **Auto-discovery**: Optionally finds and configures database tools automatically

## Syntax

```bash
dbsnapper config check [flags]
```

## Arguments

No arguments required - performs comprehensive system check by default.

## Options

```bash
      --auto-discover   Automatically discover database client tools
  -h, --help            help for check
      --save            Save discovered tool paths to configuration file (use with --auto-discover)
```

## Validation Components

### Configuration File Validation
- **File existence**: Confirms configuration file is present
- **YAML syntax**: Validates proper YAML structure
- **Required sections**: Checks for essential configuration sections
- **Loading verification**: Ensures configuration loads without errors

### Database Engine Validation

#### PostgreSQL Local Engine (pglocal)
- **`psql` client**: Command-line PostgreSQL client for testing connections
- **`pg_dump` tool**: Required for creating database snapshots
- **`pg_restore` tool**: Required for loading database snapshots
- **Version compatibility**: Ensures tool versions are supported

#### MySQL Local Engine (mylocal)
- **`mysql` client**: Command-line MySQL client for testing connections
- **`mysqldump` tool**: Required for creating database snapshots
- **Version compatibility**: Ensures tool versions are supported

#### PostgreSQL Docker Engine (pgdocker)
- **Docker connectivity**: Verifies Docker daemon is running and accessible
- **Image configuration**: Checks `docker.images.postgres` setting
- **Image availability**: Confirms PostgreSQL Docker image is present
- **Container capability**: Validates Docker container creation permissions

#### MySQL Docker Engine (mydocker)
- **Docker connectivity**: Verifies Docker daemon is running and accessible
- **Image configuration**: Checks `docker.images.mysql` setting
- **Image availability**: Confirms MySQL Docker image is present
- **Container capability**: Validates Docker container creation permissions

### Cloud Integration Validation
- **DBSnapper Cloud connectivity**: Tests connection to cloud services
- **Authentication**: Validates API token if configured
- **Service availability**: Confirms cloud services are accessible

## Auto-Discovery Feature

### Tool Discovery Process
The `--auto-discover` flag enables automatic detection of database tools:

1. **Search locations**: Scans common installation paths
2. **Tool validation**: Tests each found tool for compatibility
3. **Version checking**: Ensures tool versions meet requirements
4. **Path resolution**: Determines optimal tool paths

### Discovery Search Paths

#### PostgreSQL Tools
- **Postgres.app**: `/Applications/Postgres.app/Contents/Versions/*/bin/`
- **Homebrew**: `/opt/homebrew/bin/`, `/usr/local/bin/`
- **System paths**: `/usr/bin/`, `/usr/local/bin/`
- **Custom paths**: Checks `PATH` environment variable

#### MySQL Tools
- **Homebrew**: `/opt/homebrew/opt/mysql-client/bin/`, `/opt/homebrew/bin/`
- **System paths**: `/usr/bin/`, `/usr/local/bin/`, `/usr/local/mysql/bin/`
- **Custom paths**: Checks `PATH` environment variable

### Saving Discovered Tools
The `--save` flag (used with `--auto-discover`) updates your configuration:

```bash
# Discover tools and save to configuration
dbsnapper config check --auto-discover --save
```

This updates the `database_tools` section in your configuration file:

```yaml
database_tools:
  auto_detect: true
  postgresql:
    pg_dump: "/Applications/Postgres.app/Contents/Versions/17/bin/pg_dump"
    pg_restore: "/Applications/Postgres.app/Contents/Versions/17/bin/pg_restore"
    psql: "/Applications/Postgres.app/Contents/Versions/17/bin/psql"
  mysql:
    mysql: "/opt/homebrew/opt/mysql-client/bin/mysql"
    mysqldump: "/opt/homebrew/opt/mysql-client/bin/mysqldump"
```

## Check Output and Status Indicators

### Status Icons
- **✅ Success**: Component is properly configured and available
- **❌ Error**: Component has issues that need attention
- **⚠️ Warning**: Component works but has non-critical issues
- **🔵 Info**: Informational status about component checking
- **🔍 Discovery**: Auto-discovery process indicators

### Output Sections

#### Configuration File Status
```
✅ Config file ( /path/to/dbsnapper.yml ) found and loaded
```

#### Database Engine Status
```
🔵 Postgres Local Engine (pglocal)
  ✅ psql found at /Applications/Postgres.app/Contents/Versions/17/bin/psql
  ✅ pg_dump found at /Applications/Postgres.app/Contents/Versions/17/bin/pg_dump
  ✅ pg_restore found at /Applications/Postgres.app/Contents/Versions/17/bin/pg_restore
```

#### Auto-Discovery Results
```
🔍 Auto-discovering database client tools...
  ✅ Database tools discovery completed
  📋 Discovered tools:
    pg_dump: ✅ /Applications/Postgres.app/Contents/Versions/17/bin/pg_dump
    mysql: ✅ /opt/homebrew/opt/mysql-client/bin/mysql
  💾 Saving discovered tools to configuration file...
  ✅ Configuration saved to /path/to/dbsnapper.yml
```

#### Final Status
```
✅ All supported database engines configured
✅ DBSnapper Cloud connected
✅ Configuration OK
```

## Example Usage

### Basic Health Check
```bash
# Comprehensive system validation
dbsnapper config check
```

**Output:**
```
Checking DBSnapper Configuration
  ✅ Config file ( ~/.config/dbsnapper/dbsnapper.yml ) found and loaded
  🔵 Postgres Local Engine (pglocal)
    ✅ psql found at /usr/local/bin/psql
    ✅ pg_dump found at /usr/local/bin/pg_dump
    ✅ pg_restore found at /usr/local/bin/pg_restore
  🔵 MySQL Local Engine (mylocal)
    ✅ mysqldump found at /usr/local/bin/mysqldump
    ✅ mysql found at /usr/local/bin/mysql
  🔵 Postgres Docker Engine (pgdocker)
    ✅ Docker client connected
    ✅ docker.images.postgres set in config file
      ✅ Found Docker image: postgres:17
  🔵 Mysql Docker Engine (mydocker)
    ✅ Docker client connected
    ✅ docker.images.mysql set in config file
      ✅ Found Docker image: mysql:9
  ✅ All supported database engines configured
  ✅ DBSnapper Cloud connected

  ✅ Configuration OK
```

### Auto-Discovery and Configuration Update
```bash
# Discover tools and save to configuration
dbsnapper config check --auto-discover --save
```

### Discovery Without Saving
```bash
# Discover tools but don't update configuration
dbsnapper config check --auto-discover
```

### Custom Configuration File
```bash
# Check specific configuration file
dbsnapper config check --config /path/to/custom/config.yml
```

## Use Cases

### Initial Setup Verification
```bash
# After running `dbsnapper config init`
dbsnapper config init
dbsnapper config check

# Verify everything is working before first use
```

### Regular Health Monitoring
```bash
# Monthly system health check
dbsnapper config check

# Include in system monitoring scripts
if dbsnapper config check >/dev/null 2>&1; then
    echo "DBSnapper configuration healthy"
else
    echo "DBSnapper configuration issues detected"
    dbsnapper config check  # Show details
fi
```

### After Software Updates
```bash
# After updating PostgreSQL or MySQL
dbsnapper config check --auto-discover --save

# Verify tools still work after OS updates
dbsnapper config check
```

### Troubleshooting Workflow
```bash
# Diagnose configuration issues
dbsnapper config check

# Try to fix automatically
dbsnapper config check --auto-discover --save

# Verify fixes worked
dbsnapper config check
```

### CI/CD Pipeline Validation
```bash
# Validate build environment
dbsnapper config check

# Fail CI if configuration is invalid
dbsnapper config check || exit 1

# Auto-configure in containerized builds
dbsnapper config check --auto-discover --save
```

### Team Environment Standardization
```bash
# Verify team member setup
dbsnapper config check

# Standardize tool paths across team
dbsnapper config check --auto-discover --save
```

## Error Detection and Resolution

### Common Issues and Solutions

#### Configuration File Issues
```bash
# Error: Config file not found
❌ Config file not found

# Solution: Initialize configuration
dbsnapper config init
```

```bash
# Error: Invalid YAML syntax
❌ Config file syntax error

# Solution: Validate and fix syntax
dbsnapper config validate
```

#### Database Tool Issues
```bash
# Error: PostgreSQL tools not found
❌ psql not found
❌ pg_dump not found

# Solution: Install PostgreSQL client tools and auto-discover
brew install postgresql  # or appropriate package manager
dbsnapper config check --auto-discover --save
```

```bash
# Error: MySQL tools not found
❌ mysql not found
❌ mysqldump not found

# Solution: Install MySQL client tools and auto-discover
brew install mysql-client  # or appropriate package manager
dbsnapper config check --auto-discover --save
```

#### Docker Issues
```bash
# Error: Docker not available
❌ Docker client not connected

# Solution: Start Docker service
# macOS: Start Docker Desktop
# Linux: sudo systemctl start docker
```

```bash
# Error: Docker images missing
❌ Docker image postgres:17 not found

# Solution: Pull required images
docker pull postgres:17
docker pull mysql:9
```

#### Cloud Connectivity Issues
```bash
# Error: Cloud connection failed
❌ DBSnapper Cloud connection failed

# Solution: Check authentication and network
dbsnapper auth token <your_token>
# Check internet connectivity and firewall settings
```

### Automated Issue Resolution
The `--auto-discover --save` combination can automatically resolve many tool-related issues:

```bash
# Comprehensive auto-fix attempt
dbsnapper config check --auto-discover --save

# Manual verification after auto-fix
dbsnapper config check
```

## Integration with Other Commands

### Pre-Operation Validation
```bash
# Verify configuration before critical operations
dbsnapper config check && dbsnapper build production-db
dbsnapper config check && dbsnapper sanitize sensitive-data
```

### Automated Workflows
```bash
#!/bin/bash
# Automated backup script with health check
if ! dbsnapper config check >/dev/null 2>&1; then
    echo "Configuration issues detected, attempting auto-fix..."
    dbsnapper config check --auto-discover --save
    if ! dbsnapper config check >/dev/null 2>&1; then
        echo "Configuration issues persist, manual intervention required"
        exit 1
    fi
fi

# Proceed with backup operations
dbsnapper build production-db
```

### System Monitoring Integration
```bash
# Nagios/monitoring check script
#!/bin/bash
if dbsnapper config check >/dev/null 2>&1; then
    echo "OK - DBSnapper configuration healthy"
    exit 0
else
    echo "CRITICAL - DBSnapper configuration issues"
    exit 2
fi
```

## Performance and Optimization

### Check Performance
- **Fast validation**: Basic checks complete in seconds
- **Docker checks**: May be slower if Docker images need pulling
- **Cloud checks**: Depend on network connectivity
- **Auto-discovery**: May take longer on first run

### Optimization Strategies
```bash
# Skip cloud checks for faster validation
dbsnapper config check --nocloud

# Cache Docker images for faster checks
docker pull postgres:17 mysql:9

# Use in monitoring with appropriate timeouts
timeout 30 dbsnapper config check
```

## Best Practices

### Regular Health Checks
1. **Monthly validation**: Include in regular maintenance routine
2. **After updates**: Run after software or OS updates
3. **Before critical operations**: Validate before important snapshots
4. **Team onboarding**: Include in new team member setup

### Automation Integration
1. **CI/CD validation**: Include in build pipelines
2. **Monitoring**: Integrate with system monitoring
3. **Auto-healing**: Use auto-discover for self-healing systems
4. **Documentation**: Document configuration standards

### Error Handling
1. **Graceful failures**: Handle check failures appropriately
2. **Notification**: Alert on persistent configuration issues
3. **Recovery procedures**: Document resolution steps
4. **Escalation**: Define when manual intervention is needed

!!! tip "Regular Validation"
    Include `dbsnapper config check` in your regular maintenance routine to catch configuration issues before they impact operations.

!!! note "Auto-Discovery"
    Use `--auto-discover --save` after installing or updating database client tools to automatically update your configuration.

!!! warning "Docker Dependencies"
    Docker-based engines require Docker to be running and have the appropriate images pulled. The check will help identify missing Docker images.

## Related Commands

- [`config init`](config-init.md) - Initialize DBSnapper configuration  
- [`config discover`](config-discover.md) - Dedicated tool discovery command
- [`config validate`](config-validate.md) - Validate configuration file syntax
- [`config`](config.md) - Overview of all configuration commands

## See Also

- [Configuration Reference](../configuration.md) - Complete configuration options
- [Database Engines](../database-engines/introduction.md) - Supported databases and setup
- [Installation Guide](../installation.md) - DBSnapper installation instructions
