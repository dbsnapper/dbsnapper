# MySQL Local (`mysql://`)

<p align="center">
  <img alt="MySQL Logo" src="/static/mysql-icon.png"  />
</p>

## Overview

The MySQL Local engine provides direct connections to MySQL databases using locally installed command-line tools. This engine offers optimal performance for local development and environments where MySQL client tools are available, providing fast and efficient database operations without containerization overhead.

## Architecture

The MySQL Local engine implements the Database and DatabaseTools interfaces using:

- **Connection Management**: Direct MySQL connections via `sqlx` and MySQL drivers
- **Data Operations**: Native SQL execution with MySQL-specific optimizations
- **Tool Operations**: Local `mysqldump` and `mysql` command execution
- **Constraint Management**: Automated foreign key and unique constraint handling

## Requirements

### MySQL Client Tools

The engine requires these MySQL tools in your system PATH:

- `mysqldump` - Database backup creation and export
- `mysql` - Database restoration and SQL execution
- `mysqlshow` - Database and table inspection (optional but recommended)

### Supported Versions

- **Tested**: MySQL 8.3 and MySQL 8.0 series
- **Recommended**: MySQL 8.0+ for optimal performance and feature support
- **Compatible**: MySQL 5.7+ (with reduced feature availability)

### Installation Options

- **MySQL Official Installer** - Complete MySQL installation with all tools
- **Package Managers**: 
  - macOS: `brew install mysql`
  - Ubuntu/Debian: `apt-get install mysql-client`
  - RHEL/CentOS: `yum install mysql`
- **Docker**: Client tools via `mysql:latest` image
- **Standalone Client**: MySQL client-only packages

## URL Schemes and Usage

### Supported URL Schemes

The MySQL Local engine responds to multiple URL schemes:

| Scheme | Description | CLI Abbreviation |
|--------|-------------|------------------|
| `mysql://` | Standard MySQL scheme | `my` |
| `mylocal://` | Explicit local designation | `myl` |
| `myl://` | Shortest local form | `myl` |

### Connection String Format

```
mysql://username:password@host:port/database?parameters
```

**Example Configurations:**

```yaml
targets:
  production-mysql:
    snapshot:
      src_url: "mysql://app_user:secret@db.company.com:3306/production"
      dst_url: "mysql://dev_user:dev_pass@localhost:3306/development"
```

### SSL and Security Configuration

```yaml
targets:
  secure-mysql:
    snapshot:
      src_url: "mysql://user:pass@host:3306/db?tls=preferred&serverPubKey=server-public.pem"
```

**Common MySQL Parameters:**
- `tls=true|false|preferred` - SSL/TLS connection mode
- `charset=utf8mb4` - Character set specification
- `parseTime=true` - Enable time parsing
- `timeout=30s` - Connection timeout

## Key Differences from PostgreSQL

### Schema vs Database Structure

MySQL uses a different schema model than PostgreSQL:

```yaml
targets:
  mysql-structure:
    # MySQL: database contains tables directly
    # No schema filtering support (PostgreSQL-only feature)
    snapshot:
      src_url: "mysql://user:pass@host/database_name"
      dst_url: "mysql://user:pass@localhost/copy_database"
```

**Important Notes:**
- **No Schema Filtering**: MySQL engines do not support `schema_config` with `include_schemas`/`exclude_schemas`
- **Database-Level Operations**: MySQL operates at the database level, not schema level
- **Different Permissions Model**: MySQL uses database-level privileges

### Constraint Handling

MySQL Local engine automatically manages constraints during operations:

```sql
-- Automatically executed during operations
SET UNIQUE_CHECKS=0, FOREIGN_KEY_CHECKS=0, SQL_MODE='';
```

This allows for:
- Circumventing foreign key constraints during data loading
- Handling circular dependencies in table relationships
- Faster bulk data operations

## Target Configuration Examples

### Basic Local Development

```yaml
targets:
  local-mysql:
    snapshot:
      src_url: "mysql://root:password@localhost:3306/myapp_production"
      dst_url: "mysql://root:password@localhost:3306/myapp_development"
```

### Production to Development Copy

```yaml
targets:
  prod-to-dev:
    snapshot:
      src_url: "mysql://readonly:secret@prod.mysql.company.com:3306/application"
      dst_url: "mysql://dev:dev_pass@dev.mysql.company.com:3306/application_copy"
      
    sanitize:
      override_query: |
        UPDATE users SET 
          email = CONCAT('user', id, '@example.com'),
          phone = '555-0000',
          ssn = NULL,
          created_ip = '0.0.0.0';
```

### Multi-Database Environment

```yaml
targets:
  customer-db:
    snapshot:
      src_url: "mysql://app:prod_pass@prod-mysql:3306/customer_data"
      dst_url: "mysql://app:dev_pass@dev-mysql:3306/customer_data"
      
  analytics-db:
    snapshot:
      src_url: "mysql://analytics:prod_pass@prod-mysql:3306/analytics"
      dst_url: "mysql://analytics:dev_pass@dev-mysql:3306/analytics_copy"
```

### High-Performance Configuration

```yaml
targets:
  performance-mysql:
    # CPU configuration affects tool execution parallelism
    cpus: 4
    
    snapshot:
      src_url: "mysql://user:pass@source:3306/large_database?timeout=60s"
      dst_url: "mysql://user:pass@dest:3306/large_database_copy"
```

## Permissions and Security

### Required Database Permissions

**For Source Database (Building Snapshots):**
```sql
-- Create dedicated read-only user
CREATE USER 'dbsnapper_readonly'@'%' IDENTIFIED BY 'readonly_password';
GRANT SELECT ON production.* TO 'dbsnapper_readonly'@'%';
GRANT LOCK TABLES ON production.* TO 'dbsnapper_readonly'@'%';
GRANT SHOW VIEW ON production.* TO 'dbsnapper_readonly'@'%';
FLUSH PRIVILEGES;
```

**For Destination Database (Loading Snapshots):**
```sql
-- Create development user with full database privileges
CREATE USER 'dbsnapper_dev'@'%' IDENTIFIED BY 'dev_password';
GRANT ALL PRIVILEGES ON development.* TO 'dbsnapper_dev'@'%';
GRANT CREATE ON *.* TO 'dbsnapper_dev'@'%';  -- For database creation
FLUSH PRIVILEGES;
```

### Security Best Practices

```yaml
targets:
  secure-mysql:
    snapshot:
      # Use dedicated users with minimal required privileges
      src_url: "mysql://readonly:readonly_pass@prod:3306/app?tls=true"
      dst_url: "mysql://dev:dev_pass@dev:3306/app"
```

**Security Recommendations:**
- Create dedicated users for DBSnapper operations
- Use SSL/TLS for production connections
- Implement IP-based access restrictions
- Regular password rotation
- Monitor database access logs

## MySQL-Specific Features

### Storage Engine Handling

The engine automatically handles different MySQL storage engines:

```yaml
targets:
  mixed-engines:
    # Automatically handles InnoDB, MyISAM, and other engines
    snapshot:
      src_url: "mysql://user:pass@source:3306/mixed_database"
      dst_url: "mysql://user:pass@dest:3306/mixed_copy"
```

### Character Set Management

```yaml
targets:
  utf8-database:
    snapshot:
      src_url: "mysql://user:pass@source:3306/app?charset=utf8mb4"
      dst_url: "mysql://user:pass@dest:3306/app?charset=utf8mb4"
```

### Trigger and View Handling

MySQL Local engine preserves:
- Database triggers
- Views and their dependencies
- Stored procedures and functions
- Event schedules (when permissions allow)

## Performance Optimization

### Large Database Strategies

```yaml
targets:
  large-mysql:
    cpus: 8  # Parallel tool execution
    
    snapshot:
      src_url: "mysql://user:pass@source:3306/large_db?timeout=300s"
      dst_url: "mysql://user:pass@dest:3306/large_db"
      
    # Optimize for large datasets
    sanitize:
      override_query: |
        -- Use efficient UPDATE statements
        UPDATE users SET email = CONCAT('user', id, '@example.com') WHERE email IS NOT NULL;
```

### Network Optimization

```yaml
targets:
  network-optimized:
    snapshot:
      # Connection optimization for remote databases
      src_url: "mysql://user:pass@remote:3306/db?timeout=60s&readTimeout=45s&writeTimeout=45s"
      dst_url: "mysql://user:pass@local:3306/db"
```

### Memory Management

MySQL Local engine optimizes memory usage:
- Streaming data processing for large tables
- Efficient bulk insert operations
- Connection pooling for multiple operations

## Troubleshooting

### Common Issues

**MySQL Tools Not Found:**
```bash
# Verify MySQL tools are installed
which mysqldump
which mysql
mysql --version
```

**Connection Failures:**
```yaml
targets:
  debug-connection:
    snapshot:
      # Test with extended timeout and error details
      src_url: "mysql://user:pass@host:3306/db?timeout=30s&tls=false"
```

**Permission Errors:**
```sql
-- Check user privileges
SHOW GRANTS FOR 'username'@'hostname';

-- Verify database access
USE database_name;
SHOW TABLES;
```

**Character Set Issues:**
```sql
-- Check database character set
SHOW CREATE DATABASE database_name;

-- Verify table character sets
SELECT table_name, table_collation 
FROM information_schema.tables 
WHERE table_schema = 'database_name';
```

### Debug Commands

```bash
# Test MySQL connectivity
mysql -h hostname -P 3306 -u username -p database_name

# Verify mysqldump functionality
mysqldump --version
mysqldump -h hostname -P 3306 -u username -p --single-transaction database_name --no-data

# Test DBSnapper connection
dbsnapper build mysql-target --dry-run --verbose
```

### Performance Issues

**Slow Dump Operations:**
- Check source database server performance
- Use connection pooling for better throughput
- Consider network bandwidth limitations
- Monitor disk I/O on both source and destination

**Large Table Handling:**
```yaml
targets:
  large-table-handling:
    # Optimize for tables with millions of rows
    snapshot:
      src_url: "mysql://user:pass@source:3306/big_db?timeout=600s"
      dst_url: "mysql://user:pass@dest:3306/big_db"
```

## Limitations

### Schema Filtering Not Available

Unlike PostgreSQL engines, MySQL engines do not support schema filtering:

```yaml
targets:
  mysql-no-filtering:
    # This is NOT supported for MySQL engines
    # snapshot:
    #   schema_config:
    #     include_schemas: ["schema1", "schema2"]  # Not available for MySQL
    
    snapshot:
      src_url: "mysql://user:pass@host:3306/database"
      dst_url: "mysql://user:pass@host:3306/database_copy"
```

### Database-Level Operations Only

MySQL engines operate at the database level:
- Cannot filter individual schemas within a database
- All tables within a database are processed together
- No selective table copying (entire database or nothing)

## Related Documentation

- [MySQL Docker Engine](mysql-docker.md) - Container-based alternative
- [PostgreSQL Local Engine](postgresql-local.md) - PostgreSQL comparison
- [Database Engines Introduction](introduction.md) - Engine architecture overview