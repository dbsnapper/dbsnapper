---
title: "dbsnapper dev validate-sanitize"
description: "DBSnapper Agent Command Reference for 'dbsnapper dev validate-sanitize'"
weight: 100
---

# dbsnapper dev validate-sanitize

## dbsnapper dev validate-sanitize

Validate that sanitization was applied to a target database

### Synopsis

Validate that sanitization was applied by checking for specific metadata in the sanitized database.

This command connects to the destination database (after sanitization) and validates that:
1. The expected metadata table exists
2. The metadata contains the expected sanitization markers
3. The sanitization process completed successfully

Examples:
  # Basic validation using default metadata table
  dbsnapper dev validate-sanitize test-san-pg
  
  # Validate specific table and column
  dbsnapper dev validate-sanitize test-san-pg --table dbsnapper_info --column tags --contains "test_san_pg.sql"
  
  # Output in JSON format
  dbsnapper dev validate-sanitize test-san-pg --json
  
  # Validate against source database instead of destination
  dbsnapper dev validate-sanitize test-san-pg --connection-type src_url

```
dbsnapper dev validate-sanitize [target] [flags]
```

### Options

```
      --column string            Column to check for sanitization metadata (default "tags")
      --connection-type string   Database connection to use: dst_url (default) or src_url (default "dst_url")
      --contains string          Value that the metadata column should contain (if empty, uses target name)
  -h, --help                     help for validate-sanitize
      --json                     Output results in JSON format
      --table string             Table to check for sanitization metadata (default "dbsnapper_info")
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper dev](dbsnapper_dev.md)	 - Development utilities (requires feature flag)

