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

**Examples:**

```bash
# Start MCP server with HTTP transport (default)
dbsnapper mcp

# HTTP transport on custom port
dbsnapper mcp --port 3001

# HTTP transport with custom path
dbsnapper mcp --path /api/mcp

# Start with Server-Sent Events transport
dbsnapper mcp --transport sse

# Use stdio transport (legacy mode)
dbsnapper mcp --stdio

# Start with debug logging
DBSNAPPER_DEBUG=true dbsnapper mcp
```

## Arguments

None - the MCP server runs without additional arguments.

## Options

```bash
  -h, --help                     help for mcp
      --path string             HTTP path (for http/sse transport) (default "/mcp")
  -p, --port int                HTTP port (for http/sse transport) (default 8080)
      --stdio                   Use stdio transport (legacy mode for Claude Desktop)
  -t, --transport string        Transport type: http, sse, or stdio (default "http")

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

1. **Multiple Transport Options**: HTTP (default), Server-Sent Events (SSE), or stdio communication
2. **Tool Registry**: Exposes DBSnapper commands as callable tools
3. **Resource System**: Provides structured access to configuration and data
4. **State Management**: Maintains connection to DBSnapper configuration and working directory

### Integration Flow

```
AI Application / Web Client
        ↕ HTTP/SSE/stdio (JSON-RPC)
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

### Integration Options

#### HTTP Integration (Default - Recommended)

The MCP server now defaults to HTTP transport, making it accessible from web applications, APIs, and other HTTP clients:

```bash
# Start HTTP server (default)
dbsnapper mcp

# Custom port and path
dbsnapper mcp --port 3001 --path /api/mcp

# Server-Sent Events transport
dbsnapper mcp --transport sse
```

**HTTP Access:**

- MCP endpoint: `http://localhost:8080/mcp`
- Server info: `http://localhost:8080/`

#### Claude Desktop Integration (Stdio - Recommended)

**Claude Desktop currently works best with stdio transport** for local MCP servers:

1. **Locate Claude Desktop Configuration**

   - **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
   - **Linux**: `~/.config/claude-desktop/claude_desktop_config.json`

2. **Add DBSnapper MCP Server (Stdio Mode)**

   ```json
   {
     "mcpServers": {
       "dbsnapper": {
         "command": "dbsnapper",
         "args": ["mcp", "--stdio"],
         "description": "DBSnapper database snapshot management",
         "env": {
           "DBSNAPPER_CONFIG": "/Users/snappy/.config/dbsnapper/dbsnapper.yml"
         }
       }
     }
   }
   ```

3. **Restart Claude Desktop** and start a new conversation

#### Claude Code Integration (HTTP - Recommended)

**Claude Code supports HTTP transport with streaming capabilities** for local MCP servers:

1. **Start DBSnapper HTTP server**: `dbsnapper mcp` (runs on `http://localhost:8080/mcp`)

2. **Create Claude Code MCP configuration** (`.mcp.json` in your project root):

   ```json
   {
     "mcpServers": {
       "dbsnapper": {
         "command": "dbsnapper",
         "args": ["mcp"],
         "env": {}
       }
     }
   }
   ```

3. **Start using DBSnapper in Claude Code** with natural language commands

**Benefits:**

- Real-time HTTP streaming support
- Better performance for complex operations
- Built-in debugging capabilities
- Web-standard protocols

#### Streamable HTTP with mcp-remote Adapter

**For clients that don't yet support native Streamable HTTP**, you can use the `mcp-remote` adapter to connect any MCP client to DBSnapper's HTTP server:

**What is mcp-remote?**
The `mcp-remote` adapter enables MCP clients like Claude Desktop (which otherwise only support local connections) to work with remote MCP servers using the new Streamable HTTP transport.

**Setup Instructions:**

1. **Install mcp-remote adapter**:

   ```bash
   npm install -g mcp-remote
   ```

2. **Start DBSnapper HTTP server**:

   ```bash
   dbsnapper mcp  # runs on http://localhost:8080/mcp
   ```

3. **Configure your MCP client with mcp-remote**:
   ```json
   {
     "mcpServers": {
       "dbsnapper-remote": {
         "command": "npx",
         "args": ["mcp-remote", "http://localhost:8080/mcp"],
         "description": "DBSnapper via Streamable HTTP adapter"
       }
     }
   }
   ```

**Benefits:**

- Enables Streamable HTTP for any MCP client
- Better performance than traditional stdio transport
- Future-proof as clients adopt native Streamable HTTP
- Real-time streaming capabilities

**Use Cases:**

- Testing Streamable HTTP with Claude Desktop
- Connecting legacy MCP clients to modern HTTP servers
- Bridge between traditional MCP and new HTTP transport

#### Advanced Configuration Options

**Full Path Configuration (Stdio - Recommended for Claude Desktop):**

```json
{
  "mcpServers": {
    "dbsnapper": {
      "command": "/usr/local/bin/dbsnapper",
      "args": ["mcp", "--stdio"],
      "description": "DBSnapper Agent MCP Server",
      "env": {
        "DBSNAPPER_CONFIG": "/Users/snappy/.config/dbsnapper/dbsnapper.yml"
      }
    }
  }
}
```

**Debug Mode Configuration (Stdio):**

```json
{
  "mcpServers": {
    "dbsnapper": {
      "command": "/path/to/dbsnapper",
      "args": ["mcp", "--stdio"],
      "env": {}
    }
  }
}
```

### Transport Methods

#### Stdio Transport (Recommended for Claude Desktop)

**Best for:** Claude Desktop, legacy direct AI assistant integration

**Benefits:**

- Direct process-based communication
- No network ports or certificates needed
- Native Claude Desktop support
- Lower resource usage
- Simple configuration

**Usage:**

```bash
# Stdio transport for Claude Desktop
dbsnapper mcp --stdio
```

#### HTTP Transport (Default - Best for APIs/Web/Claude Code)

**Best for:** Claude Code, web applications, APIs, development integrations, debugging

**Benefits:**

- Accessible from any HTTP client
- RESTful endpoints for integration
- Better debugging and monitoring
- Easy testing with curl or Postman
- Future-proof for remote MCP servers

**Usage:**

```bash
# Default HTTP transport
dbsnapper mcp

# Custom port and endpoint
dbsnapper mcp --port 3001 --path /api/dbsnapper
```

**Claude Desktop Usage (Experimental):**

Requires `mcp-remote` adapter: `npx mcp-remote http://localhost:8080/mcp`

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

- **[AI Integrations Overview](../ai-integrations.md)** - Full AI capabilities and future roadmap
