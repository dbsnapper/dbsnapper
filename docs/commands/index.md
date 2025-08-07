# DBSnapper Commands

This section provides comprehensive documentation for all DBSnapper v3.0 commands. Each command includes detailed syntax, options, examples, use cases, and troubleshooting guidance.

## Command Categories

### Core Workflow Commands

These are the primary commands for database snapshot operations:

| Command | Purpose | Quick Start |
|---------|---------|-------------|
| [`build`](build.md) | Create database snapshots | `dbsnapper build my-target` |
| [`load`](load.md) | Load snapshots into databases | `dbsnapper load my-target` |
| [`sanitize`](sanitize.md) | Remove sensitive data from snapshots | `dbsnapper sanitize my-target` |
| [`subset`](subset.md) | Create smaller relational database subsets | `dbsnapper subset my-target` |
| [`pull`](pull.md) | Download cloud snapshots | `dbsnapper pull my-target` |

### Target Management Commands

Commands for managing database targets:

| Command | Purpose | Quick Start |
|---------|---------|-------------|
| [`targets`](targets.md) | List all available targets | `dbsnapper targets` |
| [`target`](target.md) | View detailed target information | `dbsnapper target my-target` |

### Configuration Commands

Setup and configuration management:

| Command | Purpose | Quick Start |
|---------|---------|-------------|
| [`config`](config.md) | Configuration command overview | `dbsnapper config --help` |
| [`config init`](config-init.md) | Initialize DBSnapper configuration | `dbsnapper config init` |
| [`config check`](config-check.md) | Validate configuration and dependencies | `dbsnapper config check` |
| [`config discover`](config-discover.md) | Auto-discover database tools | `dbsnapper config discover` |
| [`config validate`](config-validate.md) | Validate configuration file | `dbsnapper config validate` |

### Authentication Commands

Cloud integration and authentication:

| Command | Purpose | Quick Start |
|---------|---------|-------------|
| [`auth`](auth.md) | Authentication command overview | `dbsnapper auth --help` |
| [`auth token`](auth-token.md) | Configure cloud API token | `dbsnapper auth token <token>` |

### Development Commands

Testing and development utilities:

| Command | Purpose | Quick Start |
|---------|---------|-------------|
| [`dev`](dev.md) | Development utilities overview | `dbsnapper dev --help` |
| [`dev ts`](dev-ts.md) | Add timestamp for testing | `dbsnapper dev ts my-target` |

### Integration Commands

External system integrations:

| Command | Purpose | Quick Start |
|---------|---------|-------------|
| [`mcp`](mcp.md) | Start Model Context Protocol server | `dbsnapper mcp` |

## Getting Started

### New Users
If you're new to DBSnapper, start with these commands in order:

1. **[`config init`](config-init.md)** - Set up your initial configuration
2. **[`config check`](config-check.md)** - Verify your setup is working
3. **[`targets`](targets.md)** - View available database targets
4. **[`build`](build.md)** - Create your first snapshot

### Common Workflows

#### Basic Snapshot Workflow
```bash
# Initialize configuration (first time only)
dbsnapper config init

# Create snapshot
dbsnapper build production-db

# Load into development environment
dbsnapper load production-db
```

#### Sanitization Workflow
```bash
# Build original snapshot
dbsnapper build production-db

# Create sanitized version
dbsnapper sanitize production-db

# Load sanitized snapshot
dbsnapper load production-db
```

#### Cloud Integration Workflow
```bash
# Set up cloud authentication
dbsnapper auth token <your-token>

# Pull shared team snapshots
dbsnapper pull shared-team-data

# Load into local environment
dbsnapper load shared-team-data
```

## Command Structure

DBSnapper commands follow a consistent structure:

### Standard Format
```bash
dbsnapper <command> [subcommand] <target_name> [options]
```

### Global Options
All commands support these global options:
- `--config string` - Specify custom configuration file
- `--nocloud` - Disable cloud mode for faster local operations
- `--help` - Display help information

### Target Names
Most commands require a target name that corresponds to a configured database target in your configuration file. Use `dbsnapper targets` to see available targets.

## Command Output Formats

Many commands support different output formats:

### JSON Output
Use `--json` flag for programmatic access:
```bash
dbsnapper targets --json
dbsnapper config check --json
```

### Table Output
Default human-readable format:
```bash
dbsnapper targets
dbsnapper config check
```

## Error Handling

All commands provide clear error messages and exit codes:

- **Exit Code 0**: Success
- **Exit Code 1**: General error
- **Exit Code 2**: Configuration error
- **Exit Code 3**: Database connection error

## Security Considerations

### Data Protection
- Snapshots may contain sensitive data
- Use sanitization for development/testing environments
- Protect configuration files with appropriate permissions
- Rotate cloud authentication tokens regularly

### Access Control
- Database credentials stored in configuration files
- Cloud integration requires API tokens
- Shared targets may have access restrictions
- Audit trail available for cloud operations

## Configuration Requirements

### Database Tools
DBSnapper requires database client tools:
- **PostgreSQL**: `pg_dump`, `pg_restore`, `psql`
- **MySQL**: `mysql`, `mysqldump`

Use `dbsnapper config discover` to automatically find and configure these tools.

### Docker Support
Optional Docker integration for:
- Ephemeral database operations
- Isolated sanitization environments
- Consistent cross-platform behavior

## Cloud Integration

### DBSnapper Cloud Features
When authenticated with DBSnapper Cloud:
- **Cloud targets**: Centrally managed database targets
- **Shared targets**: Team collaboration via Okta SSO
- **Cloud storage**: S3-compatible snapshot storage
- **Web interface**: Browser-based target management

### Authentication Setup
```bash
# Configure cloud access
dbsnapper auth token <your-token>

# Verify connectivity
dbsnapper config check
```

## Development and Testing

### End-to-End Testing
```bash
# Enable development utilities
# Add to configuration: feature_flags.development_utilities: true

# Add test timestamp
dbsnapper dev ts my-target --test-name "e2e-test"

# Run operations
dbsnapper build my-target
dbsnapper load my-target

# Validate consistency
dbsnapper dev assert-ts my-target
```

### MCP Integration
```bash
# Start MCP server for AI integration
dbsnapper mcp

# Use with Claude Desktop or other MCP-compatible applications
```

## Troubleshooting

### Common Issues

#### Command Not Found
```bash
# Verify installation
dbsnapper --version

# Check PATH
which dbsnapper
```

#### Configuration Errors
```bash
# Validate configuration
dbsnapper config check

# Re-initialize if needed
dbsnapper config init
```

#### Database Connection Issues
```bash
# Test connectivity
dbsnapper targets

# Check credentials and network access
dbsnapper config validate
```

#### Missing Database Tools
```bash
# Auto-discover tools
dbsnapper config discover --save

# Manual verification
dbsnapper config check
```

## Getting Help

### Command-Specific Help
```bash
# Get help for any command
dbsnapper <command> --help

# Examples
dbsnapper build --help
dbsnapper config check --help
```

### Community and Support
- **Documentation**: Complete guides at [docs.dbsnapper.com](https://docs.dbsnapper.com)
- **GitHub Issues**: Report bugs and request features
- **Community**: Join discussions and get help from the community

## See Also

- [Configuration Reference](../configuration.md) - Complete configuration options
- [Database Engines](../database-engines/introduction.md) - Supported databases
- [DBSnapper Cloud](../dbsnapper-cloud/introduction.md) - Cloud features and setup
- [Quick Start Guide](../quick-start.md) - Getting started tutorial