# PostgreSQL Local (`postgres://`)

<p align="center">
  <img alt="Postgres.app Logo" src="/static/postgresapp-icon.png"  />
</p>

## Overview

The PostgreSQL Local engine provides direct connections to PostgreSQL databases using locally installed command-line tools. This engine offers the best performance for local development and environments where PostgreSQL client tools are readily available.

## Architecture

The PostgreSQL Local engine implements the Database and DatabaseTools interfaces using:

- **Connection Management**: Direct database connections via `sqlx` and `pgx` drivers
- **Data Operations**: Native SQL execution for queries and schema operations  
- **Tool Operations**: Local `pg_dump`, `pg_restore`, and `psql` command execution
- **Schema Filtering**: Advanced include/exclude patterns for selective exports

## Requirements

### PostgreSQL Client Tools

The engine requires these PostgreSQL tools in your system PATH:

- `pg_dump` - Database backup creation
- `pg_restore` - Database restoration from custom format dumps
- `psql` - SQL execution and plain text restore operations

### Supported Versions

- **Tested**: PostgreSQL 9.2 through 16.x
- **Recommended**: PostgreSQL 12+ for optimal performance
- **Compatible**: Any PostgreSQL version with client tools

### Installation Options

- **[Postgres.app](https://postgresapp.com/)** (macOS) - Includes all required tools
- **PostgreSQL Official Installer** - Full PostgreSQL installation
- **Package Managers**: `brew install postgresql`, `apt-get install postgresql-client`
- **Docker**: Client tools only via `postgres:alpine` image

## URL Schemes and Usage

### Supported URL Schemes

The PostgreSQL Local engine responds to multiple URL schemes:

| Scheme | Description | CLI Abbreviation |
|--------|-------------|------------------|
| `postgres://` | Standard PostgreSQL scheme | `pg` |
| `postgresql://` | Alternative standard scheme | `pg` |
| `pg://` | Short form | `pg` |
| `pglocal://` | Explicit local designation | `pgl` |
| `pgl://` | Shortest local form | `pgl` |

### Connection String Format

```
postgres://username:password@host:port/database?parameters
```

**Example Configurations:**

```yaml
targets:
  production-db:
    snapshot:
      src_url: "postgres://app_user:secret@db.company.com:5432/production?sslmode=require"
      dst_url: "postgres://dev_user:dev_pass@localhost:5432/development?sslmode=disable"
```

### SSL Configuration

```yaml
targets:
  secure-db:
    snapshot:
      src_url: "postgres://user:pass@host/db?sslmode=require&sslcert=client.crt&sslkey=client.key"
```

## Advanced Features

### Schema Filtering (PostgreSQL Exclusive)

PostgreSQL Local engine supports sophisticated schema filtering unavailable in other engines:

```yaml
targets:
  filtered-db:
    snapshot:
      src_url: "postgres://user:pass@host/production"
      dst_url: "postgres://user:pass@localhost/filtered_copy"
      
      schema_config:
        # Include specific schemas
        include_schemas:
          - "public"          # Exact match
          - "app_data"        # Application data schema
          - "tenant_001"      # Specific tenant schema
        
        # Exclude patterns (processed after include)
        exclude_schemas:
          - "temp_schema"     # Temporary schema
          - "backup_schema"   # Backup schema
          - "audit_logs"      # Audit schema
```

**Schema Filtering Behavior:**
1. If `include_schemas` is specified, only those schemas are processed
2. `exclude_schemas` patterns are applied after include filtering
3. Empty `include_schemas` means all schemas (before exclude filtering)
4. Schema names must match exactly (no wildcard patterns)

### CPU Configuration and Parallel Operations

PostgreSQL Local engine supports parallel operations for improved performance:

```yaml
targets:
  parallel-db:
    cpus: 4  # Use 4 parallel workers
    
    snapshot:
      src_url: "postgres://user:pass@source/large_db"
      dst_url: "postgres://user:pass@dest/large_db_copy"
```

**Parallel Operation Benefits:**
- Faster `pg_dump` operations on multi-core systems
- Concurrent table processing during export
- Reduced overall backup time for large databases
- Automatically adjusts based on available CPU cores

### Connection Pool Management

The engine automatically manages connection pools:

```yaml
targets:
  pooled-db:
    # Connection pooling handled automatically
    # Optimal for high-frequency operations
    snapshot:
      src_url: "postgres://user:pass@host/db?pool_max_conns=10"
```

## Target Configuration Examples

### Basic Local Development

```yaml
targets:
  local-dev:
    snapshot:
      src_url: "postgres://postgres:password@localhost:5432/myapp_production"
      dst_url: "postgres://postgres:password@localhost:5432/myapp_development"
```

### Production to Staging with Schema Filtering

```yaml
targets:
  prod-to-staging:
    cpus: 8
    
    snapshot:
      src_url: "postgres://readonly:secret@prod.db.company.com:5432/production?sslmode=require"
      dst_url: "postgres://staging:secret@staging.db.company.com:5432/staging?sslmode=require"
      
      schema_config:
        include_schemas: ["public", "app_data"]
        exclude_schemas: ["temp_schema", "audit_logs"]
      
    sanitize:
      override_query: |
        UPDATE users SET 
          email = 'user' || id || '@staging.company.com',
          phone = '555-0000',
          ssn = NULL;
```

### High-Performance Large Database

```yaml
targets:
  large-db:
    cpus: 16  # Maximum parallel workers
    
    snapshot:
      src_url: "postgres://dbuser:strongpass@warehouse.company.com:5432/analytics"
      dst_url: "postgres://dbuser:devpass@dev-warehouse:5432/analytics_copy"
```

## Permissions and Security

### Required Database Permissions

**For Source Database (Building Snapshots):**
- `CONNECT` on database
- `SELECT` on all tables to backup
- `USAGE` on schemas to include
- `pg_read_all_data` role (PostgreSQL 14+) or individual table permissions

**For Destination Database (Loading Snapshots):**
- `CONNECT` on database
- `CREATE` privilege to create tables
- `INSERT`, `UPDATE`, `DELETE` on destination tables
- Schema creation permissions if needed

### Security Best Practices

```yaml
targets:
  secure-setup:
    snapshot:
      # Use dedicated read-only user for source
      src_url: "postgres://dbsnapper_readonly:readonly_pass@source/db?sslmode=require"
      
      # Use dedicated user with limited privileges for destination
      dst_url: "postgres://dbsnapper_dev:dev_pass@dest/db?sslmode=disable"
```

**Recommended User Setup:**

```sql
-- Source database - read-only user
CREATE USER dbsnapper_readonly WITH PASSWORD 'readonly_password';
GRANT CONNECT ON DATABASE production TO dbsnapper_readonly;
GRANT USAGE ON SCHEMA public TO dbsnapper_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO dbsnapper_readonly;

-- Destination database - development user
CREATE USER dbsnapper_dev WITH PASSWORD 'dev_password';
GRANT CONNECT ON DATABASE development TO dbsnapper_dev;
GRANT CREATE ON DATABASE development TO dbsnapper_dev;
GRANT ALL PRIVILEGES ON SCHEMA public TO dbsnapper_dev;
```

## Performance Optimization

### Large Database Strategies

```yaml
targets:
  optimized-large-db:
    cpus: 8
    
    # Use custom format for better compression and parallel restore
    snapshot:
      src_url: "postgres://user:pass@source/large_db"
      dst_url: "postgres://user:pass@dest/large_db"
      
      # Schema filtering reduces data volume
      schema_config:
        exclude_schemas: ["audit_logs", "temp_schema", "backup_schema"]
```

### Network Optimization

```yaml
targets:
  network-optimized:
    snapshot:
      # Use connection pooling
      src_url: "postgres://user:pass@remote/db?pool_max_conns=5&pool_min_conns=1"
      
      # Optimize for network latency
      dst_url: "postgres://user:pass@local/db?connect_timeout=30"
```

## Troubleshooting

### Common Issues

**Tool Not Found Errors:**
```bash
# Verify PostgreSQL tools are installed and in PATH
which pg_dump
which pg_restore  
which psql
```

**Connection Failures:**
```yaml
# Test connection manually
targets:
  test-connection:
    snapshot:
      src_url: "postgres://user:pass@host/db?connect_timeout=10&sslmode=disable"
```

**Schema Filtering Not Working:**
- Ensure schema names match exactly (case-sensitive)
- Verify wildcard patterns are correct
- Check that user has permissions on filtered schemas

**Performance Issues:**
- Increase `cpus` setting for parallel operations
- Use schema filtering to reduce data volume
- Consider network latency for remote connections

### Validation Commands

```bash
# Test basic connectivity
dbsnapper build test-target --dry-run

# Validate schema filtering
dbsnapper build filtered-target --dry-run --verbose

# Check parallel performance
dbsnapper build large-target --cpus 8 --verbose
```

## Related Documentation

- [PostgreSQL Docker Engine](postgresql-docker.md) - Container-based alternative
- [MySQL Local Engine](mysql-local.md) - MySQL comparison
- [Database Engines Introduction](introduction.md) - Engine architecture overview