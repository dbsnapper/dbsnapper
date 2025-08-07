# MCP Command

The `mcp` command starts the Model Context Protocol (MCP) server, enabling seamless integration between DBSnapper and AI applications like Claude Desktop. This powerful integration allows you to manage database snapshots, targets, and configurations through natural language conversations with AI assistants.

## Overview

The MCP command provides AI application integration:
- **Model Context Protocol**: Standards-based integration with LLM applications
- **Tool exposure**: Makes DBSnapper operations available as AI tools
- **Resource access**: Provides structured access to configuration and snapshot data
- **Natural language interface**: Control DBSnapper through conversational AI
- **Automated workflows**: Enable AI-assisted database management tasks

## Syntax

```bash
dbsnapper mcp [flags]
```

## Arguments

None - the MCP server runs without additional arguments.

## Options

```bash
  -h, --help   help for mcp

Global Flags:
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

## What is Model Context Protocol (MCP)?

The Model Context Protocol is a standardized way for AI applications to access external tools and data sources. The DBSnapper MCP Server exposes DBSnapper's functionality through this protocol, enabling AI assistants to:

### Core Capabilities
- **Database operations**: Create, load, and manage database snapshots
- **Target management**: List and inspect database targets
- **Configuration access**: View and validate DBSnapper configuration
- **Tool discovery**: Automatically discover and configure database client tools
- **Resource browsing**: Access snapshot metadata and build history

### Integration Benefits
- **Natural language control**: Use conversational language instead of command-line syntax
- **Context awareness**: AI understands your database environment and suggests appropriate actions
- **Automated workflows**: Chain multiple operations together intelligently
- **Error assistance**: Get intelligent help with troubleshooting and configuration issues

## How It Works

### MCP Server Architecture
1. **Standard I/O Communication**: Communicates with AI applications via stdin/stdout
2. **Tool Registry**: Exposes DBSnapper commands as callable tools
3. **Resource System**: Provides structured access to configuration and data
4. **State Management**: Maintains connection to DBSnapper configuration and working directory

### Integration Flow
```
AI Application (Claude Desktop)
        ↕ stdin/stdout (JSON-RPC)
DBSnapper MCP Server
        ↕ direct calls
DBSnapper Core Operations
        ↕ database connections
Your Databases
```

### Security Model
- **Credential sanitization**: Automatically removes sensitive information from AI responses
- **Read-only access**: AI cannot modify configuration files directly
- **Operation permissions**: All operations go through DBSnapper's existing security mechanisms
- **No direct database access**: AI cannot bypass DBSnapper's authentication and access controls

## Setup and Configuration

### Prerequisites
- DBSnapper Agent installed and configured
- At least one database target configured
- AI application supporting MCP (like Claude Desktop)

### Claude Desktop Integration

#### Installation Steps
1. **Locate Claude Desktop Configuration**
   - **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`  
   - **Linux**: `~/.config/claude-desktop/claude_desktop_config.json`

2. **Add DBSnapper MCP Server**
   ```json
   {
     "mcpServers": {
       "dbsnapper": {
         "command": "dbsnapper",
         "args": ["mcp"],
         "description": "DBSnapper database snapshot management"
       }
     }
   }
   ```

3. **Restart Claude Desktop** and start a new conversation

#### Advanced Configuration Options

**Full Path Configuration:**
```json
{
  "mcpServers": {
    "dbsnapper": {
      "command": "/usr/local/bin/dbsnapper",
      "args": ["mcp"],
      "description": "DBSnapper Agent MCP Server",
      "env": {
        "DBSNAPPER_CONFIG": "/etc/dbsnapper/config.yml",
        "DBSNAPPER_LOG_LEVEL": "info"
      }
    }
  }
}
```

**Debug Mode Configuration:**
```json
{
  "mcpServers": {
    "dbsnapper": {
      "command": "/path/to/dbsnapper",
      "args": ["mcp"],
      "description": "DBSnapper with debug logging",
      "env": {
        "DBSNAPPER_HTTP_DEBUG": "true",
        "DBSNAPPER_LOG_LEVEL": "debug"
      }
    }
  }
}
```

**Database Tools Configuration:**
```json
{
  "mcpServers": {
    "dbsnapper": {
      "command": "dbsnapper",
      "args": ["mcp"],
      "description": "DBSnapper with PostgreSQL tools",
      "env": {
        "PATH": "/Applications/Postgres.app/Contents/Versions/latest/bin:/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin"
      }
    }
  }
}
```

### Environment Variables

- `DBSNAPPER_CONFIG`: Custom configuration file path
- `DBSNAPPER_WORKING_DIR`: Override working directory location
- `DBSNAPPER_LOG_LEVEL`: Set logging verbosity (debug, info, warn, error)
- `DBSNAPPER_HTTP_DEBUG`: Enable detailed HTTP debugging (true/false)
- `DBSNAPPER_MCP_MODE`: Enable MCP-specific features (automatically set when using mcp command)
- `PATH`: Include database client tool directories

## Available Tools and Resources

### Target Management Tools

#### List Targets (`list_targets`)
Lists all configured database targets with filtering options.

**Parameters:**
- `includeShared` (boolean): Include shared targets from cloud/SSO
- `format` (string): Output format - "json" or "table"

**AI Usage Examples:**
- "Show me all my database targets"
- "List targets including shared ones"
- "Display targets in table format"

#### Target Details (`get_target_details`)
Provides comprehensive information about a specific target.

**Parameters:**
- `targetName` (string, required): Name of the target
- `format` (string): Output format - "json" or "table"

**AI Usage Examples:**
- "Show details for my production database"
- "Get information about the analytics-db target"
- "Display target configuration for staging-env"

### Snapshot Management Tools

#### Build Snapshot (`build_snapshot`)
Creates a new database snapshot for specified target.

**Parameters:**
- `targetName` (string, required): Target to snapshot
- `timestamp` (integer): Custom timestamp for naming
- `format` (string): Output format preference

**AI Usage Examples:**
- "Create a snapshot of my production database"
- "Build a snapshot for the sales-db target"
- "Take a backup of the development environment"

#### Load Snapshot (`load_snapshot`)
Loads a database snapshot into target database.

**Parameters:**
- `targetName` (string, required): Target name
- `snapshotIndex` (integer): Snapshot index (0 for latest)
- `destinationUrl` (string): Override destination URL
- `useOriginal` (boolean): Use original vs sanitized snapshot
- `format` (string): Output format preference

**AI Usage Examples:**
- "Load the latest production snapshot into staging"
- "Restore snapshot index 2 for the test database"
- "Load the original (unsanitized) snapshot for development"

#### List Snapshots (`list_snapshots`)
Shows available snapshots with metadata and statistics.

**Parameters:**
- `targetName` (string): Filter for specific target
- `format` (string): Output format preference

**AI Usage Examples:**
- "Show me all available snapshots"
- "List snapshots for the production target"
- "Display snapshot history with sizes and dates"

### Configuration Management Tools

#### Database Tool Discovery (`discover_database_tools`)
Automatically finds and configures database client tools.

**Parameters:**
- `autoSave` (boolean): Save tool paths to configuration

**AI Usage Examples:**
- "Find and configure my database tools"
- "Discover PostgreSQL and MySQL clients"
- "Set up database tools automatically"

**Key Benefits:**
- Solves "executable file not found in $PATH" errors
- Finds tools like `pg_dump`, `pg_restore`, `psql`, `mysql`, `mysqldump`
- Saves configuration permanently for future use
- No manual PATH configuration required

#### Validate Database Tools (`validate_database_tools`)
Checks current database tools configuration status.

**AI Usage Examples:**
- "Check my database tools setup"
- "Validate PostgreSQL client configuration"
- "Show database tool status"

#### Configuration Information (`get_config_info`)
Provides comprehensive configuration file and setup information.

**Parameters:**
- `format` (string): Output format - "json" or "table"

**AI Usage Examples:**
- "What configuration file is DBSnapper using?"
- "Show my DBSnapper setup information"
- "Display configuration details in JSON format"

### Advanced Operations

#### Sanitize Snapshot (`sanitize_snapshot`)
Creates sanitized snapshot with sensitive data removed.

**Parameters:**
- `targetName` (string, required): Target name
- `snapshotIndex` (integer): Snapshot index to sanitize
- `ephemeral` (boolean): Force ephemeral sanitization mode
- `newSet` (boolean): Create new snapshot set
- `format` (string): Output format preference

**AI Usage Examples:**
- "Sanitize the latest production snapshot"
- "Create a sanitized version for sharing"
- "Remove sensitive data from snapshot index 1"

## MCP Resources

### Configuration Resources

#### `dbsnapper://config`
Provides sanitized DBSnapper configuration including:
- Target configurations (credentials automatically sanitized)
- Storage profiles (sensitive data removed)
- Working directory and settings
- Cloud integration status

#### `dbsnapper://targets`
Lists all configured targets with summary information:
- Target names and types (local, cloud, shared)
- Database engines and connection status
- Snapshot counts and latest activity
- Configuration validation status

### Snapshot Resources

#### `dbsnapper://snapshots`
Comprehensive snapshot information:
- File names, sizes, and creation timestamps
- Original vs sanitized snapshot indicators
- Storage locations (local and cloud)
- Metadata and build statistics

#### `dbsnapper://build-history`
Historical snapshot build information:
- Build success/failure status
- Operation timing and performance metrics
- Error logs and troubleshooting information
- Target usage patterns

## Example Usage Scenarios

### Getting Started with AI Integration

#### Initial Setup and Discovery
```
User: "What database targets do I have configured?"
AI: I'll check your configured database targets.

[AI uses list_targets tool]

AI: You have 3 targets configured:
- production-db (PostgreSQL, cloud target)
- staging-db (PostgreSQL, local target)  
- analytics-db (MySQL, local target)

Would you like to see detailed information about any of these targets?
```

#### Database Tool Setup
```
User: "I'm getting errors about pg_dump not being found"
AI: I can help you set up your database tools automatically.

[AI uses discover_database_tools with autoSave: true]

AI: ✅ Database tools discovered and configured:
- PostgreSQL: pg_dump, pg_restore, psql found at /opt/homebrew/bin/
- MySQL: mysql, mysqldump found at /usr/local/bin/
- Configuration saved to your DBSnapper config file

Your database tools are now properly configured for future operations.
```

### Snapshot Management Workflows

#### Creating Production Snapshots
```
User: "Create a snapshot of my production database and load it into staging"
AI: I'll create a production snapshot and then load it into staging.

[AI uses build_snapshot with targetName: "production-db"]

AI: ✅ Production snapshot created successfully!
- Snapshot file: /snapshots/1705123456_production-db.zip
- Size: 2.1 GB
- Build time: 45.2 seconds

[AI uses load_snapshot with targetName: "production-db", destinationUrl from staging config]

AI: ✅ Snapshot loaded into staging environment!
- Loaded sanitized snapshot (index 0)
- Destination: staging-db  
- Load time: 12.3 seconds

Your staging environment now has the latest production data.
```

#### Snapshot History and Analysis
```
User: "Show me the history of snapshots for my production database"
AI: I'll get the snapshot history for your production database.

[AI uses list_snapshots with targetName: "production-db"]

AI: Here's your production-db snapshot history:
- Index 0 (latest): 1705123456_production-db.zip (2.1 GB, sanitized available)
- Index 1: 1705037056_production-db.zip (2.0 GB, sanitized available)  
- Index 2: 1704950656_production-db.zip (1.9 GB, sanitized available)

All snapshots have sanitized versions available for safe sharing.
Would you like to load any specific snapshot?
```

### Configuration Management

#### Configuration Validation
```
User: "Is my DBSnapper configuration set up correctly?"
AI: I'll check your DBSnapper configuration and setup.

[AI uses get_config_info and validate_database_tools]

AI: Your DBSnapper configuration looks good:
✅ Configuration file: ~/.config/dbsnapper/dbsnapper.yml
✅ Working directory: /Users/you/.dbsnapper  
✅ Database tools: PostgreSQL and MySQL tools configured
✅ Cloud mode: Enabled with valid authentication
✅ Targets: 3 configured (2 local, 1 cloud)
✅ Storage profiles: 1 S3 profile configured

Everything is properly configured for database snapshot operations.
```

#### Multi-Environment Setup
```
User: "Help me set up snapshots for my development workflow"
AI: I'll help you create an efficient development snapshot workflow.

[AI uses list_targets to understand environment]

AI: I see you have production-db and staging-db targets. Here's a recommended workflow:

1. Create daily production snapshots (sanitized for security)
2. Load sanitized snapshots into staging for testing
3. Use sanitized data for development environments

Would you like me to create a sanitized snapshot from production now?

[User agrees]

[AI uses build_snapshot then sanitize_snapshot]

AI: ✅ Development-ready data prepared:
- Original production snapshot: 1705123456_production-db.zip
- Sanitized version: 1705123456_production-db.san.zip  
- Sensitive data removed, ready for development use

You can now safely use this data across development environments.
```

### Advanced Workflows

#### Automated Testing Setup
```
User: "I need to set up automated testing with fresh data each day"
AI: I'll help you understand how to set up automated testing with fresh snapshots.

[AI uses get_config_info and list_snapshots]

AI: Based on your configuration, here's an automated testing approach:

Current setup:
- Production snapshots: Available with sanitization
- Test environments: Configured for automated loading
- Database tools: Properly configured

Recommended daily workflow:
1. Build fresh production snapshot (automated via CI/CD)
2. Sanitize snapshot for test data safety  
3. Load sanitized snapshot into test databases
4. Run automated test suite

The MCP integration can be used with GitHub Actions or other CI/CD tools.
Would you like me to help set up any part of this workflow?
```

#### Data Sharing and Collaboration
```
User: "I need to share database data with my team, but it contains sensitive information"
AI: I can help you create a sanitized snapshot that's safe for team sharing.

[AI uses list_snapshots to see available data]

AI: I see you have recent production snapshots. I'll create a sanitized version for safe sharing.

[AI uses sanitize_snapshot with appropriate configuration]

AI: ✅ Team-safe snapshot created:
- Original: 1705123456_production-db.zip (contains sensitive data)
- Sanitized: 1705123456_production-db.san.zip (safe for team use)
- Sanitization: PII removed, emails masked, IDs preserved for referential integrity

The sanitized snapshot can be safely shared with your team or loaded into shared development environments. The data structure and relationships are preserved while sensitive information is protected.
```

## Troubleshooting

### Common Setup Issues

#### MCP Server Not Starting
```bash
# Test MCP server directly
dbsnapper mcp --help

# Start MCP server with debug output
DBSNAPPER_LOG_LEVEL=debug dbsnapper mcp

# Check if binary is accessible
which dbsnapper
ls -la /path/to/dbsnapper
```

**Solutions:**
1. Verify DBSnapper binary exists and is executable
2. Check configuration file exists and is readable
3. Ensure proper permissions on binary and config files
4. Use absolute paths in Claude Desktop configuration

#### Claude Desktop Integration Issues
```json
// Verify JSON syntax is correct
{
  "mcpServers": {
    "dbsnapper": {
      "command": "/full/path/to/dbsnapper",
      "args": ["mcp"],
      "description": "DBSnapper database management"
    }
  }
}
```

**Solutions:**
1. Validate JSON syntax in configuration file
2. Use absolute path to DBSnapper binary
3. Restart Claude Desktop completely after configuration changes
4. Check Claude Desktop logs for error messages

#### Database Tools Not Found
```
Error: "pg_dump": executable file not found in $PATH
```

**Recommended Solution: Use MCP Tool Discovery**
```
AI Prompt: "Find and configure my database tools automatically"
```

**Alternative: Manual PATH Configuration**
```json
{
  "mcpServers": {
    "dbsnapper": {
      "command": "dbsnapper",
      "args": ["mcp"],
      "env": {
        "PATH": "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin"
      }
    }
  }
}
```

### Configuration Issues

#### Configuration File Not Found
```bash
# Check default configuration location
ls -la ~/.config/dbsnapper/dbsnapper.yml

# Initialize configuration if missing
dbsnapper config init

# Use custom configuration path
export DBSNAPPER_CONFIG=/path/to/custom/config.yml
```

#### Permission Issues
```bash
# Fix binary permissions
chmod +x /path/to/dbsnapper

# Fix configuration file permissions
chmod 600 ~/.config/dbsnapper/dbsnapper.yml

# Check working directory permissions
ls -la ~/.dbsnapper/
```

### AI Integration Issues

#### AI Can't Execute Tools
**Symptoms:**
- AI says tools are not available
- Operations fail with "tool not found" errors

**Solutions:**
1. Restart Claude Desktop after configuration changes
2. Verify MCP server configuration in Claude Desktop settings
3. Test MCP server independently: `dbsnapper mcp --debug`
4. Check for configuration file syntax errors

#### Operations Fail or Timeout
**Symptoms:**
- Database operations hang or timeout
- "Connection refused" or similar database errors

**Solutions:**
1. Test database connectivity independently
2. Verify target configurations are correct
3. Check network connectivity and firewall settings
4. Ensure database credentials are valid

#### Inconsistent Results
**Symptoms:**
- Same operation produces different results
- Configuration appears to change between runs

**Solutions:**
1. Use absolute paths in all configurations
2. Set explicit environment variables
3. Check for multiple configuration files
4. Verify working directory consistency

### Debugging and Diagnostics

#### Enable Debug Mode
```json
{
  "mcpServers": {
    "dbsnapper": {
      "command": "dbsnapper",
      "args": ["mcp"],
      "env": {
        "DBSNAPPER_LOG_LEVEL": "debug",
        "DBSNAPPER_HTTP_DEBUG": "true"
      }
    }
  }
}
```

#### Test MCP Server Independently
```bash
# Start MCP server with debug output
DBSNAPPER_LOG_LEVEL=debug dbsnapper mcp

# Test specific operations
dbsnapper targets --help
dbsnapper build --help
dbsnapper load --help
```

#### Verify Configuration
```bash
# Validate configuration file
dbsnapper config validate

# Check configuration information
dbsnapper config check

# Test database connectivity
dbsnapper targets
```

## Performance Considerations

### Memory Usage
- **Snapshot operations**: MCP server uses same memory as direct DBSnapper operations
- **Tool discovery**: Minimal memory overhead for tool registry
- **Resource caching**: Configuration and metadata cached for performance

### Response Times
- **Simple queries**: Near-instantaneous (target lists, configuration info)
- **Snapshot operations**: Same performance as direct DBSnapper commands
- **Large snapshots**: Progress may not be visible to AI during long operations

### Concurrent Operations
- **Single operation limit**: MCP server processes one operation at a time
- **AI request queuing**: Multiple AI requests are queued and processed sequentially
- **Database connections**: Respects DBSnapper's existing connection management

## Security Considerations

### Data Protection
- **Automatic sanitization**: Sensitive configuration data automatically removed from AI responses
- **Credential isolation**: Database credentials never exposed to AI applications
- **Read-only metadata**: AI can access metadata but cannot modify sensitive configurations
- **Snapshot security**: Original vs sanitized snapshot handling preserved

### Access Control
- **Operation permissions**: All operations subject to DBSnapper's existing permissions
- **Configuration access**: AI cannot modify configuration files directly
- **Network isolation**: MCP server uses stdio communication only (no network ports)
- **Audit trail**: All operations logged through DBSnapper's logging system

### Best Practices
1. **Use sanitized snapshots**: Default to sanitized snapshots for AI-assisted operations
2. **Monitor operations**: Review MCP server logs for unusual activity
3. **Limit scope**: Configure only necessary targets for AI access
4. **Regular updates**: Keep DBSnapper and AI applications updated
5. **Configuration security**: Maintain restrictive permissions on configuration files

!!! tip "Quick Start"
    The fastest way to get started is to add DBSnapper to your Claude Desktop configuration, restart Claude, and ask: "What database targets do I have?" This will verify the integration is working correctly.

!!! note "Tool Discovery"
    If you encounter "executable file not found" errors, use the AI prompt "Find and configure my database tools automatically" to resolve PATH issues permanently.

!!! warning "Security"
    While DBSnapper automatically sanitizes configuration data in AI responses, always review snapshot contents before sharing or using in development environments.

## Related Commands

- [`config`](config.md) - Configuration management (used by MCP for setup validation)
- [`targets`](targets.md) - Target management (exposed as MCP tool)
- [`build`](build.md) - Snapshot creation (exposed as MCP tool)
- [`load`](load.md) - Snapshot loading (exposed as MCP tool)
- [`sanitize`](sanitize.md) - Data sanitization (exposed as MCP tool)

## See Also

- [Model Context Protocol](https://modelcontextprotocol.io/) - Official MCP specification
- [Claude Desktop](https://claude.ai/desktop) - Primary AI application supporting MCP integration