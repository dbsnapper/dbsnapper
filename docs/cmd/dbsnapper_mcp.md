---
title: "dbsnapper mcp"
description: "DBSnapper Agent Command Reference for 'dbsnapper mcp'"
weight: 100
---

# dbsnapper mcp

## dbsnapper mcp

Start the MCP (Model Context Protocol) server

### Synopsis

Start the MCP server to enable integration with LLM applications.

The MCP server provides standardized access to DBSnapper functionality through
the Model Context Protocol, allowing LLM applications like Claude Code to
interact with database snapshots, targets, and other operations.

The server provides multiple transport methods and includes:

- Tools: Execute DBSnapper operations (list targets, build snapshots, etc.)
- Resources: Access configuration and snapshot information
- Prompts: Templates for common database operations

Transport Options:
  dbsnapper mcp                    # Start HTTP server (default) on port 8080
  dbsnapper mcp --port 3001        # HTTP server on custom port
  dbsnapper mcp --transport sse    # Server-Sent Events transport
  dbsnapper mcp --stdio            # Legacy stdio transport for Claude Code
  
Examples:
  dbsnapper mcp                           # HTTP transport (default)
  dbsnapper mcp --port 3001 --path /api   # HTTP with custom port/path
  dbsnapper mcp --transport sse           # SSE transport
  dbsnapper mcp --stdio                   # Stdio for Claude Desktop/Code
  DBSNAPPER_DEBUG=true dbsnapper mcp      # Debug logging (any transport)
  
Integration with Claude Desktop/Code (Stdio - Recommended):
  1. Add to Claude configuration:
     {
       "mcpServers": {
         "dbsnapper": {
           "command": "dbsnapper",
           "args": ["mcp", "--stdio"],
           "description": "DBSnapper database snapshot management"
         }
       }
     }
  2. Restart Claude Desktop/Code

HTTP integration (experimental - requires mcp-remote adapter):
  1. Start DBSnapper HTTP server: dbsnapper mcp
  2. Install mcp-remote: npm install -g mcp-remote
  3. Add to Claude configuration:
     {
       "mcpServers": {
         "dbsnapper": {
           "command": "npx",
           "args": ["mcp-remote", "http://localhost:8080/mcp"],
           "description": "DBSnapper via HTTP (experimental)"
         }
       }
     }


```
dbsnapper mcp [flags]
```

### Options

```
  -h, --help                     help for mcp
      --path string             HTTP endpoint path (default "/mcp")
      --port int                HTTP server port (default 8080)
      --stdio                   Use stdio transport (legacy mode for Claude Code)
  -t, --transport string        Transport type: http, sse, or stdio (default "http")
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper](dbsnapper.md)	 - Simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.

