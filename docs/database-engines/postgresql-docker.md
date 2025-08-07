# PostgreSQL Docker (`pgdocker://`)

<p align="center">
  <img alt="PostgreSQL Logo" src="/static/postgres-icon.png"  />
</p>

## Overview

The PostgreSQL Docker engine provides containerized PostgreSQL operations using Docker images. This engine automatically manages PostgreSQL tool containers, making it ideal for environments where PostgreSQL client tools aren't locally installed or when consistent tool versions are required across different systems.

## Architecture

The PostgreSQL Docker engine implements the Database and DatabaseTools interfaces using:

- **Container Management**: Automatic Docker container creation and lifecycle management
- **Tool Execution**: PostgreSQL tools (`pg_dump`, `pg_restore`, `psql`) executed within containers
- **Network Isolation**: Dedicated Docker networks for secure database communication
- **Volume Management**: Automatic file system mounting for data transfer

## Docker Image Configuration

### Default Image

**Image**: `postgres:16-alpine`
- Lightweight Alpine Linux base
- PostgreSQL 16 with all standard tools
- Optimized for container environments
- Minimal attack surface

### Supported Images

The engine is compatible with official PostgreSQL Docker images:

- `postgres:16-alpine` (recommended)
- `postgres:15-alpine`
- `postgres:14-alpine`
- `postgres:13-alpine`
- `postgres:16`, `postgres:15`, `postgres:14` (full Debian-based images)

### Custom Image Configuration

```yaml
# In dbsnapper configuration
docker:
  images:
    postgres: "postgres:16-alpine"  # Custom image
```

## URL Schemes and Usage

### Supported URL Schemes

| Scheme | Description | CLI Abbreviation |
|--------|-------------|------------------|
| `pgdocker://` | PostgreSQL Docker engine | `pgd` |
| `pgd://` | Short form | `pgd` |

### Connection String Format

```
pgdocker://username:password@host:port/database?parameters
```

**Example Configurations:**

```yaml
targets:
  docker-postgres:
    snapshot:
      src_url: "pgdocker://user:pass@prod.db.com:5432/production?sslmode=require"
      dst_url: "pgdocker://user:pass@localhost:5432/development"
```

## Container Lifecycle Management

### Automatic Container Operations

1. **Container Creation**: Temporary containers created for each operation
2. **Network Setup**: Isolated Docker networks for secure communication
3. **Tool Execution**: PostgreSQL commands executed within containers
4. **File Transfer**: Snapshot files managed via volume mounts
5. **Cleanup**: Automatic container and network removal after operations

### Network Configuration

The engine creates isolated Docker networks:

```yaml
targets:
  networked-postgres:
    # Network automatically managed
    snapshot:
      src_url: "pgdocker://user:pass@database:5432/app"
      dst_url: "pgdocker://user:pass@dev-db:5432/app_copy"
```

**Network Features:**
- Bridge networks for container-to-container communication
- DNS resolution between containers
- Port mapping for external connectivity
- Automatic network cleanup

## Docker-in-Docker Support

### Detection and Adaptation

When DBSnapper runs inside a Docker container, the PostgreSQL Docker engine automatically detects this environment and adapts its behavior:

```yaml
targets:
  dind-aware:
    # Engine automatically detects Docker-in-Docker
    # and adjusts volume mounting strategies
    snapshot:
      src_url: "pgdocker://user:pass@host/source_db"
      dst_url: "pgdocker://user:pass@host/dest_db"
```

### Volume Mounting Strategies

**Host Environment:**
- Direct host path mounting
- Full file system access
- Standard Docker volume behavior

**Docker-in-Docker Environment:**
- Shared volume mounting between containers
- Network-based file transfer when needed
- Container-to-container communication

## Advanced Features

### Schema Filtering Support

PostgreSQL Docker engine inherits full schema filtering capabilities:

```yaml
targets:
  filtered-docker:
    snapshot:
      src_url: "pgdocker://user:pass@source/production"
      dst_url: "pgdocker://user:pass@dest/filtered_copy"
      
      schema_config:
        include_schemas: ["public", "app_data"]
        exclude_schemas: ["temp_schema", "audit_logs"]
```

### CPU Configuration

Parallel operations supported within Docker containers:

```yaml
targets:
  parallel-docker:
    cpus: 4  # Parallel pg_dump operations
    
    snapshot:
      src_url: "pgdocker://user:pass@source/large_db"
      dst_url: "pgdocker://user:pass@dest/large_db_copy"
```

### Ephemeral Database Integration

The PostgreSQL Docker engine seamlessly integrates with ephemeral databases:

```yaml
targets:
  with-ephemeral:
    # Ephemeral database automatically created for sanitization
    snapshot:
      src_url: "pgdocker://user:pass@production/source"
      dst_url: "pgdocker://user:pass@localhost/development"
      
    sanitize:
      override_query: |
        UPDATE users SET email = 'user' || id || '@example.com';
```

## Target Configuration Examples

### Basic Docker Setup

```yaml
targets:
  docker-basic:
    snapshot:
      src_url: "pgdocker://postgres:password@db.company.com:5432/production"
      dst_url: "pgdocker://postgres:password@localhost:5432/development"
```

### High-Performance Docker Configuration

```yaml
targets:
  docker-performance:
    cpus: 8
    
    snapshot:
      src_url: "pgdocker://readonly:secret@prod:5432/app?sslmode=require"
      dst_url: "pgdocker://dev:secret@dev:5432/app_copy"
      
      schema_config:
        exclude_schemas: ["audit_logs", "temp_schema"]
```

### Multi-Environment Docker Setup

```yaml
targets:
  staging-docker:
    snapshot:
      src_url: "pgdocker://app:prod_pass@production.db:5432/app"
      dst_url: "pgdocker://app:staging_pass@staging.db:5432/app"
      
  development-docker:
    snapshot:
      src_url: "pgdocker://app:staging_pass@staging.db:5432/app"
      dst_url: "pgdocker://app:dev_pass@localhost:5432/app_dev"
```

## Container Security

### Security Features

- **Isolated Networks**: Each operation uses dedicated Docker networks
- **Temporary Containers**: Containers removed after operations
- **No Persistent State**: No data stored in containers between runs
- **Limited Privileges**: Containers run with minimal required permissions

### Security Best Practices

```yaml
targets:
  secure-docker:
    snapshot:
      # Use read-only credentials for source
      src_url: "pgdocker://readonly:readonly_pass@prod:5432/app?sslmode=require"
      
      # Use limited credentials for destination
      dst_url: "pgdocker://dev:dev_pass@dev:5432/app?sslmode=disable"
```

## Performance Optimization

### Container Resource Management

```yaml
targets:
  optimized-docker:
    cpus: 6  # Utilize available CPU cores
    
    # Large database optimization
    snapshot:
      src_url: "pgdocker://user:pass@source/large_db"
      dst_url: "pgdocker://user:pass@dest/large_db"
```

### Network Performance

- **Local Networks**: Containers communicate via bridge networks
- **External Connections**: Optimized for remote database access
- **Parallel Processing**: Multiple containers for concurrent operations

## Troubleshooting

### Common Issues

**Docker Not Available:**
```bash
# Verify Docker is running
docker version
docker ps
```

**Container Creation Failures:**
```bash
# Check Docker permissions
docker run --rm hello-world

# Verify image availability
docker pull postgres:16-alpine
```

**Network Connectivity Issues:**
```yaml
targets:
  debug-network:
    snapshot:
      # Test with simplified connection
      src_url: "pgdocker://user:pass@host:5432/db?connect_timeout=30"
```

**Volume Mounting Problems:**
- Ensure Docker daemon has access to file paths
- Check Docker-in-Docker volume sharing
- Verify file permissions on host system

### Debug Commands

```bash
# Test Docker engine connectivity
dbsnapper build docker-target --dry-run --verbose

# Check container logs
docker logs $(docker ps -l -q)

# Verify network creation
docker network ls | grep dbsnapper

# Test image availability
docker images | grep postgres
```

### Performance Issues

**Slow Operations:**
- Check available system resources
- Increase `cpus` setting for parallel operations
- Use schema filtering to reduce data volume
- Consider network latency to database servers

**Container Startup Delays:**
- Pre-pull Docker images: `docker pull postgres:16-alpine`
- Use faster storage for Docker volumes
- Optimize Docker daemon configuration

## Docker Requirements

### System Requirements

- **Docker Engine**: 20.10+ recommended
- **Docker Compose**: 2.0+ (for complex deployments)
- **System Resources**: 2GB+ RAM, adequate CPU for parallel operations
- **Network**: Bridge network support

### Installation Verification

```bash
# Verify Docker installation
docker --version
docker info

# Test PostgreSQL image
docker run --rm postgres:16-alpine pg_dump --version

# Check network capabilities
docker network create test-network
docker network rm test-network
```

## Related Documentation

- [PostgreSQL Local Engine](postgresql-local.md) - Local tool alternative
- [MySQL Docker Engine](mysql-docker.md) - MySQL containerized alternative
- [Database Engines Introduction](introduction.md) - Engine architecture overview