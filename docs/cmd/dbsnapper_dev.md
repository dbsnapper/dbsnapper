---
title: "dbsnapper dev"
description: "DBSnapper Agent Command Reference for 'dbsnapper dev'"
weight: 100
---

# dbsnapper dev

## dbsnapper dev

Development utilities (requires feature flag)

### Synopsis

Development utilities for DBSnapper.

These commands are intended for development and testing purposes only.
They require the feature_flags.development_utilities configuration to be enabled.

Available subcommands:
  ts                 Add timestamp table to target database for end-to-end testing
  assert-ts          Validate timestamp table exists with expected values
  schemas            List database schemas for target validation
  validate-sanitize  Validate that sanitization was applied to target database


### Options

```
  -h, --help   help for dev
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper](dbsnapper.md)	 - Simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.
* [dbsnapper dev assert-ts](dbsnapper_dev_assert-ts.md)	 - Compare timestamp between source and destination databases
* [dbsnapper dev schemas](dbsnapper_dev_schemas.md)	 - List database schemas for target
* [dbsnapper dev ts](dbsnapper_dev_ts.md)	 - Add timestamp table to target database
* [dbsnapper dev validate-sanitize](dbsnapper_dev_validate-sanitize.md)	 - Validate that sanitization was applied to a target database

