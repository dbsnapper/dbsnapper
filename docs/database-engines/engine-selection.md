# Engine Selection Guide

## Overview

Choosing the right database engine for your DBSnapper operations depends on your environment, requirements, and constraints. This guide helps you select the optimal engine combination for your specific use case.

## Quick Selection Matrix

| Scenario | PostgreSQL | MySQL | Recommendation |
|----------|------------|--------|----------------|
| **Local tools installed** | `postgres://` | `mysql://` | Use local engines for best performance |
| **No local tools** | `pgdocker://` | `mydocker://` | Use Docker engines for convenience |
| **CI/CD pipelines** | `pgdocker://` | `mydocker://` | Docker engines for consistency |
| **Schema filtering needed** | `postgres://` or `pgdocker://` | Not available | PostgreSQL only feature |
| **Maximum performance** | `postgres://` | `mysql://` | Local engines avoid container overhead |
| **Consistent environments** | `pgdocker://` | `mydocker://` | Docker ensures version consistency |

## PostgreSQL Engine Selection

### PostgreSQL Local (`postgres://`)

**Choose when:**
- PostgreSQL client tools are installed locally
- Maximum performance is required
- Direct network access to databases
- Schema filtering is needed
- Working with large databases requiring parallel operations

```yaml
targets:
  postgres-local-optimal:
    cpus: 8  # Parallel processing available
    
    snapshot:
      src_url: "postgres://user:pass@prod:5432/app"
      dst_url: "postgres://user:pass@dev:5432/app"
      
      schema_config:
        exclude_schemas: ["audit_logs", "temp_data"]
```

**Advantages:**
- Fastest performance (no container overhead)
- Full schema filtering support
- Parallel processing with `cpus` configuration
- Direct tool execution

**Requirements:**
- `pg_dump`, `pg_restore`, `psql` in system PATH
- Network connectivity to PostgreSQL servers
- Compatible PostgreSQL client version

### PostgreSQL Docker (`pgdocker://`)

**Choose when:**
- PostgreSQL tools not installed locally
- Running in containerized environments
- Need consistent tool versions across team/environments
- Working in CI/CD pipelines

```yaml
targets:
  postgres-docker-optimal:
    snapshot:
      src_url: "pgdocker://user:pass@prod:5432/app"
      dst_url: "pgdocker://user:pass@dev:5432/app"
      
      schema_config:
        include_schemas: ["public", "app_data"]
```

**Advantages:**
- No local tool installation required
- Consistent PostgreSQL version across environments
- Full schema filtering support
- Automatic container management

**Requirements:**
- Docker Engine 20.10+
- Internet access to pull images
- Adequate system resources for containers

## MySQL Engine Selection

### MySQL Local (`mysql://`)

**Choose when:**
- MySQL client tools are installed locally
- Maximum performance is critical
- Working with local MySQL instances
- Direct database connectivity available

```yaml
targets:
  mysql-local-optimal:
    cpus: 4  # Affects tool execution
    
    snapshot:
      src_url: "mysql://user:pass@prod:3306/app"
      dst_url: "mysql://user:pass@dev:3306/app"
```

**Advantages:**
- Best performance (no containerization overhead)
- Direct tool execution
- Familiar local tool behavior

**Requirements:**
- `mysqldump`, `mysql` commands in system PATH
- Network connectivity to MySQL servers
- Compatible MySQL client version

### MySQL Docker (`mydocker://`)

**Choose when:**
- MySQL tools not available locally
- Need consistent MySQL tool versions
- Working in containerized environments
- Simplified setup requirements

```yaml
targets:
  mysql-docker-optimal:
    snapshot:
      src_url: "mydocker://user:pass@prod:3306/app"
      dst_url: "mydocker://user:pass@dev:3306/app"
```

**Advantages:**
- No local MySQL installation needed
- Includes MySQL Shell (`mysqlsh`) for advanced operations
- Consistent tool versions
- Automatic container lifecycle management

**Requirements:**
- Docker Engine 20.10+
- System resources for MySQL containers
- Network access for image pulls

## Mixed Engine Scenarios

### Cross-Database Migrations

Combine different engines for database migrations:

```yaml
targets:
  mysql-to-postgres:
    snapshot:
      # Source: MySQL with local tools
      src_url: "mysql://user:pass@legacy:3306/old_app"
      # Destination: PostgreSQL with Docker tools  
      dst_url: "pgdocker://user:pass@new:5432/modern_app"
      
    sanitize:
      override_query: |
        -- Transform data during migration
        ALTER TABLE users ALTER COLUMN id TYPE BIGINT;
```

### Environment-Specific Choices

Different engines for different environments:

```yaml
# Production backup (local tools for performance)
production-backup:
  snapshot:
    src_url: "postgres://readonly:pass@prod:5432/app"
    dst_url: "postgres://backup:pass@backup:5432/app_backup"

# Development refresh (Docker for consistency)  
dev-refresh:
  snapshot:
    src_url: "pgdocker://user:pass@staging:5432/app"
    dst_url: "pgdocker://dev:pass@localhost:5432/app_dev"
```

## Decision Framework

### Performance Requirements

**High Performance Needs:**
```yaml
# Use local engines for maximum speed
targets:
  high-performance:
    snapshot:
      src_url: "postgres://user:pass@source:5432/large_db"
      dst_url: "mysql://user:pass@dest:3306/large_db"
```

**Standard Performance Needs:**
```yaml
# Docker engines acceptable for regular operations
targets:
  standard-performance:
    snapshot:
      src_url: "pgdocker://user:pass@source:5432/app"  
      dst_url: "mydocker://user:pass@dest:3306/app"
```

### Environment Constraints

**Local Development:**
```yaml
# Mix of local and Docker based on tool availability
targets:
  local-dev:
    snapshot:
      src_url: "postgres://user:pass@localhost:5432/app"      # Local PostgreSQL
      dst_url: "mydocker://user:pass@localhost:3306/app"      # MySQL via Docker
```

**CI/CD Pipelines:**
```yaml
# Docker engines for consistent, reproducible builds
targets:
  ci-pipeline:
    snapshot:
      src_url: "pgdocker://user:pass@test-db:5432/app"
      dst_url: "mydocker://user:pass@staging-db:3306/app"
```

**Production Operations:**
```yaml
# Local engines for performance, with proper monitoring
targets:
  production-ops:
    cpus: 8
    snapshot:
      src_url: "postgres://readonly:pass@prod:5432/app"
      dst_url: "postgres://backup:pass@backup:5432/app"
```

### Feature Requirements

**Schema Filtering Required:**
```yaml
# Must use PostgreSQL engines (local or Docker)
targets:
  schema-filtering:
    snapshot:
      src_url: "postgres://user:pass@source:5432/app"  # or pgdocker://
      dst_url: "postgres://user:pass@dest:5432/app"    # or pgdocker://
      
      schema_config:
        include_schemas: ["public", "app_data"]
```

**No Schema Filtering:**
```yaml
# Any engine type works
targets:
  no-filtering:
    snapshot:
      src_url: "mysql://user:pass@source:3306/app"      # or mydocker://
      dst_url: "postgres://user:pass@dest:5432/app"     # or pgdocker://
```

## Performance Comparison

### Local vs Docker Performance

**Local Engines:**
- 10-20% faster for small databases (< 1GB)
- 20-30% faster for medium databases (1-10GB)  
- 30-40% faster for large databases (> 10GB)
- No container startup overhead
- Direct tool execution

**Docker Engines:**
- Consistent performance across environments
- Container startup adds 5-15 seconds
- Network overhead for database connections
- Memory overhead for container runtime

### Resource Usage

**Local Engine Resources:**
- Minimal memory overhead
- CPU usage only during operations
- Direct disk I/O access

**Docker Engine Resources:**
- Container memory overhead (100-500MB)
- Docker daemon CPU usage
- Volume mounting overhead

## Troubleshooting Engine Issues

### Local Engine Problems

**Tools Not Found:**
```bash
# PostgreSQL
which pg_dump pg_restore psql

# MySQL  
which mysqldump mysql
```

**Version Compatibility:**
```bash
# Check tool versions
pg_dump --version
mysql --version
```

### Docker Engine Problems

**Docker Not Available:**
```bash
docker --version
docker ps
```

**Image Pull Issues:**
```bash
docker pull postgres:16-alpine
docker pull mysql:8-oracle
```

**Resource Constraints:**
```bash
docker stats
docker system df
```

## Best Practices

### Development Workflow

```yaml
# Development: Use Docker for consistency
dev-target:
  snapshot:
    src_url: "pgdocker://user:pass@dev-db:5432/app"
    dst_url: "pgdocker://user:pass@localhost:5432/app_local"

# Testing: Use local tools for speed  
test-target:
  snapshot:
    src_url: "postgres://user:pass@test-db:5432/app"
    dst_url: "postgres://user:pass@localhost:5432/app_test"

# Production: Use local tools with monitoring
prod-target:
  cpus: 8
  snapshot:
    src_url: "postgres://readonly:pass@prod:5432/app"
    dst_url: "postgres://backup:pass@backup:5432/app"
```

### Team Collaboration

**Standardize on Docker engines** for team consistency:
```yaml
# Team standard configuration
defaults:
  # Docker engines ensure everyone uses same tool versions
  postgres_engine: "pgdocker://"
  mysql_engine: "mydocker://"

targets:
  team-standard:
    snapshot:
      src_url: "pgdocker://user:pass@shared-db:5432/app"
      dst_url: "mydocker://user:pass@dev-db:3306/app"
```

### Production Operations

**Use local engines** with proper resource allocation:
```yaml
production-operations:
  cpus: 16  # Maximize performance
  
  snapshot:
    src_url: "postgres://readonly:pass@prod:5432/app"
    dst_url: "postgres://backup:pass@backup:5432/app"
    
    schema_config:
      exclude_schemas: ["temp_data", "audit_logs"]
```

## Related Documentation

- [PostgreSQL Local Engine](postgresql-local.md) - Local PostgreSQL configuration
- [PostgreSQL Docker Engine](postgresql-docker.md) - Docker PostgreSQL setup
- [MySQL Local Engine](mysql-local.md) - Local MySQL configuration
- [MySQL Docker Engine](mysql-docker.md) - Docker MySQL setup
- [Docker Integration](docker-integration.md) - Docker system overview
- [Database Engines Introduction](introduction.md) - Engine architecture