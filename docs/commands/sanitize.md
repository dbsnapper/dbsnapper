# Sanitize Command

The `sanitize` command creates sanitized versions of database snapshots by removing or modifying sensitive data. This critical security feature ensures that production data can be safely shared with development teams while maintaining referential integrity and realistic test data.

## Overview

The sanitize command takes existing snapshots created with the `build` command and applies sanitization queries to remove sensitive information like personally identifiable information (PII), passwords, financial data, or any other confidential content. The sanitized snapshots maintain the database structure and relationships while replacing sensitive data with safe alternatives.

## Syntax

```bash
dbsnapper sanitize [flags] <target_name> [snapshot_index]
```

## Arguments

- `<target_name>`: The name of the target defined in your configuration file
- `[snapshot_index]`: Optional index of the snapshot to sanitize (default: 0 for most recent)

## Options

### Command-Specific Flags

```bash
  -e, --ephemeral   Create a snapshot using an ephemeral database via docker containers
  -n, --new         Create a new snapshot set for the target
  -h, --help        help for sanitize  
```

### Global Flags

```bash
      --config string   Config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

## Target Configuration

The sanitize command requires targets with sanitization configuration:

### Required Configuration

**Minimal Target Setup**:

```yaml
targets:
  my-target:
    snapshot:
      src_url: "postgresql://user:pass@host:5432/dbname"
      dst_url: "postgresql://user:pass@dev:5432/dbname_copy"
    sanitize:
      query_file: "sanitize.sql"
```

### Complete Target Configuration

**Target with All Sanitization Options**:

```yaml
targets:
  production-db:
    name: "Production Database"
    snapshot:
      src_url: "postgresql://user:pass@prod:5432/users"
      dst_url: "postgresql://user:pass@dev:5432/users_snap"
    sanitize:
      dst_url: "postgresql://user:pass@sanitize:5432/users_clean"
      query_file: "user_sanitize.sql"
      # Optional: Override with direct SQL
      override_query: |
        UPDATE users SET 
          email = 'user' || id || '@example.com',
          phone = '555-0100',
          ssn = NULL;
        DELETE FROM user_sessions;
```

## Example Usage

### Basic Sanitization

```bash
# Sanitize the most recent snapshot
dbsnapper sanitize production-db

# Sanitize a specific snapshot by index
dbsnapper sanitize production-db 2
```

### Ephemeral Sanitization

```bash
# Force ephemeral mode regardless of configuration
dbsnapper sanitize production-db --ephemeral

# Combine ephemeral with new snapshot set
dbsnapper sanitize production-db --ephemeral --new
```

### New Snapshot Set Creation

```bash
# Create a new sanitized snapshot set
dbsnapper sanitize user-db --new

# Create multiple sanitized versions for testing
dbsnapper sanitize production-db --new
dbsnapper sanitize production-db --new  # Different timestamp
```

## How It Works

1. **Snapshot Selection**: Identifies the snapshot to sanitize based on index (0 = most recent)
2. **Database Preparation**: Sets up the sanitization environment (ephemeral or configured database)
3. **Snapshot Loading**: Loads the original snapshot into the sanitization database
4. **Query Execution**: Applies sanitization queries to remove/modify sensitive data
5. **Sanitized Snapshot Creation**: Creates a new `.san.zip` file with the sanitized data
6. **Cleanup**: Removes temporary files and databases (for ephemeral mode)

## Output Files

### Sanitized Snapshots

- **Format**: `<timestamp>_<target_name>.san.zip`
- **Location**: Working directory specified in configuration file
- **Example**: `1752444532_production.san.zip`

### Snapshot Set Behavior

**Default Behavior**
- Original: `1234567890_target.zip`
- Sanitized: `1234567890_target.san.zip` (same timestamp)

**New Snapshot Set (`--new` flag)**
- Original: `1234567890_target.zip` (unchanged)
- New Sanitized: `9876543210_target.san.zip` (new timestamp)

## Sanitization Query Priority System

DBSnapper uses a **three-level priority system** for specifying sanitization queries, allowing flexible configuration from global defaults to target-specific overrides:

### Priority Levels (Highest to Lowest)

1. **Target-level `override_query`** (Highest Priority)
   - Specified in individual target's sanitize configuration
   - Overrides all other query sources
   - Use for target-specific sanitization requirements

2. **Global `override.san_query`** (Medium Priority) 
   - Specified in global override configuration
   - Applied when no target-level override exists
   - Use for organization-wide sanitization standards

3. **Target `query_file`** (Lowest Priority)
   - SQL file path in target configuration
   - Used when no override queries are specified
   - Supports absolute and relative file paths

### Configuration Examples

**Target-Level Override Query**:

```yaml
targets:
  financial-db:
    name: "Financial Database"
    sanitize:
      dst_url: "postgresql://user:pass@host:5432/finance_clean"
      override_query: |
        UPDATE accounts SET account_number = 'ACCT-' || id;
        UPDATE transactions SET amount = ROUND(RANDOM() * 1000, 2);
        DELETE FROM audit_logs WHERE created_at < NOW() - INTERVAL '30 days';
```

**Global Override Configuration**:

```yaml
# Global sanitization standard for all targets
override:
  san_query: |
    -- Standard PII sanitization
    UPDATE users SET 
      email = 'user' || id || '@example.com',
      phone = '555-0100',
      ssn = NULL;
    DELETE FROM user_sessions;

targets:
  app-db:
    sanitize:
      dst_url: "postgresql://user:pass@host:5432/app_clean"
      # Uses global override.san_query
```

**Base64 Encoded Complex Query**:

```yaml
targets:
  complex-db:
    sanitize:
      # Base64 encoded multi-line query with special characters
      override_query: "VVBEQVRFIHVzZXJzIFNFVCBlbWFpbCA9ICd1c2VyJyB8fCBpZCB8fCAnQGV4YW1wbGUuY29tJywgcGFzc3dvcmQgPSAnZmFrZV9oYXNoJzsKVVBEQVRFIGN1c3RvbWVycyBTRVQgY2NfbnVtYmVyID0gJyoqKioqKioqKioqKioqJyBXSEVSRSBjY19udW1iZXIgSVMgTk9UIE5VTEw7"
```

### Query File Path Resolution

The `query_file` path resolution follows these rules:

- **Absolute paths** (e.g., `/path/to/sanitize.sql`): Used as-is
- **Relative paths** (e.g., `scripts/sanitize.sql`): Resolved relative to the working directory
- **Filenames only** (e.g., `sanitize.sql`): Looked up in the working directory

## Sanitization Modes

### Persistent Database Mode (Default)

Uses a configured database specified in `sanitize.dst_url`:

```yaml
targets:
  production-db:
    sanitize:
      dst_url: "postgresql://user:pass@host:5432/sanitize_db"
      query_file: "sanitize.sql"
```

**Advantages:**
- Faster execution (no container startup)
- Persistent database for debugging
- Full control over database configuration

**Disadvantages:**
- Requires pre-configured database
- Database will be dropped/recreated
- Manual cleanup required

### Ephemeral Database Mode (`--ephemeral` flag)

Uses temporary Docker containers for sanitization:

```bash
dbsnapper sanitize production-db --ephemeral
```

**Advantages:**
- No database setup required
- Automatic cleanup after completion
- Isolated sanitization environment
- No risk to existing databases

**Disadvantages:**
- Slower execution (container overhead)
- Requires Docker installation
- Limited debugging capabilities

### Automatic Ephemeral Mode

If no `sanitize.dst_url` is configured, the command automatically uses ephemeral mode:

```yaml
targets:
  simple-target:
    sanitize:
      query_file: "basic_sanitize.sql"
      # No dst_url specified - will use ephemeral mode
```

## Base64 Query Encoding

All sanitization queries support **base64 encoding** for complex SQL containing special characters, multi-line queries, or YAML formatting issues:

```yaml
targets:
  secure-target:
    sanitize:
      # Base64-encoded complex sanitization query
      override_query: "VVBEQVRFIHVzZXJzIFNFVCBlbWFpbCA9ICd1c2VyJyB8fCBpZCB8fCAnQGV4YW1wbGUuY29tJywgcGFzc3dvcmQgPSAnZmFrZV9oYXNoJzsKVVBEQVRFIGN1c3RvbWVycyBTRVQgY2NfbnVtYmVyID0gJyoqKioqKioqKioqKioqJyBXSEVSRSBjY19udW1iZXIgSVMgTk9UIE5VTEw7"
```

The system automatically detects and decodes base64 content, making it transparent to users while enabling complex query handling.

## Use Cases

### Development Environment Protection

```bash
# Create sanitized snapshots for safe development use
dbsnapper build production-db
dbsnapper sanitize production-db
dbsnapper load dev-db  # Loads sanitized version by default
```

### Multiple Sanitization Levels

```bash
# Create different sanitization levels
dbsnapper sanitize user-data --new    # Light sanitization
# Update sanitize config for heavy sanitization
dbsnapper sanitize user-data --new    # Heavy sanitization
```

### Testing Sanitization Queries

```bash
# Test sanitization with ephemeral mode
dbsnapper sanitize test-db --ephemeral
# Review results, adjust query, repeat
```

### Team Sharing

```bash
# Sanitize and share via cloud storage
dbsnapper sanitize shared-db
# Team members can pull sanitized version
```

## Troubleshooting

### Common Issues

**Query File Not Found**
```
Error: sanitization query file not found: sanitize.sql
```
- Verify file path is correct
- Check file exists in working directory
- Use absolute paths if needed

**Database Connection Failed**
```
Error: failed to connect to sanitize database
```
- Verify `sanitize.dst_url` connection string
- Check database server availability
- Confirm credentials and permissions

**Docker Not Available (Ephemeral Mode)**
```
Error: Docker is not available for ephemeral mode
```
- Install Docker Desktop or Docker Engine
- Ensure Docker daemon is running
- Check Docker permissions

**SQL Syntax Errors**
```
Error: syntax error in sanitization query
```
- Review SQL syntax in query_file or override_query
- Test queries manually against database
- Consider using base64 encoding for complex queries

**Permission Denied**
```
Error: insufficient privileges to modify data
```
- Ensure database user has UPDATE/DELETE privileges
- Check table-specific permissions
- Verify connection credentials

### Best Practices

1. **Test Sanitization Queries**: Always test on non-production data first
2. **Use Ephemeral Mode**: For testing and one-off sanitizations
3. **Version Control**: Keep sanitization SQL files in version control
4. **Data Integrity**: Maintain referential integrity after sanitization
5. **Regular Updates**: Keep sanitization queries updated with schema changes

!!! warning "Irreversible Operation"
    Sanitization permanently modifies data in the sanitized snapshot. The original snapshot remains unchanged, but the sanitized data cannot be recovered without re-running from the original.

!!! tip "Query Priority System"
    Use the three-level priority system effectively:
    - Global queries for organization standards
    - Query files for target-specific needs  
    - Override queries for special cases

## Related Commands

- [`build`](build.md) - Create snapshots to sanitize
- [`load`](load.md) - Load sanitized snapshots (prioritizes .san.zip by default)
- [`target`](target.md) - View available snapshots including sanitized versions
- [`targets`](targets.md) - List all available targets

## See Also

- [Sanitization Guide](../sanitize/introduction.md) - Comprehensive sanitization strategies
- [Configuration](../configuration.md) - Target and sanitization configuration
- [Security Best Practices](../sanitize/configuration.md) - Safe sanitization practices