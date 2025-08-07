---
title: "dbsnapper dev assert-ts"
description: "DBSnapper Agent Command Reference for 'dbsnapper dev assert-ts'"
weight: 100
---

# dbsnapper dev assert-ts

## dbsnapper dev assert-ts

Compare timestamp between source and destination databases

### Synopsis

Validates that the timestamp in the dbsnapper_info table matches between 
the target's src_url and dst_url databases.

This command:
- Reads the dbsnapper_info table from both source and destination databases
- Compares timestamps and tags between the two
- Reports whether the data was properly preserved during snapshot/load operations

Target Support:
- Local targets: Defined in local configuration files
- Cloud targets: Retrieved from DBSnapper cloud API  
- Shared targets: Accessible via Okta SSO integration (read-only comparison)

Database URL Resolution:
- Source database: Always uses target's src_url (required for comparison)
- Destination database: Uses same URL determination logic as load command
- Supports configuration overrides and shared target defaults

Examples:
  # Basic workflow with local target
  dbsnapper dev ts my-local-target --test-name "e2e-build-123"
  dbsnapper build my-local-target
  dbsnapper load my-local-target  
  dbsnapper dev assert-ts my-local-target
  
  # Cloud target comparison
  dbsnapper dev assert-ts my-cloud-target --json
  
  # Shared target validation (read-only)
  dbsnapper dev assert-ts shared-target-name


```
dbsnapper dev assert-ts <target_name> [flags]
```

### Options

```
  -h, --help   help for assert-ts
      --json   Output result as JSON
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper dev](dbsnapper_dev.md)	 - Development utilities (requires feature flag)

