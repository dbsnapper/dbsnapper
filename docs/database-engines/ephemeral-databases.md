# Ephemeral Databases

## Overview

Ephemeral databases are temporary database containers that DBSnapper creates automatically for specific operations like sanitization and data transformation. These short-lived containers provide isolated environments for processing sensitive data without affecting your production or development databases.

## How Ephemeral Databases Work

### Automatic Creation

DBSnapper automatically creates ephemeral databases when needed:

```yaml
targets:
  sanitize-example:
    snapshot:
      src_url: "postgres://user:pass@production:5432/app"
      dst_url: "postgres://user:pass@localhost:5432/app_dev"
      
    sanitize:
      override_query: |
        UPDATE users SET email = 'user' || id || '@example.com';
```

**Behind the scenes:**
1. DBSnapper creates a temporary database container
2. Loads your snapshot data into the ephemeral database
3. Runs sanitization queries in the isolated environment
4. Exports sanitized data to your destination
5. Automatically destroys the temporary container

### Container Lifecycle

**Creation Phase:**
- Unique container name generated automatically
- Isolated Docker network created
- Database credentials configured securely

**Operation Phase:**
- Data loaded from source snapshot
- Sanitization or transformation queries executed
- Processed data prepared for export

**Cleanup Phase:**
- Container stopped and removed
- Network resources cleaned up
- No persistent data remains

## Use Cases

### Data Sanitization

The most common use case for ephemeral databases:

```yaml
targets:
  customer-data-sanitization:
    snapshot:
      src_url: "postgres://readonly:pass@prod:5432/customers"
      dst_url: "postgres://dev:pass@dev:5432/customers_sanitized"
      
    sanitize:
      override_query: |
        -- Remove sensitive customer data
        UPDATE customers SET 
          email = 'customer' || id || '@example.com',
          phone = '555-0000',
          ssn = NULL,
          credit_card = NULL;
          
        -- Anonymize addresses
        UPDATE addresses SET 
          street = 'Test Street',
          city = 'Test City',
          postal_code = '00000';
```

### Data Transformation

Transform data structure during the copy process:

```yaml
targets:
  legacy-migration:
    snapshot:
      src_url: "mysql://user:pass@legacy:3306/old_app"
      dst_url: "postgres://user:pass@new:5432/modernized_app"
      
    sanitize:
      override_query: |
        -- Transform legacy data formats
        UPDATE user_profiles SET 
          full_name = CONCAT(first_name, ' ', last_name),
          created_date = DATE(created_timestamp);
```

### Testing Data Preparation

Create specialized datasets for testing:

```yaml
targets:
  test-data-prep:
    snapshot:
      src_url: "postgres://user:pass@staging:5432/app"
      dst_url: "postgres://test:pass@test:5432/app_testing"
      
    sanitize:
      override_query: |
        -- Create predictable test data
        UPDATE users SET 
          email = 'test' || id || '@testcompany.com',
          status = 'active'
        WHERE id <= 100;
        
        -- Remove non-test data
        DELETE FROM users WHERE id > 100;
```

## Database Engine Support

### PostgreSQL Ephemeral Databases

Uses PostgreSQL Docker containers with automatic configuration:

```yaml
targets:
  postgres-ephemeral:
    snapshot:
      src_url: "pgdocker://user:pass@source:5432/app"
      dst_url: "postgres://user:pass@dest:5432/app_clean"
      
    # Ephemeral PostgreSQL container created automatically
    sanitize:
      override_query: |
        UPDATE sensitive_table SET data = 'sanitized';
```

**Features:**
- Full PostgreSQL feature support
- Schema filtering compatibility
- Parallel processing capabilities

### MySQL Ephemeral Databases

Uses MySQL Docker containers with root access for operations:

```yaml
targets:
  mysql-ephemeral:
    snapshot:
      src_url: "mydocker://user:pass@source:3306/app"
      dst_url: "mysql://user:pass@dest:3306/app_clean"
      
    # Ephemeral MySQL container created automatically  
    sanitize:
      override_query: |
        UPDATE users SET email = CONCAT('user', id, '@example.com');
```

**Features:**
- MySQL 8.0 feature support
- All storage engines supported
- Automatic constraint handling

## Configuration Options

### Network Isolation

Ephemeral databases run in isolated Docker networks by default:

```yaml
targets:
  isolated-processing:
    # Network automatically managed for security
    snapshot:
      src_url: "postgres://user:pass@production:5432/sensitive_db"
      dst_url: "postgres://user:pass@dev:5432/sanitized_db"
```

### Resource Allocation

Control resources for ephemeral database containers:

```yaml
targets:
  resource-controlled:
    cpus: 4  # Affects ephemeral container resources
    
    snapshot:
      src_url: "postgres://user:pass@large-db:5432/warehouse"
      dst_url: "postgres://user:pass@dest:5432/warehouse_processed"
      
    sanitize:
      override_query: |
        -- Resource-intensive sanitization
        UPDATE large_table SET sensitive_data = 'redacted';
```

## Security Benefits

### Data Isolation

**Complete Isolation:**
- Ephemeral databases run in separate containers
- No network access to production systems
- Temporary credentials with limited scope

**No Persistent State:**
- Containers destroyed after operations
- No data left on disk after processing
- Automatic cleanup of all resources

### Credential Management

```yaml
targets:
  secure-ephemeral:
    snapshot:
      # Production uses read-only credentials
      src_url: "postgres://readonly:limited@prod:5432/app"
      # Destination uses dev credentials  
      dst_url: "postgres://dev:dev_pass@dev:5432/app"
    
    # Ephemeral database uses auto-generated credentials
    sanitize:
      override_query: |
        UPDATE users SET email = 'safe' || id || '@example.com';
```

## Performance Considerations

### Optimization Strategies

**For Large Datasets:**
```yaml
targets:
  large-ephemeral:
    cpus: 8  # More resources for ephemeral processing
    
    snapshot:
      src_url: "postgres://user:pass@warehouse:5432/analytics"
      dst_url: "postgres://user:pass@dev:5432/analytics_sample"
      
    sanitize:
      override_query: |
        -- Process in batches for better performance
        UPDATE large_table SET 
          sensitive_col = 'redacted'
        WHERE id BETWEEN 1 AND 10000;
```

**Memory Management:**
- Ephemeral containers automatically sized based on data volume
- Streaming processing for large datasets
- Automatic cleanup prevents resource leaks

### Processing Time

**Factors Affecting Performance:**
- Source data size
- Complexity of sanitization queries  
- Available CPU and memory resources
- Network bandwidth for data transfer

**Typical Processing Times:**
- Small databases (< 1GB): 1-5 minutes
- Medium databases (1-10GB): 5-30 minutes
- Large databases (> 10GB): 30+ minutes

## Troubleshooting

### Common Issues

**Container Creation Failures:**
```bash
# Verify Docker is working
docker version
docker ps

# Check available resources
docker system df
docker system info
```

**Memory Issues:**
```yaml
targets:
  memory-optimized:
    # Reduce resource usage for large datasets
    cpus: 2
    
    sanitize:
      override_query: |
        -- Process in smaller batches
        UPDATE users SET email = 'user' || id || '@example.com' 
        WHERE id <= 1000;
```

**Network Connectivity:**
```bash
# Check Docker networks
docker network ls

# Verify container networking
docker run --rm --network bridge postgres:16-alpine ping -c 1 google.com
```

### Debug Commands

```bash
# Monitor ephemeral container creation
dbsnapper sanitize target-name --verbose

# Check Docker container logs (while running)
docker logs $(docker ps -l -q)

# Monitor resource usage
docker stats $(docker ps -l -q)
```

### Performance Issues

**Slow Processing:**
- Increase `cpus` setting for more resources
- Simplify sanitization queries
- Process data in smaller batches
- Check available system memory

**Container Startup Delays:**
- Pre-pull required Docker images
- Ensure adequate disk space
- Optimize Docker daemon settings

## Best Practices

### Query Optimization

```yaml
targets:
  optimized-sanitization:
    sanitize:
      override_query: |
        -- Use efficient updates with indexes
        UPDATE users SET 
          email = 'user' || id || '@example.com'
        WHERE email IS NOT NULL;
        
        -- Batch process large tables
        UPDATE large_table SET 
          sensitive_data = NULL 
        WHERE created_date < '2023-01-01'
        LIMIT 10000;
```

### Resource Management

```yaml
targets:
  resource-aware:
    # Match resources to data size
    cpus: 4
    
    sanitize:
      override_query: |
        -- Monitor progress with row counts
        UPDATE users SET email = 'test' || id || '@example.com';
        -- Returns: UPDATE 15000 (example)
```

### Security Practices

- **Use read-only source credentials** when possible
- **Limit sanitization queries** to necessary operations only
- **Verify sanitization results** before deploying to development
- **Monitor ephemeral container resource usage** to detect issues

## Related Documentation

- [Database Engines Introduction](introduction.md) - Engine architecture overview
- [PostgreSQL Docker Engine](postgresql-docker.md) - PostgreSQL containerization
- [MySQL Docker Engine](mysql-docker.md) - MySQL containerization