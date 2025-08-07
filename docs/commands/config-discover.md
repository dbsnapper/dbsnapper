# Config Discover Command

The `config discover` command automatically finds and validates database client tools on your system. This intelligent discovery tool scans common installation paths, validates tool compatibility, and optionally updates your DBSnapper configuration with the discovered tools.

## Overview

The config discover command provides automated database tool management:
- **Automatic detection**: Scans system for PostgreSQL and MySQL client tools
- **Path validation**: Verifies tools are executable and compatible
- **Version checking**: Ensures discovered tools meet DBSnapper requirements
- **Configuration integration**: Optionally updates configuration with discovered paths
- **Multi-format output**: Supports both human-readable and JSON output formats

## Syntax

```bash
dbsnapper config discover [flags]
```

## Arguments

No arguments required - performs comprehensive tool discovery by default.

## Options

```bash
  -h, --help   help for discover
      --json   Output results as JSON
      --save   Save discovered tool paths to configuration file
```

## Discovery Process

### Search Strategy
The discover command uses a comprehensive search strategy to find database tools:

1. **Priority-based search**: Checks high-priority locations first
2. **Path validation**: Tests each discovered tool for executability
3. **Version compatibility**: Verifies tool versions are supported
4. **Conflict resolution**: Chooses optimal tools when multiple versions found

### Search Locations

#### PostgreSQL Tools
The discovery process searches for `psql`, `pg_dump`, and `pg_restore` in:

**macOS:**
- **Postgres.app**: `/Applications/Postgres.app/Contents/Versions/*/bin/`
- **Homebrew**: `/opt/homebrew/bin/`, `/usr/local/bin/`
- **MacPorts**: `/opt/local/bin/`
- **System paths**: `/usr/bin/`, `/usr/local/bin/`

**Linux:**
- **Package manager paths**: `/usr/bin/`, `/usr/local/bin/`
- **PostgreSQL official**: `/usr/pgsql-*/bin/`
- **Custom installations**: User-defined paths in `$PATH`

**Windows:**
- **Program Files**: `C:\Program Files\PostgreSQL\*\bin\`
- **System paths**: Paths defined in `%PATH%`

#### MySQL Tools
The discovery process searches for `mysql` and `mysqldump` in:

**macOS:**
- **Homebrew**: `/opt/homebrew/opt/mysql-client/bin/`, `/opt/homebrew/bin/`
- **MySQL official**: `/usr/local/mysql/bin/`
- **System paths**: `/usr/bin/`, `/usr/local/bin/`

**Linux:**
- **Package manager paths**: `/usr/bin/`, `/usr/local/bin/`
- **MySQL official**: `/usr/local/mysql/bin/`
- **MariaDB**: `/usr/local/mariadb/bin/`

**Windows:**
- **Program Files**: `C:\Program Files\MySQL\MySQL Server *\bin\`
- **System paths**: Paths defined in `%PATH%`

### Tool Validation
Each discovered tool undergoes validation:

1. **Executable test**: Verifies the tool can be executed
2. **Version check**: Ensures version compatibility with DBSnapper
3. **Functionality test**: Basic functionality verification
4. **Permission check**: Confirms appropriate execution permissions

## Output Formats

### Human-Readable Output (Default)
```bash
dbsnapper config discover
```

**Output:**
```
DBSnapper v2.7.0-dev - Database Tools Discovery
──────────────────────────────────────────────────

→ Database Tools Discovery
  ✅ pg_dump: /Applications/Postgres.app/Contents/Versions/17/bin/pg_dump
  ✅ pg_restore: /Applications/Postgres.app/Contents/Versions/17/bin/pg_restore
  ✅ psql: /Applications/Postgres.app/Contents/Versions/17/bin/psql
  ✅ mysql: /opt/homebrew/opt/mysql-client/bin/mysql
  ✅ mysqldump: /opt/homebrew/opt/mysql-client/bin/mysqldump

💡 To save this configuration, run:
   dbsnapper config discover --save

  ✓ Database Tools Discovery completed
```

### JSON Output
```bash
dbsnapper config discover --json
```

**Output:**
```json
{
  "success": true,
  "discovered_tools": {
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
  "message": "Database tools discovery completed successfully"
}
```

## Configuration Integration

### Save Discovered Tools
The `--save` flag updates your configuration file with discovered tools:

```bash
dbsnapper config discover --save
```

This updates the `database_tools` section:

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

### Configuration Merge Behavior
- **Preserves existing settings**: Manual tool paths are not overwritten
- **Updates auto-detected paths**: Only updates paths that were auto-detected
- **Maintains structure**: Preserves other configuration sections
- **Backup safety**: Creates backup before making changes

## Example Usage

### Basic Tool Discovery
```bash
# Discover tools without saving
dbsnapper config discover
```

### Discovery and Configuration Update
```bash
# Discover tools and save to configuration
dbsnapper config discover --save
```

### JSON Output for Automation
```bash
# Get discovery results in JSON format
dbsnapper config discover --json

# Save JSON results and update configuration
dbsnapper config discover --json --save
```

### Custom Configuration File
```bash
# Discover tools for specific configuration file
dbsnapper config discover --config /path/to/custom.yml --save
```

## Use Cases

### Initial System Setup
```bash
# After installing DBSnapper
dbsnapper config init
dbsnapper config discover --save
dbsnapper config check
```

### After Database Software Updates
```bash
# After upgrading PostgreSQL or MySQL
dbsnapper config discover --save

# Verify new tools work correctly
dbsnapper config check
```

### System Migration
```bash
# On new system or container
dbsnapper config discover --json > discovered-tools.json
dbsnapper config discover --save

# Verify migration success
dbsnapper config check
```

### CI/CD Environment Setup
```bash
# Automated pipeline setup
dbsnapper config discover --save
if ! dbsnapper config check >/dev/null 2>&1; then
    echo "Database tools not properly configured"
    exit 1
fi
```

### Team Environment Standardization
```bash
# Generate standardized tool configuration
dbsnapper config discover --json --save

# Share configuration template with team
cp ~/.config/dbsnapper/dbsnapper.yml team-template.yml
```

### Troubleshooting Tool Issues
```bash
# Diagnose tool problems
dbsnapper config discover

# Compare with current configuration
dbsnapper config check

# Update if tools have moved
dbsnapper config discover --save
```

## Advanced Usage

### Programmatic Integration
```bash
#!/bin/bash
# Script to ensure tools are available

# Discover tools and get JSON output
DISCOVERY_RESULT=$(dbsnapper config discover --json)

# Check if discovery was successful
if echo "$DISCOVERY_RESULT" | jq -e '.success' >/dev/null; then
    echo "Tools discovered successfully"
    dbsnapper config discover --save
else
    echo "Tool discovery failed"
    exit 1
fi
```

### Automated Configuration Updates
```bash
#!/bin/bash
# Automated maintenance script

# Check current configuration
if ! dbsnapper config check >/dev/null 2>&1; then
    echo "Configuration issues detected"
    
    # Try to fix with discovery
    dbsnapper config discover --save
    
    # Verify fix worked
    if dbsnapper config check >/dev/null 2>&1; then
        echo "Configuration fixed automatically"
    else
        echo "Manual intervention required"
        exit 1
    fi
fi
```

### Multi-Environment Discovery
```bash
# Discover tools for different environments
for env in dev staging prod; do
    echo "Discovering tools for $env environment"
    dbsnapper config discover --config "configs/$env.yml" --save
done
```

## Tool Priority and Selection

### Version Preference
When multiple versions are found, the discovery process prioritizes:

1. **Newer versions**: More recent tool versions are preferred
2. **Standard locations**: Tools in standard locations over custom paths
3. **Package manager installations**: Tools installed via package managers
4. **Consistency**: Tools from the same installation/version when possible

### Path Selection Logic
```bash
# Example priority order for PostgreSQL on macOS:
# 1. /Applications/Postgres.app/Contents/Versions/17/bin/pg_dump
# 2. /opt/homebrew/bin/pg_dump  
# 3. /usr/local/bin/pg_dump
# 4. /usr/bin/pg_dump
```

## Error Handling and Troubleshooting

### Common Discovery Issues

#### No Tools Found
```bash
# Output when no tools are found
❌ No PostgreSQL tools found
❌ No MySQL tools found
```

**Solutions:**
1. Install database client tools using package manager
2. Add tool locations to system PATH
3. Install from official database websites

#### Permission Issues
```bash
# Output for permission problems
⚠️ pg_dump found but not executable
```

**Solutions:**
```bash
# Fix permissions
chmod +x /path/to/pg_dump

# Check ownership
ls -la /path/to/pg_dump
```

#### Version Compatibility Issues
```bash
# Output for incompatible versions
⚠️ pg_dump version too old (requires 12+)
```

**Solutions:**
1. Upgrade to supported version
2. Install newer version alongside existing
3. Use package manager to update

### Discovery Failures
```json
{
  "success": false,
  "error": "No database tools found",
  "discovered_tools": {},
  "validation_results": {},
  "message": "Database tools discovery failed"
}
```

## Performance Considerations

### Discovery Speed
- **Fast search**: Optimized path scanning
- **Cached results**: Results can be cached between runs
- **Parallel validation**: Tools validated concurrently when possible
- **Early termination**: Stops searching when optimal tools found

### System Impact
- **Low overhead**: Minimal system resource usage
- **No installation**: Only discovers existing tools
- **Safe operation**: Read-only system scanning
- **Respect limits**: Honours system resource constraints

## Best Practices

### Discovery Strategy
1. **Regular discovery**: Run after software updates
2. **Validation**: Always validate discovered tools with `config check`
3. **Version tracking**: Monitor tool versions for compatibility
4. **Documentation**: Document custom tool installations

### Configuration Management
1. **Backup first**: Backup configuration before auto-saving
2. **Version control**: Track configuration changes
3. **Testing**: Test configuration after discovery updates
4. **Rollback plan**: Have rollback procedures for failed updates

### Automation Integration
1. **CI/CD integration**: Include in pipeline setup
2. **Health monitoring**: Regular discovery in monitoring scripts
3. **Error handling**: Robust error handling in automated scripts
4. **Logging**: Log discovery results for troubleshooting

!!! tip "After Software Updates"
    Run `dbsnapper config discover --save` after updating PostgreSQL or MySQL to ensure DBSnapper uses the latest tools.

!!! note "JSON Output"
    Use `--json` flag for programmatic integration and automation scripts. The JSON output provides structured data for parsing and validation.

!!! warning "Tool Compatibility"
    The discover command validates tool compatibility with DBSnapper. Very old or very new tool versions may not be supported.

## Related Commands

- [`config init`](config-init.md) - Initialize configuration (includes discovery)
- [`config check`](config-check.md) - Validate discovered tools and configuration
- [`config validate`](config-validate.md) - Validate configuration file syntax
- [`config`](config.md) - Overview of configuration management commands

## See Also

- [Database Engines](../database-engines/introduction.md) - Supported databases and client requirements
- [Installation Guide](../installation.md) - Installing database client tools
- [Configuration Reference](../configuration.md) - Database tools configuration options
