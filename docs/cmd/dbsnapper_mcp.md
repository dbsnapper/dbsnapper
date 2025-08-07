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

The server communicates over stdin/stdout and provides:

- Tools: Execute DBSnapper operations (list targets, build snapshots, etc.)
- Resources: Access configuration and snapshot information
- Prompts: Templates for common database operations

Examples:
  dbsnapper mcp                    # Start MCP server with stdio transport
  DBSNAPPER_DEBUG=true dbsnapper mcp     # Start with debug logging
  
Integration with Claude Code:
  1. Add to ~/.config/claude-code/config.json:
     {
       "mcpServers": {
         "dbsnapper": {
           "command": "dbsnapper",
           "args": ["mcp"],
           "description": "DBSnapper database snapshot management"
         }
       }
     }
  2. Restart Claude Code
  3. Use DBSnapper tools in conversations


```
dbsnapper mcp [flags]
```

### Options

```
  -h, --help   help for mcp
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper](dbsnapper.md)	 - Simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.

