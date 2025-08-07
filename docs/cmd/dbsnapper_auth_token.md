---
title: "dbsnapper auth token"
description: "DBSnapper Agent Command Reference for 'dbsnapper auth token'"
weight: 100
---

# dbsnapper auth token

## dbsnapper auth token

Add DBSnapper Cloud Api Token

### Synopsis

This comand adds the DBSnapper Cloud API token to your configuration file. 
	You can find this token on the Get Started page of the DBSnapper Cloud app.

```
dbsnapper auth token <api_token> [flags]
```

### Examples

```
dbsnapper auth token hNsdelkjcxgoiuwalkmcgoi...
```

### Options

```
  -h, --help   help for token
```

### Options inherited from parent commands

```
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

### SEE ALSO

* [dbsnapper auth](dbsnapper_auth.md)	 - Login to DBSnapper Cloud

