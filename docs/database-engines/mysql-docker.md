# MySQL Docker (`mydocker://`)

<p align="center">
  <img alt="MySQL Logo" src="/static/mysql-icon.png"  />
</p>

## Overview

The MySQL Docker engine provides containerized MySQL operations using Docker images. This engine automatically manages MySQL tool containers, making it perfect for environments where MySQL client tools aren't locally installed or when you need consistent tool versions across different systems.

## Docker Image Configuration

### Default Image

**Image**: `mysql:8-oracle`
- Official Oracle MySQL 8.0 image
- Includes `mysqlsh` (MySQL Shell) for advanced operations
- Optimized for production workloads
- Full MySQL 8.0 feature set

### Supported Images

The engine works with official MySQL Docker images:

- `mysql:8-oracle` (recommended)
- `mysql:8.0`
- `mysql:latest`
- Custom MySQL images with `mysqlsh` included

### Custom Image Configuration

```yaml
# In dbsnapper configuration
docker:
  images:
    mysql: "mysql:8-oracle"  # Custom image
```

## URL Schemes and Usage

### Supported URL Schemes

| Scheme | Description | CLI Abbreviation |
|--------|-------------|------------------|
| `mydocker://` | MySQL Docker engine | `myd` |
| `myd://` | Short form | `myd` |

### Connection String Format

```
mydocker://username:password@host:port/database?parameters
```

**Example Configurations:**

```yaml
targets:
  docker-mysql:
    snapshot:
      src_url: "mydocker://app:pass@prod.mysql.com:3306/production"
      dst_url: "mydocker://dev:pass@localhost:3306/development"
```

## Container Management

### Automatic Operations

The MySQL Docker engine handles all container operations automatically:

1. **Container Creation**: Temporary containers for each operation
2. **Tool Execution**: MySQL commands run inside containers
3. **File Management**: Snapshot files transferred via volume mounts
4. **Cleanup**: Containers removed after operations complete

### Network Configuration

```yaml
targets:
  networked-mysql:
    # Network automatically managed
    snapshot:
      src_url: "mydocker://user:pass@mysql-server:3306/app"
      dst_url: "mydocker://user:pass@dev-mysql:3306/app_copy"
```

## Target Configuration Examples

### Basic Docker Setup

```yaml
targets:
  docker-basic:
    snapshot:
      src_url: "mydocker://root:password@db.company.com:3306/production"
      dst_url: "mydocker://root:password@localhost:3306/development"
```

### Production to Development Copy

```yaml
targets:
  prod-to-dev:
    snapshot:
      src_url: "mydocker://readonly:secret@prod.mysql.company.com:3306/application"
      dst_url: "mydocker://dev:dev_pass@dev.mysql.company.com:3306/application_copy"
      
    sanitize:
      override_query: |
        UPDATE users SET 
          email = CONCAT('user', id, '@example.com'),
          phone = '555-0000',
          ssn = NULL;
```

### Multi-Database Environment

```yaml
targets:
  customer-docker:
    snapshot:
      src_url: "mydocker://app:prod_pass@prod-mysql:3306/customers"
      dst_url: "mydocker://app:dev_pass@dev-mysql:3306/customers"
      
  analytics-docker:
    snapshot:
      src_url: "mydocker://analytics:prod_pass@prod-mysql:3306/analytics"
      dst_url: "mydocker://analytics:dev_pass@dev-mysql:3306/analytics_copy"
```

### High-Performance Configuration

```yaml
targets:
  performance-docker:
    cpus: 4  # Affects container resource allocation
    
    snapshot:
      src_url: "mydocker://user:pass@source:3306/large_database"
      dst_url: "mydocker://user:pass@dest:3306/large_database_copy"
```

## SSL and Security Configuration

### SSL Connections

```yaml
targets:
  secure-docker:
    snapshot:
      src_url: "mydocker://user:pass@host:3306/db?tls=preferred"
      dst_url: "mydocker://user:pass@localhost:3306/db"
```

### Security Best Practices

```yaml
targets:
  secure-setup:
    snapshot:
      # Use read-only credentials for source
      src_url: "mydocker://readonly:readonly_pass@prod:3306/app?tls=true"
      
      # Use limited credentials for destination
      dst_url: "mydocker://dev:dev_pass@dev:3306/app"
```

**Security Features:**
- Isolated container environments
- Temporary containers with no persistent state
- Automatic cleanup after operations
- Network isolation between operations

## Performance Optimization

### Container Resource Management

```yaml
targets:
  optimized-docker:
    cpus: 6  # Container CPU allocation
    
    snapshot:
      src_url: "mydocker://user:pass@source:3306/large_db"
      dst_url: "mydocker://user:pass@dest:3306/large_db"
```

### Large Database Handling

```yaml
targets:
  large-database:
    cpus: 8
    
    snapshot:
      src_url: "mydocker://user:pass@source:3306/warehouse?timeout=300s"
      dst_url: "mydocker://user:pass@dest:3306/warehouse_copy"
      
    sanitize:
      override_query: |
        -- Efficient updates for large datasets
        UPDATE users SET email = CONCAT('user', id, '@example.com') 
        WHERE email IS NOT NULL LIMIT 10000;
```

## MySQL-Specific Features

### Storage Engine Support

Automatically handles all MySQL storage engines:
- InnoDB (default)
- MyISAM
- Memory
- Archive
- Custom storage engines

### Character Set Handling

```yaml
targets:
  utf8-docker:
    snapshot:
      src_url: "mydocker://user:pass@source:3306/app?charset=utf8mb4"
      dst_url: "mydocker://user:pass@dest:3306/app?charset=utf8mb4"
```

### MySQL 8.0 Features

The Docker engine supports modern MySQL features:
- JSON data types
- Window functions
- Common table expressions (CTEs)
- Roles and dynamic privileges

## Troubleshooting

### Common Issues

**Docker Not Available:**
```bash
# Verify Docker is running
docker version
docker ps
```

**MySQL Image Issues:**
```bash
# Pull the MySQL image
docker pull mysql:8-oracle

# Test MySQL tools in container
docker run --rm mysql:8-oracle mysql --version
docker run --rm mysql:8-oracle mysqldump --version
```

**Connection Problems:**
```yaml
targets:
  debug-connection:
    snapshot:
      # Test with extended timeout
      src_url: "mydocker://user:pass@host:3306/db?timeout=30s&tls=false"
```

**Container Creation Failures:**
```bash
# Check Docker permissions
docker run --rm hello-world

# Verify available resources
docker system info
```

### Debug Commands

```bash
# Test Docker engine connectivity
dbsnapper build mysql-docker-target --dry-run --verbose

# Check MySQL connectivity manually
docker run --rm -it mysql:8-oracle mysql -h hostname -P 3306 -u username -p

# Verify mysqldump works
docker run --rm mysql:8-oracle mysqldump --version

# Test container networking
docker network ls
```

### Performance Issues

**Slow Operations:**
- Check available system resources
- Increase `cpus` setting
- Consider network latency to database servers
- Monitor disk I/O performance

**Container Startup Delays:**
- Pre-pull Docker image: `docker pull mysql:8-oracle`
- Use faster storage for Docker volumes
- Optimize Docker daemon settings

## Limitations

### Schema Filtering Not Available

Like MySQL Local engine, MySQL Docker does not support schema filtering:

```yaml
targets:
  mysql-docker-no-filtering:
    # This is NOT supported for MySQL engines
    # snapshot:
    #   schema_config:
    #     include_schemas: ["schema1"]  # Not available
    
    snapshot:
      src_url: "mydocker://user:pass@host:3306/database"
      dst_url: "mydocker://user:pass@host:3306/database_copy"
```

### Database-Level Operations

MySQL Docker engine operates at the database level:
- Processes entire databases, not individual schemas
- Cannot selectively copy specific tables
- All tables within a database are included

## Docker Requirements

### System Requirements

- **Docker Engine**: 20.10+ recommended
- **System Resources**: 2GB+ RAM for MySQL containers
- **Network**: Bridge network support
- **Storage**: Adequate space for temporary containers and data

### Installation Verification

```bash
# Verify Docker installation
docker --version
docker info

# Test MySQL image
docker run --rm mysql:8-oracle mysql --version
docker run --rm mysql:8-oracle mysqldump --version

# Test container creation
docker run --rm --name test-mysql -e MYSQL_ROOT_PASSWORD=test mysql:8-oracle echo "MySQL container works"
```

## Related Documentation

- [MySQL Local Engine](mysql-local.md) - Local tool alternative
- [PostgreSQL Docker Engine](postgresql-docker.md) - PostgreSQL containerized comparison
- [Database Engines Introduction](introduction.md) - Engine architecture overview