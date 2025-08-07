# Database Engines Architecture

<p align="center">
  <img alt="Database Engines" src="/static/database-engines.png"  />
</p>

DBSnapper uses a sophisticated engine architecture that provides unified database operations across different vendors and deployment models. Each database engine implements standardized interfaces while handling vendor-specific operations and optimizations.

## Engine Architecture Overview

### Interface-Based Design

All database engines implement two core interfaces:

- **Database Interface**: Connection management, query execution, schema operations
- **DatabaseTools Interface**: Dump, restore, and utility operations using command-line tools

This design ensures consistent behavior across all supported database types while allowing for vendor-specific optimizations.

### Engine Factory Pattern

DBSnapper uses URL schemes to automatically select the appropriate engine:

```yaml
# URL Scheme → Engine Selection
postgres://     → PostgreSQL Local
pgdocker://     → PostgreSQL Docker  
mysql://        → MySQL Local
mydocker://     → MySQL Docker
```

The engine factory examines the connection string scheme and instantiates the correct engine implementation automatically.

## Supported Database Engines

### PostgreSQL Engines

- **PostgreSQL Local** (`postgres://`, `postgresql://`, `pg://`, `pglocal://`, `pgl://`)
  - Uses locally installed `pg_dump`, `pg_restore`, and `psql` tools
  - Supports schema filtering and parallel operations
  - Best performance for local development

- **PostgreSQL Docker** (`pgdocker://`, `pgd://`)
  - Uses `postgres:16-alpine` Docker image (configurable)
  - Automatic container management and networking
  - Ideal when PostgreSQL tools aren't locally installed

### MySQL Engines

- **MySQL Local** (`mysql://`, `mylocal://`, `myl://`)
  - Uses locally installed `mysqldump` and `mysql` tools
  - Direct connection to MySQL servers
  - Optimal performance for local MySQL instances

- **MySQL Docker** (`mydocker://`, `myd://`)
  - Uses `mysql:8-oracle` Docker image with `mysqlsh`
  - Container-based tool execution
  - Perfect for environments without MySQL client tools

## Key Engine Features

### Schema Filtering (PostgreSQL Only)

PostgreSQL engines support advanced schema filtering:

```yaml
targets:
  my-postgres:
    snapshot:
      schema_config:
        include_schemas: ["public", "app_data"]
        exclude_schemas: ["temp_schema", "backup_schema"]
```

### CPU Configuration

Configure parallel operations for improved performance:

```yaml
targets:
  my-target:
    cpus: 4  # Parallel dump/restore operations
```

### Ephemeral Databases

All Docker engines can create temporary database containers for operations like sanitization and testing:

- Automatic container lifecycle management
- Isolated network environments
- Automatic cleanup after operations

## Engine Selection Guidelines

### Use Local Engines When:
- Database tools are installed locally (`pg_dump`, `mysqldump`)
- Maximum performance is required
- Direct network connectivity to databases
- Minimal Docker overhead desired

### Use Docker Engines When:
- Database tools aren't installed locally
- Running in containerized environments
- Consistent tool versions across environments
- Isolation from host system required

## Docker Integration

### Docker-in-Docker Support
When DBSnapper runs inside a container, engines automatically detect this and adjust their behavior:
- Volume mounting strategies adapt for container environments
- Network connectivity uses container networking
- Tool execution switches between local and Docker modes

### Network Management
Docker engines create isolated networks for database containers:
- Bridge networks for container-to-container communication
- Port mapping for external connectivity
- Automatic network cleanup

## URL Scheme Reference

| Scheme | Engine Type | Description |
|--------|-------------|-------------|
| `postgres://`, `postgresql://`, `pg://` | PostgreSQL Local | Local PostgreSQL tools |
| `pglocal://`, `pgl://` | PostgreSQL Local | Explicit local designation |
| `pgdocker://`, `pgd://` | PostgreSQL Docker | Containerized PostgreSQL tools |
| `mysql://` | MySQL Local | Local MySQL tools |
| `mylocal://`, `myl://` | MySQL Local | Explicit local designation |
| `mydocker://`, `myd://` | MySQL Docker | Containerized MySQL tools |

## Advanced Capabilities

### Connection String Parsing
Engines automatically parse connection strings and extract:
- Database credentials and connection parameters
- SSL settings and connection options
- Database names for operations

### Error Handling
Standardized error handling across all engines:
- Connection validation and retry logic
- Detailed error messages with troubleshooting hints
- Graceful handling of network interruptions

### Performance Optimization
- Connection pooling for database operations
- Parallel processing where supported
- Efficient memory usage during large data transfers

## Next Steps

- [PostgreSQL Local Engine](postgresql-local.md) - Local PostgreSQL tool configuration
- [PostgreSQL Docker Engine](postgresql-docker.md) - Containerized PostgreSQL setup
- [MySQL Local Engine](mysql-local.md) - Local MySQL tool configuration  
- [MySQL Docker Engine](mysql-docker.md) - Containerized MySQL setup