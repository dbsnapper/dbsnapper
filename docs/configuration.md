---
title: Configuration Settings
description: Complete guide to DBSnapper configuration file settings, environment variables, and advanced features
---

# Configuration Settings

The DBSnapper configuration system provides comprehensive control over database snapshot operations, cloud integration, security settings, and workflow automation. This guide covers all configuration options from basic setup to advanced features.

## Quick Start

### Default Configuration Location

DBSnapper looks for configuration files in this order:

1. `./dbsnapper.yml` (current directory)
2. `~/.config/dbsnapper/dbsnapper.yml` (user config directory)
3. Environment variables (see [Environment Variables](#environment-variables))

### Minimal Configuration

```yaml title="~/.config/dbsnapper/dbsnapper.yml"
# Required for encryption
secret_key: c614a689a559d1b517c28a5e4fcdc059

# Define your first target
targets:
  myapp-dev:
    name: "My App Development"
    snapshot:
      src_url: "postgresql://user:pass@localhost:5432/myapp_prod"
      dst_url: "postgresql://user:pass@localhost:5432/myapp_dev"
```

## Complete Configuration Reference

<!-- prettier-ignore-start -->
!!! example "Complete Configuration Example"

    ```yaml linenums="1"
    # Core Settings
    authtoken: jREZkinFQpSJb4TWZAKioHuCD7KG2GV3xZqiUwbZfeJVTS7V
    secret_key: c614a689a559d1b517c28a5e4fcdc059
    working_directory: ~/.dbsnapper
    env: development
    debug: true
    trace: false
    
    
    # Docker Configuration
    docker:
      images:
        mysql: mysql:9
        postgres: postgres:latest
    
    # Global Overrides
    override:
      san_query: RFJPUCBUQUJMRSBJRiBFWElTVFMgZGJzbmFwcGVyX2luZm87...
      dst_db_url: postgres://postgres:postgres@localhost:15432/override_db
    
    # Default Settings
    defaults:
      shared_target_dst_db_url: postgres://postgres:postgres@localhost:15432/shared_default
    
    # SSO Configuration
    sso:
      okta:
        provider_url: https://dev-89837244.okta.com
        client_id: 0oag95e2z5BhLZ5AI5d7
        redirect_url: http://localhost:8080/callback
    
    # Storage Profiles
    storage_profiles:
      aws-s3-prod:
        provider: s3
        awscli_profile: default
        region: us-west-2
        bucket: dbsnapper-production
        prefix: snapshots
      
      cloudflare-r2:
        provider: r2
        awscli_profile: r2_profile
        bucket: dbsnapper-r2
        prefix: sanitized
    
    # Target Definitions
    targets:
      myapp-prod:
        name: "Production Database"
        snapshot:
          src_url: postgres://{{`DB_USER` | env}}:{{`DB_PASSWORD` | env}}@prod-host:5432/myapp
          dst_url: postgres://{{`DB_USER` | env}}:{{`DB_PASSWORD` | env}}@dev-host:5432/myapp_dev
          schema_config:
            include_schemas: ["public", "app_data"]
            exclude_schemas: ["temp_logs"]
        storage_profile: aws-s3-prod
        
        sanitize:
          dst_url: postgres://user:pass@localhost:5432/myapp_sanitized
          query_file: sanitize.sql
          override_query: "UPDATE users SET email = 'user@example.com';"
        
        subset:
          src_url: postgres://user:pass@prod-host:5432/myapp
          dst_url: postgres://user:pass@localhost:5432/myapp_subset
          subset_tables:
            - table: public.users
              where: "created_at > '2023-01-01'"
            - table: public.orders
              percent: 20
          copy_tables:
            - public.settings
          excluded_tables:
            - public.temp_data
          added_relationships:
            - fk_table: public.orders
              fk_columns: user_id
              ref_table: public.users
              ref_columns: id
          excluded_relationships:
            - fk_table: public.audit_log
              ref_table: public.users
      
      shared-team-target:
        name: "Team Shared Target"
        share:
          dst_url: postgres://user:pass@localhost:5432/team_shared
          storage_profile_name: cloudflare-r2
    ```
<!-- prettier-ignore-end -->

## Configuration Sections

### Global Settings

#### Core Settings

```yaml
# Required: Encryption key for sensitive data
secret_key: c614a689a559d1b517c28a5e4fcdc059

# Optional: Authentication token for DBSnapper Cloud
authtoken: jREZkinFQpSJb4TWZAKioHuCD7KG2GV3...

# Optional: Local storage location (default: ~/.dbsnapper)
working_directory: ~/.dbsnapper

# Optional: Environment mode (development/staging/production)
env: development
```

**Core Settings Details**:

`secret_key` _(required string[32])_
: 16-byte hexadecimal string (32 characters) used for encrypting sensitive configuration data. Generated automatically with `dbsnapper config init`.

`authtoken` _(optional string)_
: Authentication token for DBSnapper Cloud integration. Enables cloud targets, storage profiles, and team sharing features.

`working_directory` _(optional string)_
: Directory for storing local snapshots and temporary files. Defaults to `~/.dbsnapper`.

`env` _(optional string)_
: Environment mode that configures API endpoints automatically: - `development` → `http://app.dbsnapper.local:3000/api/v3` - `staging` → `https://stg4777.dbsnapper.com/api/v3` - `production` → `https://app.dbsnapper.com/api/v3`

### Docker Configuration

Configure Docker images for database engines:

```yaml
docker:
  images:
    mysql: mysql:9
    postgres: postgres:latest
```

**Supported Database Images**:

- `mysql`: MySQL database engine
- `postgres`: PostgreSQL database engine

### Environment Variables

DBSnapper supports comprehensive environment variable configuration using the `DBSNAPPER_` prefix. Convert configuration keys by replacing periods with double underscores `__`.

#### Common Environment Variables

```bash
# Core Settings
DBSNAPPER_SECRET_KEY=c614a689a559d1b517c28a5e4fcdc059
DBSNAPPER_AUTHTOKEN=jREZkinFQpSJb4TWZAKioHuCD7KG2GV3...
DBSNAPPER_WORKING_DIRECTORY=/opt/dbsnapper
DBSNAPPER_ENV=production

# Automation & CI
DBSNAPPER_NO_CONFIRM=true          # Skip confirmation prompts (i.e. when loading snapshots)

# Configuration Overrides
DBSNAPPER_OVERRIDE__SAN_QUERY=RFJPUCBUQUJMRSBJRi...
DBSNAPPER_OVERRIDE__DST_DB_URL=postgres://user:pass@localhost:5432/override_db
DBSNAPPER_DEFAULTS__SHARED_TARGET_DST_DB_URL=postgres://user:pass@localhost:5432/shared_default

# Docker Images
DBSNAPPER_DOCKER__IMAGES__POSTGRES=postgres:16-alpine
DBSNAPPER_DOCKER__IMAGES__MYSQL=mysql:8-oracle
```

#### Configuration Priority

Environment variables take precedence over configuration file settings, allowing for deployment-specific overrides without modifying configuration files.

### URL Templates

DBSnapper supports Go template syntax in all connection URLs, enabling dynamic configuration with environment variables:

```yaml
targets:
  dynamic-target:
    snapshot:
      src_url: "postgres://{{`DB_USER` | env}}:{{`DB_PASSWORD` | env}}@{{`DB_HOST` | env}}:5432/{{`DB_NAME` | env}}"
      dst_url: "postgres://{{`DB_USER` | env}}:{{`DB_PASSWORD` | env}}@localhost:5432/{{`DB_NAME` | env}}_snapshot"
```

**Template Functions**:

- `{{`ENV_VAR` | env}}` - Substitute environment variable value
- `{{`CONSTANT`}}` - Use literal string value

**Supported URL Fields**:

- `snapshot.src_url` and `snapshot.dst_url`
- `sanitize.dst_url`
- `subset.src_url` and `subset.dst_url`
- `share.dst_url`

### Targets

Targets define database connections and operations. Each target can support multiple operations: snapshot, sanitize, subset, and share.

#### Basic Target Configuration

```yaml
targets:
  myapp-prod:
    name: "Production Database"
    snapshot:
      src_url: "postgresql://user:pass@prod-host:5432/myapp"
      dst_url: "postgresql://user:pass@dev-host:5432/myapp_dev" # Will be created if it doesn't exist or overwritten if it does.
    storage_profile: aws-s3-production
```

#### Schema Configuration

**PostgreSQL Only Feature**: Control which database schemas are included in snapshots. Schema filtering is only supported for PostgreSQL databases - MySQL uses standard dump without filtering.

```yaml
targets:
  postgres-target:
    name: "PostgreSQL with Schema Filtering"
    snapshot:
      src_url: "postgresql://user:pass@host:5432/myapp"
      dst_url: "postgresql://user:pass@host:5432/myapp_snap"
      schema_config:
        # Option 1: Include only specific schemas
        include_schemas: ["public", "app_data", "reports"]

        # Option 2: Exclude specific schemas (alternative to include_schemas)
        # exclude_schemas: ["temp_logs", "analytics", "debug"]

        # Option 3: Control default behavior
        use_default_schema: false # true = default schema only, false = all schemas
```

**Schema Configuration Behavior**:

| Configuration                        | Build Behavior                        | Load Behavior                           |
| ------------------------------------ | ------------------------------------- | --------------------------------------- |
| No `schema_config`                   | Builds `public` schema only           | Loads `public` schema only              |
| `use_default_schema: true`           | Builds default schema (`public`)      | Loads default schema from snapshot      |
| `use_default_schema: false`          | Builds **all available schemas**      | Loads all schemas present in snapshot   |
| `include_schemas: ["public", "app"]` | Builds only listed schemas that exist | Loads only listed schemas from snapshot |
| `exclude_schemas: ["temp", "logs"]`  | Builds all schemas except listed ones | Loads all schemas except excluded ones  |

**Schema Configuration Options**:

`include_schemas` _(optional array)_
: Include only specified schemas in build/load operations. Takes precedence over `exclude_schemas`. Only existing schemas will be processed.

`exclude_schemas` _(optional array)_
: Exclude specified schemas from build/load operations. Processes all other available schemas.

`use_default_schema` _(optional boolean)_
: Controls default schema behavior: - `true`: Process only the default schema (`public` for PostgreSQL) - `false`: Process all available schemas in the database - Not set: Defaults to `public` schema only

**Important Notes**:

- **Dynamic Analysis**: Schema filtering analyzes available schemas in the source database at build time
- **Load Consistency**: Load operations respect the schemas that were included in the snapshot
- **Non-existent Schemas**: Requesting schemas that don't exist will show warnings but won't fail the operation
- **MySQL Limitation**: Schema filtering is PostgreSQL-only; MySQL targets ignore `schema_config`
- **Validation**: Schemas cannot appear in both `include_schemas` and `exclude_schemas`

!!! warning "Schema Validation"
The system prevents schemas from appearing in both `include_schemas` and `exclude_schemas` lists. Configuration validation will fail if this occurs.

!!! tip "Best Practices" - Use `include_schemas` for explicit control over which schemas to process - Use `exclude_schemas` when you want most schemas except a few (logs, temp data) - Set `use_default_schema: false` to capture all schemas in complex applications - Test schema configuration with small databases first to verify expected behavior

#### Sanitization Configuration

Configure data sanitization for security and compliance:

```yaml
targets:
  sanitized-target:
    sanitize:
      dst_url: "postgresql://user:pass@localhost:5432/myapp_sanitized"
      query_file: "sanitization.sql"
      override_query: "UPDATE users SET email = CONCAT('user', id, '@example.com');"
```

**Sanitization Priority System**

DBSnapper uses a three-level priority system for sanitization queries:

1. **Target-level `override_query`** (Highest Priority)
2. **Global `override.san_query`** (Medium Priority)
3. **Target `query_file`** (Lowest Priority)

**Base64 Encoding Support**

All sanitization queries support base64 encoding for complex SQL:

```yaml
# Base64 encoded query (automatically detected and decoded)
override:
  san_query: "VVBEQVRFIHVzZXJzIFNFVCBlbWFpbCA9ICd1c2VyQGV4YW1wbGUuY29tJzs="
```

#### Subset Configuration

Create smaller, referentially intact database subsets:

```yaml
targets:
  subset-target:
    subset:
      src_url: "postgresql://user:pass@prod:5432/myapp"
      dst_url: "postgresql://user:pass@dev:5432/myapp_subset"

      # Tables to subset with specific criteria
      subset_tables:
        - table: public.users
          where: "created_at > '2023-01-01'"
        - table: public.orders
          percent: 20

      # Tables to copy completely
      copy_tables:
        - public.settings
        - public.configuration

      # Tables to exclude entirely
      excluded_tables:
        - public.temp_data
        - public.audit_logs

      # Add missing foreign key relationships
      added_relationships:
        - fk_table: public.orders
          fk_columns: user_id
          ref_table: public.users
          ref_columns: id

      # Exclude problematic relationships (circular dependencies)
      excluded_relationships:
        - fk_table: public.audit_log
          ref_table: public.users
```

**Subset Configuration Details**:

`subset_tables`
: Initial tables of interest with filtering criteria. Use either `where` clause or `percent` (not both).

`copy_tables`
: Tables copied in their entirety to maintain referential integrity.

`excluded_tables`
: Tables completely excluded from the subset.

`added_relationships`
: Foreign key relationships not defined in database schema but needed for referential integrity.

`excluded_relationships`
: Relationships to ignore (useful for breaking circular dependencies).

#### Share Configuration

Configure targets for team sharing via cloud storage:

```yaml
targets:
  shared-target:
    name: "Team Shared Target"
    share:
      dst_url: "postgresql://user:pass@localhost:5432/shared_db"
      storage_profile_name: "s3-sanitized-bucket"
```

**Share Configuration Details**:

`dst_url`
: Database connection where shared snapshots will be loaded.

`storage_profile_name`
: Reference to storage profile containing shared snapshots.

### Storage Profiles

Configure cloud storage for snapshot backup and sharing:

#### Amazon S3

```yaml
storage_profiles:
  s3-production:
    provider: s3
    region: us-west-2
    bucket: dbsnapper-prod-snapshots
    prefix: team-snapshots

    # Option 1: Use AWS CLI profile
    awscli_profile: production

    # Option 2: Direct credentials (not recommended for production)
    access_key: AKIA...
    secret_key: xyz...
```

#### Cloudflare R2

```yaml
storage_profiles:
  r2-sanitized:
    provider: r2
    bucket: dbsnapper-r2-sanitized
    prefix: sanitized

    # Option 1: Use AWS CLI profile configured for R2
    awscli_profile: r2_profile

    # Option 2: Direct credentials with account ID
    access_key: your-r2-access-key
    secret_key: your-r2-secret-key
    account_id: your-cloudflare-account-id
```

#### MinIO

```yaml
storage_profiles:
  minio-local:
    provider: minio
    endpoint: http://localhost:9000
    bucket: dbsnapper-minio
    prefix: snapshots
    access_key: minioadmin
    secret_key: minioadmin
```

#### DigitalOcean Spaces

```yaml
storage_profiles:
  do-spaces:
    provider: dospaces
    endpoint: https://nyc3.digitaloceanspaces.com
    bucket: dbsnapper-do-spaces
    prefix: production
    access_key: your-spaces-key
    secret_key: your-spaces-secret
```

### Override System

Global overrides provide system-wide defaults and overrides:

```yaml
override:
  # Global sanitization query (base64 encoded)
  san_query: "VVBEQVRFIHVzZXJzIFNFVCBlbWFpbCA9ICd1c2VyQGV4YW1wbGUuY29tJzs="

  # Global destination database override
  dst_db_url: "postgresql://postgres:postgres@localhost:5432/global_override"
```

**Override Priority**:

- Override settings apply to all operations unless specifically overridden in target configuration
- Target-level settings always take precedence over global overrides

### Defaults System

Configure default values for shared and team operations:

```yaml
defaults:
  # Default destination for shared team snapshots
  shared_target_dst_db_url: "postgresql://user:pass@localhost:5432/shared_default"
```

### SSO Configuration

Configure Single Sign-On for team sharing and authentication:

```yaml
sso:
  okta:
    provider_url: https://dev-89837244.okta.com
    client_id: 0oag95e2z5BhLZ5AI5d7
    redirect_url: http://localhost:8080/callback
    # Tokens are encrypted automatically when stored
    access_token: $aes$fdff26aae3438c29...
    refresh_token: $aes$6d42e5fd73fea9de...
    expires: 1715292878
```

## Configuration Modes

### Local Mode (Standalone)

Local mode uses only the configuration file without cloud integration:

```yaml
# Minimal local configuration
secret_key: c614a689a559d1b517c28a5e4fcdc059
working_directory: ~/.dbsnapper

targets:
  local-target:
    name: "Local Development"
    snapshot:
      src_url: "postgresql://user:pass@localhost:5432/myapp"
      dst_url: "postgresql://user:pass@localhost:5432/myapp_dev"
```

**Local Mode Features**:

- Local snapshot storage only
- No cloud integration
- No team sharing capabilities
- Faster operations (no network overhead)

### Cloud Mode

Cloud mode enables full feature set with DBSnapper Cloud integration:

```yaml
# Cloud configuration
secret_key: c614a689a559d1b517c28a5e4fcdc059
authtoken: jREZkinFQpSJb4TWZAKioHuCD7KG2GV3...
env: production

storage_profiles:
  cloud-storage:
    provider: s3
    bucket: dbsnapper-cloud
    awscli_profile: production

targets:
  cloud-target:
    name: "Production with Cloud"
    snapshot:
      src_url: "postgresql://user:pass@prod:5432/myapp"
      dst_url: "postgresql://user:pass@dev:5432/myapp"
    storage_profile: cloud-storage
```

**Cloud Mode Features**:

- Cloud snapshot storage and backup
- Team sharing with SSO integration
- Cloud target management
- Enhanced security with encryption
- Cross-team collaboration

## Security Considerations

### Credential Management

**Best Practices**:

- Use AWS CLI profiles instead of hardcoded credentials
- Store sensitive values as environment variables
- Use DBSnapper's encryption for stored tokens
- Regularly rotate authentication tokens

**Example Secure Configuration**:

```yaml
# Use environment variables for sensitive data
authtoken: # Set via DBSNAPPER_AUTHTOKEN
secret_key: # Set via DBSNAPPER_SECRET_KEY

targets:
  secure-target:
    snapshot:
      # Use templates with environment variables
      src_url: "postgres://{{`DB_USER` | env}}:{{`DB_PASS` | env}}@{{`DB_HOST` | env}}/myapp"
      dst_url: "postgres://{{`DB_USER` | env}}:{{`DB_PASS` | env}}@localhost/myapp_dev"

storage_profiles:
  secure-s3:
    provider: s3
    # Use AWS CLI profile instead of credentials
    awscli_profile: dbsnapper-production
    bucket: secure-snapshots
```

### Encryption

DBSnapper automatically encrypts sensitive configuration values:

- SSO tokens (access_token, refresh_token)
- Optionally other sensitive strings using `$aes$` prefix
- All encryption uses AES with the configured `secret_key`

## Troubleshooting

### Configuration Validation

Validate your configuration:

```bash
dbsnapper config check
```

### Common Issues

**Invalid Schema Configuration**

```
Error: schema 'analytics' cannot be in both include and exclude lists
```

- Remove schema from one of the lists
- Use either `include_schemas` OR `exclude_schemas`, not both for the same schema

**Template Parsing Errors**

```
Error: template parsing failed for src_url
```

- Check template syntax: `{{`VARIABLE` | env}}`
- Ensure environment variables exist
- Verify no typos in variable names

**Storage Profile Issues**

```
Error: storage profile 'missing-profile' not found
```

- Verify storage profile name matches configuration
- Check storage profile is properly configured
- Ensure required credentials are provided

### Debug Mode

For troubleshooting, you can enable additional logging by adding `debug: true` to your configuration file or using environment variables as needed.

## Migration Guide

### Upgrading from Earlier Versions

**v2.6.0+ Override System**:

- Added `override.san_query` and `override.dst_db_url`
- Migrate target-specific overrides to global overrides if desired

**v2.7.0+ Defaults System**:

- Added `defaults.shared_target_dst_db_url`
- Configure default shared target destination

**Schema Configuration**:

- New `schema_config` section replaces database-specific schema filtering
- Update existing schema filtering configurations

## See Also

- [Build Command](commands/build.md) - Creating database snapshots
- [Load Command](commands/load.md) - Restoring snapshots
- [Cloud Storage Setup](cloud-storage-engines/introduction.md)
- [Database Engines](database-engines/introduction.md)
- [Sanitization](sanitize/introduction.md)
