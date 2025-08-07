---
title: "dbsnapper dev schemas"
description: "DBSnapper Agent Command Reference for 'dbsnapper dev schemas'"
weight: 100
---

# dbsnapper dev schemas

## dbsnapper dev schemas

List database schemas for target

### Synopsis

List all database schemas for the specified target.

This command connects to the target database and retrieves a list of all
available schemas. This is useful for validating schema filtering operations
and ensuring that only expected schemas are present after build/load operations.

Target Support:
- Local targets: Defined in local configuration files
- Cloud targets: Retrieved from DBSnapper cloud API
- Shared targets: Accessible via Okta SSO integration

Database Selection:
- By default, inspects the destination database (dst_url)
- Use --src_url to explicitly inspect the source database instead
- Database URL determination follows the same logic as load/sanitize commands

Database Support:
- PostgreSQL: Queries information_schema.schemata
- MySQL: Queries information_schema.schemata

Output Format:
- JSON format includes success status, target name, schema list, and connection info
- Schemas are returned in alphabetical order for consistent comparison
- Use --json flag for programmatic access (recommended for e2e tests)

Examples:
  # List schemas in destination database
  dbsnapper dev schemas my-target --json
  
  # List schemas in source database
  dbsnapper dev schemas my-target --src_url --json
  
  # Example JSON output:
  {
    "success": true,
    "target": "my-target",
    "schemas": ["analytics", "public", "reporting"],
    "database_type": "postgresql",
    "connection_info": {
      "url_type": "destination",
      "description": "destination database (dst_url)"
    }
  }

E2E Test Integration:
This command is designed for use in end-to-end testing to validate schema
filtering operations. Test scripts can parse the JSON output to verify
that only expected schemas are present after build/load operations.


```
dbsnapper dev schemas <target_name> [flags]
```

### Options

```
  -h, --help      help for schemas
      --json      Output result as JSON (recommended for programmatic use)
      --src_url   Inspect the source database (src_url) instead of destination (dst_url)
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper dev](dbsnapper_dev.md)	 - Development utilities (requires feature flag)

