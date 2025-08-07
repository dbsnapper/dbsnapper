# Config Validate Command

The `config validate` command validates that database client tools specified in your configuration file are properly installed and executable. This focused validation tool ensures your configured tools are functional and ready for DBSnapper operations.

## Overview

The config validate command provides targeted tool validation:
- **Configuration-based validation**: Tests only tools specified in your configuration
- **Executable verification**: Confirms tools can be executed and respond correctly
- **Version checking**: Validates tool versions meet DBSnapper requirements
- **Path verification**: Ensures configured paths point to valid executables
- **Structured output**: Provides both human-readable and JSON output formats

## Syntax

```bash
dbsnapper config validate [flags]
```

## Arguments

No arguments required - validates all configured database tools.

## Options

```bash
  -h, --help   help for validate
      --json   Output results as JSON
```

## Validation Process

### Configuration Loading
1. **Configuration file**: Loads your DBSnapper configuration file
2. **Tool extraction**: Extracts database tool paths from configuration
3. **Path resolution**: Resolves relative paths and environment variables
4. **Tool identification**: Identifies PostgreSQL and MySQL tools to validate

### Tool Validation Steps
For each configured tool, the validation process:

1. **Path verification**: Confirms the tool exists at the specified path
2. **Permission check**: Verifies the tool has executable permissions
3. **Execution test**: Runs the tool with `--version` flag to test functionality
4. **Version analysis**: Analyzes version output for compatibility
5. **Status reporting**: Reports validation success or failure with details

### Validated Tools

#### PostgreSQL Tools
- **`psql`**: PostgreSQL command-line client
- **`pg_dump`**: PostgreSQL database backup tool
- **`pg_restore`**: PostgreSQL database restore tool

#### MySQL Tools
- **`mysql`**: MySQL command-line client
- **`mysqldump`**: MySQL database backup tool

## Output Formats

### Human-Readable Output (Default)
```bash
dbsnapper config validate
```

**Successful Validation Output:**
```
DBSnapper v2.7.0-dev - Database Tools Validation
──────────────────────────────────────────────────

Configuration file: ~/.config/dbsnapper/dbsnapper.yml

→ Database Tools Validation

  ✅ pg_dump: /Applications/Postgres.app/Contents/Versions/17/bin/pg_dump
  ✅ pg_restore: /Applications/Postgres.app/Contents/Versions/17/bin/pg_restore
  ✅ psql: /Applications/Postgres.app/Contents/Versions/17/bin/psql
  ✅ mysql: /opt/homebrew/opt/mysql-client/bin/mysql
  ✅ mysqldump: /opt/homebrew/opt/mysql-client/bin/mysqldump

  ✓ All database tools are properly configured and validated
```

**Validation with Issues:**
```
DBSnapper v2.7.0-dev - Database Tools Validation
──────────────────────────────────────────────────

Configuration file: ~/.config/dbsnapper/dbsnapper.yml

→ Database Tools Validation

  ✅ pg_dump: /usr/local/bin/pg_dump
  ❌ pg_restore: /usr/local/bin/pg_restore (not found)
  ✅ psql: /usr/local/bin/psql
  ❌ mysql: /usr/local/bin/mysql (permission denied)
  ✅ mysqldump: /usr/local/bin/mysqldump

  ⚠️ 2 database tools have configuration issues
```

### JSON Output
```bash
dbsnapper config validate --json
```

**Successful Validation JSON:**
```json
{
  "success": true,
  "current_tools": {
    "auto_detect": true,
    "postgresql": {
      "pg_dump": "/Applications/Postgres.app/Contents/Versions/17/bin/pg_dump",
      "pg_restore": "/Applications/Postgres.app/Contents/Versions/17/bin/pg_restore",
      "psql": "/Applications/Postgres.app/Contents/Versions/17/bin/psql"
    },
    "mysql": {
      "mysql": "/opt/homebrew/opt/mysql-client/bin/mysql",
      "mysqldump": "/opt/homebrew/opt/mysql-client/bin/mysqldump"
    }
  },
  "validation_results": {
    "mysql": "✅ /opt/homebrew/opt/mysql-client/bin/mysql",
    "mysqldump": "✅ /opt/homebrew/opt/mysql-client/bin/mysqldump",
    "pg_dump": "✅ /Applications/Postgres.app/Contents/Versions/17/bin/pg_dump",
    "pg_restore": "✅ /Applications/Postgres.app/Contents/Versions/17/bin/pg_restore",
    "psql": "✅ /Applications/Postgres.app/Contents/Versions/17/bin/psql"
  },
  "missing_tools_count": 0,
  "message": "All database tools are properly configured and validated"
}
```

**Validation with Issues JSON:**
```json
{
  "success": false,
  "current_tools": {
    "auto_detect": false,
    "postgresql": {
      "pg_dump": "/usr/local/bin/pg_dump",
      "pg_restore": "/usr/local/bin/pg_restore",
      "psql": "/usr/local/bin/psql"
    },
    "mysql": {
      "mysql": "/usr/local/bin/mysql",
      "mysqldump": "/usr/local/bin/mysqldump"
    }
  },
  "validation_results": {
    "pg_dump": "✅ /usr/local/bin/pg_dump",
    "pg_restore": "❌ /usr/local/bin/pg_restore (not found)",
    "psql": "✅ /usr/local/bin/psql",
    "mysql": "❌ /usr/local/bin/mysql (permission denied)",
    "mysqldump": "✅ /usr/local/bin/mysqldump"
  },
  "missing_tools_count": 2,
  "message": "2 database tools have configuration issues",
  "errors": [
    {
      "tool": "pg_restore",
      "path": "/usr/local/bin/pg_restore",
      "error": "file not found"
    },
    {
      "tool": "mysql", 
      "path": "/usr/local/bin/mysql",
      "error": "permission denied"
    }
  ]
}
```

## Validation Results Interpretation

### Status Indicators
- **✅ Success**: Tool is properly configured and functional
- **❌ Error**: Tool has issues that prevent operation
- **⚠️ Warning**: Tool works but has non-critical issues
- **ℹ️ Info**: Informational status about the tool

### Common Error Types
- **File not found**: Tool path doesn't exist or is incorrect
- **Permission denied**: Tool exists but lacks executable permissions
- **Version incompatible**: Tool version doesn't meet requirements
- **Execution failed**: Tool exists but fails to run properly

## Example Usage

### Basic Validation
```bash
# Validate all configured database tools
dbsnapper config validate
```

### JSON Output for Automation
```bash
# Get validation results in JSON format
dbsnapper config validate --json

# Use in scripts for automated validation
if dbsnapper config validate --json | jq -e '.success'; then
    echo "All tools validated successfully"
else
    echo "Tool validation failed"
fi
```

### Custom Configuration File
```bash
# Validate specific configuration file
dbsnapper config validate --config /path/to/custom.yml
```

### Validation in Scripts
```bash
#!/bin/bash
# Script to ensure tools are valid before operations

if ! dbsnapper config validate >/dev/null 2>&1; then
    echo "Database tools validation failed"
    dbsnapper config validate  # Show details
    exit 1
fi

echo "Tools validated, proceeding with operations..."
```

## Use Cases

### Pre-Operation Validation
```bash
# Validate configuration before critical operations
dbsnapper config validate && dbsnapper build production-db
dbsnapper config validate && dbsnapper sanitize sensitive-data
```

### Configuration File Changes
```bash
# After manual configuration edits
nano ~/.config/dbsnapper/dbsnapper.yml
dbsnapper config validate

# Verify changes didn't break tool configuration
```

### System Maintenance
```bash
# After system updates or tool installations
sudo apt update && sudo apt upgrade postgresql-client mysql-client
dbsnapper config validate

# Check if tool paths are still valid
```

### CI/CD Pipeline Validation
```bash
# In automated pipelines
dbsnapper config validate --json > validation-results.json

# Check validation status
if ! cat validation-results.json | jq -e '.success'; then
    echo "Tool validation failed in CI"
    exit 1
fi
```

### Team Environment Verification
```bash
# Verify team member setup
dbsnapper config validate

# Ensure consistent tool configuration across team
for member in alice bob charlie; do
    echo "Validating $member's configuration"
    ssh $member "dbsnapper config validate"
done
```

### Troubleshooting Workflow
```bash
# Diagnose tool issues
dbsnapper config validate

# Get detailed error information
dbsnapper config validate --json | jq '.errors[]'

# Fix issues and re-validate
dbsnapper config discover --save
dbsnapper config validate
```

## Error Resolution

### Common Issues and Solutions

#### Tool Not Found
```bash
❌ pg_dump: /usr/local/bin/pg_dump (not found)
```

**Solutions:**
1. **Install the tool**: Use package manager to install PostgreSQL client
2. **Update path**: Use `config discover` to find correct path
3. **Fix configuration**: Manually update configuration with correct path

```bash
# Auto-fix with discovery
dbsnapper config discover --save
dbsnapper config validate
```

#### Permission Denied
```bash
❌ mysql: /usr/local/bin/mysql (permission denied)
```

**Solutions:**
```bash
# Fix permissions
sudo chmod +x /usr/local/bin/mysql

# Or fix ownership
sudo chown $USER:$USER /usr/local/bin/mysql

# Verify fix
dbsnapper config validate
```

#### Version Incompatible
```bash
⚠️ pg_dump: /usr/local/bin/pg_dump (version too old)
```

**Solutions:**
1. **Update tool**: Upgrade to supported version
2. **Install newer version**: Install alongside existing version
3. **Use package manager**: Update via package manager

```bash
# Update via Homebrew (macOS)
brew upgrade postgresql

# Update via apt (Ubuntu/Debian)
sudo apt update && sudo apt upgrade postgresql-client

# Re-validate
dbsnapper config validate
```

#### Execution Failed
```bash
❌ mysqldump: /usr/local/bin/mysqldump (execution failed)
```

**Solutions:**
1. **Check dependencies**: Ensure required libraries are installed
2. **Verify installation**: Reinstall the tool if corrupted
3. **Test manually**: Run the tool manually to diagnose issues

```bash
# Test tool manually
/usr/local/bin/mysqldump --version

# Reinstall if needed
sudo apt reinstall mysql-client

# Re-validate
dbsnapper config validate
```

## Integration with Other Commands

### Validation Workflow
```bash
# Complete configuration workflow
dbsnapper config init           # Initialize configuration
dbsnapper config discover --save # Auto-discover tools
dbsnapper config validate       # Validate configuration
dbsnapper config check          # Comprehensive health check
```

### Pre-Operation Checks
```bash
# Validate before important operations
dbsnapper config validate && {
    dbsnapper build production-db
    dbsnapper sanitize production-db
    dbsnapper load staging-db
}
```

### Automated Maintenance
```bash
#!/bin/bash
# Automated maintenance script

echo "Validating database tools configuration..."
if ! dbsnapper config validate --json | jq -e '.success'; then
    echo "Attempting to fix tool configuration..."
    dbsnapper config discover --save
    
    if ! dbsnapper config validate --json | jq -e '.success'; then
        echo "Manual intervention required"
        exit 1
    fi
fi

echo "Configuration validated successfully"
```

## Performance and Considerations

### Validation Speed
- **Fast validation**: Typically completes in seconds
- **Executable tests**: Quick version checks for each tool
- **Minimal system impact**: Low CPU and memory usage
- **Network independence**: No network connectivity required

### System Requirements
- **File system access**: Requires read access to tool paths
- **Execute permissions**: Needs ability to run tools with --version
- **Path resolution**: Requires access to resolve configured paths

## Best Practices

### Regular Validation
1. **After configuration changes**: Always validate after manual edits
2. **System updates**: Validate after OS or database tool updates
3. **Team onboarding**: Include validation in setup procedures
4. **Automated checks**: Include in monitoring and CI/CD pipelines

### Configuration Management
1. **Version control**: Track configuration file changes
2. **Testing**: Test configurations in non-production environments first
3. **Documentation**: Document custom tool installations and paths
4. **Rollback**: Have rollback procedures for broken configurations

### Error Handling
1. **Graceful failures**: Handle validation failures appropriately
2. **Detailed logging**: Log validation results for troubleshooting
3. **Automated fixing**: Use auto-discovery to fix common issues
4. **Manual procedures**: Document manual resolution steps

## Differences from Other Config Commands

### `config validate` vs `config check`
- **`config validate`**: Validates only configured tools
- **`config check`**: Comprehensive system health check including Docker, cloud connectivity

### `config validate` vs `config discover`
- **`config validate`**: Tests existing configuration
- **`config discover`**: Finds new tools and optionally updates configuration

### When to Use Each
```bash
# After manual configuration changes
dbsnapper config validate

# After installing new database software
dbsnapper config discover --save

# For comprehensive system health check
dbsnapper config check
```

!!! tip "Configuration Changes"
    Always run `dbsnapper config validate` after manually editing your configuration file to ensure tool paths are still valid.

!!! note "JSON Output"
    Use `--json` flag for programmatic integration. The JSON output includes detailed error information for troubleshooting.

!!! warning "Tool Dependencies"
    Validation only checks that tools are executable. It doesn't validate that all required libraries and dependencies are available.

## Related Commands

- [`config check`](config-check.md) - Comprehensive configuration and system validation
- [`config discover`](config-discover.md) - Auto-discover and configure database tools
- [`config init`](config-init.md) - Initialize configuration with auto-discovery
- [`config`](config.md) - Overview of configuration management commands

## See Also

- [Configuration Reference](../configuration.md) - Database tools configuration options
- [Database Engines](../database-engines/introduction.md) - Supported databases and requirements
- [Installation Guide](../installation.md) - Installing and configuring database client tools
