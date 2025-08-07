# Config Init Command

The `config init` command initializes DBSnapper by creating a configuration file and working directory structure. This is typically the first command new users run to set up their DBSnapper environment with sensible defaults and automatically discovered database tools.

## Overview

The config init command bootstraps a complete DBSnapper installation by:
- Creating a YAML configuration file with default settings
- Generating a secure encryption key for sensitive data
- Setting up the working directory structure
- Auto-discovering database client tools on your system
- Providing a foundation for adding targets and storage profiles

## Syntax

```bash
dbsnapper config init [flags]
```

## Arguments

No arguments required - all configuration is automatic with sensible defaults.

## Options

```bash
  -h, --help   Show help for the config init command
```

## How It Works

1. **Configuration File Creation**: Creates a YAML configuration file at the specified or default location
2. **Working Directory Setup**: Creates the working directory for storing snapshots and temporary files
3. **Secret Key Generation**: Generates a secure 32-character secret key for encrypting sensitive data
4. **Database Tool Discovery**: Automatically discovers and configures PostgreSQL and MySQL client tools
5. **Default Settings**: Applies sensible defaults for Docker images, CPU usage, and feature flags
6. **Validation**: Validates discovered tools and reports their status

## Configuration File Location

The configuration file location follows this priority order:

1. **`--config` flag**: Explicitly specified file path
2. **`DBSNAPPER_CONFIG` environment variable**: Environment-specified path
3. **Development directory**: If running from the DBSnapper repository, uses `dbsnapper.yml` in repo root
4. **Default location**: `~/.config/dbsnapper/dbsnapper.yml`

### Examples

```bash
# Use default location (~/.config/dbsnapper/dbsnapper.yml)
dbsnapper config init

# Specify custom location with flag
dbsnapper config init --config /path/to/custom/config.yml

# Specify custom location with environment variable
DBSNAPPER_CONFIG=/path/to/custom/config.yml dbsnapper config init
```

## Generated Configuration Structure

The config init command creates a complete configuration file with these sections:

### Authentication and Security
```yaml
authtoken: ""                                    # DBSnapper Cloud API token
secret_key: 37493065d18a05a4df222decea24e956     # Generated encryption key
```

### Database Tools (Auto-Discovered)
```yaml
database_tools:
  auto_detect: true                              # Enable automatic tool discovery
  postgresql:
    pg_dump: /Applications/Postgres.app/Contents/Versions/17/bin/pg_dump
    pg_restore: /Applications/Postgres.app/Contents/Versions/17/bin/pg_restore
    psql: /Applications/Postgres.app/Contents/Versions/17/bin/psql
  mysql:
    mysql: /opt/homebrew/opt/mysql-client/bin/mysql
    mysqldump: /opt/homebrew/opt/mysql-client/bin/mysqldump
```

### Default Settings
```yaml
defaults:
  working_directory: /Users/username/.dbsnapper  # Snapshot storage location
  cpus: 2                                        # CPU cores for operations
  shared_target_dst_db_url: ""                   # Default destination for shared targets
```

### Docker Configuration
```yaml
docker:
  images:
    mysql: mysql:9                               # MySQL Docker image
    postgres: postgres:17                        # PostgreSQL Docker image
```

### Feature Flags and Overrides
```yaml
debug: false                                     # Debug logging disabled
feature_flags:
  development_utilities: false                   # Development tools disabled by default
override:
  dst_db_url: ""                                # Global destination database override
  nocloud: false                                # Cloud mode enabled by default
  san_query: ""                                 # Global sanitization query override
```

## Working Directory Structure

The initialized working directory (`~/.dbsnapper` by default) contains:

```
~/.dbsnapper/
├── snapshots/          # Created automatically when first snapshot is built
└── temp/              # Created automatically for temporary operations
```

**Snapshot Files**: When you create snapshots, they're stored as:
- `{timestamp}_{target_name}.zip` - Original snapshots
- `{timestamp}_{target_name}.san.zip` - Sanitized snapshots

## Database Tool Discovery

The init command automatically discovers database client tools in these locations:

### PostgreSQL Tools
- **Postgres.app**: `/Applications/Postgres.app/Contents/Versions/*/bin/`
- **Homebrew**: `/opt/homebrew/bin/`, `/usr/local/bin/`
- **System paths**: `/usr/bin/`, `/usr/local/bin/`
- **Custom installations**: Checks PATH environment variable

### MySQL Tools  
- **Homebrew**: `/opt/homebrew/opt/mysql-client/bin/`, `/opt/homebrew/bin/`
- **System installations**: `/usr/bin/`, `/usr/local/bin/`, `/usr/local/mysql/bin/`
- **Custom installations**: Checks PATH environment variable

### Tool Validation

Each discovered tool is validated by:
- Checking executable permissions
- Verifying the tool responds to version queries
- Ensuring compatibility with DBSnapper requirements

## Configuration File Format Evolution

DBSnapper v3.0 introduces an improved configuration format while maintaining backwards compatibility:

### New Format (Recommended)
```yaml
defaults:
  working_directory: ~/.dbsnapper    # New location under defaults
  cpus: 2
```

### Legacy Format (Still Supported)
```yaml
working_directory: ~/.dbsnapper      # Legacy root-level location
```

### Priority Rules
- If both formats exist, `defaults.working_directory` takes precedence
- Legacy format is automatically upgraded on next configuration update
- No manual migration required

## Example Usage

### Basic Initialization
```bash
# Initialize with default settings
dbsnapper config init
```

**Output:**
```
DBSnapper v2.7.0-dev - Configuration Initialization
──────────────────────────────────────────────────────

→ Config file location: /Users/username/.config/dbsnapper/dbsnapper.yml
→ Working directory: /Users/username/.dbsnapper

→ Configuration Initialization

  ✓ Configuration file initialized: /Users/username/.config/dbsnapper/dbsnapper.yml
  ✓ Working directory created: /Users/username/.dbsnapper
```

### Custom Location Initialization
```bash
# Initialize with custom configuration location
dbsnapper config init --config /project/dbsnapper.yml
```

### Development Setup
```bash
# Initialize for development (from DBSnapper repo directory)
cd /path/to/dbsnapper-repo
dbsnapper config init  # Creates dbsnapper.yml in repo root
```

## Post-Initialization Steps

After running `config init`, you can:

### 1. Verify Installation
```bash
# Check configuration and discovered tools
dbsnapper config check
```

### 2. Add Database Targets
Edit your configuration file to add targets:

```yaml
targets:
  my-database:
    name: "My Database"
    snapshot:
      src_url: "postgresql://user:pass@localhost:5432/mydb"
      dst_url: "postgresql://user:pass@localhost:5432/mydb_snap"
```

### 3. Configure Cloud Integration (Optional)
```bash
# Add DBSnapper Cloud authentication
dbsnapper auth token <your_token>
```

### 4. Add Storage Profiles (Optional)
```yaml
storage_profiles:
  aws-s3:
    type: s3
    region: us-west-2
    bucket: my-snapshots
```

## Configuration File Management

### Manual Editing
The generated configuration file is standard YAML and can be edited manually:

```bash
# Edit with your preferred editor
nano ~/.config/dbsnapper/dbsnapper.yml
vim ~/.config/dbsnapper/dbsnapper.yml
```

### Validation
```bash
# Validate configuration after manual changes
dbsnapper config validate
```

### Tool Re-discovery
```bash
# Re-discover database tools after installing new versions
dbsnapper config discover --save
```

## Use Cases

### New User Setup
```bash
# Complete new installation
dbsnapper config init
dbsnapper config check
# Add targets via manual configuration or DBSnapper Cloud
```

### Project-Specific Configuration
```bash
# Setup per-project configuration
cd /path/to/project
dbsnapper config init --config ./dbsnapper.yml
# Configure project-specific targets and settings
```

### Development Environment
```bash
# Setup development environment
cd /path/to/dbsnapper-repo
dbsnapper config init
# Enables development features and uses repo-local config
```

### CI/CD Pipeline Setup
```bash
# Automated pipeline initialization
DBSNAPPER_CONFIG=/ci/config/dbsnapper.yml dbsnapper config init
# Configure targets via environment variables or mounted config
```

### Multiple Environment Management
```bash
# Separate configurations for different environments
dbsnapper config init --config ~/.config/dbsnapper/dev.yml
dbsnapper config init --config ~/.config/dbsnapper/staging.yml
dbsnapper config init --config ~/.config/dbsnapper/prod.yml
```

## Security Considerations

### Secret Key Protection
- The generated `secret_key` encrypts sensitive data in the configuration
- **Keep secure**: Don't commit to version control
- **Backup safely**: Store in secure password manager
- **Rotate regularly**: Generate new keys periodically

### Configuration File Security
```bash
# Set appropriate file permissions
chmod 600 ~/.config/dbsnapper/dbsnapper.yml

# Verify permissions
ls -la ~/.config/dbsnapper/dbsnapper.yml
# Should show: -rw------- (read/write for owner only)
```

### Environment Variables
```bash
# Use environment variables for sensitive data in CI/CD
export DBSNAPPER_AUTHTOKEN="your_token_here"
export DBSNAPPER_SECRET_KEY="your_secret_key_here"
```

## Troubleshooting

### Common Issues

**Permission Denied**
```
Error: permission denied creating configuration directory
```
- Check write permissions to configuration directory
- Try with appropriate permissions: `sudo dbsnapper config init`
- Use custom location with write access

**Database Tools Not Found**
```
Warning: no PostgreSQL tools found
Warning: no MySQL tools found
```
- Install required database client tools
- Check PATH environment variable
- Run `dbsnapper config discover --save` after installation

**Configuration File Already Exists**
```
Error: configuration file already exists
```
- The init command won't overwrite existing configurations
- Delete existing file if you want to reinitialize: `rm ~/.config/dbsnapper/dbsnapper.yml`
- Use different location: `dbsnapper config init --config /path/to/new/config.yml`

**Working Directory Creation Failed**
```
Error: failed to create working directory
```
- Check disk space availability
- Verify write permissions to parent directory
- Try different working directory location in configuration

**Invalid Configuration Path**
```
Error: invalid configuration file path
```
- Ensure parent directories exist
- Use absolute paths when possible
- Check path syntax and permissions

### Recovery Steps

1. **Check Permissions**: Ensure write access to configuration and working directories
2. **Verify Tools**: Install required PostgreSQL and MySQL client tools
3. **Clear State**: Remove existing configuration files if reinitializing
4. **Use Custom Paths**: Try different configuration or working directory locations
5. **Check Dependencies**: Ensure all required system dependencies are installed

!!! tip "First Command"
    `config init` is typically the first command new users run. It sets up everything needed to start using DBSnapper with sensible defaults.

!!! warning "Existing Configuration"
    The init command will not overwrite existing configuration files. Delete the existing file first if you want to reinitialize.

!!! note "Auto-Discovery"
    Database tools are automatically discovered during initialization. Run `dbsnapper config discover --save` after installing new database clients.

## Related Commands

- [`config check`](config-check.md) - Validate configuration and dependencies
- [`config discover`](config-discover.md) - Re-discover database tools
- [`config validate`](config-validate.md) - Validate configuration file syntax
- [`auth token`](auth-token.md) - Add cloud authentication

## See Also

- [Configuration](../configuration.md) - Complete configuration reference
- [Database Engines](../database-engines/introduction.md) - Supported databases and setup
- [Installation](../installation.md) - DBSnapper installation guide
- [Quick Start](../quick-start.md) - Getting started tutorial