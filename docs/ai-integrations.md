# AI Integrations

!!! success "New in v3.0.0-pr1 Preview"

    DBSnapper v3.0.0-pr1 introduces AI integration capabilities through Model Context Protocol (MCP) support. This preview release demonstrates the core AI functionality with comprehensive MCP server integration for AI assistants.

## Overview

DBSnapper's AI integration capabilities enable developers and teams to interact with their database snapshots, targets, and configurations through natural language conversations with AI assistants. Instead of memorizing command-line syntax, you can simply tell your AI assistant what you want to accomplish.

**Key Benefits:**

- **Natural Language Interface**: Interact with DBSnapper using plain English
- **Include in Workflows**: Seamlessly integrate dbsnapper operations into your AI workflows
- **Intelligent Discovery**: AI assistants can help discover and configure database tools automatically
- **Context-Aware Operations**: AI can understand your workflow and suggest appropriate actions
- **Enhanced Productivity**: Faster snapshot operations through conversational interfaces
- **Error Prevention**: AI validation helps catch configuration issues before they cause problems

## Model Context Protocol (MCP) Integration

The Model Context Protocol provides a standardized way for AI applications to access external tools and data sources. DBSnapper's MCP server exposes comprehensive functionality through this protocol.

### Supported AI Applications

- **Claude Desktop** - Anthropic's desktop AI application
- **Claude Code** - Anthropic's VS Code-integrated AI assistant
- **Any MCP-compatible AI client** - The standard protocol ensures broad compatibility

### Core Capabilities

#### 🎯 **Target Management**

- List and explore all configured database targets
- Get detailed information about specific targets including snapshots
- Safely update target configurations with validation
- Access sanitized configuration data

#### 📸 **Snapshot Operations**

- Create new database snapshots with natural language commands
- Load snapshots into target databases
- Browse available snapshots with rich metadata
- Access snapshot creation history and statistics

#### 🔒 **Sanitization & Security**

- Create sanitized snapshots with sensitive data removed
- Validate configurations including sanitization queries
- Access sanitization templates for common use cases
- Real-time validation with detailed error reporting

#### 🛠️ **Database Tools Management**

- Automatically discover database client tools (pg_dump, mysql, etc.)
- Validate database tools configuration
- Resolve PATH and configuration issues automatically

### Quick Start

1. **Install DBSnapper v3.0+** with MCP support
2. **Configure your AI application** to use the DBSnapper MCP server
3. **Start conversing** with your AI assistant about database operations

#### Claude Desktop Configuration

For Claude Desktop, use dbsnapper in stdio mode. Add the following configuration:

```json
{
  "mcpServers": {
    "dbsnapper": {
      "command": "dbsnapper",
      "args": ["mcp", "--stdio"],
      "env": {
        "DBSNAPPER_CONFIG": "/Users/snappy/.config/dbsnapper/dbsnapper.yml"
      }
    }
  }
}
```

#### Claude Code Configuration

For Claude Code, use dbsnapper in http streaming mode (default). Use the following configuration:

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

Start the dbsnapper MCP server:

```bash
dbsnapper mcp
```

### Example Conversations

#### Getting Started

!!! quote "What database targets do I have configured?"

    **AI:**

    I'll check your database targets using the DBSnapper tool to see what targets you have configured.

    You have **15 database targets** configured in DBSnapper. Here's a summary of your targets:

    <h2>Target Overview</h2>

    **Database Types:**

    - **PostgreSQL**: 14 targets
    - **MySQL**: 1 target (myd-ghost)

    **Storage Profiles:**

    - **r2**: 9 targets
    - **s3**: 2 targets
    - **No storage profile**: 4 targets

    <h2>Key Targets by Purpose</h2>

    **Production/Live Systems:**

    - `tprofile-prod-local` - Twitter profile production system (4 snapshots)
    - `daisy_mastodon` - Mastodon instance (4 snapshots)
    - `myd-ghost` - Ghost blog MySQL database (2 snapshots)

    **Development/Testing:**

    - `dvdrental-cloud` - Most active with 28 snapshots
    - `dvdrental-san` - 16 snapshots
    - `app_dbsnapper_com_development` - DBSnapper app development

    **Data Migration/Syncing:**

    - `pg-remote2docker` - Remote to Docker migration
    - `pg-docker2host` - Docker to host migration
    - `pg-docker2host-local` - Local Docker migrations

    The `dvdrental` targets appear to be your primary testing databases, with `dvdrental-cloud` being the most actively used (28 snapshots).

    Would you like me to get more details about any specific target or check the snapshots for a particular database?

#### Get Target Details

!!! quote "Get the target details for dvdrental-cloud"

    **AI:**

    Here are the details for your `dvdrental-cloud` target:

    <h2>Target Configuration</h2>

    - **Type**: Cloud target (PostgreSQL)
    - **Source**: `localhost:15432/dvdrental`
    - **Destination**: `localhost:5432/dvdrental_snap`
    - **Storage**: R2 cloud storage
    - **Total Snapshots**: 28

    <h2>Snapshot History</h2>

    **Recent Activity (2025):**

    - **Latest snapshot**: July 25, 2025 at 8:25 PM (613,644 bytes)
    - **Previous**: July 25, 2025 at 1:10 PM (613,646 bytes)
    - **Earlier**: May 19, 2025 at 7:12 PM (613,514 bytes)

    **Historical Activity:**

    - **2024**: Most activity was in May 2024 (16 snapshots) with heavy testing on May 15th
    - **Earlier snapshots**: March 2024 and August 2024

    <h2>Key Observations</h2>

    1. **Consistent Size**: Most snapshots are around 613KB, indicating a stable database size
    2. **Testing Pattern**: Heavy snapshot activity on May 15, 2024 suggests extensive testing/development
    3. **Recent Usage**: Two snapshots in July 2025 show recent activity
    4. **Storage Location**: All snapshots are stored in cloud (R2), none local
    5. **File Format**: All snapshots are original type (no sanitized versions)

    This appears to be your primary test database for DBSnapper development and testing, given the high number of snapshots and consistent usage pattern over time.

    Would you like me to show you details about any other targets or perform operations on this target?

#### Creating a snapshot

!!! quote "Create a snapshot for dvdrental-cloud"

    **AI:**

    ✅ **Snapshot created successfully!**

    <h2>Snapshot Details</h2>

    - **Target**: dvdrental-cloud
    - **Local file**: `/Users/snappy/.dbsnapper/1755314735_dvdrental-cloud.zip`
    - **Cloud storage**: `fc6dcab5-7f68-4c01-b6f8-629f8215baa5.zip`
    - **File size**: 613,560 bytes (~599 KB)
    - **Build duration**: ~203ms (very fast!)
    - **Timestamp**: 1755314735 (August 15, 2025)

    The snapshot has been successfully created and uploaded to your R2 cloud storage. This is now your **29th snapshot** for the dvdrental-cloud target. The file size (613,560 bytes) is consistent with your other recent snapshots, indicating the database structure and data remain stable.

    The snapshot is available both locally and in the cloud, so you can load it to your destination database or use it for testing purposes whenever needed.

## Available MCP Tools

### Target Management

- **`list_targets`** - List all configured database targets
- **`get_target_details`** - Get detailed target information with snapshots
- **`update_target`** - Safely update target configurations (dry-run supported)

### Snapshot Operations

- **`build_snapshot`** - Create new database snapshots
- **`load_snapshot`** - Load snapshots into target databases
- **`list_snapshots`** - Browse available snapshots with metadata

### Sanitization & Security

- **`sanitize_snapshot`** - Create sanitized snapshots with sensitive data removed
- **`validate_config`** - Validate targets, queries, and database connections

### Database Tools

- **`discover_database_tools`** - Auto-discover and configure database client tools
- **`validate_database_tools`** - Validate database tools configuration
- **`get_config_info`** - Get comprehensive configuration information

## MCP Resources

DBSnapper provides rich contextual information through MCP resources:

### Configuration Resources

- **`dbsnapper://config`** - Sanitized configuration including targets and profiles
- **`dbsnapper://targets`** - Target list with summary statistics
- **`dbsnapper://validation-results`** - Real-time validation status

### Snapshot Resources

- **`dbsnapper://snapshots`** - Comprehensive snapshot information with metadata
- **`dbsnapper://build-history`** - Snapshot creation history with success/failure details

### Sanitization Resources

- **`dbsnapper://sanitization-queries`** - Query configuration with execution status
- **`dbsnapper://query-templates`** - Common sanitization templates (PII, GDPR, etc.)

## Available in v3.0.0-pr1

**✅ Model Context Protocol (MCP) Server**

- Full MCP server implementation with HTTP and SSE transport
- Complete tool exposure for all core DBSnapper operations
- Resource access for configuration and snapshot data
- Compatible with Claude Desktop, Claude Code, and other MCP clients

**✅ Core AI Operations**

- Target management through natural language
- Snapshot creation, loading, and sanitization
- Configuration validation and database tool discovery
- Real-time error handling and troubleshooting assistance

Ready to embrace AI-powered database management? Here's how to get started:

1. **[Install DBSnapper v3.0+](installation.md)** with the latest AI features
2. **[Follow the MCP Setup Guide](commands/mcp.md)** for detailed configuration
3. **[Explore Example Workflows](articles/creating-sanitized-database-snapshots-with-dbsnapper.md)** to see AI integration in action
4. **[Join the Community](https://github.com/joescharf/dbsnapper-agent)** to share feedback and use cases

---

!!! tip "Ready to Experience AI-Powered Database Management?"

    DBSnapper v3.0's AI integration represents just the beginning. We're building toward a future where managing database snapshots is as natural as having a conversation. [Get started today](installation.md) and be part of the AI-powered database revolution.
