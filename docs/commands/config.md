# Config Command

The `config` command provides comprehensive configuration management for DBSnapper, offering tools to initialize, validate, check, and maintain your DBSnapper environment. This command serves as the central hub for all configuration-related operations.

## Overview

The config command encompasses essential configuration management functions:
- **Initialization**: Bootstrap new DBSnapper installations
- **Validation**: Verify configuration file syntax and structure
- **Health Checking**: Test database connectivity and tool availability
- **Auto-Discovery**: Find and configure database client tools automatically
- **Troubleshooting**: Diagnose and resolve configuration issues

## Syntax

```bash
dbsnapper config [command]
```

## Available Subcommands

### Core Configuration Commands

| Command | Purpose | Use When |
|---------|---------|----------|
| [`init`](config-init.md) | Initialize configuration and working directory | Setting up new DBSnapper installation |
| [`check`](config-check.md) | Check configuration and dependencies | Troubleshooting or health monitoring |
| [`discover`](config-discover.md) | Auto-discover database tools | After installing new database clients |
| [`validate`](config-validate.md) | Validate configuration file syntax | After manual configuration edits |

## Configuration Management Workflow

### New Installation Setup
```bash
# 1. Initialize DBSnapper configuration
dbsnapper config init

# 2. Verify installation and auto-discover tools
dbsnapper config check

# 3. Validate configuration structure
dbsnapper config validate
```

### Maintenance and Updates
```bash
# Regular health check
dbsnapper config check

# After installing new database versions
dbsnapper config discover --save

# After manual configuration changes
dbsnapper config validate
```

### Troubleshooting Workflow
```bash
# Comprehensive system check
dbsnapper config check

# Verify configuration syntax
dbsnapper config validate

# Re-discover tools if paths changed
dbsnapper config discover --save --json
```

## Configuration File Management

### Configuration File Locations
The config commands work with configuration files in this priority order:

1. **`--config` flag**: Explicitly specified file path
2. **`DBSNAPPER_CONFIG` environment variable**: Environment-specified path  
3. **Development directory**: `dbsnapper.yml` in current directory (if inside DBSnapper repo)
4. **Default location**: `~/.config/dbsnapper/dbsnapper.yml`

### Configuration File Structure
```yaml
# Authentication and Security
authtoken: ""                    # DBSnapper Cloud API token
secret_key: "generated-key"      # Encryption key for sensitive data

# Database Tools Configuration
database_tools:
  auto_detect: true
  postgresql:
    pg_dump: "/path/to/pg_dump"
    pg_restore: "/path/to/pg_restore"  
    psql: "/path/to/psql"
  mysql:
    mysql: "/path/to/mysql"
    mysqldump: "/path/to/mysqldump"

# Default Settings
defaults:
  working_directory: "~/.dbsnapper"
  cpus: 2
  shared_target_dst_db_url: ""

# Docker Configuration  
docker:
  images:
    mysql: "mysql:9"
    postgres: "postgres:17"

# Feature Flags
feature_flags:
  development_utilities: false

# Global Overrides
override:
  dst_db_url: ""
  nocloud: false
  san_query: ""

# Storage Profiles (optional)
storage_profiles:
  aws-s3:
    type: "s3"
    region: "us-west-2"
    bucket: "my-snapshots"

# Target Definitions
targets:
  my-target:
    name: "My Database"
    snapshot:
      src_url: "postgresql://user:pass@host:5432/db"
      dst_url: "postgresql://user:pass@host:5432/db_snap"
```

## Common Configuration Scenarios

### Basic Local Setup
```bash
# Initialize for local development
dbsnapper config init

# The generated configuration includes:
# - Auto-discovered database tools
# - Default working directory
# - Local target definitions
```

### Cloud Integration Setup
```bash
# Initialize with cloud features
dbsnapper config init

# Add cloud authentication
dbsnapper auth token <your_token>

# Add storage profiles for cloud snapshots
# (Edit configuration file to add storage_profiles section)
```

### Team/Enterprise Setup
```bash
# Initialize with shared configuration
dbsnapper config init --config /shared/config/dbsnapper.yml

# Validate team configuration
dbsnapper config validate

# Ensure all team members have required tools
dbsnapper config check
```

### Development Environment
```bash
# Setup from DBSnapper repository
cd /path/to/dbsnapper-repo
dbsnapper config init  # Creates dbsnapper.yml in repo root

# Enable development utilities
# Edit config to set: feature_flags.development_utilities: true
```

## Configuration Components

### Database Tools Configuration
- **Auto-detection**: Automatically finds PostgreSQL and MySQL client tools
- **Manual configuration**: Override auto-detected paths
- **Validation**: Verifies tools are executable and compatible
- **Updates**: Re-discovery when tools are updated or moved

### Working Directory Management  
- **Creation**: Automatic creation of working directory structure
- **Permissions**: Proper permissions for snapshot storage
- **Location**: Configurable location with sensible defaults
- **Cleanup**: Management of temporary files and old snapshots

### Cloud Integration Settings
- **Authentication**: API token management for DBSnapper Cloud
- **Storage profiles**: S3-compatible cloud storage configuration
- **Shared targets**: Okta SSO integration for team collaboration
- **Security**: Encryption keys and secure credential storage

### Target Configuration
- **Database connections**: Source and destination database URLs
- **Schema filtering**: PostgreSQL schema inclusion/exclusion rules
- **Sanitization**: Query files and sanitization settings
- **Subsetting**: Table subsetting and relationship configuration

## Security Considerations

### Configuration File Protection
```bash
# Set appropriate permissions
chmod 600 ~/.config/dbsnapper/dbsnapper.yml

# Verify permissions
ls -la ~/.config/dbsnapper/dbsnapper.yml
# Should show: -rw------- (read/write for owner only)
```

### Sensitive Data Management
- **Secret key**: Generated automatically, keep secure
- **Database credentials**: Use environment variables when possible
- **API tokens**: Store securely, rotate regularly
- **Version control**: Never commit configuration files with secrets

### Environment Variable Usage
```bash
# Use environment variables for sensitive data
export DBSNAPPER_SECRET_KEY="your-secret-key"
export DBSNAPPER_AUTHTOKEN="your-api-token"
export DATABASE_URL="postgresql://user:pass@host:5432/db"

# Reference in configuration
targets:
  my-target:
    snapshot:
      src_url: "${DATABASE_URL}"
```

## Example Usage

### Initial Setup
```bash
# Complete new installation setup
dbsnapper config init
dbsnapper config check
dbsnapper config validate

# Check results
cat ~/.config/dbsnapper/dbsnapper.yml
```

### Regular Maintenance
```bash
# Monthly configuration health check
dbsnapper config check

# After database software updates
dbsnapper config discover --save

# Before important operations
dbsnapper config validate
```

### Troubleshooting
```bash
# Comprehensive troubleshooting
dbsnapper config check --json > config-status.json
dbsnapper config validate --json > config-validation.json

# Fix issues and re-check
dbsnapper config discover --save
dbsnapper config check
```

### Team Collaboration
```bash
# Validate shared team configuration
dbsnapper config --config /team/shared/dbsnapper.yml validate

# Check tool availability across team
dbsnapper config --config /team/shared/dbsnapper.yml check
```

## Integration with Other Commands

### Configuration-Dependent Operations
Most DBSnapper commands depend on proper configuration:

```bash
# These commands require valid configuration
dbsnapper targets        # Needs target definitions
dbsnapper build my-db    # Needs target and tool configuration
dbsnapper sanitize my-db # Needs sanitization queries and tools
```

### Configuration Verification Before Operations
```bash
# Verify configuration before critical operations
dbsnapper config check && dbsnapper build production-db
dbsnapper config validate && dbsnapper sanitize sensitive-data
```

## Use Cases

### Development Team Onboarding
```bash
# New developer setup
dbsnapper config init
# Manual: Add team-specific targets to configuration
dbsnapper config check    # Verify everything works
```

### CI/CD Pipeline Setup
```bash
# Automated pipeline configuration
DBSNAPPER_CONFIG=/ci/config/dbsnapper.yml dbsnapper config init
dbsnapper config check --json | jq '.status' # Verify in scripts
```

### Database Migration Projects
```bash
# Setup for migration project
dbsnapper config init --config ./migration-config.yml
# Manual: Configure source and destination databases
dbsnapper config validate
```

### Multi-Environment Management
```bash
# Separate configurations per environment
dbsnapper config init --config ~/.config/dbsnapper/dev.yml
dbsnapper config init --config ~/.config/dbsnapper/staging.yml
dbsnapper config init --config ~/.config/dbsnapper/prod.yml

# Validate each environment
for env in dev staging prod; do
  dbsnapper config --config ~/.config/dbsnapper/$env.yml validate
done
```

### Disaster Recovery Preparation
```bash
# Verify disaster recovery configuration
dbsnapper config --config /backup/dr-config.yml check
dbsnapper config --config /backup/dr-config.yml validate
```

## Best Practices

### Configuration Management
1. **Version control**: Keep configuration templates in version control (without secrets)
2. **Environment separation**: Use different config files for different environments
3. **Regular validation**: Run `config check` regularly to catch issues early
4. **Documentation**: Document custom configuration choices
5. **Backup**: Keep secure backups of working configurations

### Security Best Practices
1. **File permissions**: Restrict configuration file access to owner only
2. **Environment variables**: Use env vars for sensitive data
3. **Secret rotation**: Regularly rotate API tokens and secret keys
4. **Audit trail**: Log configuration changes and access
5. **Principle of least privilege**: Configure minimal required permissions

### Operational Excellence
1. **Health monitoring**: Integrate `config check` into monitoring
2. **Change management**: Test configuration changes before deployment
3. **Automation**: Automate configuration validation in CI/CD
4. **Troubleshooting**: Use JSON output for programmatic analysis
5. **Team coordination**: Share configuration standards across team

## Troubleshooting Common Issues

### Configuration File Issues
```bash
# File not found
Error: configuration file not found
# Solution: Run `dbsnapper config init` or specify correct path

# Permission denied  
Error: permission denied reading configuration
# Solution: Check file permissions with `ls -la`

# Invalid YAML syntax
Error: configuration file syntax error
# Solution: Run `dbsnapper config validate` for details
```

### Database Tool Issues
```bash
# Tools not found
Warning: PostgreSQL tools not found
# Solution: Install tools, then run `dbsnapper config discover --save`

# Tool version incompatible
Error: pg_dump version too old
# Solution: Update PostgreSQL client tools
```

### Permission and Access Issues
```bash
# Working directory creation failed
Error: cannot create working directory
# Solution: Check parent directory permissions and disk space

# Cloud authentication failed  
Error: invalid API token
# Solution: Update token with `dbsnapper auth token <new_token>`
```

!!! tip "Regular Health Checks"
    Run `dbsnapper config check` regularly to catch configuration issues before they impact operations. Consider adding it to your regular maintenance routine.

!!! note "Subcommand Details"
    Each config subcommand has detailed documentation. Use `dbsnapper config [subcommand] --help` for specific options and examples.

!!! warning "Configuration Security"
    Never commit configuration files containing secrets to version control. Use environment variables or secure secret management systems for sensitive data.

## Related Commands

- [`config init`](config-init.md) - Initialize new DBSnapper configuration
- [`config check`](config-check.md) - Check configuration and dependencies
- [`config discover`](config-discover.md) - Auto-discover database tools
- [`config validate`](config-validate.md) - Validate configuration syntax

## See Also

- [Configuration Reference](../configuration.md) - Complete configuration file reference
- [Database Engines](../database-engines/introduction.md) - Supported databases and client setup
- [Installation Guide](../installation.md) - DBSnapper installation instructions  
- [Cloud Integration](../dbsnapper-cloud/introduction.md) - Cloud features and setup