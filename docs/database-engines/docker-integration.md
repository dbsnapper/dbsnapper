# Docker Integration

## Overview

DBSnapper provides seamless Docker integration for database operations, whether you're running DBSnapper on your local machine or within containerized environments. The Docker integration handles everything from container lifecycle management to network configuration automatically.

## Docker Engine Types

### Local Docker Usage

When DBSnapper runs on your local machine with Docker engines:

```yaml
targets:
  local-docker-usage:
    snapshot:
      # DBSnapper creates temporary containers for these operations
      src_url: "pgdocker://user:pass@remote-db:5432/production"
      dst_url: "mydocker://user:pass@local-mysql:3306/development"
```

**How it works:**
- DBSnapper creates temporary containers with PostgreSQL/MySQL tools
- Containers connect to your databases over the network
- Data is processed within isolated containers
- Containers are automatically cleaned up after operations

### Docker-in-Docker (DinD) Support

When DBSnapper itself runs inside a Docker container:

```yaml
# DBSnapper automatically detects Docker-in-Docker scenarios
targets:
  dind-aware:
    snapshot:
      src_url: "pgdocker://user:pass@database:5432/app"
      dst_url: "postgres://user:pass@localhost:5432/app_copy"
```

**Automatic adaptations:**
- Volume mounting strategies adjust for container environments
- Network connectivity uses container-to-container communication
- File system access adapts to container constraints

## Container Lifecycle Management

### Automatic Container Operations

DBSnapper handles all container operations transparently:

```yaml
targets:
  managed-containers:
    snapshot:
      src_url: "pgdocker://user:pass@prod:5432/app"
      dst_url: "mydocker://user:pass@dev:3306/app"
```

**Lifecycle stages:**
1. **Image Pull**: Automatically pulls required Docker images
2. **Container Creation**: Creates containers with appropriate configurations
3. **Network Setup**: Configures isolated networks for operations
4. **Tool Execution**: Runs database tools within containers
5. **Data Transfer**: Manages file transfers between containers and host
6. **Cleanup**: Removes containers and networks after completion

### Resource Management

Control container resources for optimal performance:

```yaml
targets:
  resource-controlled:
    cpus: 6  # Affects container CPU allocation
    
    snapshot:
      src_url: "pgdocker://user:pass@source:5432/large_db"
      dst_url: "mydocker://user:pass@dest:3306/large_db_copy"
```

## Network Configuration

### Automatic Network Creation

DBSnapper creates isolated networks for secure container communication:

```yaml
targets:
  networked-operation:
    # Network automatically created and managed
    snapshot:
      src_url: "pgdocker://user:pass@postgres-server:5432/app"
      dst_url: "mydocker://user:pass@mysql-server:3306/app"
```

**Network features:**
- Bridge networks for container-to-container communication
- DNS resolution between containers
- Port mapping for external database connections
- Automatic network cleanup after operations

### Custom Network Configuration

For advanced scenarios, DBSnapper adapts to existing networks:

```yaml
targets:
  existing-network:
    # DBSnapper works with existing Docker networks
    snapshot:
      src_url: "pgdocker://user:pass@existing-postgres:5432/app"
      dst_url: "mydocker://user:pass@existing-mysql:3306/app"
```

## Volume Management

### File System Access

DBSnapper automatically handles file system mounting:

```yaml
targets:
  file-system-managed:
    # Snapshot files automatically managed
    snapshot:
      src_url: "pgdocker://user:pass@source:5432/db"
      dst_url: "mydocker://user:pass@dest:3306/db"
```

**Volume handling:**
- Host directories mounted into containers
- Temporary files managed automatically
- Cross-platform path resolution
- Docker-in-Docker volume sharing

### Working Directory Configuration

Configure where snapshot files are stored:

```yaml
# In dbsnapper configuration
defaults:
  working_directory: "./snapshots"

targets:
  custom-location:
    snapshot:
      # Files stored in configured working directory
      src_url: "pgdocker://user:pass@source:5432/app"
      dst_url: "mydocker://user:pass@dest:3306/app"
```

## Docker Image Configuration

### Default Images

DBSnapper uses optimized default images:

```yaml
# Default image configuration (automatic)
docker:
  images:
    postgres: "postgres:16-alpine"  # PostgreSQL operations
    mysql: "mysql:8-oracle"         # MySQL operations
```

### Custom Images

Configure custom Docker images for specific needs:

```yaml
# Custom image configuration
docker:
  images:
    postgres: "postgres:15-alpine"  # Use PostgreSQL 15
    mysql: "mysql:8.0"              # Use different MySQL image

targets:
  custom-images:
    snapshot:
      # Uses configured custom images
      src_url: "pgdocker://user:pass@source:5432/app"
      dst_url: "mydocker://user:pass@dest:3306/app"
```

### Image Requirements

**PostgreSQL Images:**
- Must include `pg_dump`, `pg_restore`, and `psql`
- Recommended: `postgres:16-alpine` or `postgres:15-alpine`
- Compatible with PostgreSQL 9.2+

**MySQL Images:**
- Must include `mysqldump` and `mysql` commands
- Recommended: `mysql:8-oracle` (includes `mysqlsh`)
- Compatible with MySQL 5.7+

## Security Considerations

### Container Isolation

Docker engines provide strong security isolation:

```yaml
targets:
  secure-isolation:
    snapshot:
      # Each operation runs in isolated containers
      src_url: "pgdocker://readonly:pass@prod:5432/sensitive_db"
      dst_url: "mydocker://dev:pass@dev:3306/sanitized_db"
```

**Security features:**
- Containers run with minimal required permissions
- No persistent state between operations
- Isolated networks prevent unauthorized access
- Automatic credential management

### Credential Handling

DBSnapper securely manages database credentials in containers:

```yaml
targets:
  credential-security:
    snapshot:
      # Credentials passed securely to containers
      src_url: "pgdocker://app_readonly:${PROD_PASSWORD}@prod:5432/app"
      dst_url: "mydocker://app_dev:${DEV_PASSWORD}@dev:3306/app"
```

**Best practices:**
- Use environment variables for sensitive credentials
- Employ read-only database users for source connections
- Limit destination database privileges
- Rotate credentials regularly

## Performance Optimization

### Container Performance

Optimize container performance for large operations:

```yaml
targets:
  performance-optimized:
    cpus: 8  # Allocate more CPU resources
    
    snapshot:
      src_url: "pgdocker://user:pass@warehouse:5432/analytics"
      dst_url: "mydocker://user:pass@dest:3306/analytics_copy"
```

### Resource Allocation Strategies

**For Small Databases (< 1GB):**
```yaml
targets:
  small-db:
    cpus: 2  # Minimal resources sufficient
```

**For Medium Databases (1-10GB):**
```yaml
targets:
  medium-db:
    cpus: 4  # Balanced resource allocation
```

**For Large Databases (> 10GB):**
```yaml
targets:
  large-db:
    cpus: 8  # Maximum resources for best performance
```

### Memory Management

DBSnapper automatically manages container memory:
- Containers sized based on data volume
- Streaming processing for large datasets
- Automatic garbage collection
- Memory-efficient data transfer

## Troubleshooting

### Common Docker Issues

**Docker Not Available:**
```bash
# Verify Docker installation
docker --version
docker info
docker ps
```

**Image Pull Failures:**
```bash
# Manually pull required images
docker pull postgres:16-alpine
docker pull mysql:8-oracle

# Verify image availability
docker images | grep postgres
docker images | grep mysql
```

**Container Creation Problems:**
```bash
# Check Docker daemon status
sudo systemctl status docker  # Linux
# or
open -a Docker  # macOS

# Verify Docker permissions
docker run --rm hello-world
```

**Network Connectivity Issues:**
```bash
# Test container networking
docker network ls
docker run --rm postgres:16-alpine ping -c 1 google.com

# Check port availability
netstat -tlnp | grep :5432  # PostgreSQL
netstat -tlnp | grep :3306  # MySQL
```

### Resource Issues

**Insufficient Memory:**
```bash
# Check available system resources
free -h  # Linux
top      # macOS/Linux

# Monitor Docker resource usage
docker stats
```

**Disk Space Problems:**
```bash
# Check Docker disk usage
docker system df
docker system prune  # Clean up unused containers/images

# Check available disk space
df -h
```

### Debug Commands

```bash
# Verbose operation logging
dbsnapper build target-name --verbose

# Monitor container creation
docker events --filter type=container

# Check container logs (during operation)
docker logs $(docker ps -l -q)

# Inspect container configuration
docker inspect $(docker ps -l -q)
```

## Best Practices

### Performance Best Practices

```yaml
targets:
  optimized-setup:
    # Match CPU allocation to data size
    cpus: 4
    
    snapshot:
      # Use efficient connection strings
      src_url: "pgdocker://user:pass@source:5432/db?connect_timeout=30"
      dst_url: "mydocker://user:pass@dest:3306/db?timeout=60s"
```

### Security Best Practices

```yaml
targets:
  secure-setup:
    snapshot:
      # Use read-only credentials for source
      src_url: "pgdocker://readonly:${READONLY_PASS}@prod:5432/app"
      
      # Use limited credentials for destination
      dst_url: "mydocker://dev:${DEV_PASS}@dev:3306/app"
```

### Resource Management

- **Pre-pull Docker images** during setup to avoid delays
- **Monitor container resource usage** during operations
- **Clean up unused images** regularly to save disk space
- **Use appropriate CPU allocation** based on database size

### Network Configuration

- **Use isolated networks** for sensitive operations
- **Configure connection timeouts** for unreliable networks
- **Test connectivity** before running large operations
- **Monitor network traffic** during data transfers

## Docker Requirements

### System Requirements

- **Docker Engine**: 20.10+ recommended
- **Docker Compose**: 2.0+ (for complex setups)
- **System Resources**: 
  - 4GB+ RAM for medium databases
  - 8GB+ RAM for large databases
  - 2+ CPU cores recommended
- **Network**: Stable internet for image pulls
- **Storage**: Adequate space for images and temporary data

### Installation Verification

```bash
# Comprehensive Docker verification
docker --version
docker info
docker run --rm hello-world

# Test required images
docker run --rm postgres:16-alpine pg_dump --version
docker run --rm mysql:8-oracle mysql --version

# Test networking
docker network create test-network
docker network rm test-network

# Test volume mounting
docker run --rm -v $(pwd):/data postgres:16-alpine ls -la /data
```

## Related Documentation

- [PostgreSQL Docker Engine](postgresql-docker.md) - PostgreSQL containerization details
- [MySQL Docker Engine](mysql-docker.md) - MySQL containerization details  
- [Ephemeral Databases](ephemeral-databases.md) - Temporary container databases
- [Database Engines Introduction](introduction.md) - Engine architecture overview