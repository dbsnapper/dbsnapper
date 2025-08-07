# Dev Command

The `dev` command provides development and testing utilities for DBSnapper operations. These specialized tools are designed for end-to-end testing, validation workflows, and development assistance. They enable thorough testing of database operations and validation of expected behaviors.

## Overview

The dev command provides development utilities:
- **End-to-end testing**: Tools for comprehensive testing workflows
- **Database validation**: Verify database states and schema configurations
- **Sanitization testing**: Validate data sanitization processes
- **Development assistance**: Debugging and development support tools
- **Feature flag controlled**: Requires explicit enablement for security

## Syntax

```bash
dbsnapper dev [subcommand] [flags]
```

## Subcommands

| Subcommand | Purpose | Use Case |
|------------|---------|----------|
| [`ts`](#ts-subcommand) | Add timestamp table for testing | End-to-end test verification |
| [`assert-ts`](#assert-ts-subcommand) | Validate timestamp consistency | Test workflow validation |
| [`schemas`](#schemas-subcommand) | List database schemas | Schema filtering validation |
| [`validate-sanitize`](#validate-sanitize-subcommand) | Verify sanitization application | Data protection verification |

## Options

```bash
  -h, --help   help for dev

Global Flags:
      --config string   config file (default is ~/.config/dbsnapper/dbsnapper.yml)
      --nocloud         Disable cloud mode to speed up operations by skipping cloud API calls
```

## Feature Flag Requirement

Development utilities require explicit enablement in configuration:

```yaml
# Enable development utilities
feature_flags:
  development_utilities: true
```

**Security Note**: Development utilities are disabled by default and should only be enabled in development and testing environments.

## Target Support

All dev subcommands support multiple target types:

### Local Targets
- Defined in local configuration files
- Full read/write access for testing operations
- Complete control over database operations

### Cloud Targets
- Retrieved from DBSnapper Cloud API
- Subject to cloud permissions and access controls
- May have restrictions on certain operations

### Shared Targets
- Accessible via Okta SSO integration
- Typically read-only for validation purposes
- Limited to non-destructive operations

## Database URL Resolution

Development commands use consistent database URL resolution:

### Default Behavior
- **Destination database**: Uses target's `dst_url` (load/sanitize target)
- **Source database**: Uses target's `src_url` when explicitly requested
- **Shared targets**: Uses appropriate defaults for the target type

### URL Override Options
Most commands support `--src_url` flag to inspect source database instead of destination.

## TS Subcommand

### Purpose
Adds a timestamp table (`dbsnapper_info`) to target database for end-to-end testing verification.

### Syntax
```bash
dbsnapper dev ts <target_name> [flags]
```

### Arguments
- `<target_name>`: Name of target to timestamp

### Options
```bash
  -h, --help                help for ts
      --json                Output result as JSON
      --location string     Location to include in tags (default: "cloud")
      --query-name string   Query name to include in tags (optional)
      --src_url             Timestamp the source database instead of destination
      --test-name string    Test name to include in tags (optional)
```

### Timestamp Table Structure
```sql
CREATE TABLE dbsnapper_info (
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    tags TEXT[]  -- Array containing metadata
);
```

### Metadata Tags
The timestamp table includes structured metadata:
- **Target name**: Name of the DBSnapper target
- **Test name**: Custom test identifier (via --test-name)
- **Query name**: Associated query identifier (via --query-name)  
- **Location**: Execution location (via --location)

### Example Usage
```bash
# Basic timestamp for end-to-end testing
dbsnapper dev ts production-replica --test-name e2e-build-001

# Timestamp source database with custom metadata
dbsnapper dev ts staging-db --src_url --test-name integration-test --location local

# JSON output for programmatic use
dbsnapper dev ts analytics-db --json --test-name nightly-test
```

**Output:**
```json
{
  "success": true,
  "target": "analytics-db",
  "timestamp": "2024-01-15T10:30:45Z",
  "database_type": "postgresql",
  "tags": ["analytics-db", "nightly-test", "", "cloud"],
  "connection_info": {
    "url_type": "destination",
    "description": "destination database (dst_url)"
  }
}
```

## Assert-TS Subcommand

### Purpose
Validates timestamp consistency between source and destination databases for end-to-end testing.

### Syntax
```bash
dbsnapper dev assert-ts <target_name> [flags]
```

### Arguments  
- `<target_name>`: Name of target to validate

### Options
```bash
  -h, --help   help for assert-ts
      --json   Output result as JSON
```

### Validation Process
1. **Read source database**: Queries `dbsnapper_info` table from src_url
2. **Read destination database**: Queries `dbsnapper_info` table from dst_url  
3. **Compare timestamps**: Ensures timestamps match between databases
4. **Validate tags**: Confirms metadata tags are preserved correctly
5. **Report results**: Provides success/failure status with details

### End-to-End Testing Workflow
```bash
# Complete testing workflow
dbsnapper dev ts my-target --test-name "e2e-build-123"
dbsnapper build my-target
dbsnapper load my-target  
dbsnapper dev assert-ts my-target  # Validates the complete cycle
```

### Example Usage
```bash
# Basic timestamp validation
dbsnapper dev assert-ts production-replica

# JSON output for test automation
dbsnapper dev assert-ts staging-db --json

# Validate cloud target consistency  
dbsnapper dev assert-ts cloud-production --json
```

**Output:**
```json
{
  "success": true,
  "target": "staging-db",
  "source_timestamp": "2024-01-15T10:30:45Z",
  "destination_timestamp": "2024-01-15T10:30:45Z",
  "timestamps_match": true,
  "tags_match": true,
  "source_tags": ["staging-db", "e2e-build-123", "", "cloud"],
  "destination_tags": ["staging-db", "e2e-build-123", "", "cloud"]
}
```

## Schemas Subcommand

### Purpose
Lists all database schemas for target validation and schema filtering verification.

### Syntax
```bash
dbsnapper dev schemas <target_name> [flags]
```

### Arguments
- `<target_name>`: Name of target to inspect

### Options
```bash
  -h, --help      help for schemas
      --json      Output result as JSON (recommended)
      --src_url   Inspect source database instead of destination
```

### Database Support
- **PostgreSQL**: Queries `information_schema.schemata`
- **MySQL**: Queries `information_schema.schemata`
- **Schema ordering**: Results returned in alphabetical order

### Use Cases
- **Schema filtering validation**: Verify only expected schemas exist after operations
- **Database inspection**: Understand database structure
- **Test verification**: Confirm schema-specific operations worked correctly
- **Migration validation**: Ensure schema changes applied correctly

### Example Usage
```bash
# List schemas in destination database
dbsnapper dev schemas my-target --json

# Inspect source database schemas
dbsnapper dev schemas production-db --src_url --json

# Quick schema overview (human-readable)
dbsnapper dev schemas analytics-target
```

**Output:**
```json
{
  "success": true,
  "target": "analytics-target",
  "schemas": [
    "analytics",
    "customer_data", 
    "public",
    "reporting"
  ],
  "database_type": "postgresql",
  "connection_info": {
    "url_type": "destination",
    "description": "destination database (dst_url)"
  }
}
```

## Validate-Sanitize Subcommand

### Purpose
Validates that data sanitization was successfully applied by checking for sanitization metadata.

### Syntax
```bash
dbsnapper dev validate-sanitize <target_name> [flags]
```

### Arguments
- `<target_name>`: Name of target to validate

### Options
```bash
      --column string            Column to check for metadata (default "tags")
      --connection-type string   Database connection: dst_url or src_url (default "dst_url")
      --contains string          Expected value in metadata column
  -h, --help                     help for validate-sanitize
      --json                     Output results in JSON format
      --table string             Table containing metadata (default "dbsnapper_info")
```

### Validation Process
1. **Connect to database**: Uses specified connection type (default: destination)
2. **Query metadata table**: Checks for sanitization markers
3. **Validate content**: Ensures expected sanitization metadata exists
4. **Report status**: Success/failure with detailed information

### Sanitization Metadata
The validation looks for:
- **Metadata table existence**: Confirms sanitization process created tracking table
- **Expected markers**: Validates specific sanitization identifiers exist
- **Process completion**: Ensures sanitization completed successfully

### Example Usage
```bash
# Basic sanitization validation
dbsnapper dev validate-sanitize production-sanitized

# Custom validation criteria
dbsnapper dev validate-sanitize my-target \
    --table sanitization_log \
    --column process_id \
    --contains "sanitize-2024-01-15"

# JSON output for automation
dbsnapper dev validate-sanitize test-target --json

# Validate source database sanitization
dbsnapper dev validate-sanitize my-target --connection-type src_url
```

**Output:**
```json
{
  "success": true,
  "target": "test-target",
  "table_exists": true,
  "metadata_found": true,
  "expected_value": "test-target",
  "actual_value": "test-target",
  "validation_passed": true,
  "connection_info": {
    "connection_type": "dst_url",
    "description": "destination database"
  }
}
```

## Development Workflows

### End-to-End Testing Workflow
```bash
#!/bin/bash
set -e

TARGET="test-environment"
TEST_NAME="e2e-$(date +%s)"

echo "Starting end-to-end test: $TEST_NAME"

# 1. Add timestamp to source database
dbsnapper dev ts $TARGET --src_url --test-name "$TEST_NAME" --json

# 2. Build snapshot
dbsnapper build $TARGET

# 3. Load snapshot into destination  
dbsnapper load $TARGET

# 4. Validate timestamp consistency
if ! dbsnapper dev assert-ts $TARGET --json; then
    echo "ERROR: Timestamp validation failed"
    exit 1
fi

# 5. Validate schemas
EXPECTED_SCHEMAS='["public", "analytics"]'
ACTUAL_SCHEMAS=$(dbsnapper dev schemas $TARGET --json | jq -r '.schemas')

if [[ "$ACTUAL_SCHEMAS" != "$EXPECTED_SCHEMAS" ]]; then
    echo "ERROR: Schema validation failed"
    echo "Expected: $EXPECTED_SCHEMAS"
    echo "Actual: $ACTUAL_SCHEMAS"
    exit 1
fi

echo "✅ End-to-end test completed successfully"
```

### Sanitization Testing Workflow
```bash
#!/bin/bash
set -e

TARGET="production-data"
TEST_NAME="sanitization-$(date +%s)"

echo "Testing sanitization workflow: $TEST_NAME"

# 1. Add timestamp for tracking
dbsnapper dev ts $TARGET --src_url --test-name "$TEST_NAME"

# 2. Build original snapshot
dbsnapper build $TARGET

# 3. Create sanitized snapshot
dbsnapper sanitize $TARGET

# 4. Load sanitized snapshot
dbsnapper load $TARGET

# 5. Validate sanitization was applied
if ! dbsnapper dev validate-sanitize $TARGET --json; then
    echo "ERROR: Sanitization validation failed"
    exit 1
fi

# 6. Check timestamp preservation
if ! dbsnapper dev assert-ts $TARGET --json; then
    echo "ERROR: Timestamp consistency failed"
    exit 1
fi

echo "✅ Sanitization workflow validated successfully"
```

### Schema Filtering Validation
```bash
#!/bin/bash
set -e

TARGET="multi-schema-db"
EXPECTED_SCHEMAS=("public" "analytics" "reporting")

echo "Validating schema filtering for $TARGET"

# Get actual schemas
ACTUAL=$(dbsnapper dev schemas $TARGET --json | jq -r '.schemas[]' | sort)
EXPECTED=$(printf '%s\n' "${EXPECTED_SCHEMAS[@]}" | sort)

if [[ "$ACTUAL" == "$EXPECTED" ]]; then
    echo "✅ Schema filtering validation passed"
else
    echo "❌ Schema filtering validation failed"
    echo "Expected schemas: ${EXPECTED_SCHEMAS[*]}"
    echo "Actual schemas: $ACTUAL"
    exit 1
fi
```

### Cloud Target Testing
```bash
#!/bin/bash
set -e

CLOUD_TARGET="production-cloud"
TEST_NAME="cloud-integration-$(date +%s)"

echo "Testing cloud target integration: $TEST_NAME"

# 1. Timestamp cloud target
dbsnapper dev ts $CLOUD_TARGET --test-name "$TEST_NAME" --location cloud --json

# 2. Build cloud snapshot
dbsnapper build $CLOUD_TARGET

# 3. Validate schemas match expectations
SCHEMAS=$(dbsnapper dev schemas $CLOUD_TARGET --json)
if ! echo "$SCHEMAS" | jq -e '.success'; then
    echo "ERROR: Schema validation failed"
    exit 1
fi

# 4. Validate timestamp consistency (if loading to destination)
if dbsnapper load $CLOUD_TARGET; then
    if ! dbsnapper dev assert-ts $CLOUD_TARGET --json; then
        echo "ERROR: Cloud target timestamp validation failed"
        exit 1
    fi
fi

echo "✅ Cloud target integration test completed"
```

## Integration with CI/CD

### GitHub Actions Integration
```yaml
name: DBSnapper E2E Tests

on: [push, pull_request]

jobs:
  e2e-tests:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - uses: actions/checkout@v4
    
    - name: Setup DBSnapper
      run: |
        go build -o bin/dbsnapper ./
        chmod +x bin/dbsnapper
    
    - name: Configure development utilities
      run: |
        cat > test-config.yml << EOF
        feature_flags:
          development_utilities: true
        targets:
          test-db:
            snapshot:
              src_url: "postgres://postgres:test@localhost:5432/test_source"
              dst_url: "postgres://postgres:test@localhost:5432/test_dest"
        EOF
    
    - name: Run E2E tests
      run: |
        export DBSNAPPER_CONFIG=test-config.yml
        
        # Create test databases
        createdb -h localhost -U postgres test_source
        createdb -h localhost -U postgres test_dest
        
        # Run development workflow
        ./bin/dbsnapper dev ts test-db --test-name ci-${{ github.run_id }}
        ./bin/dbsnapper build test-db
        ./bin/dbsnapper load test-db
        
        # Validate results
        ./bin/dbsnapper dev assert-ts test-db --json
        ./bin/dbsnapper dev schemas test-db --json
```

### Test Automation Scripts
```bash
#!/bin/bash
# automated-test-suite.sh

set -e

TARGETS=("local-test" "staging-test" "integration-test")
TEST_SESSION="automated-$(date +%Y%m%d-%H%M%S)"

echo "Starting automated test suite: $TEST_SESSION"

for TARGET in "${TARGETS[@]}"; do
    echo "Testing target: $TARGET"
    
    # Individual test workflow
    TEST_NAME="${TEST_SESSION}-${TARGET}"
    
    dbsnapper dev ts "$TARGET" --test-name "$TEST_NAME" --json > "/tmp/${TARGET}-timestamp.json"
    
    if dbsnapper build "$TARGET"; then
        echo "✅ Build successful for $TARGET"
    else
        echo "❌ Build failed for $TARGET"
        continue
    fi
    
    if dbsnapper load "$TARGET"; then
        echo "✅ Load successful for $TARGET"
    else
        echo "❌ Load failed for $TARGET" 
        continue
    fi
    
    if dbsnapper dev assert-ts "$TARGET" --json > "/tmp/${TARGET}-assert.json"; then
        echo "✅ Timestamp validation passed for $TARGET"
    else
        echo "❌ Timestamp validation failed for $TARGET"
    fi
    
    # Store test results
    jq -s '.[0] + .[1]' "/tmp/${TARGET}-timestamp.json" "/tmp/${TARGET}-assert.json" > "/tmp/${TARGET}-results.json"
done

echo "Automated test suite completed: $TEST_SESSION"
```

## Troubleshooting

### Feature Flag Not Enabled
```
Error: development utilities are not enabled
```

**Solution:**
```yaml
# Add to configuration file
feature_flags:
  development_utilities: true
```

### Database Connection Issues
```
Error: failed to connect to database
```

**Solutions:**
1. Verify target configuration is correct
2. Test database connectivity independently
3. Check database credentials and permissions
4. Ensure database is accessible from current network

### Timestamp Table Issues
```
Error: dbsnapper_info table not found
```

**Solutions:**
1. Run `dbsnapper dev ts` before validation commands
2. Check database permissions for table creation
3. Verify target database URL is correct
4. Ensure database supports required table structure

### Schema Query Failures
```
Error: failed to query database schemas
```

**Solutions:**
1. Verify database user has schema access permissions
2. Check if `information_schema` is accessible
3. Ensure database engine is supported (PostgreSQL/MySQL)
4. Test with simpler database queries first

### JSON Parsing Issues
```
Error: invalid JSON output
```

**Solutions:**
1. Use `--json` flag for programmatic access
2. Check for command errors before parsing JSON
3. Validate JSON output with `jq` or similar tools
4. Handle error conditions in automation scripts

## Best Practices

### Development Environment Setup
1. **Separate configuration**: Use dedicated configuration files for development
2. **Feature flag isolation**: Only enable development utilities where needed
3. **Test data management**: Use non-production data for development testing
4. **Version control**: Track test configurations alongside code changes

### Test Automation
1. **Consistent naming**: Use structured test naming conventions
2. **Error handling**: Implement proper error handling in test scripts  
3. **Result storage**: Save test results for analysis and debugging
4. **Parallel testing**: Be careful with concurrent operations on shared resources

### CI/CD Integration
1. **Environment isolation**: Use dedicated test environments
2. **Resource cleanup**: Clean up test resources after use
3. **Result reporting**: Integrate test results with CI/CD reporting
4. **Failure analysis**: Provide detailed failure information for debugging

!!! tip "Development Workflow"
    Start with simple timestamp validation before building complex test workflows. The `ts` and `assert-ts` commands provide a solid foundation for end-to-end testing.

!!! note "Feature Flag Required"
    Development utilities must be explicitly enabled via configuration. This prevents accidental use in production environments.

!!! warning "Testing Data"
    Development commands can modify database state. Always use dedicated test environments and avoid production databases.

## Related Commands

- [`build`](build.md) - Create snapshots (tested by dev workflow)
- [`load`](load.md) - Load snapshots (validated by dev tools) 
- [`sanitize`](sanitize.md) - Sanitize snapshots (verified by validate-sanitize)
- [`config`](config.md) - Configuration management (controls dev feature flags)

## See Also

