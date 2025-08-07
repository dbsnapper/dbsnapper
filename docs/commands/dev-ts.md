# Dev TS Command

The `dev ts` command adds a timestamp table to target databases for end-to-end testing verification. This essential testing utility creates structured metadata that enables comprehensive validation of database operations and workflows.

## Overview

The dev ts command provides timestamp-based testing:
- **Test metadata creation**: Adds structured metadata table for test tracking
- **End-to-end validation**: Enables verification of complete database workflows  
- **Flexible targeting**: Supports source or destination database timestamping
- **Custom metadata**: Allows test-specific tags and identifiers
- **JSON integration**: Provides structured output for automation

## Syntax

```bash
dbsnapper dev ts <target_name> [flags]
```

## Arguments

- `<target_name>`: Name of the target database to timestamp

## Options

```bash
  -h, --help                help for ts
      --json                Output result as JSON
      --location string     Location to include in tags (default "cloud")
      --query-name string   Query name to include in tags (optional)
      --src_url             Timestamp the source database instead of destination
      --test-name string    Test name to include in tags (optional)
```

## Database Table Structure

The command creates a `dbsnapper_info` table with the following structure:

### PostgreSQL
```sql
CREATE TABLE dbsnapper_info (
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    tags TEXT[]
);
```

### MySQL
```sql
CREATE TABLE dbsnapper_info (
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    tags JSON
);
```

## Metadata Tags Structure

The tags field contains structured metadata in order:
1. **Target name**: Name of the DBSnapper target
2. **Test name**: Custom test identifier (from --test-name)
3. **Query name**: Associated query identifier (from --query-name)
4. **Location**: Execution location (from --location)

**Example tags array**: `["production-replica", "e2e-build-001", "sanitize-query", "cloud"]`

## Database Selection Logic

### Default Behavior (Destination Database)
```bash
# Uses target's dst_url by default
dbsnapper dev ts my-target --test-name "workflow-test"
```

**Use cases:**
- Testing load operations  
- Validating destination database state
- Verifying data after transformations

### Source Database Targeting  
```bash
# Uses target's src_url with --src_url flag
dbsnapper dev ts my-target --src_url --test-name "source-validation"
```

**Use cases:**
- Testing build operations
- Validating source database state
- Pre-operation state verification

## Example Usage

### Basic End-to-End Testing
```bash
# Add timestamp for simple e2e test
dbsnapper dev ts production-replica --test-name "e2e-build-001"
```

**Output:**
```
✅ Timestamp table added successfully
Target: production-replica
Database: destination (dst_url)  
Timestamp: 2024-01-15T10:30:45Z
Tags: ["production-replica", "e2e-build-001", "", "cloud"]
```

### Source Database Validation
```bash
# Timestamp source database for build testing
dbsnapper dev ts staging-db --src_url --test-name "integration-test" --location "local"
```

**Output:**
```
✅ Timestamp table added successfully
Target: staging-db
Database: source (src_url)
Timestamp: 2024-01-15T10:30:45Z  
Tags: ["staging-db", "integration-test", "", "local"]
```

### Complete Metadata Specification
```bash
# Full metadata for comprehensive tracking
dbsnapper dev ts analytics-db \
    --test-name "nightly-validation" \
    --query-name "sanitize-pii" \
    --location "production" \
    --json
```

**JSON Output:**
```json
{
  "success": true,
  "target": "analytics-db",
  "timestamp": "2024-01-15T10:30:45Z",
  "database_type": "postgresql",
  "tags": [
    "analytics-db",
    "nightly-validation", 
    "sanitize-pii",
    "production"
  ],
  "connection_info": {
    "url_type": "destination",
    "description": "destination database (dst_url)"
  }
}
```

## Target Type Support

### Local Targets
```bash
# Local target with full control
dbsnapper dev ts my-local-db --test-name "local-development"
```

**Capabilities:**
- Full read/write access
- Complete control over database operations
- No cloud API dependencies

### Cloud Targets
```bash  
# Cloud target with cloud integration
dbsnapper dev ts my-cloud-db --test-name "cloud-validation" --location "cloud"
```

**Capabilities:**
- Uses DBSnapper Cloud API for target resolution
- Subject to cloud permissions and access controls
- Enhanced metadata for cloud environments

### Shared Targets
```bash
# Shared target (typically read-only)
dbsnapper dev ts shared-team-db --test-name "shared-validation"
```

**Capabilities:**
- Accessed via Okta SSO integration
- May have restricted permissions
- Used for validation rather than modification

## Testing Workflows

### Complete End-to-End Workflow
```bash
#!/bin/bash
set -e

TARGET="test-environment"
TEST_NAME="e2e-$(date +%s)"

echo "Starting E2E test workflow: $TEST_NAME"

# 1. Add timestamp to track test
dbsnapper dev ts $TARGET --test-name "$TEST_NAME" --json > /tmp/timestamp.json

# 2. Execute DBSnapper operations  
dbsnapper build $TARGET
dbsnapper load $TARGET

# 3. Validate timestamp consistency
dbsnapper dev assert-ts $TARGET --json > /tmp/validation.json

# 4. Check results
if jq -e '.success' /tmp/validation.json > /dev/null; then
    echo "✅ E2E test completed successfully"
else
    echo "❌ E2E test failed"
    jq '.error' /tmp/validation.json
    exit 1
fi
```

### Build Operation Testing
```bash
#!/bin/bash
set -e

TARGET="source-database"
TEST_NAME="build-validation-$(date +%s)"

echo "Testing build operation: $TEST_NAME"

# 1. Timestamp source database before build
dbsnapper dev ts $TARGET --src_url --test-name "$TEST_NAME" --location "build-test"

# 2. Perform build operation
if dbsnapper build $TARGET; then
    echo "✅ Build operation successful"
else
    echo "❌ Build operation failed"
    exit 1
fi

# 3. Load and validate timestamp preservation  
dbsnapper load $TARGET

# 4. Assert timestamp consistency
if dbsnapper dev assert-ts $TARGET --json; then
    echo "✅ Timestamp preserved through build/load cycle"
else
    echo "❌ Timestamp consistency check failed"
    exit 1
fi
```

### Sanitization Testing
```bash
#!/bin/bash
set -e

TARGET="production-data"  
TEST_NAME="sanitization-$(date +%s)"

echo "Testing sanitization workflow: $TEST_NAME"

# 1. Add timestamp with sanitization metadata
dbsnapper dev ts $TARGET --src_url \
    --test-name "$TEST_NAME" \
    --query-name "pii-sanitization" \
    --location "sanitize-test"

# 2. Build original snapshot
dbsnapper build $TARGET

# 3. Create sanitized version
dbsnapper sanitize $TARGET

# 4. Load sanitized snapshot
dbsnapper load $TARGET

# 5. Validate both timestamp preservation and sanitization
dbsnapper dev assert-ts $TARGET --json
dbsnapper dev validate-sanitize $TARGET --json

echo "✅ Sanitization workflow validated"
```

## Automation Integration

### JSON Output Processing
```bash
#!/bin/bash

TARGET="automation-test"
TEST_NAME="automated-$(date +%s)"

# Execute with JSON output
RESULT=$(dbsnapper dev ts $TARGET --test-name "$TEST_NAME" --json)

# Extract key information
SUCCESS=$(echo "$RESULT" | jq -r '.success')
TIMESTAMP=$(echo "$RESULT" | jq -r '.timestamp')
TARGET_NAME=$(echo "$RESULT" | jq -r '.target')

if [[ "$SUCCESS" == "true" ]]; then
    echo "Timestamp added successfully for $TARGET_NAME at $TIMESTAMP"
    echo "Test name: $TEST_NAME"
    
    # Store for later validation
    echo "$RESULT" > "/tmp/timestamp-${TARGET_NAME}-${TEST_NAME}.json"
else
    echo "Failed to add timestamp"
    exit 1
fi
```

### CI/CD Pipeline Integration
```yaml
# GitHub Actions example
- name: Prepare E2E Test Environment
  run: |
    TEST_NAME="ci-${{ github.run_id }}-${{ github.run_attempt }}"
    
    # Add timestamps to all test targets
    for target in test-db-1 test-db-2 test-db-3; do
      ./bin/dbsnapper dev ts $target \
        --test-name "$TEST_NAME" \
        --location "github-actions" \
        --json > "timestamp-${target}.json"
    done

- name: Execute Database Operations
  run: |
    # Run build/load operations on timestamped databases
    for target in test-db-1 test-db-2 test-db-3; do
      ./bin/dbsnapper build $target
      ./bin/dbsnapper load $target
    done

- name: Validate Test Results
  run: |
    # Validate all timestamps were preserved
    for target in test-db-1 test-db-2 test-db-3; do
      if ! ./bin/dbsnapper dev assert-ts $target --json; then
        echo "Timestamp validation failed for $target"
        exit 1
      fi
    done
    echo "All E2E validations passed"
```

### Test Result Aggregation
```bash
#!/bin/bash

TEST_SESSION="batch-$(date +%Y%m%d-%H%M%S)"
TARGETS=("app-db" "analytics-db" "cache-db")

echo "Starting batch test session: $TEST_SESSION"

RESULTS_DIR="/tmp/test-results-$TEST_SESSION"
mkdir -p "$RESULTS_DIR"

for TARGET in "${TARGETS[@]}"; do
    echo "Processing target: $TARGET"
    
    TEST_NAME="${TEST_SESSION}-${TARGET}"
    
    # Add timestamp and capture result
    if dbsnapper dev ts "$TARGET" --test-name "$TEST_NAME" --json > "$RESULTS_DIR/${TARGET}-timestamp.json"; then
        echo "✅ Timestamp added for $TARGET"
        
        # Extract timestamp for tracking
        TIMESTAMP=$(jq -r '.timestamp' "$RESULTS_DIR/${TARGET}-timestamp.json")
        echo "$TARGET: $TIMESTAMP" >> "$RESULTS_DIR/session-summary.txt"
    else
        echo "❌ Failed to add timestamp for $TARGET"
        echo "$TARGET: FAILED" >> "$RESULTS_DIR/session-summary.txt"
    fi
done

echo "Batch timestamping completed. Results in: $RESULTS_DIR"
```

## Error Handling and Troubleshooting

### Common Issues

#### Feature Flag Not Enabled
```
Error: development utilities are not enabled
```

**Solution:**
```yaml
# Add to configuration file
feature_flags:
  development_utilities: true
```

#### Database Connection Failure
```
Error: failed to connect to destination database
```

**Solutions:**
1. Verify target configuration is correct
2. Test database connectivity: `dbsnapper targets`
3. Check database credentials and permissions
4. Ensure target database exists and is accessible

#### Table Creation Permission Error
```
Error: permission denied to create table "dbsnapper_info"
```

**Solutions:**
1. Verify database user has CREATE TABLE permissions
2. Check database user has appropriate schema permissions  
3. Ensure target database allows table creation
4. Use a database user with administrative privileges for testing

#### Target Not Found Error
```
Error: target "my-target" not found
```

**Solutions:**
1. Verify target exists: `dbsnapper targets`
2. Check target name spelling
3. Ensure configuration file contains target definition
4. Verify cloud targets are accessible (if using cloud integration)

### Validation and Debugging

#### Verify Timestamp Table Creation
```sql
-- PostgreSQL
SELECT * FROM dbsnapper_info ORDER BY created_at DESC LIMIT 1;

-- MySQL  
SELECT * FROM dbsnapper_info ORDER BY created_at DESC LIMIT 1;
```

#### Check Table Structure
```sql
-- PostgreSQL
\d dbsnapper_info

-- MySQL
DESCRIBE dbsnapper_info;
```

#### Validate Tags Content
```sql
-- PostgreSQL - view tags array
SELECT created_at, tags FROM dbsnapper_info;

-- MySQL - view JSON tags
SELECT created_at, JSON_EXTRACT(tags, '$') FROM dbsnapper_info;
```

#### Debug JSON Output
```bash
# Capture and examine JSON output
dbsnapper dev ts my-target --test-name "debug-test" --json | jq .

# Validate specific fields
dbsnapper dev ts my-target --test-name "debug-test" --json | jq -r '.success'
dbsnapper dev ts my-target --test-name "debug-test" --json | jq -r '.timestamp'
dbsnapper dev ts my-target --test-name "debug-test" --json | jq -r '.tags[]'
```

## Performance Considerations

### Operation Speed
- **Fast execution**: Timestamp table creation is typically very fast (< 1 second)
- **Minimal overhead**: Small table with simple structure has negligible impact
- **Database agnostic**: Performance consistent across PostgreSQL and MySQL

### Resource Usage
- **Low memory**: Minimal memory footprint for table creation
- **Small storage**: Timestamp table uses minimal disk space
- **Connection efficiency**: Uses existing database connections efficiently

### Concurrent Operations
- **Thread safety**: Safe to run concurrently on different targets
- **Database locks**: Brief table-level lock during creation (typically milliseconds)
- **Parallel testing**: Can be used in parallel test execution

## Best Practices

### Test Naming Conventions
```bash
# Use structured test names for better tracking
dbsnapper dev ts my-target --test-name "e2e-build-$(date +%s)"
dbsnapper dev ts my-target --test-name "integration-test-pr-123" 
dbsnapper dev ts my-target --test-name "nightly-validation-$(date +%Y%m%d)"
```

### Metadata Management
```bash
# Include comprehensive metadata for complex workflows
dbsnapper dev ts production-replica \
    --test-name "release-validation-v2.1.0" \
    --query-name "production-sanitization" \
    --location "staging-environment"
```

### Automation Integration
```bash
# Use JSON output for all automation
if RESULT=$(dbsnapper dev ts $TARGET --test-name "$TEST_NAME" --json); then
    # Process successful result
    echo "$RESULT" | jq -r '.timestamp'
else
    # Handle failure case
    echo "Timestamp creation failed"
    exit 1
fi
```

### Clean Environment Testing
```bash
# Ensure clean test environment
TARGET="test-db"

# Remove existing timestamp table if present
psql -h localhost -U test -d test_dest -c "DROP TABLE IF EXISTS dbsnapper_info;"

# Add fresh timestamp
dbsnapper dev ts $TARGET --test-name "clean-environment-test"
```

!!! tip "Test Workflow Foundation"
    The `dev ts` command is the foundation of DBSnapper's end-to-end testing. Always start your test workflows by adding timestamps before performing operations.

!!! note "Database Selection"
    By default, timestamps are added to the destination database (dst_url). Use `--src_url` when you need to track the source database state.

!!! warning "Development Only"
    Development utilities require explicit feature flag enablement and should only be used in development and testing environments.

## Related Commands

- [`dev assert-ts`](../cmd/dbsnapper_dev_assert-ts.md) - Validate timestamp consistency between databases
- [`dev schemas`](../cmd/dbsnapper_dev_schemas.md) - List database schemas for validation
- [`dev validate-sanitize`](../cmd/dbsnapper_dev_validate-sanitize.md) - Verify sanitization application
- [`build`](build.md) - Create snapshots (tested with ts workflow)
- [`load`](load.md) - Load snapshots (validated with assert-ts)

## See Also

- [Dev Command Overview](dev.md) - Complete development utilities overview  
