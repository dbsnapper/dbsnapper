# Engine Configuration Reference

## Overview

This reference provides comprehensive configuration examples for all database engines, covering common scenarios from basic setups to advanced production deployments. Use these examples as starting points for your specific requirements.

## Target Configuration Structure

All database engines use the same basic target structure:

```yaml
targets:
  target-name:
    # CPU allocation for parallel operations
    cpus: 4
    
    # Snapshot configuration
    snapshot:
      src_url: "engine://user:pass@host:port/database"
      dst_url: "engine://user:pass@host:port/database" 
      
      # PostgreSQL only: Schema filtering
      schema_config:
        include_schemas: ["schema1", "schema2"]
        exclude_schemas: ["temp_schema"]
    
    # Optional: Data sanitization
    sanitize:
      dst_url: "engine://user:pass@host:port/sanitized_database"
      override_query: |
        UPDATE sensitive_table SET data = 'sanitized';
```

## PostgreSQL Local Engine

### Basic Configuration

```yaml
targets:
  postgres-local-basic:
    snapshot:
      src_url: "postgres://user:password@localhost:5432/production"
      dst_url: "postgres://user:password@localhost:5432/development"
```

### Production with Schema Filtering

```yaml
targets:
  postgres-production:
    cpus: 8
    
    snapshot:
      src_url: "postgres://readonly:${PROD_PASSWORD}@prod.db.company.com:5432/application?sslmode=require"
      dst_url: "postgres://dev:${DEV_PASSWORD}@dev.db.company.com:5432/application_dev?sslmode=disable"
      
      schema_config:
        include_schemas:
          - "public"
          - "app_data"
          - "user_profiles"
        exclude_schemas:
          - "audit_logs"
          - "temp_processing"
          - "migration_backup"
    
    sanitize:
      dst_url: "postgres://dev:${DEV_PASSWORD}@dev.db.company.com:5432/application_sanitized"
      override_query: |
        -- Remove sensitive user data
        UPDATE users SET 
          email = 'user' || id || '@example.com',
          phone = regexp_replace(phone, '\d', 'X', 'g'),
          ssn = NULL,
          date_of_birth = '1990-01-01';
        
        -- Anonymize addresses
        UPDATE addresses SET 
          street_address = 'Test Street ' || id,
          city = 'Test City',
          postal_code = '00000',
          latitude = NULL,
          longitude = NULL;
        
        -- Clear payment information
        UPDATE payment_methods SET
          card_number = 'XXXX-XXXX-XXXX-1234',
          expiry_date = '12/25',
          cvv = NULL;
```

### SSL Configuration

```yaml
targets:
  postgres-ssl:
    snapshot:
      src_url: "postgres://app:pass@secure.db.com:5432/app?sslmode=require&sslcert=client.crt&sslkey=client.key&sslrootcert=ca.crt"
      dst_url: "postgres://dev:pass@localhost:5432/app_copy?sslmode=disable"
```

### High-Performance Large Database

```yaml
targets:
  postgres-warehouse:
    cpus: 16  # Maximum parallel processing
    
    snapshot:
      src_url: "postgres://analytics:${WAREHOUSE_PASSWORD}@warehouse.company.com:5432/analytics?connect_timeout=60&statement_timeout=300000"
      dst_url: "postgres://dev:${DEV_PASSWORD}@dev-warehouse:5432/analytics_sample"
      
      schema_config:
        # Only include essential schemas for development
        include_schemas:
          - "public"
          - "core_data"  
          - "reporting"
        exclude_schemas:
          - "raw_imports"      # Large temporary data
          - "historical_archive"  # Old data not needed for dev
          - "audit_trail"      # Sensitive audit data
```

## PostgreSQL Docker Engine

### Basic Docker Configuration

```yaml
targets:
  postgres-docker-basic:
    snapshot:
      src_url: "pgdocker://user:password@remote.db.com:5432/production"
      dst_url: "pgdocker://user:password@localhost:5432/development"
```

### CI/CD Pipeline Configuration

```yaml
targets:
  postgres-ci-pipeline:
    cpus: 4
    
    snapshot:
      src_url: "pgdocker://ci_user:${CI_DB_PASSWORD}@test-db:5432/integration_tests"
      dst_url: "pgdocker://ci_user:${CI_DB_PASSWORD}@localhost:5432/test_results"
      
      schema_config:
        include_schemas: ["public", "test_data"]
    
    sanitize:
      dst_url: "pgdocker://ci_user:${CI_DB_PASSWORD}@localhost:5432/sanitized_tests"
      override_query: |
        -- Sanitize test data for sharing
        UPDATE test_users SET 
          email = 'test' || id || '@testcompany.com',
          name = 'Test User ' || id;
```

### Custom Docker Image

```yaml
# Global Docker configuration
docker:
  images:
    postgres: "postgres:15-alpine"  # Use PostgreSQL 15

targets:
  postgres-custom-image:
    snapshot:
      src_url: "pgdocker://user:pass@source:5432/app"
      dst_url: "pgdocker://user:pass@dest:5432/app_copy"
```

## MySQL Local Engine

### Basic Configuration

```yaml
targets:
  mysql-local-basic:
    snapshot:
      src_url: "mysql://user:password@localhost:3306/production"
      dst_url: "mysql://user:password@localhost:3306/development"
```

### Production Configuration

```yaml
targets:
  mysql-production:
    cpus: 6
    
    snapshot:
      src_url: "mysql://readonly:${PROD_PASSWORD}@prod-mysql.company.com:3306/ecommerce?tls=true&charset=utf8mb4"
      dst_url: "mysql://dev:${DEV_PASSWORD}@dev-mysql.company.com:3306/ecommerce_dev?charset=utf8mb4"
    
    sanitize:
      dst_url: "mysql://dev:${DEV_PASSWORD}@dev-mysql.company.com:3306/ecommerce_sanitized"
      override_query: |
        -- Sanitize customer data
        UPDATE customers SET 
          email = CONCAT('customer', id, '@example.com'),
          phone = REGEXP_REPLACE(phone, '[0-9]', 'X'),
          first_name = CONCAT('Customer', id),
          last_name = 'Test',
          date_of_birth = '1990-01-01';
        
        -- Clear sensitive order information
        UPDATE orders SET
          billing_address = 'Test Address',
          shipping_address = 'Test Address',
          payment_method = 'Test Card';
        
        -- Anonymize payment data
        UPDATE payment_transactions SET
          card_last_four = '1234',
          transaction_id = CONCAT('TEST_', id),
          processor_response = 'TEST_APPROVED';
```

### Multi-Database Configuration

```yaml
targets:
  mysql-customers:
    snapshot:
      src_url: "mysql://app:${PROD_PASSWORD}@prod-cluster:3306/customers"
      dst_url: "mysql://dev:${DEV_PASSWORD}@dev-mysql:3306/customers_dev"
  
  mysql-orders:
    snapshot:
      src_url: "mysql://app:${PROD_PASSWORD}@prod-cluster:3306/orders"
      dst_url: "mysql://dev:${DEV_PASSWORD}@dev-mysql:3306/orders_dev"
  
  mysql-inventory:
    snapshot:
      src_url: "mysql://app:${PROD_PASSWORD}@prod-cluster:3306/inventory"
      dst_url: "mysql://dev:${DEV_PASSWORD}@dev-mysql:3306/inventory_dev"
```

### Performance-Optimized Configuration

```yaml
targets:
  mysql-performance:
    cpus: 8
    
    snapshot:
      src_url: "mysql://analytics:${WAREHOUSE_PASSWORD}@mysql-warehouse:3306/analytics?timeout=300s&readTimeout=120s&writeTimeout=120s"
      dst_url: "mysql://dev:${DEV_PASSWORD}@dev-mysql:3306/analytics_sample"
    
    sanitize:
      override_query: |
        -- Batch process large tables for better performance
        UPDATE user_events SET 
          user_id = FLOOR(RAND() * 10000),
          ip_address = '192.168.1.1',
          user_agent = 'Test Browser'
        WHERE created_date >= '2024-01-01'
        LIMIT 100000;
```

## MySQL Docker Engine

### Basic Docker Configuration

```yaml
targets:
  mysql-docker-basic:
    snapshot:
      src_url: "mydocker://user:password@remote.mysql.com:3306/production"
      dst_url: "mydocker://user:password@localhost:3306/development"
```

### Development Team Configuration

```yaml
targets:
  mysql-team-dev:
    snapshot:
      src_url: "mydocker://developer:${SHARED_DEV_PASSWORD}@shared-mysql:3306/team_app"
      dst_url: "mydocker://developer:${SHARED_DEV_PASSWORD}@localhost:3306/local_app"
    
    sanitize:
      dst_url: "mydocker://developer:${SHARED_DEV_PASSWORD}@localhost:3306/local_app_sanitized"
      override_query: |
        -- Standard development data sanitization
        UPDATE users SET 
          email = CONCAT('dev', id, '@devcompany.com'),
          password = '$2y$10$test.hash.for.development.use',
          api_token = NULL;
```

### Custom MySQL Image

```yaml
# Global Docker configuration  
docker:
  images:
    mysql: "mysql:8.0"  # Use standard MySQL 8.0

targets:
  mysql-custom-image:
    snapshot:
      src_url: "mydocker://user:pass@source:3306/app"
      dst_url: "mydocker://user:pass@dest:3306/app_copy"
```

## Mixed Engine Configurations

### Cross-Database Migration

```yaml
targets:
  mysql-to-postgres-migration:
    snapshot:
      # Source: MySQL production with local tools for performance
      src_url: "mysql://migration:${MYSQL_PASSWORD}@legacy-mysql:3306/legacy_app"
      # Destination: PostgreSQL with Docker for consistent tooling
      dst_url: "pgdocker://migration:${POSTGRES_PASSWORD}@new-postgres:5432/modern_app"
    
    sanitize:
      # Use PostgreSQL engine for destination sanitization
      dst_url: "pgdocker://dev:${DEV_PASSWORD}@new-postgres:5432/modern_app_dev"
      override_query: |
        -- PostgreSQL-specific sanitization after migration
        UPDATE users SET 
          email = 'migrated_user' || id || '@newcompany.com',
          legacy_id = NULL,
          migration_date = NOW();
```

### Environment-Specific Engine Selection

```yaml
# Local development: Use whatever tools are available
local-development:
  snapshot:
    src_url: "postgres://dev:dev@localhost:5432/app"          # Local PostgreSQL
    dst_url: "mydocker://dev:dev@localhost:3306/app_mysql"    # MySQL via Docker

# Staging: Use Docker for consistency  
staging-refresh:
  snapshot:
    src_url: "pgdocker://staging:${STAGING_PASSWORD}@staging-db:5432/app"
    dst_url: "pgdocker://staging:${STAGING_PASSWORD}@staging-db:5432/app_refreshed"

# Production: Use local tools for maximum performance
production-backup:
  cpus: 12
  snapshot:
    src_url: "postgres://backup:${BACKUP_PASSWORD}@prod-primary:5432/app"
    dst_url: "postgres://backup:${BACKUP_PASSWORD}@backup-server:5432/app_backup"
    
    schema_config:
      exclude_schemas: ["temp_data", "session_store"]
```

## Advanced Configuration Patterns

### Multi-Target Operations

```yaml
# Backup multiple environments
backup-all-environments:
  snapshot:
    src_url: "postgres://backup:${PROD_PASSWORD}@prod:5432/app"
    dst_url: "postgres://backup:${BACKUP_PASSWORD}@backup:5432/prod_backup_${DATE}"

backup-staging:
  snapshot:
    src_url: "postgres://backup:${STAGING_PASSWORD}@staging:5432/app"  
    dst_url: "postgres://backup:${BACKUP_PASSWORD}@backup:5432/staging_backup_${DATE}"

backup-dev:
  snapshot:
    src_url: "postgres://backup:${DEV_PASSWORD}@dev:5432/app"
    dst_url: "postgres://backup:${BACKUP_PASSWORD}@backup:5432/dev_backup_${DATE}"
```

### Conditional Environment Configuration

```yaml
# Production-like performance for staging
staging-performance-test:
  cpus: 8
  
  snapshot:
    src_url: "postgres://perf_test:${PERF_PASSWORD}@prod-replica:5432/app"
    dst_url: "postgres://staging:${STAGING_PASSWORD}@staging-performance:5432/app"
    
    schema_config:
      # Include all schemas for realistic performance testing
      exclude_schemas: ["audit_logs"]  # Only exclude truly unnecessary data
  
  sanitize:
    dst_url: "postgres://staging:${STAGING_PASSWORD}@staging-performance:5432/app_sanitized"
    override_query: |
      -- Minimal sanitization to preserve performance characteristics
      UPDATE users SET 
        email = 'perf_test' || id || '@staging.com'
      WHERE email LIKE '%@production-domain.com';
```

### Resource-Optimized Configuration

```yaml
# Small database - minimal resources
small-app-refresh:
  cpus: 2
  
  snapshot:
    src_url: "pgdocker://user:pass@small-db:5432/small_app"
    dst_url: "pgdocker://user:pass@localhost:5432/small_app_dev"

# Medium database - balanced resources  
medium-app-refresh:
  cpus: 4
  
  snapshot:
    src_url: "postgres://user:pass@medium-db:5432/medium_app"
    dst_url: "postgres://user:pass@dev:5432/medium_app_dev"

# Large database - maximum resources
large-app-refresh:
  cpus: 16
  
  snapshot:
    src_url: "postgres://user:pass@warehouse:5432/analytics"
    dst_url: "postgres://user:pass@dev-warehouse:5432/analytics_sample"
    
    schema_config:
      include_schemas: ["public", "core_analytics"]  # Reduce data volume
```

## Global Configuration

### Default Settings

```yaml
# Global defaults applied to all targets
defaults:
  cpus: 4
  working_directory: "./snapshots"
  
# Docker image configuration
docker:
  images:
    postgres: "postgres:16-alpine"
    mysql: "mysql:8-oracle"

# Database tool paths (if using local engines)
database_tools:
  postgresql:
    pg_dump: "/usr/local/bin/pg_dump"
    pg_restore: "/usr/local/bin/pg_restore"
    psql: "/usr/local/bin/psql"
  mysql:
    mysql: "/usr/local/bin/mysql"
    mysqldump: "/usr/local/bin/mysqldump"
```

### Environment Variables

```yaml
targets:
  secure-config:
    snapshot:
      # Use environment variables for sensitive data
      src_url: "postgres://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}"
      dst_url: "postgres://${DEV_USER}:${DEV_PASSWORD}@${DEV_HOST}:${DEV_PORT}/${DEV_NAME}"
```

## Validation and Testing

### Configuration Validation

```bash
# Test configuration without executing
dbsnapper build target-name --dry-run

# Verbose output for troubleshooting
dbsnapper build target-name --dry-run --verbose

# Validate multiple targets
dbsnapper config check
```

### Connection Testing

```yaml
targets:
  connection-test:
    # Minimal configuration for testing connectivity
    snapshot:
      src_url: "postgres://user:pass@host:5432/db?connect_timeout=10"
      dst_url: "postgres://user:pass@localhost:5432/test_db"
```

## Related Documentation

- [PostgreSQL Local Engine](postgresql-local.md) - PostgreSQL local configuration details
- [PostgreSQL Docker Engine](postgresql-docker.md) - PostgreSQL Docker setup  
- [MySQL Local Engine](mysql-local.md) - MySQL local configuration details
- [MySQL Docker Engine](mysql-docker.md) - MySQL Docker setup
- [Engine Selection Guide](engine-selection.md) - Choosing the right engines
- [Database Engines Introduction](introduction.md) - Architecture overview