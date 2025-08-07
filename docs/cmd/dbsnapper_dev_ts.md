---
title: "dbsnapper dev ts"
description: "DBSnapper Agent Command Reference for 'dbsnapper dev ts'"
weight: 100
---

# dbsnapper dev ts

## dbsnapper dev ts

Add timestamp table to target database

### Synopsis

Add a timestamp table to the target database for end-to-end testing verification.

This command creates a dbsnapper_info table with metadata including:
- created_at: Current timestamp  
- tags: Array containing target name, test name, query name, and location

This is used to verify end-to-end tests by checking that the timestamp 
table exists with the expected metadata after operations.

Target Support:
- Local targets: Defined in local configuration files
- Cloud targets: Retrieved from DBSnapper cloud API
- Shared targets: Accessible via Okta SSO integration

Database Selection:
- By default, timestamps the destination database (dst_url)
- Use --src_url to explicitly timestamp the source database instead
- Database URL determination follows the same logic as load/sanitize commands

Examples:
  # Local target with default destination database
  dbsnapper dev ts my-local-target --test-name e2e-build
  
  # Cloud target with source database timestamping
  dbsnapper dev ts my-cloud-target --src_url --test-name e2e-build
  
  # With additional metadata tags
  dbsnapper dev ts my-target --test-name e2e-build --query-name san_query_override --location cloud


```
dbsnapper dev ts <target_name> [flags]
```

### Options

```
  -h, --help                help for ts
      --json                Output result as JSON
      --location string     Location to include in tags (default: cloud) (default "cloud")
      --query-name string   Query name to include in tags (optional)
      --src_url             Timestamp the source database (src_url) instead of destination (dst_url)
      --test-name string    Test name to include in tags (optional)
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper dev](dbsnapper_dev.md)	 - Development utilities (requires feature flag)

